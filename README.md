# COVID-19 Data Exploration with SQL

This document outlines the data analysis project that I did following a YouTube video "Data Analyst Portfolio Project | SQL Data Exploration | Project 1/4" by Alex The Analyst. The project focuses on exploring a COVID-19 dataset using SQL for data exploration and preparation for visualization.

## 1. Dataset Description

The dataset used in this project contains COVID-19 related information from early 2020 to the present . It includes various metrics such as:

*   **Total Cases** 
*   **New Cases** 
*   **Total Deaths** 
*   **New Deaths** 
*   **Total Vaccinations** 
*   **People Vaccinated** 
*   **Population** 
*   Other data about smokers, diabetes and other random stuff 

The dataset is divided into two Excel files:
*   **covid deaths** contains data on deaths, cases and population 
*   **covid vaccinations** contains data on vaccinations 

The data is initially stored in two separate excel files, one containing death-related information and the other containing vaccination information .

## 2. Data Preparation

### 2.1. Data Download and Formatting
*   The dataset is downloaded from a source that allows access to the complete historical data, starting from January 1, 2020 .
*   The population data is moved to the beginning of the *covid deaths* table for easier access without needing joins in early queries .
*   Unnecessary columns are removed from each excel sheet to prepare for import .
*   The two excel files are saved as excel workbooks and not as csv files .

### 2.2. Importing Data into SQL Server
*   A new database named "Portfolio Project" is created in SQL Server .
*   The two excel files are imported as tables into the "Portfolio Project" database . I used MySQL Workbench.  
*   The two tables are named `covid deaths` and `covid vaccinations` .

## 3. Data Exploration and Analysis with SQL

The following SQL queries were used for data exploration and analysis.

### 3.1. Basic Data Selection
The following query selects key information from the `covid deaths` table and orders by location and date .

```sql
--Selecting location, date, total cases, new cases, total deaths and population
SELECT
    location,
    date,
    total_cases,
    new_cases,
    total_deaths,
    population
FROM
    PortfolioProject..coviddeaths
ORDER BY
    location,
    date;
```

This query retrieves the location, date, total cases, new cases, total deaths, and population from the coviddeaths table and orders the result by location and date.

### 3.2. Calculating Death Percentage
This query calculates the death percentage for each location, showing the likelihood of dying if infected with COVID-19.

```
-- Calculating death percentage
SELECT
    location,
    date,
    total_cases,
    total_deaths,
    (total_deaths/total_cases)*100 AS death_percentage
FROM
    PortfolioProject..coviddeaths
ORDER BY
    location,
    date;
```

This query calculates the percentage of deaths out of the total cases, providing a death percentage for each entry.

## 3.3. Calculating Percentage of Population Infected
This query calculates the percentage of the population infected with COVID-19.

```
-- Calculating percentage of population infected
SELECT
    location,
    date,
    total_cases,
    population,
    (total_cases/population)*100 AS percent_population_infected
FROM
    PortfolioProject..coviddeaths
ORDER BY
    location,
    date;
```

This query calculates the percentage of the population infected with COVID-19 by dividing the total cases by the population for each entry.

## 3.4. Countries with Highest Infection Rate
This query identifies countries with the highest infection rates compared to their population.
```
--Countries with highest infection rate
SELECT
    location,
    MAX(total_cases) AS highest_infection_count,
    MAX((total_cases/population))*100 AS percent_population_infected
FROM
    PortfolioProject..coviddeaths
GROUP BY
    location,
    population
ORDER BY
    percent_population_infected DESC;
```

This query finds the maximum total cases for each country and calculates the percentage of the population infected, then orders them in descending order based on the infection percentage.

## 3.5. Countries with Highest Death Count
This query identifies countries with the highest death counts per population. The total_deaths column is cast as an integer due to data type issues. The query filters out continent-level data by excluding rows where the continent is null.

```
-- Countries with highest death count
SELECT
    location,
    MAX(CAST(total_deaths as int)) AS total_death_count
FROM
    PortfolioProject..coviddeaths
WHERE continent is not null
GROUP BY
    location
ORDER BY
    total_death_count DESC;
```

This query identifies countries with the highest total death counts and filters out continent-level data. The total_deaths column is cast as an integer to enable aggregate functions.

## 3.6. Death Count by Continent
This query shows the highest death count by continent.
```
--Death count by continent
SELECT
    continent,
    MAX(CAST(total_deaths as int)) AS total_death_count
FROM
    PortfolioProject..coviddeaths
WHERE continent is not null
GROUP BY
    continent
ORDER BY
    total_death_count DESC;
```
This query calculates the highest death count per continent, using the same data type conversion and filtering to exclude continent-level locations.

## 3.7. Global Numbers
This query calculates global COVID-19 numbers by summing new cases and new deaths, and then calculating the death percentage.
```
-- Global numbers
SELECT
    date,
    SUM(new_cases) AS total_cases,
    SUM(CAST(new_deaths as int)) AS total_deaths,
    (SUM(CAST(new_deaths as int))/SUM(new_cases))*100 AS death_percentage
FROM
    PortfolioProject..coviddeaths
WHERE continent is not null
GROUP BY
    date;
```
This query calculates the daily total cases, total deaths, and the death percentage across all locations. The new_deaths column is cast as an integer to enable aggregation.
This query calculates the overall death percentage across the world without considering the date.

```
-- Overall global numbers
SELECT
    SUM(new_cases) AS total_cases,
    SUM(CAST(new_deaths as int)) AS total_deaths,
    (SUM(CAST(new_deaths as int))/SUM(new_cases))*100 AS death_percentage
FROM
    PortfolioProject..coviddeaths
WHERE continent is not null;
```
## 3.8. Joining Deaths and Vaccinations Tables
This query joins the coviddeaths and covidvaccinations tables to analyze vaccination data.
```
-- Joining deaths and vaccinations tables
SELECT
    dea.continent,
    dea.location,
    dea.date,
    dea.population,
    vac.new_vaccinations
FROM
    PortfolioProject..coviddeaths dea
JOIN
    PortfolioProject..covidvaccinations vac
ON
    dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent is not null
ORDER BY dea.location, dea.date
```

This query joins the two tables on location and date, retrieves the continent, location, date, population from the deaths table and the new vaccinations from the vaccinations table.

## 3.9. Calculating Rolling Vaccination Count
This query calculates the cumulative number of vaccinations for each location. The new vaccinations column is cast as an integer to enable aggregate functions.
```
-- Calculating rolling vaccination count
SELECT
    dea.continent,
    dea.location,
    dea.date,
    dea.population,
    vac.new_vaccinations,
    SUM(CONVERT(int,vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_people_vaccinated
FROM
    PortfolioProject..coviddeaths dea
JOIN
    PortfolioProject..covidvaccinations vac
ON
    dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent is not null
ORDER BY dea.location, dea.date
```
This query uses a window function to calculate the rolling sum of new vaccinations, partitioned by location and ordered by location and date. The new_vaccinations column is also cast as an integer to enable aggregation.

## 3.10. Using a CTE to calculate Percent Population Vaccinated
This query uses a Common Table Expression (CTE) to calculate the percentage of the population vaccinated. It uses the rolling count of vaccinations calculated in the previous query.
```
-- Using CTE to calculate percent population vaccinated
WITH pop_vs_vac (continent, location, date, population, new_vaccinations, rolling_people_vaccinated)
AS
(
    SELECT
        dea.continent,
        dea.location,
        dea.date,
        dea.population,
        vac.new_vaccinations,
    SUM(CONVERT(int,vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_people_vaccinated
    FROM
        PortfolioProject..coviddeaths dea
    JOIN
        PortfolioProject..covidvaccinations vac
    ON
        dea.location = vac.location
        AND dea.date = vac.date
    WHERE dea.continent is not null
)
SELECT *, (rolling_people_vaccinated/population)*100 as percent_population_vaccinated
FROM pop_vs_vac
```
This query defines a CTE pop_vs_vac to store intermediate results, and then calculates the percentage of the population vaccinated using the rolling people vaccinated values.

## 3.11. Using a Temp Table to calculate Percent Population Vaccinated
This query uses a temporary table to calculate the percentage of the population vaccinated. It demonstrates an alternative to using CTEs.
```
-- Using a temp table to calculate percent population vaccinated
DROP TABLE IF EXISTS #percent_population_vaccinated
CREATE TABLE #percent_population_vaccinated (
    continent nvarchar(255),
    location nvarchar(255),
    date datetime,
    population numeric,
    new_vaccinations numeric,
    rolling_people_vaccinated numeric
)
INSERT INTO #percent_population_vaccinated
    SELECT
        dea.continent,
        dea.location,
        dea.date,
        dea.population,
        vac.new_vaccinations,
        SUM(CONVERT(int,vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_people_vaccinated
    FROM
        PortfolioProject..coviddeaths dea
    JOIN
        PortfolioProject..covidvaccinations vac
    ON
        dea.location = vac.location
        AND dea.date = vac.date
    WHERE dea.continent is not null

SELECT *, (rolling_people_vaccinated/population)*100 as percent_population_vaccinated
FROM #percent_population_vaccinated
```
This query creates a temporary table, populates it with the required data, and then calculates the vaccination percentage. It includes a DROP TABLE IF EXISTS statement to handle re-execution and avoid errors.

## 3.12. Creating a View
This query creates a view named percent_population_vaccinated to store the results of the percentage of the population vaccinated, calculated with the rolling count of vaccinations.
```
--Creating a view
CREATE VIEW percent_population_vaccinated AS
SELECT
    dea.continent,
    dea.location,
    dea.date,
    dea.population,
    vac.new_vaccinations,
    SUM(CONVERT(int,vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_people_vaccinated
    FROM
        PortfolioProject..coviddeaths dea
    JOIN
        PortfolioProject..covidvaccinations vac
    ON
        dea.location = vac.location
        AND dea.date = vac.date
    WHERE dea.continent is not null
```
This query creates a view to store the vaccination percentage, which can be used for further analysis and visualization. The view can be queried like a table.

