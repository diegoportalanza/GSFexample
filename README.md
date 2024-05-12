# GSFexample
a very easy tool to PLOT

# This example was created using Conda/Spyder with the aid of AI.

#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Created on Sun May 12 10:48:55 2024

@author: diego
"""

from siphon.catalog import TDSCatalog
import xarray as xr

base_url = 'http://thredds.ucar.edu/thredds/catalog/grib/NCEP/GFS/Global_0p25deg/catalog.xml'
catalog = TDSCatalog(base_url)
print(list(catalog.datasets))

# access the data

gfs_dataset = catalog.datasets['Latest Collection for GFS Quarter Degree Forecast']


#subset service

ncss = gfs_dataset.subset()

query = ncss.query()

# Geographic R for Equador

query.lonlat_box(north=2, south=-5, east=-75, west=-81)

# parameters
query.variables('Temperature_surface', 'Precipitation_rate_surface', 'u-component_of_wind_isobaric', 'v-component_of_wind_isobaric')

# format
query.accept('netcdf4')

#retrieve data
data = ncss.get_data(query)

# import dataset

dataset = xr.open_dataset(xr.backends.NetCDF4DataStore(data))

# print data

print(dataset)


temperature = dataset['Temperature_surface']
precipitation = dataset['Precipitation_rate_surface']



import matplotlib.pyplot as plt
import cartopy.crs as ccrs
import cartopy.feature as cfeature



# Create a new plot with a specific projection
fig, ax = plt.subplots(figsize=(10, 10), subplot_kw={'projection': ccrs.PlateCarree()})
ax.coastlines()

# Convert dataset longitudes from 0-360 to -180 to +180 format for plotting
longitude = dataset['longitude'] % 360
dataset.coords['longitude'] = longitude

# Adjust the plot call to match the dataset's coordinate names
temperature = dataset['Temperature_surface'].isel(time1=0)  # Selecting the first time step for simplicity
temperature.plot(ax=ax, transform=ccrs.PlateCarree(), x='longitude', y='latitude', add_colorbar=True, cmap='coolwarm')

# Set the map extent to tightly focus on Ecuador
ax.set_extent([-81, -75, -5, 2], crs=ccrs.PlateCarree())

# Add features for better visualization
ax.add_feature(cfeature.BORDERS, linestyle=':')
ax.add_feature(cfeature.COASTLINE)

# Optionally, refine the display of gridlines and labels
gl = ax.gridlines(draw_labels=True, linewidth=0.5, color='gray', alpha=0.5, linestyle='--')
gl.top_labels = False
gl.right_labels = False

# Show plot
plt.show()




import matplotlib.pyplot as plt
import cartopy.crs as ccrs
import cartopy.feature as cfeature

# Create a figure with two subplots
fig, axes = plt.subplots(nrows=1, ncols=2, figsize=(20, 10), subplot_kw={'projection': ccrs.PlateCarree()})

# Convert dataset longitudes from 0-360 to -180 to +180 format for plotting
longitude = dataset['longitude'] % 360
dataset.coords['longitude'] = longitude

# Convert temperature from Kelvin to Celsius
temperature_celsius = dataset['Temperature_surface'].isel(time1=0) - 273.15
precipitation = dataset['Precipitation_rate_surface'].isel(time1=0)

# Plot temperature (Celsius) on the first subplot
axes[0].coastlines(resolution='10m')
axes[0].add_feature(cfeature.BORDERS, linestyle=':')
axes[0].add_feature(cfeature.COASTLINE)
axes[0].set_extent([-81, -75, -5, 2], crs=ccrs.PlateCarree())
temperature_plot = temperature_celsius.plot(ax=axes[0], transform=ccrs.PlateCarree(), x='longitude', y='latitude', 
                                            add_colorbar=False, cmap='coolwarm')
cbar_temp = fig.colorbar(temperature_plot, ax=axes[0], orientation='vertical', shrink=0.8)
cbar_temp.set_label('Temperature (°C)')
gl_temp = axes[0].gridlines(draw_labels=True, linewidth=0.5, color='gray', alpha=0.5, linestyle='--')
gl_temp.top_labels = False
gl_temp.right_labels = False

# Plot precipitation (mm/hour) on the second subplot
axes[1].coastlines(resolution='10m')
axes[1].add_feature(cfeature.BORDERS, linestyle=':')
axes[1].add_feature(cfeature.COASTLINE)
axes[1].set_extent([-81, -75, -5, 2], crs=ccrs.PlateCarree())
precipitation_plot = precipitation.plot(ax=axes[1], transform=ccrs.PlateCarree(), x='longitude', y='latitude', 
                                        add_colorbar=False, cmap='Greens')
cbar_precip = fig.colorbar(precipitation_plot, ax=axes[1], orientation='vertical', shrink=0.8)
cbar_precip.set_label('Precipitation Rate (mm/hr)')
gl_precip = axes[1].gridlines(draw_labels=True, linewidth=0.5, color='gray', alpha=0.5, linestyle='--')
gl_precip.top_labels = False
gl_precip.right_labels = False

# Add signature
fig.text(0.8, 0.08, '@diegoportalanza', ha='center', va='center', fontsize=12, color='gray')

# Add main title
fig.suptitle('GFS Forecast - Temperature and Precipitation', fontsize=30)

# Adjust layout to prevent overlapping elements
plt.tight_layout(rect=[0, 0.03, 1, 0.95])  # Adjust the rect to make room for the suptitle
plt.show()




