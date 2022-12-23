---
title: "Access to Teen Driver Education"
date: 2022-12-22
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

## Geocoding Ohio Teen Driving Schools

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

After multiple rounds of cleaning and back-ward geocoding, all of the 390 driving schools in Ohio are geocoded as point features by Google Maps API. As the map below shows, there seems a clustering pattern of driving schools. Hovering over the driving schools in the interactive map, we can see many driving schools are loacted in or affiliated with high schools. By keyword searching, we can find at least 37 high school-based driving schools.

<div id="folium-chart-1"></div>

<br/>

## Creating 10-Mile Buffers of Teen Driving Schools

In order to identify areas with easy to moderate spatial access to teen driver education and areas with low spatial access, I create buffers around driving schools using euclidean distance and travel distance, and travel time. Ten miles roughly correspond to 20-25 mins driving in urban places and shorter driving in less dense places. Therefore, I use 10 miles as a proxy for the threshold between moderate-high access and low access to teen driving schools. In the future, I will continue testing other distance barriers.

### Euclidean Distance
It's computationally easy to calculate euclidena distance in Python using the code below. Circular buffers are created around driving schools with overlapping. Many driving schools **are** clustered in urban cores. Their service areas (assume 10 miles) significantly overlap in big cities, including Columbus, Cincinnati, and Cleveland.

```python
# EPSG 26917 is NAD83 / UTM zone 17N (meters)
miles = 10
buffer_euc = schools_gdf.to_crs(epsg=26917).buffer(miles * 1609.34).to_crs(epsg=4326)
```

<div id="folium-chart-2"></div>

In the following analysis, I dissolve all buffers into one geometry by assuming that the density of driving schools is less important than the binary variable: with or withnot access to driver education. In the discussion of availability, it's fundamental that teens need only one available to them.

To dissolve buffers, you can use this code:

```python
buffer_euc_gdf = gpd.GeoDataFrame(buffer_euc)
buffer_euc_gdf = buffer_euc_gdf.rename(columns={0:'geometry'})
buffer_euc_gdf['new_column'] = 0
buffer_euc_dissolved = buffer_euc_gdf.dissolve(by='new_column')
buffer_euc_dissolved.crs='epsg:4326'
```

### Travel Distance

Euclidean distances are less preferable in urban studies as they lose accuracy in many situations. Travel distances based on street network, or even travel time, are better to represent people's actual spatial impedence. Therefore, I attempt to improve my teen diver education access zones by using OpenStreetMap via `osmnx` functions. There are conceptually two steps to calculate 10-mile tavel radius: (1) to calculate how much of the network we can reach within 10 miles of driving schools, and (2) to draw an accurate boundary around this region. [Reference](https://pythoncharmers.com/blog/travel-distance-python-with-geopandas-folium-alphashape-osmnx-buffer.html)

```python
# generate 10-mile travel boundaries for all teen driving schools
miles = 10
buffer_td = gpd.GeoDataFrame()


for index, row in bounds_euc.iterrows():
    n = int(row['index'])
    print("Completing travel distance calculation for school # "+str(n))

    region = ox.graph_from_bbox(row['maxy'], row['miny'], 
                                row['minx'], row['maxx'])
    
    center_node = ox.nearest_nodes(G = region, 
                                   Y = schools_gdf.loc[n, 'geometry'].y, 
                                   X = schools_gdf.loc[n, 'geometry'].x)
    
    region = ox.project_graph(region, to_crs='epsg:26917')
    
    
    distances = np.arange(0, miles * 1609.34, 804.67)
    
    next(iter(region.edges(data=True)))
    
    subgraph = nx.ego_graph(region, center_node, radius= (miles * 1609.34), distance='length')
    
    points = pd.DataFrame([(data['x'], data['y']) for _, data in subgraph.nodes(data=True)], columns=['x', 'y'])
    point_geoms = gpd.GeoSeries(points.apply(Point, axis='columns'))
    point_geoms.crs='epsg:26917'
    
    
    bounding_poly = alphashape.alphashape(point_geoms, 0.001)
    
    boundings = gpd.GeoDataFrame(
    [
        {'index': n, 
         'geometry': bounding_poly}
    ],
    crs='epsg:26917',
    geometry='geometry'
    )

    
    buffer_td = pd.concat([buffer_td, boundings], axis=0)
```
Due to time constraints, I wasn't able to calculate all boundaries using travel time. One travel time buffer calculation costs roughly 8-10 minutes on my machine, and the estimated total time to calculate all buffers is over 60 hours. I will give it another strike when the computing power is freed up. The map below shows travel time buffers of the first 22 driving schools in comparison to euclidean distance buffers of all driving schools.

When zooming to the driving school at topright coner near Cleveland City, we can see how much the travel distance and euclidean distance are different. It will be worthy to complete the travel distance calculation in the future.

<div id="folium-chart-3"></div>



