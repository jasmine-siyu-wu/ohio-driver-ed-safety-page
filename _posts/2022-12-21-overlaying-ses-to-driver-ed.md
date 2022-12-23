---
title: "Overlaying Socioeconomic Status to Access to Driver Education"
date: 2022-12-21
published: true
tags: [dataviz, folium, acs, ses, drivered]
excerpt: " "
altair-loader:
  altair-chart-1: "charts/measlesAltair.json"
folium-loader:
  folium-chart-1: ["charts/PovertyRateMap.html", "400"] # second argument is the height
  folium-chart-2: ["charts/LowAccessTracts_Map.html", "400"] # second argument is the height
toc: false
toc_sticky: false
---



This post will show examples of embedding interactive charts produced using [Altair](https://altair-viz.github.io) and [Hvplot](https://hvplot.pyviz.org/).

## Altair Example

Below is a chart of the incidence of measles since 1928 for the 50 US states.

<div id="altair-chart-1"></div>

This was produced using Altair and embedded in this static web page. Note that you can also display Python code on this page:

```python
import altair as alt
alt.renderers.enable('notebook')
```

## HvPlot Example

Lastly, the measles incidence produced using the HvPlot package:

<div id="folium-chart-1"></div>



<div id="folium-chart-2"></div>
