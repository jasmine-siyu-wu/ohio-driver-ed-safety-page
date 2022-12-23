---
title: "Overlaying Socioeconomic Status to Access to Driver Education"
date: 2022-12-21
published: true
tags: [dataviz, folium, acs, ses, drivered]
excerpt: " "
folium-loader:
  folium-chart-1: ["charts/Euc10Mile_PovRate25_Map.html", "400"] # second argument is the height
  folium-chart-1: ["charts/LowAccessTracts_Map.html", "400"] # second argument is the height
read_time: true
toc: false
toc_sticky: false
---


*Accessibility* is defined more than just spatial impedence but other socioeconomic variables, such as poverty level, household income, and vehicle ownership. Paying for 450-530 dollars for driver education and training may be a financial burden for a low-income family. In this section, I will overly socioeconomic characteristics to driver education access zones.

## Reading Socioeconomic Data

**Ohio Census Tract Socioeconomic Data** is retrived from the 2017-2021 American Community Surveys (ACS) five-year estimates that contains socioeconomic variables at tract level. The data can be read via Census API into Python using `cenpy`. This study is interested in socioeconomic variables, including

- B01001_001E: TOTAL POPULATION

- B19013_001E: MEDIAN HOUSEHOLD INCOME IN THE PAST 12 MONTHS (IN 2021 INFLATION-ADJUSTED DOLLARS)

- B08201_001E: TOTAL HOUSEHOLD

- B08201_002E: TOTAL HOUSEHOLDS NO VEHICLE AVAILABLE

- B17001_001E: TOTAL POPULATION

- B17001_002E: TOTAL POPULATION INCOME IN THE PAST 12 MONTHS BELOW POVERTY LEVEL

To read in and process ACS data, use the code below:

```python
acs = cenpy.remote.APIConnection("ACSDT5Y2021") 
variables = ["NAME", "B19013_001E", "B08201_001E", "B08201_002E", "B17001_001E", "B17001_002E"]

# Pull data by census tract
census_data = acs.query(
    cols=variables,
    geo_unit="tract:*",
    geo_filter={"state": "39",
                "county": "*"},
)

# Convert to the data to floats
for col in variables[1:]:
    census_data[col] = census_data[col].astype(float)

# The poverty rate and percent HH without vehicles
census_data["poverty_rate"] = census_data["B17001_002E"] / census_data["B17001_001E"]
census_data["pct_no_veh"] = census_data["B08201_002E"] / census_data["B08201_001E"]

census_data.rename(columns = {'B19013_001E':'med_income', 'B08201_001E':'tot_hh', 
                              'B08201_002E':'hh_no_veh', 'B17001_001E':'tot_pop',
                              'B17001_002E':'bl_poverty'}, inplace = True)
```

**Poverty Rate Map**

![poverty-rate-map]({{ site.url }}{{ site.baseurl }}/charts/PovertyRateMap_basic.png)


**Median Household Income Map**

![median-household-income-map]({{ site.url }}{{ site.baseurl }}/charts/MedianIncomeMap_basic.png)

**Percent Household without Vehicles**

![percent-household-no-vehicles-map]({{ site.url }}{{ site.baseurl }}/charts/PctNoVehMap_basic.png)

Based on the maps, there seem to be spatial correlation of these socioeconomic variables. In the next section, I will test their interdependencies and choose one variable to represent teens' socioeconomic access to/affordability of driver education.

<br/>

## Inderdependency of Socioeconomic Variables




## Identifying Low-Access and High-Poverty Tracts


<div id="folium-chart-1"></div>



<div id="folium-chart-2"></div>
