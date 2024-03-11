# COVID-19 Data Exploration with SQL

This repository contains SQL queries for exploring COVID-19 data. Various SQL techniques and functions are utilized to analyze and extract insights from the data.

## Skills Utilized
- Joins
- Common Table Expressions (CTEs)
- Temporary Tables
- Window Functions
- Aggregate Functions
- Creating Views
- Converting Data Types

## Queries Explanation

### 1. Selecting Initial Data
- Retrieves basic COVID-19 data including location, date, total cases, new cases, total deaths, and population.
```sql
Select Location, date, total_cases, new_cases, total_deaths, population
From PortfolioProject..CovidDeaths
Where continent is not null 
order by 1,2
```

### 2. Total Cases vs Total Deaths
- Calculates the death percentage based on total cases and total deaths for each location, providing insight into the likelihood of dying if one contracts COVID-19 in a specific country.
```sql
-- Shows likelihood of dying if you contract covid in your country

Select Location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
Where location like '%states%'
and continent is not null 
order by 1,2
```
### 3. Total Cases vs Population
- Calculates the percentage of the population infected with COVID-19 based on total cases and population for each location.
```sql

Select Location, date, Population, total_cases,  (total_cases/population)*100 as PercentPopulationInfected
From PortfolioProject..CovidDeaths
--Where location like '%states%'
order by 1,2

```
### 4. Countries with Highest Infection Rate compared to Population
- Identifies countries with the highest infection rates relative to their population, showing the highest infection count and the corresponding percentage of the population infected.
```sql
Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From PortfolioProject..CovidDeaths
--Where location like '%states%'
Group by Location, Population
order by PercentPopulationInfected desc

```
### 5. Countries with Highest Death Count per Population
- Determines countries with the highest death count per population, displaying the total death count for each location.
```sql
Select Location, MAX(cast(Total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
--Where location like '%states%'
Where continent is not null 
Group by Location
order by TotalDeathCount desc

```
### 6. Breaking Down Data by Continent
- Shows continents with the highest death count per population, providing insights into the global distribution of COVID-19 deaths.
```sql
Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
--Where location like '%states%'
Where continent is not null 
Group by continent
order by TotalDeathCount desc
```
### 7. Global Numbers
- Provides global statistics including total cases, total deaths, and the death percentage based on new cases and new deaths.
```sql
Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
--Where location like '%states%'
where continent is not null 
--Group By date
order by 1,2
```
### 8. Total Population vs Vaccinations
- Compares the total population with the number of vaccinations administered, showing the percentage of the population that has received at least one COVID-19 vaccine.
```sql
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3
```
### 9. Using CTE to Perform Calculation
- Utilizes a Common Table Expression (CTE) to calculate the percentage of the population vaccinated based on the number of vaccinations administered.
```sql
With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
--order by 2,3
)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac
```
### 10. Using Temporary Table to Perform Calculation
- Creates a temporary table to store data and then calculates the percentage of the population vaccinated using the rolling sum of new vaccinations.
```sql
DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated
```
### 11. Creating a View to Store Data
- Creates a view to store data for later visualizations or further analysis.
```sql
Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
```
