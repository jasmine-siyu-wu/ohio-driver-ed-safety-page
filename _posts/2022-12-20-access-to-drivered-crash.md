---
title: "Access to Driver Education in Relation to Safety"
date: 2022-12-20
published: true
tags: [dataviz, folium, crash, heatmap, drivered]
excerpt: " "
altair-loader:
  altair-chart-1: "charts/measlesAltair.json"
folium-loader:
  folium-chart-1: ["charts/CrashesHeatMap.html", "400"] # second argument is the height
  folium-chart-2: ["charts/CrashesHeatMap_YoungDrivers.html", "400"] # second argument is the height
  folium-chart-3: ["charts/LowAccessTracts_CrashMap.html", "400"] # second argument is the height
  folium-chart-4: ["charts/LowAccessTracts_HighRiskTracts.html", "400"] # second argument is the height
read_time: true
toc: true
toc_sticky: true
---

After identifing low-driver-education-access neighborhoods, in the last step, I will introduce crash data to understand the relationship between driver education and traffic safety in Ohio, especially for teens.

## Loading Crash Data in 2021

**Ohio Crash Data** contains Ohio traffic crashes’ coordinates, time, crash type, crash severity, locations, emphasis areas, drivers, vehicles, etc. In 2021, there were 71,024 non-property-damage-only crashes in Ohio, including fatal, severe injury suspected, minor injury suspected, and injury possible. The data were downloaded and stored as .csv files from [Ohio Department of Transportation TIMS GIS Crash Analysis Tool (GCAT)](https://gis.dot.state.oh.us/tims/CrashAnalytics/Search_).

The first look at Ohio crashes point map can already show some clustering of crashes in major cities.

![crashes-point-map]({{ site.url }}{{ site.baseurl }}/charts/Crashes_PointMap.png)

Comparing young-driver crashes with older-driver crashes, we can see that young driver-involved crashes almost account for a third to half of all crashes. The share of young-driver crashes is especially high for fatal crashes.

![young-driver-crash-severity]({{ site.url }}{{ site.baseurl }}/charts/YoungDrivers_CrashSeverity.png)


<br/>

## Crash Hot Spots


Plotting heatmaps of crashes based on severity level, we can see that there are discrepencies between the hotspots of different types of crashes. The common sens in transportation safety planning is to focus on fatal and severely injured crashes. However, for teen drivers, less severe crashes are also important as they are more likely to take risks and react inappropriately to potential hazards.

<div id="folium-chart-1"></div>

Looking specifically at young-driver crashes' heatmap, we can see the hotspots of young-driver crashes roughly align with all crashes' hotspots.

<div id="folium-chart-2"></div>


<br/>


## K-Means Clustering

The last step of investigating crashes' spatial distribution pattern in Ohio State is K-means clustering analysis. I use different severity crashes and teen driver crashes as features to classify 10 clusters in Ohio (10 clusters due to a large geography).

```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler


# Scale the data features we want
scaler = StandardScaler()
scaled_crash_data = scaler.fit_transform(crash_tract[['Fatal Injuries', 'Severe Injuries', 
                                                      'Minor Injuries', 'Possible Injuries', 'Young Drivers']])
```

![kmeans-map]({{ site.url }}{{ site.baseurl }}/charts/Kmeans_Map.png)

To better visualize and understand clusters, I categorize 10 clusters into 3 groups base on the average tract-level crashes in each cluster: low risk, moderate risk, and high risk. The final map highlights different levels of traffic crash considering densities of teen driver crashes.


![kmeans-3level-map]({{ site.url }}{{ site.baseurl }}/charts/Kmeans_3level_Map.png)



<br/>


## Driver Education Access Zones vs. Crash Risk Zones

Overlaying low-driver-education-access tracts over crash heatmap, we can see that low access zones appear to be located at the edges of hotspots.

<div id="folium-chart-3"></div>

Comparing low-driver-education-access  zones versus crash risk zones, we can more clearly see spatial relationships between the two communities. There is a diverity of crash risk zones of the identified low-driver-education-access zones. Yet, most of the low-access zones don't fall into the high-crash-risk category.

<div id="folium-chart-4"></div>

Of the 50 identified low-driver-education-access tracts, 3 are of moderate crash risk, 13 are of low crash risk, and 34 are of no crash risk. That is, the tracts of low access to driver education and of moderate crash risk need to be prioritized when agencies plan for teen traffic safety interventions. Their driving behaviors may be influenced by the crash environment, or they may be directly endangered by the environment.

![low-access-y-risk]({{ site.url }}{{ site.baseurl }}/charts/LowAccess-byRisk.png)


## Takeaways






