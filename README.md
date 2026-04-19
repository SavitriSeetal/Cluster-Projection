# Title : California City Population Visualization

```{r}
import geopandas as gpd
import pandas as pd
import matplotlib.pyplot as plt
import folium
from folium.plugins import MarkerCluster
```
```{r}
gdf = gpd.read_file("C:/SSM/GES 775/Assignments/output_shapefile.shp")
```

# Create a GeoDataFrame with point geometries
```{r}
geometry = gpd.points_from_xy(gdf['lon'], gdf['lat'])
gdf_with_points = gpd.GeoDataFrame(gdf, geometry=geometry, crs="EPSG:4326")

gdf_with_points['population'] = pd.to_numeric(gdf_with_points['population'], errors='coerce')
```

# Plotting the GeoDataFrame
```{r}
world = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))
north_america = world[world['continent'] == 'North America']

gdf_with_points['pop_level'] = pd.cut(gdf_with_points['population'],
                                      bins=[0, 100000, 1000000, float('inf')],
                                      labels=['small', 'medium', 'large'])
```

# Define colors for different population levels
```{r}
pop_level_colors = {'small': 'blue', 'medium': 'green', 'large': 'red'}
gdf_with_points['pop_color'] = gdf_with_points['pop_level'].map(pop_level_colors)

world = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))
north_america = world[world['continent'] == 'North America']
```

# Zoom in on California
```{r}
california_extent = [-125, 32, -114, 43]  # [minx, miny, maxx, maxy]
ax = north_america.plot(figsize=(10, 6), color='lightgray', edgecolor='black')
ax.set_xlim(california_extent[0], california_extent[2])
ax.set_ylim(california_extent[1], california_extent[3])

gdf_with_points.plot(ax=ax, marker='o', color=gdf_with_points['pop_color'], markersize=7, alpha=0.7)
plt.show()

plt.title('Cities in California - Population Levels')
plt.show(block=True)
plt.interactive(False)
```

# Create a folium map centered on California
```{r}
m = folium.Map(location=[36.7783, -119.4179], zoom_start=6)
marker_cluster = MarkerCluster().add_to(m)

for idx, row in gdf_with_points.iterrows():
    popup_text = f"{row['city']}, Population: {row['population']}"
    tooltip_text = f"City: {row['city']} | Population: {row['population']:,} | Population Level: {row['pop_level']}"
    folium.Marker(
        location=[row['lat'], row['lon']],
        popup=popup_text,
        tooltip=tooltip_text,
        icon=folium.Icon(color=row['pop_color'])
    ).add_to(marker_cluster)
```
# Save the map as an HTML file or display it
m.save("C:/SSM/GES 775/Assignments/map_with_markers_popups_tooltips.html")
