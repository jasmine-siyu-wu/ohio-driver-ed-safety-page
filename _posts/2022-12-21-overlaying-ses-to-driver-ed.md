---
title: "Overlaying Socioeconomic Status to Access to Driver Education"
date: 2022-12-21
published: true
tags: [dataviz, folium, acs, ses, drivered]
excerpt: " "
folium-loader:
  folium-chart-1: ["charts/Euc10Mile_PovRate20_Map.html", "400"] # second argument is the height
  folium-chart-2: ["charts/LowAccessTracts_Map.html", "400"] # second argument is the height
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



<br/>

## Inderdependency of Socioeconomic Variables

Based on the maps above, there seem to be spatial correlation of these socioeconomic variables. Therefore, before identifying low-access tracts, I test their interdependencies and choose the most representative variable to represent teens' socioeconomic access to/affordability of driver education. 


```python
# Compute the correlation matrix
corr = census_data[["poverty_rate100", "med_income", "pct_no_veh100"]].corr()
labels = ['Poverty Rate', 'Median Household Income', 'Pct Household without Vehicles']
corr.index = labels
corr.columns = labels

# Generate a custom diverging colormap
cmap = sns.diverging_palette(230, 20, as_cmap=True)

#plotting the heatmap for correlation
ax = sns.heatmap(corr, cmap=cmap, annot=True, 
                 square=True, linewidths=.5, cbar_kws={"shrink": .5})
```

![ses-heatmap]({{ site.url }}{{ site.baseurl }}/charts/SES_Heatmap.png)


All three varibles are moderately to highly correlated with each other. Amongst them, poverty rate is highly correlated with both other two variables: median household income and percent household without vehicles. It can been seen as representative the other two variables. That is, I will use poverty rate to measure socioeconomic inacessibility to teen driver education.

The 75th percentile of poverty rate over all Ohio Census tracts is 21.4%, meaning that about a quarter of tracts have a poverty rate over 20%. Living in such a tract may indicate a centain level of financial disadvantages.

## Identifying Long-Distance and High-Poverty Tracts

So far, I have got my two measures of low access to driver education at Census tract level: living 10 miles away from a teen driving school and living in a high-poverty neighborhood (poverty rate > 20%). The map below displays the areas qualifying these two criteria.

<div id="folium-chart-1"></div>

By spatially joining the two layers, we can identifying Census tracts qualifying both criteria, which are then classified as low-driver-education-access neighborhoods.

```python
low_access_tract = gpd.sjoin(tract_pov20, buffer_euc_clip, predicate='intersects', how='left') 
low_access_tract = low_access_tract.drop(columns = 'index_right', axis=1)
low_access_tract['Euc_10Mi'] = low_access_tract['Euc_10Mi'].fillna("N")
low_access_tract = low_access_tract[low_access_tract["Euc_10Mi"] == "Y"]
```

<div id="folium-chart-2"></div>
