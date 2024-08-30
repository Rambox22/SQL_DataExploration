SELECT *
FROM dbo.CovidVacccinations$

--SELECT *
--FROM dbo.Death$

-- likeliHood of Dying in Australia 
SELECT Location, date, total_cases, total_deaths, Cast(total_deaths As decimal)/Cast(total_cases AS decimal)  * 100 AS deathpercent 
FROM dbo.Death$
WHERE location LIKE '%Australia'
ORDER BY deathpercent DESC 

-- Total_cases vs Population
-- Percentage of  population that got covid
SELECT Location, population, MAX(total_cases) AS HighestInfectedCount,  MAX((total_cases/population))*100 AS 
percentofpopulation_covid
FROM dbo.Death$
--WHERE location LIKE '%Australia'
GROUP BY location, population
ORDER BY percentofpopulation_covid DESC

SELECT location,  MAX(total_deaths) AS Maxdeathcount,  MAX((total_cases/population))*100 AS 
percentofpopulation_covid
FROM dbo.Death$
WHERE continent is NULL
--WHERE location LIKE '%Australia'
GROUP BY location

ORDER BY Maxdeathcount DESC

-- Select the Global Covid Deaths
SELECT SUM(new_cases) AS totalcases, SUM(CAST(new_deaths AS int)) as total_deaths, 
SUM(CAST(new_deaths AS int)) / SUM(new_cases) * 100 AS percentage_death_per_case
FROM dbo.Death$
WHERE continent IS NOT NULL

SELECT dea.location, dea.date
FROM dbo.Death$ dea
JOIN dbo.CovidVacccinations$ vac
ON dea.location = vac.location
AND dea.date = vac.date
ORDER BY dea.location DESC 


SELECT dea.continent, dea.location, dea.date , dea.population, vac.new_vaccinations, SUM(CAST(vac.new_vaccinations AS decimal)) OVER 
(PARTITION by dea.location ORDER BY dea.location, DEA.DATE) AS RollingPeopleVaccinated
FROM dbo.Death$ dea
JOIN dbo.CovidVacccinations$ vac
ON dea.location = vac.location
AND dea.date = vac.date
ORDER BY dea.location DESC 

--CTE
WITH PopvsVac(continent, location, date , population, new_vaccinations, RollingPeopleVaccinated)
AS(

SELECT dea.continent, dea.location, dea.date , dea.population, vac.new_vaccinations, SUM(CAST(vac.new_vaccinations AS decimal)) OVER 
(PARTITION by dea.location ORDER BY dea.location, DEA.DATE) AS RollingPeopleVaccinated
FROM dbo.Death$ dea
JOIN dbo.CovidVacccinations$ vac
ON dea.location = vac.location
AND dea.date = vac.date
--ORDER BY dea.location DESC 
)

SELECT *, (RollingPeopleVaccinated/Population) 
FROM PopvsVac

--temp table
DROP TABLE if exists #percentPopulationVaccinated
CREATE TABLE #percentPopulationVaccinated(
continent nvarchar(255),
location nvarchar(255),
date datetime,
population numeric,
new_vaccinations numeric,
RollingPeopleVaccinated numeric)

INSERT INTO #percentPopulationVaccinated
 SELECT dea.continent, dea.location, dea.date , dea.population, vac.new_vaccinations, SUM(CAST(vac.new_vaccinations AS decimal)) OVER 
(PARTITION by dea.location ORDER BY dea.location, DEA.DATE) AS RollingPeopleVaccinated
FROM dbo.Death$ dea
JOIN dbo.CovidVacccinations$ vac
ON dea.location = vac.location
AND dea.date = vac.date
--ORDER BY dea.location DESC 

SELECT *, (RollingPeopleVaccinated/Population) 
FROM #percentPopulationVaccinated


 
-- For Tablaeu Visualization
CREATE VIEW percentpopulationvaccinated AS
SELECT dea.continent, dea.location, dea.date , dea.population, vac.new_vaccinations, SUM(CAST(vac.new_vaccinations AS decimal)) OVER 
(PARTITION by dea.location ORDER BY dea.location, DEA.DATE) AS RollingPeopleVaccinated
FROM dbo.Death$ dea
JOIN dbo.CovidVacccinations$ vac
ON dea.location = vac.location
AND dea.date = vac.date

