---
title: "Access to Teen Driver Education"
date: 2022-12-20
published: true
tags: [dataviz, folium, drivered, geocode, osm]
excerpt: ""
folium-loader:
  folium-chart-1: ["charts/TeenDrivingSchools.html", "400"] # second argument is the height
  folium-chart-2: ["charts/EuclideanDist_10miles.html", "400"] # second argument is the height
  folium-chart-3: ["charts/TravelDist_10miles_Dissolved.html", "400"] # second argument is the height
toc: true
toc_sticky: true
read_time: true
---

# Geocoding Ohio Teen Driving Schools

**Ohio Teen Driving Schools Data** is downloaded from the Ohio Department of Public Safety [website](https://apps.dps.ohio.gov/DETS/public/schools) as a list . It includes the active teen driving schools' names and addresses in Ohio State. In Dec. 2022, there are 391 schools registed in Ohio that offers teen driver education and Behind-the-Wheel training services.

From addresses to points, I used Google Maps API to geocode teen driving schools. [Code reference](https://www.natasshaselvaraj.com/a-step-by-step-guide-on-geocoding-in-python/)

<br/>

```python

# geocode the entire dataframe
def geocode(add):
    g = gmaps_key.geocode(add)
    lat = g[0]["geometry"]["location"]["lat"]
    long = g[0]["geometry"]["location"]["lng"]
    return (lat, long)

schools['Geocoded'] = schools['Address'].apply(geocode)


```

After multiple rounds of cleaning and back-ward geocoding, all of the 390 driving schools in Ohio are geocoded as point features by Google Maps API. As the map below shows, there seems a clustering pattern of driving schools.

<div id="folium-chart-1"></div>

<br/>

## Creating 10-Mile Buffers of Teen Driving Schools






<div id="folium-chart-2"></div>





<div id="folium-chart-3"></div>



