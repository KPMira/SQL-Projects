---This COVID-19 SQL Analytics project will contribute to a better understanding of the pandemic's dynamics by shedding light on mortality, infection rates, population correlations, vaccination trends, and global statistics. ---
/* Skills used: 
		Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types*/
/* You can view an interactive Tableau visualization of this SQL project by following this link: https://public.tableau.com/app/profile/kimberly.mira/vizzes 
Link to the Dataset: https://ourworldindata.org/covid-deaths */


---Data Overview---
SELECT *
FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths]
ORDER BY 3,4; 

SELECT *
FROM [Covid-19_Portfolio Project]..[Covid-19_Vaccinations]
ORDER BY 3,4;


--- Overview of the data that will be used, focusing on COVID-19_Deaths table. ---
SELECT location,date, total_cases, new_cases, total_deaths, population
FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths]
ORDER BY 1,2


--- Total cases and deaths overview ---
/* Likelihood of dying from COVID-19 in 2023. (Sample Country: Philppines) */
SELECT location, date, total_cases, total_deaths,
	ROUND((CONVERT(FLOAT, total_deaths) / NULLIF(CONVERT(FLOAT, total_cases), 0)) * 100,5) as 'DeathPerCase(in%)'
FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths]
WHERE date LIKE '2023%' and location = 'Philippines'
ORDER BY 5 DESC;


--- Comparison between total cases and population ---
/* Percentage of the population infected by COVID-19 in 2023. (Sample Country: Philppines) */
SELECT location, date, population, total_cases, 
	ROUND((CONVERT(FLOAT, total_cases) / NULLIF(CONVERT(FLOAT, population), 0)) * 100,5) as 'CasesPerPopulation(%)'
FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths]
WHERE date like '2023%' and location = 'Philippines'
ORDER BY 'CasesPerPopulation(%)' DESC;


--- Correlation between total cases and population ---
/* Likelihood of contracting from COVID-19 in highly populated areas. */
SELECT continent, location, population, MAX(total_cases) as MaxInfectionCases,
	ROUND((CONVERT(FLOAT, MAX(total_cases)) / NULLIF(CONVERT(FLOAT, population), 0)) * 100,5) as 'CasesPerPopulation(%)'
FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths]
GROUP BY continent, location, population
ORDER BY 'CasesPerPopulation(%)' DESC;


--- Death rates per population ---
/* Countries with highest death rates per population. */
SELECT location,  MAX(cast(total_deaths as int)) as MaxDeathCases
FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths]
WHERE LEN(iso_code)<4
	AND continent is not NULL 
GROUP BY location
ORDER BY 2 DESC;


/* Continents with highest death rates per population. */
SELECT continent, MAX(CONVERT(INT,total_deaths)) as MaxDeathCases
FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths]
WHERE LEN(iso_code)<4
	AND continent is not NULL 
GROUP BY continent
ORDER BY 1;


--- Global Numbers---
/* COVID-19 Global death percentage, total cases and total deaths. (Using CTEs) */
WITH GlobalNumbers AS (
    SELECT 
        CONVERT(FLOAT, SUM(CASE WHEN ISNUMERIC(new_deaths) = 1 THEN CAST(new_deaths AS FLOAT) ELSE 0 END)) AS total_deaths,
        CONVERT(FLOAT, SUM(CASE WHEN ISNUMERIC(new_cases) = 1 THEN CAST(new_cases AS FLOAT) ELSE 0 END)) AS total_cases
    FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths]
    WHERE continent is not NULL 
)
SELECT total_cases, total_deaths,
		CONVERT(FLOAT, total_deaths / NULLIF(total_cases, 0) * 100) AS GlobalDeathPercentage
FROM GlobalNumbers
ORDER BY GlobalDeathPercentage;


/* COVID-19 Global death Percentage for the date range from January 1, 2020, to October 5, 2023. (Using CTEs) */
WITH GlobalNumbers AS (
    SELECT 
        CONVERT(FLOAT, SUM(CASE WHEN ISNUMERIC(new_deaths) = 1 THEN CAST(new_deaths AS FLOAT) ELSE 0 END)) AS total_deaths,
        CONVERT(FLOAT, SUM(CASE WHEN ISNUMERIC(new_cases) = 1 THEN CAST(new_cases AS FLOAT) ELSE 0 END)) AS total_cases
    FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths]
    WHERE continent is not NULL 
),
DateRange AS (
    SELECT 
        MIN(date) AS EarliestDate,
        MAX(date) AS LatestDate
    FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths]
    WHERE continent is not NULL 
)
SELECT EarliestDate, LatestDate, total_cases, total_deaths,
    CONVERT(FLOAT, total_deaths / NULLIF(total_cases, 0) * 100) AS GlobalDeathPercentage
FROM DateRange
CROSS JOIN GlobalNumbers
ORDER BY GlobalDeathPercentage;


--- Overview of the Data that will be used, focusing on Covid-19_Vaccinations. ---
SELECT *
FROM [Covid-19_Portfolio Project]..[Covid-19_Vaccinations]
ORDER BY 3,4;

--- Total Population vs Vaccinations---
/* Shows the increasing vacccination count per population. (Disregarding the dates with no issued vaccination)*/
SELECT dth.continent, dth.location, dth.date, dth.population, vcc.new_vaccinations,
	SUM(CONVERT(FLOAT,vcc.new_vaccinations)) 
	OVER (PARTITION BY dth.location Order by dth.location, dth.date) AS RollingVaccinationCount
FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths] AS dth
JOIN [Covid-19_Portfolio Project]..[Covid-19_Vaccinations] AS vcc
	ON dth.location = vcc.location AND dth.date = vcc.date
WHERE LEN(dth.iso_code)<4
	AND dth.continent is not NULL
	AND vcc.new_vaccinations <> 0
ORDER BY 2, 3;


/* Shows the increasing vacccination count per day. (Disregarding the dates with no issued vaccination)*/

WITH PVRatio as
(
SELECT dth.continent, dth.location, dth.date, dth.population, vcc.new_vaccinations,
	SUM(CONVERT(FLOAT,vcc.new_vaccinations)) 
	OVER (PARTITION BY dth.location Order by dth.location, dth.date) AS RollingVaccinationCount
FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths] AS dth
JOIN [Covid-19_Portfolio Project]..[Covid-19_Vaccinations] AS vcc
	ON dth.location = vcc.location AND dth.date = vcc.date
WHERE LEN(dth.iso_code)<4
	AND dth.continent is not NULL
	AND vcc.new_vaccinations <> 0
)

SELECT *, 
	CONVERT(FLOAT,(RollingVaccinationCount/population)*100) AS RollingPercentagePerPopulation
FROM PVRatio


/* Shows the total percentage of the population that is vaccinated per country. */
WITH Total_PVRatio as
(
SELECT dth.location, dth.population,
       SUM(CONVERT(FLOAT,vcc.new_vaccinations)) AS TotalVaccinations
FROM [Covid-19_Portfolio Project]..[Covid-19_Deaths] AS dth
JOIN [Covid-19_Portfolio Project]..[Covid-19_Vaccinations] AS vcc
    ON dth.location = vcc.location AND dth.date = vcc.date
WHERE LEN(dth.iso_code)<4
	AND dth.location is not NULL
    AND vcc.new_vaccinations <> 0
GROUP BY dth.population, dth.location
)

SELECT *, 
	ROUND(CONVERT(FLOAT,(TotalVaccinations/population)*100),5) AS PercentagePerPopulation
FROM Total_PVRatio
ORDER BY 3;


