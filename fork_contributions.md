# New Notebooks & Docs in MAGICA-School-2026 Forks

_Source: [OpenScienceComputing/MAGICA-School-2026](https://github.com/OpenScienceComputing/MAGICA-School-2026) · 24 forks checked · 82 new file(s) found_


## [SilverFox275/MAGICA-School-2026](https://github.com/SilverFox275/MAGICA-School-2026)

### `cmems_salinity_tunisia (1).ipynb`

[View on GitHub](https://github.com/SilverFox275/MAGICA-School-2026/blob/main/cmems_salinity_tunisia (1).ipynb)

# Copernicus Marine — Sea Water Salinity, Gulf of Tunisia

**Product:** `MEDSEA_MULTIYEAR_PHY_006_004`  
**Dataset:** `cmems_mod_med_phy-sal_my_4.2km_P1D-m`  
**Variable:** `so` — sea water practical salinity (PSU) at 1 m depth  
**Period:** 2026-01-01 → 2026-03-31  
**Domain:** Gulf of Tunisia — lon 10°–10.9°E, lat 36.6°–37.2°N  
**Model:** Mediterranean Sea Physics Reanalysis (MFS E3R1I, 4.2 km, daily mean)

```python
%pip install copernicusmarine hvplot matplotlib xarray --quiet
```

```python
import copernicusmarine

# Saves credentials to ~/.copernicusmarine so you are not prompted again
copernicusmarine.login(
    username="mazzabi1",
    password="YOUR_PASSWORD_HERE",  # <-- replace with your Copernicus Marine password
    force_overwrite=True,
)
```

## 1. Open the salinity dataset (lazy, no download yet)

```python
import warnings
warnings.filterwarnings("ignore")

ds_sal = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-sal_my_4.2km_P1D-m",
    variables=["so"],
    minimum_longitude=10.0,
    maximum_longitude=10.9,
    minimum_latitude=36.6,
    maximum_latitude=37.2,
    start_datetime="2026-01-01T00:00:00",
    end_datetime="2026-03-31T00:00:00",
)
ds_sal
```

## 2. Inspect the dataset

```python
print("Variables :", list(ds_sal.data_vars))
print("Dimensions:", dict(ds_sal.sizes))
print("Time range:", str(ds_sal.time.values[0])[:10], "->", str(ds_sal.time.values[-1])[:10])
print("Depth levels:", ds_sal.depth.values)
```

## 3. Select surface level (1 m depth) and load into memory

```python
%%time
so = ds_sal["so"].isel(depth=0).load()

print(f"Salinity min : {float(so.min()):.2f} PSU")
print(f"Salinity max : {float(so.max()):.2f} PSU")
print(f"Salinity mean: {float(so.mean()):.2f} PSU")
```

## 4. Spatial map — salinity on the first day (2026-01-01)

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(11, 4.5))
so.isel(time=0).plot(ax=ax, cmap="Blues_r", cbar_kwargs={"label": "PSU"})
ax.set_title(f"Surface salinity (so, 1 m) — {str(so.time.values[0])[:10]}")
ax.set_xlabel("Longitude")
ax.set_ylabel("Latitude")
plt.tight_layout()
plt.show()
```

---

### `copernicus_marine-Tunisia.ipynb`

[View on GitHub](https://github.com/SilverFox275/MAGICA-School-2026/blob/main/copernicus_marine-Tunisia.ipynb)

# Copernicus Marine Service: Access Earth Observation Data
* Need to have an account on Copernicus Marine
* The ~/.netrc file contains the account credentials: your login and password for [Copernicus Marine](https://marine.copernicus.eu/)
```
$ more ~/.netrc
machine auth.marine.copernicus.eu
  login xxxxxxxxxx 
  password xxxxxxxxxxxxx
```

```python
import copernicusmarine
import hvplot.xarray
```

```python
# turn of some annoying warnings
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
dataset_id = 'SST_MED_SST_L3S_NRT_OBSERVATIONS_010_012_b'
```

```python
ds = copernicusmarine.open_dataset(dataset_id)
```

```python
%%time
da = ds['adjusted_sea_surface_temperature'].sel(time='2025-10-01 16:00', method='nearest').load() - 273.15
```

```python
da.hvplot(x='longitude', y='latitude', rasterize=True, geo=True, cmap='turbo', tiles='OSM')
```

## Local CMEMS file: Mediterranean Physics (thetao & bottomT)

**Dataset**: `cmems_mod_med_phy-temp_my_4.2km_P1D-m` (Mediterranean Multiyear Physics, 4.2 km, daily).

Contains **90 days (Jan–Mar 2026)** of:
- `thetao` — potential temperature at 1 m depth
- `bottomT` — bottom temperature

```python
import xarray as xr
import numpy as np
import matplotlib.pyplot as plt
import hvplot.xarray  # noqa: F401

local_nc = 'cmems_mod_med_phy-temp_my_4.2km_P1D-m_1776863773209.nc'
ds_med = xr.open_dataset(local_nc)
ds_med
```

```python
# Inspect available variables, dimensions, and time range
print('Variables :', list(ds_med.data_vars))
print('Dimensions:', dict(ds_med.sizes))
print('Time range:', str(ds_med.time.values[0])[:10], '->', str(ds_med.time.values[-1])[:10])
```

### 1. Spatial map — surface temperature (`thetao` at 1 m) on the first day

```python
# Select first time step; squeeze depth if present
sst0 = ds_med['thetao'].isel(time=0)
if 'depth' in sst0.dims:
    sst0 = sst0.isel(depth=0)

fig, ax = plt.subplots(figsize=(11, 4.5))
im = sst0.plot(ax=ax, cmap='turbo', cbar_kwargs={'label': '°C'})
ax.set_title(f"Surface potential temperature (thetao, 1 m) — {str(sst0.time.values)[:10]}")
ax.set_xlabel('Longitude'); ax.set_ylabel('Latitude')
plt.tight_layout(); plt.show()
```

---

### `copernicus_marine_EO_Tunis.ipynb`

[View on GitHub](https://github.com/SilverFox275/MAGICA-School-2026/blob/main/copernicus_marine_EO_Tunis.ipynb)

# Copernicus Marine Service: Access Earth Observation Data
* Need to have an account on Copernicus Marine
* The ~/.netrc file contains the account credentials: your login and password for [Copernicus Marine](https://marine.copernicus.eu/)
```
$ more ~/.netrc
machine auth.marine.copernicus.eu
  login gmozzi
  password the_usual_one
```

```python
import copernicusmarine
import hvplot.xarray
```

```python
# turn of some annoying warnings
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
catalog = copernicusmarine.describe(contains=["Mediterranean"])                                                          
for product in catalog.products:                                                                                         
  for dataset in product.datasets:                                                                                     
      print(dataset.dataset_id)
```

```python
dataset_id = 'cmems_obs-sl_eur_phy-ssh_nrt_allsat-l4-duacs-0.0625deg_P1D'
```

```python
ds = copernicusmarine.open_dataset(dataset_id)
```

```python
list(ds.data_vars)
```

```python
%%time
da = ds['adt'].sel(time='2026-04-22 16:00', method='nearest').load()
```

```python
da.hvplot(x='longitude', y='latitude', rasterize=True, geo=True, cmap='turbo', tiles='OSM')
```

```python
bbox = [10.0, 36.6, 10.9, 37.2]                                                                                          
                                                                                                                       
da_tunis = da.sel(                                                                                                       
  longitude=slice(bbox[0], bbox[2]),                                                                                   
  latitude=slice(bbox[1], bbox[3]),                                                                                    
)                                                                                                                        
                                                                                                                       
da_tunis.hvplot(x='longitude', y='latitude', rasterize=True, geo=True, cmap='turbo', tiles='OSM')
```

---

### `gulf_of_tunis_tutorial.ipynb`

[View on GitHub](https://github.com/SilverFox275/MAGICA-School-2026/blob/main/gulf_of_tunis_tutorial.ipynb)

# Gulf of Tunis — Multi-Source Ocean Data Tutorial

This notebook shows how to access and visualize ocean observations over the **Gulf of Tunis** from two cloud-native data services:

| # | Variable | Sensor / Product | Service |
|---|----------|------------------|---------|
| 1 | Sea Surface Temperature (gridded, L3) | CMEMS SST MED NRT | Copernicus Marine |
| 2 | Sea water salinity (gridded, model) | CMEMS MED Physics MY 4.2 km | Copernicus Marine |
| 3 | Sea Surface Temperature (swath, L2) | Sentinel-3A SLSTR | Planetary Computer (STAC) |
| 4 | Chlorophyll-a (swath, L2) | Sentinel-3A OLCI | Planetary Computer (STAC) |

**Learning objectives:**
- Use the `copernicusmarine` Python client to stream gridded satellite and model data
- Plot basin-averaged time series and interactive spatial maps of a model variable (salinity)
- Use `pystac_client` to search a STAC API and open assets with `xarray`
- Clip, mask, and visualize 2-D swath data over a small area of interest
- Compare SST from two independent sources over the same region

## 0. Shared setup

We define the **bounding box** once and reuse it throughout. All three sections clip their data to this box.

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)

import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import hvplot.xarray
import hvplot.pandas

# Area of interest: Gulf of Tunis  [lon_min, lat_min, lon_max, lat_max]
bbox = [10.0, 36.6, 10.9, 37.2]
print(f"Gulf of Tunis bbox: lon [{bbox[0]}, {bbox[2]}]  lat [{bbox[1]}, {bbox[3]}]")
```

---
## 1. CMEMS gridded SST (Copernicus Marine Service)

The **Copernicus Marine Service** (CMEMS) distributes processed, gridded satellite products via an OPeNDAP/Zarr backend. The `copernicusmarine` Python client lets us stream only the data we need — no download required.

We use:
- **Dataset:** `SST_MED_SST_L3S_NRT_OBSERVATIONS_010_012_b`
- **Variable:** `adjusted_sea_surface_temperature` (Kelvin → °C)
- **Grid:** ~0.01° regular lat/lon over the Mediterranean

> **Credentials:** the `~/.netrc` file must contain your Copernicus Marine username and password.

```python
import copernicusmarine

dataset_id = "SST_MED_SST_L3S_NRT_OBSERVATIONS_010_012_b"

ds_cmems = copernicusmarine.open_dataset(dataset_id)
print(list(ds_cmems.data_vars))
```

Select a single time step and clip to the Gulf of Tunis bbox using `.sel()`.

> If the latitude slice returns empty data, the axis may be stored in descending order — swap the limits: `slice(bbox[3], bbox[1])`.

```python
time_val="2026-01-01 16:00"
da_sst_cmems = (
    ds_cmems["adjusted_sea_surface_temperature"]
    .sel(time=time_val, method="nearest")
    .sel(
        longitude=slice(bbox[0], bbox[2]),
        latitude=slice(bbox[1], bbox[3]),
    )
    .load()
) - 273.15  # Kelvin → Celsius

print(f"Shape: {da_sst_cmems.shape}")
print(f"SST range: {float(da_sst_cmems.min()):.1f} – {float(da_sst_cmems.max()):.1f} °C")
```

---

### `planetary_computer_Tunis.ipynb`

[View on GitHub](https://github.com/SilverFox275/MAGICA-School-2026/blob/main/planetary_computer_Tunis.ipynb)

# Cloud Native Satellite Data Demo 1
* Use pystac_client to search the STAC API at Planetary Computer
* Use odc.stac to load the data
* Use hvplot to visualize the data

```python
import pystac_client
import planetary_computer
from rich.table import Table
import hvplot.xarray

# Connect to the Planetary Computer STAC API
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)
```

```python
collections = list(catalog.get_all_collections())
collections.sort(key=lambda c: c.id)
table = Table("ID", "Title", title="Planetary Computer collections")
for collection in collections:
    table.add_row(collection.id, collection.title)
#table
```

```python
# Define our area of interest (AOI) - a rough box around Cape Cod gulf of tunisia
bbox = [10.0, 36.6, 10.9, 37.2]
```

```python
# Let's search for Sentinel-3 SLSTR WST (Sea Surface Temperature) Level-2 data  NADIA
search = catalog.search(
    collections=["sentinel-3-slstr-wst-l2-netcdf"],
    bbox=bbox,
    datetime="2026-01-01/2026-04-21",
)

# Get the results and see what we found
items = search.item_collection()
print(f"Found {len(items)} scenes matching your criteria.")

# Let's inspect the assets of the first item to see the available data bands
if items:
    first_item = items[0]
    print("\nAssets for the first scene:")
    for asset_key, asset in first_item.assets.items():
        print(f"- {asset_key}: {asset.title}")
```

```python
import xarray as xr
import fsspec

# Prendre le premier item
item = items[0]

# Signer l'URL
asset = item.assets["l2p"]
signed_asset = planetary_computer.sign(asset)

print(f"URL : {signed_asset.href}")

# Ouvrir avec xarray
ds = xr.open_dataset(fsspec.open(signed_asset.href).open())
print(ds)
```

---

### `planetary_computer_Tunis_chl.ipynb`

[View on GitHub](https://github.com/SilverFox275/MAGICA-School-2026/blob/main/planetary_computer_Tunis_chl.ipynb)

# Cloud Native Satellite Data Demo 1
* Use pystac_client to search the STAC API at Planetary Computer
* Use xarray to load Sentinel-3 OLCI WFR chlorophyll data
* Use hvplot to visualize chlorophyll-a (CHL-NN) over the Gulf of Tunis

**Future goal:** compare with Copernicus Marine gridded ocean colour (CMEMS L4) — same physical variable, different processing level.

```python
import pystac_client
import planetary_computer
from rich.table import Table
import hvplot.xarray

# Connect to the Planetary Computer STAC API
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)
```

```python
collections = list(catalog.get_all_collections())
collections.sort(key=lambda c: c.id)
table = Table("ID", "Title", title="Planetary Computer collections")
for collection in collections:
    table.add_row(collection.id, collection.title)
table
```

```python
# Area of interest — Gulf of Tunis
bbox = [10.0, 36.6, 10.9, 37.2]
```

```python
search = catalog.search(
    collections=["sentinel-3-olci-wfr-l2-netcdf"],
    bbox=bbox,
    datetime="2026-01-01/2026-04-21",
)

items = search.item_collection()
print(f"Found {len(items)} Sentinel-3 SRAL WAT passes.")

if items:
    first_item = items[0]
    print(f"\nFirst pass: {first_item.datetime}")
    print("\nAssets:")
    for asset_key, asset in first_item.assets.items():
        print(f"- {asset_key}: {asset.title}")
```

```python
import xarray as xr
import fsspec

# Take the first item
item = items[0]

def open_asset(item, key):
    asset = planetary_computer.sign(item.assets[key])
    return xr.open_dataset(fsspec.open(asset.href).open())

ds = open_asset(item, "chl-nn")
geo = open_asset(item, "geo-coordinates")

# Assign lat/lon from the geo-coordinates file as coordinates
ds = ds.assign_coords(
    latitude=(["rows", "columns"], geo["latitude"].values),
    longitude=(["rows", "columns"], geo["longitude"].values),
)

print(ds)
```

---

### `taranto-icechunk-append-glomoz.ipynb`

[View on GitHub](https://github.com/SilverFox275/MAGICA-School-2026/blob/main/taranto-icechunk-append-glomoz.ipynb)

# Create or append Virtual Icechunk Store from SHYFEM forecast NetCDF files

* This notebook appends Taranto SHYFEM forecast data to Icechunk store using date-based set logic.
* If no repo exists, a new one will be created with references to all the existing NetCDF files. 

**Append Methodology:**
1. **`set_repo`**: extract all dates currently present in the Icechunk store's `time` coordinate
2. **`set_cloud`**: Scan the S3 bucket for all available NOS files and extract their dates.
3. **`new_dates`**: Calculate the difference (`set_cloud - set_repo`) to determine exactly which days need to be written for creation or appended.

```python
import warnings
import os
import pandas as pd
import fsspec
import xarray as xr
from dotenv import load_dotenv

warnings.filterwarnings("ignore", category=UserWarning)
```

```python
import icechunk
from obstore.store import from_url
from virtualizarr import open_virtual_dataset
from virtualizarr.parsers import HDFParser
from obspec_utils.registry import ObjectStoreRegistry
from obstore.store import S3Store
```

```python
# Load credentials
_ = load_dotenv(f'{os.environ["HOME"]}/dotenv/protocoast.env')

# Configuration
storage_endpoint = os.environ['ENDPOINT_URL']

# Icechunk repo destination
storage_bucket = 'glomoz'
storage_name = 'taranto-icechunk-tubitak-v2'
bucket_url = f"s3://{storage_bucket}"

# Source NetCDF data location
data_bucket = 'protocoast-data'
data_bucket_url = f"s3://{data_bucket}"

# Setup Filesystem
fs = fsspec.filesystem('s3', anon=False, endpoint_url=storage_endpoint, 
                       skip_instance_cache=True, use_listings_cache=False)
```

```python
try:
    fs.ls(f'{storage_bucket}/icechunk')
except:
    pass
```

```python
# Define Icechunk Storage & Config
storage = icechunk.s3_storage(
    bucket=storage_bucket,
    prefix=f"icechunk/{storage_name}",
    from_env=True,
    endpoint_url=storage_endpoint,
    region='not-used',
    force_path_style=True,
)

config = icechunk.RepositoryConfig.default()
config.set_virtual_chunk_container(
    icechunk.VirtualChunkContainer(
        url_prefix=f"{data_bucket_url}/",
        store=icechunk.s3_store(region="not-used", anonymous=False, s3_compatible=True, 
                                force_path_style=True, endpoint_url=storage_endpoint),
    ),
)

credentials = icechunk.containers_credentials({f"{data_bucket_url}/": icechunk.s3_credentials(anonymous=False)})

store_obj = S3Store(
    bucket=data_bucket,
    endpoint=storage_endpoint,
    region="not-used",
)

registry = ObjectStoreRegistry({data_bucket_url: store_obj})
parser = HDFParser()
```

---


## [nadia-mkh/MAGICA-School-2026](https://github.com/nadia-mkh/MAGICA-School-2026)

### `cmems_salinity_tunisia.ipynb`

[View on GitHub](https://github.com/nadia-mkh/MAGICA-School-2026/blob/main/cmems_salinity_tunisia.ipynb)

# Copernicus Marine — Sea Water Salinity, Gulf of Tunisia

**Product:** `MEDSEA_MULTIYEAR_PHY_006_004`  
**Dataset:** `cmems_mod_med_phy-sal_my_4.2km_P1D-m`  
**Variable:** `so` — sea water practical salinity (PSU) at 1 m depth  
**Period:** 2026-01-01 → 2026-03-31  
**Domain:** Gulf of Tunisia — lon 10°–10.9°E, lat 36.6°–37.2°N  
**Model:** Mediterranean Sea Physics Reanalysis (MFS E3R1I, 4.2 km, daily mean)

```python
#%pip install copernicusmarine hvplot matplotlib xarray --quiet
```

```python
import copernicusmarine

# Saves credentials to ~/.copernicusmarine so you are not prompted again
copernicusmarine.login(
    username="nmkhinini",
    password="Nadiamkh73261260?",  # <-- replace with your Copernicus Marine password
    force_overwrite=True,
)
```

## 1. Open the salinity dataset (lazy, no download yet)

```python
import warnings
warnings.filterwarnings("ignore")

ds_sal = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-sal_my_4.2km_P1D-m",
    variables=["so"],
    minimum_longitude=10.0,
    maximum_longitude=10.9,
    minimum_latitude=36.6,
    maximum_latitude=37.2,
    start_datetime="2026-01-01T00:00:00",
    end_datetime="2026-03-31T00:00:00",
)
ds_sal
```

## 2. Inspect the dataset

```python
print("Variables :", list(ds_sal.data_vars))
print("Dimensions:", dict(ds_sal.sizes))
print("Time range:", str(ds_sal.time.values[0])[:10], "->", str(ds_sal.time.values[-1])[:10])
print("Depth levels:", ds_sal.depth.values)
```

## 3. Select surface level (1 m depth) and load into memory

```python
%%time
so = ds_sal["so"].isel(depth=0).load()

print(f"Salinity min : {float(so.min()):.2f} PSU")
print(f"Salinity max : {float(so.max()):.2f} PSU")
print(f"Salinity mean: {float(so.mean()):.2f} PSU")
```

## 4. Spatial map — salinity on the first day (2026-01-01)

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(11, 4.5))
so.isel(time=0).plot(ax=ax, cmap="Blues_r", cbar_kwargs={"label": "PSU"})
ax.set_title(f"Surface salinity (so, 1 m) — {str(so.time.values[0])[:10]}")
ax.set_xlabel("Longitude")
ax.set_ylabel("Latitude")
plt.tight_layout()
plt.show()
```

---

### `copernicus_marine-Tunisia.ipynb`

[View on GitHub](https://github.com/nadia-mkh/MAGICA-School-2026/blob/main/copernicus_marine-Tunisia.ipynb)

(could not fetch content)

---

### `gulf_of_tunis_tutorial.ipynb`

[View on GitHub](https://github.com/nadia-mkh/MAGICA-School-2026/blob/main/gulf_of_tunis_tutorial.ipynb)

# Gulf of Tunis — Multi-Source Ocean Data Tutorial

This notebook shows how to access and visualize ocean observations over the **Gulf of Tunis** from two cloud-native data services:

| # | Variable | Sensor / Product | Service |
|---|----------|------------------|---------|
| 1 | Sea Surface Temperature (gridded, L3) | CMEMS SST MED NRT | Copernicus Marine |
| 2 | Sea Surface Temperature (swath, L2) | Sentinel-3A SLSTR | Planetary Computer (STAC) |
| 3 | Chlorophyll-a (swath, L2) | Sentinel-3A OLCI | Planetary Computer (STAC) |

**Learning objectives:**
- Use the `copernicusmarine` Python client to stream gridded satellite data
- Use `pystac_client` to search a STAC API and open assets with `xarray`
- Clip, mask, and visualize 2-D swath data over a small area of interest
- Compare SST from two independent sources over the same region

## 0. Shared setup

We define the **bounding box** once and reuse it throughout. All three sections clip their data to this box.

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)

import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import hvplot.xarray
import hvplot.pandas

# Area of interest: Gulf of Tunis  [lon_min, lat_min, lon_max, lat_max]
bbox = [10.0, 36.6, 10.9, 37.2]
print(f"Gulf of Tunis bbox: lon [{bbox[0]}, {bbox[2]}]  lat [{bbox[1]}, {bbox[3]}]")
```

---
## 1. CMEMS gridded SST (Copernicus Marine Service)

The **Copernicus Marine Service** (CMEMS) distributes processed, gridded satellite products via an OPeNDAP/Zarr backend. The `copernicusmarine` Python client lets us stream only the data we need — no download required.

We use:
- **Dataset:** `SST_MED_SST_L3S_NRT_OBSERVATIONS_010_012_b`
- **Variable:** `adjusted_sea_surface_temperature` (Kelvin → °C)
- **Grid:** ~0.01° regular lat/lon over the Mediterranean

> **Credentials:** the `~/.netrc` file must contain your Copernicus Marine username and password.

```python
import copernicusmarine

dataset_id = "SST_MED_SST_L3S_NRT_OBSERVATIONS_010_012_b"

ds_cmems = copernicusmarine.open_dataset(dataset_id)
print(list(ds_cmems.data_vars))
```

Select a single time step and clip to the Gulf of Tunis bbox using `.sel()`.

> If the latitude slice returns empty data, the axis may be stored in descending order — swap the limits: `slice(bbox[3], bbox[1])`.

```python
time_val="2026-01-01 16:00"
da_sst_cmems = (
    ds_cmems["adjusted_sea_surface_temperature"]
    .sel(time=time_val, method="nearest")
    .sel(
        longitude=slice(bbox[0], bbox[2]),
        latitude=slice(bbox[1], bbox[3]),
    )
    .load()
) - 273.15  # Kelvin → Celsius

print(f"Shape: {da_sst_cmems.shape}")
print(f"SST range: {float(da_sst_cmems.min()):.1f} – {float(da_sst_cmems.max()):.1f} °C")
```

---

### `marine_heatwave_gulf_of_tunis.ipynb`

[View on GitHub](https://github.com/nadia-mkh/MAGICA-School-2026/blob/main/marine_heatwave_gulf_of_tunis.ipynb)

# Marine Heatwave Detection — Gulf of Tunis (2016–2025)

This notebook detects and analyses **marine heatwave (MHW) events** in the Gulf of Tunis over ten summer seasons (JAS: July–August–September, 2016–2025).

**Definition used:** a pixel is classified as *in heatwave* on day *t* if its sea surface temperature has been ≥ 27.5 °C for at least 5 consecutive days ending on day *t* (i.e., the 5-day trailing rolling minimum ≥ threshold).  An *event* is a contiguous temporal period in which at least one pixel satisfies this condition.

| Step | Description |
|------|-------------|
| 1 | Load daily SST from CMEMS reprocessed L3S (Gulf of Tunis bbox) |
| 2 | Detect heatwave pixels — rolling-minimum approach |
| 3 | Label and characterise events |
| 4 | Interactive statistics plots (hvplot) |
| 5 | SST map of the 3rd event (peak day) |
| 6 | Chlorophyll-a map for the same date (Sentinel-3 OLCI via Planetary Computer) |

---
## 0. Setup

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)

import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import copernicusmarine
import pystac_client
import planetary_computer
import hvplot.xarray
import hvplot.pandas
import panel as pn
pn.extension()

# Area of interest: Gulf of Tunis  [lon_min, lat_min, lon_max, lat_max]
bbox = [10.0, 36.6, 10.9, 37.2]

# Marine heatwave parameters (Hobday et al. 2016)
PERCENTILE = 90        # threshold percentile per pixel
MIN_DAYS   = 5         # consecutive days above threshold
MONTHS     = [7, 8, 9]              # JAS
YEARS      = list(range(2016, 2026))

print(f"Gulf of Tunis bbox : lon [{bbox[0]}, {bbox[2]}]  lat [{bbox[1]}, {bbox[3]}]")
print(f"MHW threshold      : {PERCENTILE}th percentile per pixel (Hobday et al. 2016)")
print(f"Min duration       : >= {MIN_DAYS} consecutive days")
print(f"Season / years     : JAS  {YEARS[0]}-{YEARS[-1]}")
```

---
## 1. Load CMEMS SST (JAS 2016–2025)

We use the **Mediterranean Sea SST L3S multi-year reprocessed** product, which provides daily gap-filled satellite-composite SST at ~0.01° resolution.

- **Dataset ID:** `cmems_SST_MED_PHY_L3S_MY_010_042_202211_P1D-m`
- **Variable:** `adjusted_sea_surface_temperature` (Kelvin)

> If the dataset ID raises a `ServiceNotFound` error, run `copernicusmarine.describe(contains=["SST_MED", "L3S", "MY"])` to find the current name.

```python
DATASET_ID = "SST_MED_SST_L3S_NRT_OBSERVATIONS_010_012_b"
VAR_NAME   = "adjusted_sea_surface_temperature"   # Kelvin

ds_sst = copernicusmarine.open_dataset(
    DATASET_ID,
    variables=[VAR_NAME],
    minimum_longitude=bbox[0],
    maximum_longitude=bbox[2],
    minimum_latitude=bbox[1],
    maximum_latitude=bbox[3],
    start_datetime="2016-07-01T00:00:00",
    end_datetime="2025-09-30T23:59:59",
)
print(f"Dataset dims : {dict(ds_sst.sizes)}")
print(f"Time range   : {str(ds_sst.time.values[0])[:10]} -> {str(ds_sst.time.values[-1])[:10]}")
```

---

### `planetary_computer_Tunis.ipynb`

[View on GitHub](https://github.com/nadia-mkh/MAGICA-School-2026/blob/main/planetary_computer_Tunis.ipynb)

# Cloud Native Satellite Data Demo 1
* Use pystac_client to search the STAC API at Planetary Computer
* Use odc.stac to load the data
* Use hvplot to visualize the data

```python
import pystac_client
import planetary_computer
from rich.table import Table
import hvplot.xarray

# Connect to the Planetary Computer STAC API
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)
```

```python
collections = list(catalog.get_all_collections())
collections.sort(key=lambda c: c.id)
table = Table("ID", "Title", title="Planetary Computer collections")
for collection in collections:
    table.add_row(collection.id, collection.title)
#table
```

```python
# Define our area of interest (AOI) - a rough box around Cape Cod gulf of tunisia
bbox = [10.0, 36.6, 10.9, 37.2]
```

```python
# Let's search for Sentinel-3 SLSTR WST (Sea Surface Temperature) Level-2 data  NADIA
search = catalog.search(
    collections=["sentinel-3-slstr-wst-l2-netcdf"],
    bbox=bbox,
    datetime="2026-01-01/2026-04-21",
)

# Get the results and see what we found
items = search.item_collection()
print(f"Found {len(items)} scenes matching your criteria.")

# Let's inspect the assets of the first item to see the available data bands
if items:
    first_item = items[0]
    print("\nAssets for the first scene:")
    for asset_key, asset in first_item.assets.items():
        print(f"- {asset_key}: {asset.title}")
```

```python
import xarray as xr
import fsspec

# Prendre le premier item
item = items[0]

# Signer l'URL
asset = item.assets["l2p"]
signed_asset = planetary_computer.sign(asset)

print(f"URL : {signed_asset.href}")

# Ouvrir avec xarray
ds = xr.open_dataset(fsspec.open(signed_asset.href).open())
print(ds)
```

---


## [rsignell/MAGICA-School-2026](https://github.com/rsignell/MAGICA-School-2026)

### `fork_contributions.md`

[View on GitHub](https://github.com/rsignell/MAGICA-School-2026/blob/main/fork_contributions.md)

# New Notebooks & Docs in MAGICA-School-2026 Forks

_Source: [OpenScienceComputing/MAGICA-School-2026](https://github.com/OpenScienceComputing/MAGICA-School-2026) · 24 forks checked · 19 new file(s) found across 4 forks_

---

## [RakeshSwain2604/MAGICA-School-2026](https://github.com/RakeshSwain2604/MAGICA-School-2026)

### [`Project/Spain.ipynb`](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/Spain.ipynb)
**Ebro Delta Coastal Erosion Analysis (2016–2024)**

Tracks coastline change on the Ebro Delta (Tarragona, Spain) using Sentinel-2 imagery from Microsoft Planetary Computer and Copernicus Marine environmental variables. Searches for low-cloud-cover scenes across 5 years (2016–2024), loads RGB + NIR bands via `odc.stac`, and computes coastal change metrics with `hvplot` visualizations.

---

### [`Project/copernicus_marine_model.ipynb`](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/copernicus_marine_model.ipynb)
**Copernicus Marine Wave Model Access**

Demonstrates accessing IBI wave climatology (`cmems_mod_ibi_wav_my_0.027deg-climatology_P1M-m`) via `copernicusmarine.open_dataset`. Selects significant wave height (`VHM0_mean`) over a multi-year time range and creates interactive map visualizations with `hvplot` and an OSM tile background.

---

### [`Project/eopf_geozarr_demo.ipynb`](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/eopf_geozarr_demo.ipynb)
**EOPF GeoZarr Demo** _(content unavailable — file too large to fetch via API)_

---

### [`Project/planetary_computer_1.ipynb`](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/planetary_computer_1.ipynb)
**Planetary Computer Demo 1** _(content unavailable — file too large to fetch via API)_

---

### [`Project/planetary_computer_2.ipynb`](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/planetary_computer_2.ipynb)
**Cloud-Native Satellite Data Explorer (Panel App)**

A Panel-based interactive app (noted as generated by Gemini Pro 2.5) that loads Sentinel-2 RGB imagery over Cape Cod from Planetary Computer via `pystac_client` and `odc.stac`. Uses `pn.cache` to avoid reloading data on each interaction and displays true-color imagery with a date dropdown for scene selection.

---

### [`Project/project_Alacranes_reef_island.ipynb`](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/project_Alacranes_reef_island.ipynb)

---


## [sofiaalovato/MAGICA-School-2026](https://github.com/sofiaalovato/MAGICA-School-2026)

### `cyclone_mocha_bay_of_bengal.ipynb`

[View on GitHub](https://github.com/sofiaalovato/MAGICA-School-2026/blob/main/cyclone_mocha_bay_of_bengal.ipynb)

# Cyclone Mocha — Bay of Bengal: Atmosphere, Ocean & Icechunk

Cyclone Mocha made landfall on **14 May 2023** near Sittwe, Rakhine State, Myanmar — one of the strongest Bay of Bengal cyclones on record (peak wind ~74 m/s, minimum pressure ~908 hPa).

This notebook provides an integrated view of the event across the full **Bay of Bengal + Andaman Sea** domain, combining:
- **Atmospheric forcing** (ERA5 via Open-Meteo): surface pressure, 10 m wind, precipitation
- **Ocean response** (CMEMS reanalysis): sea surface height anomaly / storm surge
- **Icechunk on S3**: both datasets are committed to a versioned store immediately after fetching; all maps and time series are read back from that store

**Region:** Bay of Bengal + Andaman Sea — `bbox = [85, 10, 100, 25]` (lon_min, lat_min, lon_max, lat_max)  
**Time window:** 10–18 May 2023 (approach → landfall → aftermath)  

**Data sources:**
| Source | Variable(s) | Access |
|---|---|---|
| [Open-Meteo](https://open-meteo.com/) archive API | wind, precipitation, pressure, CAPE | free, no auth |
| [IBTrACS](https://www.ncei.noaa.gov/products/international-best-track-archive) v04r01 | cyclone track | free, no auth |
| CMEMS `cmems_mod_glo_phy_my_0.083deg_P1D-m` | SSH (`zos`) | `~/.netrc` credentials |

**Workflow:**
1. Create Icechunk store on S3
2. IBTrACS cyclone track→ commit snapshot 0
3. Fetch ERA5 atmospheric data → commit snapshot 1
4. Load SSH from CMEMS → commit snapshot 2
5. Read back from Icechunk + inspect
6. Commit history & time-travel
7. Surface pressure maps
8. Wind speed maps
9. Precipitation maps
10. SSH anomaly maps
11. Cyclone track map
12. Synchronized time series at Sittwe

```python
import warnings
import json
import io
import os
import zarr
from urllib.request import urlopen

import numpy as np
import pandas as pd
import xarray as xr
import holoviews as hv
import icechunk
import copernicusmarine
import hvplot.xarray   # noqa
import hvplot.pandas   # noqa
import panel as pn
from dotenv import load_dotenv

warnings.filterwarnings('ignore')
pn.extension()
```

```python
load_dotenv(f'{os.environ["HOME"]}/dotenv/protocoast.env', override=True)
storage_endpoint = os.environ['ENDPOINT_URL']

icechunk_bucket = 'sofiaalovato'
storage_name    = 'mocha-atmosphere-v1'

bbox = [85.0, 10.0, 100.0, 25.0]   # [lon_min, lat_min, lon_max, lat_max]
LON_MIN, LAT_MIN, LON_MAX, LAT_MAX = bbox
START_DATE, END_DATE = '2023-05-10', '2023-05-18'
SITTWE_LAT, SITTWE_LON = 20.15, 92.90
LANDFALL = np.datetime64('2023-05-14')

print(f"Icechunk store : s3://{icechunk_bucket}/icechunk/{storage_name}")
print(f"Endpoint       : {storage_endpoint}")
print(f"Bbox           : {bbox}")
print(f"Time window    : {START_DATE} → {END_DATE}")
```

---


## [emmaventura2102-ship-it/MAGICA-School-2026](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026)

### `Copernicus_Chlorophyll.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/Copernicus_Chlorophyll.ipynb)

# Copernicus Marine Service: Access Model Output
* Need to have an account on Copernicus Marine
* The ~/.netrc file contains the account credentials: your login and password for [Copernicus Marine](https://marine.copernicus.eu/)
```
$ more ~/.netrc
machine auth.marine.copernicus.eu
  login xxxxxxxxxx 
  password xxxxxxxxxxxxx
```

```python
import copernicusmarine
import os
```

```python
%%time 
ds = copernicusmarine.open_dataset(dataset_id='cmems_mod_med_bgc-plankton_my_4.2km_P1D-m')
```

```python
ds
```

```python
#here you create a subset in which you select the latitude based on the values (sel) and the depth based on the index (isel)
ds_subset = ds['chl'].sel(latitude=slice(30, 47), longitude=slice(-6, 42)).isel(depth=0) 
ds_subset
```

```python
#ds_subset.groupby('time.dayofyear').mean('time')
ds_subset.groupby('time.year').mean('time')
```

```python
cluster_type = 'Coiled'
#cluster_type = 'Gateway'
```

```python
if cluster_type == 'Coiled':
    import coiled
    cluster = coiled.Cluster(
        region="eu-central-1",
        arm=True,   # run on ARM to save energy & cost
        worker_vm_types=["t4g.large"],  # cheap, small ARM instances, 2cpus, 2GB RAM
        n_workers=30, 
        name = "Emma",
        wait_for_workers=False,
        compute_purchase_option="spot_with_fallback",
        software='protocoast-notebook-arm',  # Conda environment name
        workspace='esip-lab',
        timeout=180   # leave cluster running for 3 min in case we want to use it again
    )

    client = cluster.get_client()
```

```python
%%time
if cluster_type == 'Gateway':   # Protocoast JupyterHub
    from dask_gateway import Gateway

    gateway = Gateway()  # instantiate Dask gateway 

    # Cluster options on Nebari 
    options = gateway.cluster_options()
    options.image = os.environ['JUPYTER_IMAGE']

    # Create a Dask Gateway cluster
    cluster = gateway.new_cluster(options)
    # Scale the cluster
    cluster.adapt(minimum=2, maximum=24)
    # Get the Dask client for the Dask Gateway cluster
    client = cluster.get_client()
```

---

### `Cpernicus_Temperature.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/Cpernicus_Temperature.ipynb)

# Copernicus Marine Service: Access Model Output
* Need to have an account on Copernicus Marine
* The ~/.netrc file contains the account credentials: your login and password for [Copernicus Marine](https://marine.copernicus.eu/)
```
$ more ~/.netrc
machine auth.marine.copernicus.eu
  login xxxxxxxxxx 
  password xxxxxxxxxxxxx
```

```python
import copernicusmarine
import os
```

```python
%%time 
ds = copernicusmarine.open_dataset(dataset_id='cmems_mod_med_phy-temp_my_4.2km_P1D-m')
```

```python
ds
```

```python
#here you create a subset in which you select the latitude based on the values (sel) and the depth based on the index (isel)
ds_subset = ds['thetao'].sel(latitude=slice(30, 47), longitude=slice(-6, 42)).isel(depth=0) 
ds_subset
```

```python
#ds_subset.groupby('time.dayofyear').mean('time')
ds_subset.groupby('time.year').mean('time')
```

```python
cluster_type = 'Coiled'
#cluster_type = 'Gateway'
```

```python
if cluster_type == 'Coiled':
    import coiled
    cluster = coiled.Cluster(
        region="eu-central-1",
        arm=True,   # run on ARM to save energy & cost
        worker_vm_types=["t4g.large"],  # cheap, small ARM instances, 2cpus, 2GB RAM
        n_workers=30, 
        name = "Emma",
        wait_for_workers=False,
        compute_purchase_option="spot_with_fallback",
        software='protocoast-notebook-arm',  # Conda environment name
        workspace='esip-lab',
        timeout=180   # leave cluster running for 3 min in case we want to use it again
    )

    client = cluster.get_client()
```

```python
%%time
if cluster_type == 'Gateway':   # Protocoast JupyterHub
    from dask_gateway import Gateway

    gateway = Gateway()  # instantiate Dask gateway 

    # Cluster options on Nebari 
    options = gateway.cluster_options()
    options.image = os.environ['JUPYTER_IMAGE']

    # Create a Dask Gateway cluster
    cluster = gateway.new_cluster(options)
    # Scale the cluster
    cluster.adapt(minimum=2, maximum=24)
    # Get the Dask client for the Dask Gateway cluster
    client = cluster.get_client()
```

---

### `Group Project/1.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/Group Project/1.ipynb)

# Mediterranean Sea Surface Temperature — Climatology, Anomalies & Extremes (1987–2025)

**Workflow**
1. Open the Mediterranean Sea Physics Reanalysis (`cmems_mod_med_phy-temp_my_4.2km_P1M-m`) — monthly means, lazy (dask).
2. Compute the **yearly mean SST** at every pixel and explore it with a year slider.
3. Compute the **long-term climatological mean** at every pixel (mean over 1987–2025).
4. Compute the **monthly SST anomaly** = monthly value − pixel climatological mean.
5. Visualise the monthly anomaly with an interactive time slider showing the current month.
6. At every pixel, find the **minimum** and **maximum** of the anomaly time-series.
7. Plot spatial maps of the min-anomaly and max-anomaly fields.
8. Export an **MP4 video** of the annual mean SST anomaly (1987–2025).

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import cmocean
import copernicusmarine
import hvplot.xarray
import numpy as np
import warnings

warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
# Mediterranean bounding box
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)

# The reanalysis starts January 1987; request through end of 2025
TIME_START = "1987-01-01"
TIME_END   = "2025-12-31"
```

## 1 — Open the SST dataset (lazy)

```python
%%time
ds = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-temp_my_4.2km_P1M-m",
    start_datetime=TIME_START,
    end_datetime=TIME_END,
    **MED_BBOX,
)
ds
```

```python
# Surface SST: select depth nearest to 0 m — result is (time, latitude, longitude), still lazy
sst = ds["thetao"].sel(depth=0, method="nearest")
print(f"SST shape: {sst.dims} = {dict(zip(sst.dims, sst.shape))}")
sst
```

## 2 — Yearly mean SST with time slider

Resample monthly → annual, compute (≈ 39 frames × 380 × 1016 ≈ 60 MB), then display with a year scrubber.

```python
%%time
sst_yearly = sst.resample(time="YE").mean().compute()

# Replace the datetime coordinate with plain integer years for a cleaner slider
years = sst_yearly.time.dt.year.values
sst_yearly = (
    sst_yearly
    .assign_coords(year=("time", years))
    .swap_dims({"time": "year"})
    .drop_vars("time")
)
sst_yearly.attrs["long_name"] = "Annual Mean SST"
sst_yearly.attrs["units"] = "\u00b0C"
print(f"Yearly SST shape: {sst_yearly.dims} = {dict(zip(sst_yearly.dims, sst_yearly.shape))}")
sst_yearly
```

---

### `Group Project/2.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/Group Project/2.ipynb)

# Mediterranean SST — Monthly Time Series at Two Pixels (1987–2025)

This notebook follows the same data loading and anomaly approach as
`copernicus_sst_anomaly_mediterranean.ipynb` and then extracts pixel-level
time series at two representative locations:

| Label | Location | Approx. coordinates |
|---|---|---|
| North Adriatic (Venice) | Northern Adriatic Sea, offshore Venice | 45.3 °N, 12.5 °E |
| Gulf of Lion (near Italy) | Eastern Gulf of Lion, near French-Italian Riviera | 43.3 °N, 7.5 °E |

**Plots produced**
1. Map showing the actual model grid-points selected.
2. Raw SST monthly time series (1987–2025) for both pixels.
3. SST anomaly monthly time series for both pixels (with ±1 σ envelope).
4. Monthly climatology (mean seasonal cycle) for both pixels.

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)

import numpy as np
import pandas as pd
import xarray as xr
import copernicusmarine
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

pn.extension()
```

```python
# Mediterranean bounding box and time range — identical to the SST anomaly notebook
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)
TIME_START = "1987-01-01"
TIME_END   = "2025-12-31"

# Pixel definitions — (lat, lon) chosen to lie in open-water model cells
PIXELS = {
    "North Adriatic (Venice)": {"lat": 45.3, "lon": 12.5},
    "Gulf of Lion (near Italy)": {"lat": 43.3, "lon": 7.5},
}
COLORS = {
    "North Adriatic (Venice)": "#1f77b4",   # blue
    "Gulf of Lion (near Italy)": "#d62728",  # red
}
```

## 1 — Open the SST dataset (lazy)

Same dataset and subsetting as `copernicus_sst_anomaly_mediterranean.ipynb`.

---

### `Group Project/3.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/Group Project/3.ipynb)

# Mediterranean Sea Surface Salinity — Climatology, Anomalies & Extremes (1987–2025)

**Workflow**
1. Open the Mediterranean Sea Physics Reanalysis (`cmems_mod_med_phy-sal_my_4.2km_P1M-m`) — monthly means, lazy (dask).
2. Compute the **yearly mean salinity** at every pixel and explore it with a year slider.
3. Compute the **long-term climatological mean** at every pixel (mean over 1987–2025).
4. Compute the **monthly salinity anomaly** = monthly value − pixel climatological mean.
5. Visualise the monthly anomaly with an interactive time slider showing the current month.
6. At every pixel, find the **minimum** and **maximum** of the anomaly time-series.
7. Plot spatial maps of the min-anomaly and max-anomaly fields.

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import cmocean
import copernicusmarine
import hvplot.xarray
import numpy as np
import warnings

warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
# Mediterranean bounding box
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)

# The reanalysis starts January 1987; request through end of 2025
TIME_START = "1987-01-01"
TIME_END   = "2025-12-31"
```

## 1 — Open the salinity dataset (lazy)

```python
%%time
ds = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-sal_my_4.2km_P1M-m",
    start_datetime=TIME_START,
    end_datetime=TIME_END,
    **MED_BBOX,
)
ds
```

```python
# Surface salinity: select depth nearest to 0 m — result is (time, latitude, longitude), still lazy
sal = ds["so"].sel(depth=0, method="nearest")
print(f"Salinity shape: {sal.dims} = {dict(zip(sal.dims, sal.shape))}")
sal
```

## 2 — Yearly mean salinity with time slider

Resample monthly → annual, compute (≈ 39 frames × 380 × 1016 ≈ 60 MB), then display with a year scrubber.

```python
%%time
sal_yearly = sal.resample(time="YE").mean().compute()

# Replace the datetime coordinate with plain integer years for a cleaner slider
years = sal_yearly.time.dt.year.values
sal_yearly = (
    sal_yearly
    .assign_coords(year=("time", years))
    .swap_dims({"time": "year"})
    .drop_vars("time")
)
sal_yearly.attrs["long_name"] = "Annual Mean Salinity"
sal_yearly.attrs["units"] = "PSU"
print(f"Yearly salinity shape: {sal_yearly.dims} = {dict(zip(sal_yearly.dims, sal_yearly.shape))}")
sal_yearly
```

---

### `Group Project/4.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/Group Project/4.ipynb)

# Mediterranean Sea Surface Chlorophyll — Climatology, Anomalies & Extremes (1999–2025)

**Workflow**
1. Open the Mediterranean Sea BGC Reanalysis (`cmems_mod_med_bgc-plankton_my_4.2km_P1D-m`) — daily means, lazy (dask).
2. Resample daily → monthly means.
3. Compute the **yearly mean chlorophyll** at every pixel and explore it with a year slider.
4. Compute the **long-term climatological mean** at every pixel (mean over 1999–2025).
5. Compute the **monthly chlorophyll anomaly** = monthly value − pixel climatological mean.
6. Visualise the monthly anomaly with an interactive time slider showing the current month.
7. At every pixel, find the **minimum** and **maximum** of the anomaly time-series.
8. Plot spatial maps of the min-anomaly and max-anomaly fields.

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import cmocean
import copernicusmarine
import hvplot.xarray
import numpy as np
import warnings

warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
# Mediterranean bounding box
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)

# The BGC reanalysis starts January 1999; request through end of 2025
TIME_START = "1999-01-01"
TIME_END   = "2025-12-31"
```

## 1 — Open the chlorophyll dataset (lazy)

```python
%%time
ds = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_bgc-plankton_my_4.2km_P1M-m",
    variables=["chl"],
    start_datetime=TIME_START,
    end_datetime=TIME_END,
    **MED_BBOX,
)
ds
```

```python
# Surface chlorophyll: select depth nearest to 0 m — result is (time, latitude, longitude), still lazy
chl_monthly = ds["chl"].sel(depth=0, method="nearest")
chl_monthly.attrs["long_name"] = "Monthly Mean Chlorophyll"
chl_monthly.attrs["units"] = "mg m-3"
print(f"Monthly chlorophyll shape: {chl_monthly.dims} = {dict(zip(chl_monthly.dims, chl_monthly.shape))}")
chl_monthly
```

## 2 — Monthly dataset (no resampling needed)

The `P1M-m` product is already at monthly resolution, so `chl_monthly` is ready to use directly.

```python
# No resampling required — dataset is already monthly (P1M-m)
```

---

### `Group Project/5.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/Group Project/5.ipynb)

# Mediterranean Surface Chlorophyll — Monthly Time Series at Two Pixels (1999–2025)

This notebook follows the same structure as
`copernicus_sst_timeseries_graphs.ipynb` but uses chlorophyll instead of
temperature, drawn from the Mediterranean BGC Reanalysis.

| Label | Location | Approx. coordinates |
|---|---|---|
| North Adriatic (Venice) | Northern Adriatic Sea, offshore Venice | 45.3 °N, 12.5 °E |
| Gulf of Lion (near Italy) | Eastern Gulf of Lion, near French-Italian Riviera | 43.3 °N, 7.5 °E |

**Plots produced**
1. Map showing the actual model grid-points selected.
2. Raw chlorophyll monthly time series (1999–2025) for both pixels.
3. Chlorophyll anomaly monthly time series for both pixels (with ±1 σ envelope).
4. Monthly climatology (mean seasonal cycle) for both pixels.
5. Annual mean chlorophyll and annual anomaly bar charts.
6. Per-pixel detailed two-panel figure (raw + anomaly bars).

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)

import numpy as np
import pandas as pd
import xarray as xr
import copernicusmarine
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

pn.extension()
```

```python
# Mediterranean bounding box and time range
# BGC reanalysis starts January 1999
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)
TIME_START = "1999-01-01"
TIME_END   = "2025-12-31"

# Pixel definitions — (lat, lon) chosen to lie in open-water model cells
PIXELS = {
    "North Adriatic (Venice)": {"lat": 45.3, "lon": 12.5},
    "Gulf of Lion (near Italy)": {"lat": 43.3, "lon": 7.5},
}
COLORS = {
    "North Adriatic (Venice)": "#1f77b4",   # blue
    "Gulf of Lion (near Italy)": "#d62728",  # red
}
```

---

### `copernicus_chl_timeseries_graphs.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/copernicus_chl_timeseries_graphs.ipynb)

# Mediterranean Surface Chlorophyll — Monthly Time Series at Two Pixels (1999–2025)

This notebook follows the same structure as
`copernicus_sst_timeseries_graphs.ipynb` but uses chlorophyll instead of
temperature, drawn from the Mediterranean BGC Reanalysis.

| Label | Location | Approx. coordinates |
|---|---|---|
| North Adriatic (Venice) | Northern Adriatic Sea, offshore Venice | 45.3 °N, 12.5 °E |
| Gulf of Lion (near Italy) | Eastern Gulf of Lion, near French-Italian Riviera | 43.3 °N, 7.5 °E |

**Plots produced**
1. Map showing the actual model grid-points selected.
2. Raw chlorophyll monthly time series (1999–2025) for both pixels.
3. Chlorophyll anomaly monthly time series for both pixels (with ±1 σ envelope).
4. Monthly climatology (mean seasonal cycle) for both pixels.
5. Annual mean chlorophyll and annual anomaly bar charts.
6. Per-pixel detailed two-panel figure (raw + anomaly bars).

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)

import numpy as np
import pandas as pd
import xarray as xr
import copernicusmarine
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

pn.extension()
```

```python
# Mediterranean bounding box and time range
# BGC reanalysis starts January 1999
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)
TIME_START = "1999-01-01"
TIME_END   = "2025-12-31"

# Pixel definitions — (lat, lon) chosen to lie in open-water model cells
PIXELS = {
    "North Adriatic (Venice)": {"lat": 45.3, "lon": 12.5},
    "Gulf of Lion (near Italy)": {"lat": 43.3, "lon": 7.5},
}
COLORS = {
    "North Adriatic (Venice)": "#1f77b4",   # blue
    "Gulf of Lion (near Italy)": "#d62728",  # red
}
```

---

### `copernicus_chlorophyll_anomaly_mediterranean.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/copernicus_chlorophyll_anomaly_mediterranean.ipynb)

# Mediterranean Sea Surface Chlorophyll — Climatology, Anomalies & Extremes (1999–2025)

**Workflow**
1. Open the Mediterranean Sea BGC Reanalysis (`cmems_mod_med_bgc-plankton_my_4.2km_P1D-m`) — daily means, lazy (dask).
2. Resample daily → monthly means.
3. Compute the **yearly mean chlorophyll** at every pixel and explore it with a year slider.
4. Compute the **long-term climatological mean** at every pixel (mean over 1999–2025).
5. Compute the **monthly chlorophyll anomaly** = monthly value − pixel climatological mean.
6. Visualise the monthly anomaly with an interactive time slider showing the current month.
7. At every pixel, find the **minimum** and **maximum** of the anomaly time-series.
8. Plot spatial maps of the min-anomaly and max-anomaly fields.

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import cmocean
import copernicusmarine
import hvplot.xarray
import numpy as np
import warnings

warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
# Mediterranean bounding box
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)

# The BGC reanalysis starts January 1999; request through end of 2025
TIME_START = "1999-01-01"
TIME_END   = "2025-12-31"
```

## 1 — Open the chlorophyll dataset (lazy)

```python
%%time
ds = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_bgc-plankton_my_4.2km_P1M-m",
    variables=["chl"],
    start_datetime=TIME_START,
    end_datetime=TIME_END,
    **MED_BBOX,
)
ds
```

```python
# Surface chlorophyll: select depth nearest to 0 m — result is (time, latitude, longitude), still lazy
chl_monthly = ds["chl"].sel(depth=0, method="nearest")
chl_monthly.attrs["long_name"] = "Monthly Mean Chlorophyll"
chl_monthly.attrs["units"] = "mg m-3"
print(f"Monthly chlorophyll shape: {chl_monthly.dims} = {dict(zip(chl_monthly.dims, chl_monthly.shape))}")
chl_monthly
```

## 2 — Monthly dataset (no resampling needed)

The `P1M-m` product is already at monthly resolution, so `chl_monthly` is ready to use directly.

```python
# No resampling required — dataset is already monthly (P1M-m)
```

---

### `copernicus_salinity_anomaly_mediterranean.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/copernicus_salinity_anomaly_mediterranean.ipynb)

# Mediterranean Sea Surface Salinity — Climatology, Anomalies & Extremes (1987–2025)

**Workflow**
1. Open the Mediterranean Sea Physics Reanalysis (`cmems_mod_med_phy-sal_my_4.2km_P1M-m`) — monthly means, lazy (dask).
2. Compute the **yearly mean salinity** at every pixel and explore it with a year slider.
3. Compute the **long-term climatological mean** at every pixel (mean over 1987–2025).
4. Compute the **monthly salinity anomaly** = monthly value − pixel climatological mean.
5. Visualise the monthly anomaly with an interactive time slider showing the current month.
6. At every pixel, find the **minimum** and **maximum** of the anomaly time-series.
7. Plot spatial maps of the min-anomaly and max-anomaly fields.

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import cmocean
import copernicusmarine
import hvplot.xarray
import numpy as np
import warnings

warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
# Mediterranean bounding box
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)

# The reanalysis starts January 1987; request through end of 2025
TIME_START = "1987-01-01"
TIME_END   = "2025-12-31"
```

## 1 — Open the salinity dataset (lazy)

```python
%%time
ds = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-sal_my_4.2km_P1M-m",
    start_datetime=TIME_START,
    end_datetime=TIME_END,
    **MED_BBOX,
)
ds
```

```python
# Surface salinity: select depth nearest to 0 m — result is (time, latitude, longitude), still lazy
sal = ds["so"].sel(depth=0, method="nearest")
print(f"Salinity shape: {sal.dims} = {dict(zip(sal.dims, sal.shape))}")
sal
```

## 2 — Yearly mean salinity with time slider

Resample monthly → annual, compute (≈ 39 frames × 380 × 1016 ≈ 60 MB), then display with a year scrubber.

```python
%%time
sal_yearly = sal.resample(time="YE").mean().compute()

# Replace the datetime coordinate with plain integer years for a cleaner slider
years = sal_yearly.time.dt.year.values
sal_yearly = (
    sal_yearly
    .assign_coords(year=("time", years))
    .swap_dims({"time": "year"})
    .drop_vars("time")
)
sal_yearly.attrs["long_name"] = "Annual Mean Salinity"
sal_yearly.attrs["units"] = "PSU"
print(f"Yearly salinity shape: {sal_yearly.dims} = {dict(zip(sal_yearly.dims, sal_yearly.shape))}")
sal_yearly
```

---

### `copernicus_sst_timeseries_graphs.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/copernicus_sst_timeseries_graphs.ipynb)

# Mediterranean SST — Monthly Time Series at Two Pixels (1987–2025)

This notebook follows the same data loading and anomaly approach as
`copernicus_sst_anomaly_mediterranean.ipynb` and then extracts pixel-level
time series at two representative locations:

| Label | Location | Approx. coordinates |
|---|---|---|
| North Adriatic (Venice) | Northern Adriatic Sea, offshore Venice | 45.3 °N, 12.5 °E |
| Gulf of Lion (near Italy) | Eastern Gulf of Lion, near French-Italian Riviera | 43.3 °N, 7.5 °E |

**Plots produced**
1. Map showing the actual model grid-points selected.
2. Raw SST monthly time series (1987–2025) for both pixels.
3. SST anomaly monthly time series for both pixels (with ±1 σ envelope).
4. Monthly climatology (mean seasonal cycle) for both pixels.

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)

import numpy as np
import pandas as pd
import xarray as xr
import copernicusmarine
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

pn.extension()
```

```python
# Mediterranean bounding box and time range — identical to the SST anomaly notebook
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)
TIME_START = "1987-01-01"
TIME_END   = "2025-12-31"

# Pixel definitions — (lat, lon) chosen to lie in open-water model cells
PIXELS = {
    "North Adriatic (Venice)": {"lat": 45.3, "lon": 12.5},
    "Gulf of Lion (near Italy)": {"lat": 43.3, "lon": 7.5},
}
COLORS = {
    "North Adriatic (Venice)": "#1f77b4",   # blue
    "Gulf of Lion (near Italy)": "#d62728",  # red
}
```

## 1 — Open the SST dataset (lazy)

Same dataset and subsetting as `copernicus_sst_anomaly_mediterranean.ipynb`.

---

### `taranto-icechunk-append-emmaventura.ipynb`

[View on GitHub](https://github.com/emmaventura2102-ship-it/MAGICA-School-2026/blob/main/taranto-icechunk-append-emmaventura.ipynb)

# Create or append Virtual Icechunk Store from SHYFEM forecast NetCDF files

* This notebook appends Taranto SHYFEM forecast data to an Icechunk store written to the **emmaventura** bucket.
* Virtual chunk references point back to source NetCDF files in `protocoast-data`.
* If no repo exists, a new one will be created with references to all the existing NetCDF files.

**Append Methodology:**
1. **`set_repo`**: extract all dates currently present in the Icechunk store's `time` coordinate
2. **`set_cloud`**: Scan the S3 bucket for all available NOS files and extract their dates.
3. **`new_dates`**: Calculate the difference (`set_cloud - set_repo`) to determine exactly which days need to be written for creation or appended.

```python
import warnings
import os
import pandas as pd
import fsspec
import xarray as xr
from dotenv import load_dotenv

warnings.filterwarnings("ignore", category=UserWarning)
```

```python
import icechunk
from obstore.store import from_url
from virtualizarr import open_virtual_dataset
from virtualizarr.parsers import HDFParser
from obspec_utils.registry import ObjectStoreRegistry
from obstore.store import S3Store
```

```python
# Load credentials
_ = load_dotenv(f'{os.environ["HOME"]}/dotenv/protocoast.env', override=True)

# Configuration
storage_endpoint = os.environ['ENDPOINT_URL']

# Source data bucket (NetCDF files for virtual chunk references)
source_bucket = 'protocoast-data'
source_bucket_url = f"s3://{source_bucket}"

# Destination bucket (Icechunk repo written here)
icechunk_bucket = 'emmaventura'
storage_name = 'taranto-icechunk-tubitak-v1'

# Setup Filesystem (for scanning source NetCDF files)
fs = fsspec.filesystem('s3', anon=False, endpoint_url=storage_endpoint,
                       skip_instance_cache=True, use_listings_cache=False)
```

```python
# Define Icechunk Storage & Config
# Repo is stored in emmaventura; virtual chunks reference files in protocoast-data
storage = icechunk.s3_storage(
    bucket=icechunk_bucket,
    prefix=f"icechunk/{storage_name}",
    from_env=True,
    endpoint_url=storage_endpoint,
    region='not-used',
    force_path_style=True,
)

config = icechunk.RepositoryConfig.default()
config.set_virtual_chunk_container(
    icechunk.VirtualChunkContainer(
        url_prefix=f"{source_bucket_url}/",
        store=icechunk.s3_store(region="not-used", anonymous=False, s3_compatible=True,
                                force_path_style=True, endpoint_url=storage_endpoint),
    ),
)

credentials = icechunk.containers_credentials({f"{source_bucket_url}/": icechunk.s3_credentials(anonymous=False)})

store_obj = S3Store(
    bucket=source_bucket,
    endpoint=storage_endpoint,
    region="not-used",
)

registry = ObjectStoreRegistry({source_bucket_url: store_obj})
parser = HDFParser()
```

---


## [cirozugar/MAGICA-School-2026](https://github.com/cirozugar/MAGICA-School-2026)

### `repo-ciri/alacranes.ipynb`

[View on GitHub](https://github.com/cirozugar/MAGICA-School-2026/blob/main/repo-ciri/alacranes.ipynb)

# Coral Island Dynamics at Arrecife Alacranes
## A Cloud-Native Time Series Analysis of Shoreline Change

**Open Science in Practice FERS School, Bertinoro, April 2026**

---

### Scientific Question

>Have the islands of Arrecife Alacranes changed in extent and morphology between 2017 and 2024,
> and if so, can those changes be linked to sea level anomaly or sea surface temperature?

### Study Site

Arrecife Alacranes is the largest coral reef in the southern Gulf of Mexico, located ~130 km
north of Progreso, Yucatán, México (≈22.4°N, 89.7°W). It hosts five small coral islands
(Isla Pérez, Isla Chica, Isla Pájaros, Isla Desertora, Isla Desterrada) that are sensitive
indicators of sea level change and wave energy dynamics.

### Data sources (all open, cloud-native)
| Variable | Source | Access |
|---|---|---|
| Multispectral imagery | Sentinel-2 L2A | Planetary Computer STAC |
| Sea Level Anomaly (SLA) | CMEMS SEALEVEL_GLO | `copernicusmarine` Python client |
| Sea Surface Temperature | CMEMS SST_MED or GLO | `copernicusmarine` Python client |

### FAIR compliance
- All data accessed via public APIs — no local files
- Notebook fully reproducible on JupyterHub
- Outputs documented with standard metadata
- Results to be published via Zenodo + STAC catalog

### Section 0 Environment Setup

```python
# Core scientific stack
import numpy as np
import xarray as xr
import rioxarray  # raster extension for xarray
import pandas as pd

# STAC access
import pystac_client
import planetary_computer

# CMEMS access
import copernicusmarine  # pip install copernicusmarine if not available

# Visualization
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn
from bokeh.models import HoverTool

hv.extension('bokeh')
pn.extension()

print(' All imports successful')
```

### Section 1 Study Area Definition

```python
# Study Area definition
# Arrecife Alacranes bounding box
# Tighter bbox centered on southern islands (Pérez, Pájaros, Chica)
BBOX = [0.55, 40.55, 1.05, 40.90]  # [lon_min, lat_min, lon_max, lat_max]
YEARS = list(range(2017, 2026))         # 2017–2024 (Sentinel-2 full archive)
CLOUD_COVER_MAX = 10                    # % — coral reef, mostly clear sky

# Month range for 'dry season' composites — reduces cloud cover and vegetation variability
# Yucatán dry season: February–April
DATE_WINDOWS = [(f'{y}-01-01', f'{y}-05-30') for y in YEARS]

print(f'Study area: {BBOX}')
print(f'Time windows: {DATE_WINDOWS[0]} → {DATE_WINDOWS[-1]}')
```

---

### `repo-ciri/blksea-bgc-icechunk.ipynb`

[View on GitHub](https://github.com/cirozugar/MAGICA-School-2026/blob/main/repo-ciri/blksea-bgc-icechunk.ipynb)

# Create Virtual Icechunk Store from Copernicus Marine Black Sea BGC NetCDF files

Uses VirtualiZarr to create an Icechunk store pointing at the first 5 files of
`cmems_mod_blk_bgc-nut_anfc_2.5km_P1D-m` (Nitrate, Phosphate, ODU — daily mean, 3D).

Source files are on Copernicus Marine S3 (`s3.waw3-1.cloudferro.com`), accessed anonymously.
The Icechunk repo is stored on the protocoast S3.

```python
import warnings
import os
import pandas as pd
import xarray as xr
from dotenv import load_dotenv

import icechunk
from virtualizarr import open_virtual_dataset
from virtualizarr.parsers import HDFParser
from obspec_utils.registry import ObjectStoreRegistry
from obstore.store import S3Store

warnings.filterwarnings('ignore', category=UserWarning)
```

```python
# --- Icechunk store (protocoast S3) ---
_ = load_dotenv(f'{os.environ["HOME"]}/dotenv/protocoast.env', override=True)
storage_endpoint = os.environ['ENDPOINT_URL']
storage_bucket   = 'protocoast-data'
storage_name     = 'blksea-bgc-nut-icechunk-v1'

# --- Source files (Copernicus Marine S3, anonymous) ---
CMEMS_ENDPOINT = 'https://s3.waw3-1.cloudferro.com'
CMEMS_BUCKET   = 'mdl-native-11'
CMEMS_PREFIX   = 'native/BLKSEA_ANALYSISFORECAST_BGC_007_010/cmems_mod_blk_bgc-nut_anfc_2.5km_P1D-m_202511'
```

```python
import fsspec
fs = fsspec.filesystem('s3', anon=True, endpoint_url=CMEMS_ENDPOINT)
```

```python
listed_files = fs.ls('s3://mdl-native-11/native/BLKSEA_ANALYSISFORECAST_BGC_007_010/cmems_mod_blk_bgc-nut_anfc_2.5km_P1D-m_202511/2024/02/')
listed_files
```

```python
listed_files = [f's3://{f}' for f in listed_files]
```

```python
listed_files[0]
```

```python
xr.open_dataset(fs.open(listed_files[0]))
```

### Step 1: Set up Icechunk repo with virtual chunk container pointing at Copernicus S3

```python
# Icechunk store on protocoast
storage = icechunk.s3_storage(
    bucket=storage_bucket,
    prefix=f'icechunk/{storage_name}',
    from_env=True,
    endpoint_url=storage_endpoint,
    region='not-used',
    force_path_style=True,
)

config = icechunk.RepositoryConfig.default()

# NetCDF file chunks live on Copernicus S3 (anonymous)
cmems_s3_prefix = f's3://{CMEMS_BUCKET}/'
config.set_virtual_chunk_container(
    icechunk.VirtualChunkContainer(
        url_prefix=cmems_s3_prefix,
        store=icechunk.s3_store(
            region='not-used',
            anonymous=True,
            s3_compatible=True,
            force_path_style=True,
            endpoint_url=CMEMS_ENDPOINT,
        ),
    )
)

credentials = icechunk.containers_credentials(
    {cmems_s3_prefix: icechunk.s3_credentials(anonymous=True)}
)

# obstore registry for VirtualiZarr — keyed with s3:// prefix to match listed_files
cmems_store = S3Store(
    bucket=CMEMS_BUCKET,
    endpoint_url=CMEMS_ENDPOINT,
    region='not-used',
    skip_signature=True
)
registry = ObjectStoreRegistry({cmems_s3_prefix: cmems_store})
parser = HDFParser()
```

---

### `repo-ciri/eopf_geozarr_demo.ipynb`

[View on GitHub](https://github.com/cirozugar/MAGICA-School-2026/blob/main/repo-ciri/eopf_geozarr_demo.ipynb)

# EOPF GeoZarr Format — Interactive Exploration

This notebook demonstrates the usefulness of the **EOPF Zarr format** for cloud-native Sentinel-2 analysis:

- **No download required** — data streams directly from object storage
- **Analysis-ready** — the `eopf-zarr` xarray backend delivers properly georeferenced, scaled data
- **Resolution-aware** — request any resolution on open; bands are resampled on-the-fly
- **Interactive visualization** — explore imagery and indices with `hvplot`

We'll work over the **Italian Alps** (Aosta Valley region) using Sentinel-2 L2A data.

## 1. Search the STAC catalog

```python
import warnings
warnings.filterwarnings('ignore')

import pystac_client

catalog = pystac_client.Client.open("https://stac.core.eopf.eodc.eu/")

# Aosta Valley, Italian Alps
bbox = [-89.6, 22.25, -89.50, 22.65]

search = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=bbox,
    datetime="2025-04-01/2025-06-01",
    query={"eo:cloud_cover": {"lt": 20}},
)

items = list(search.items())
print(f"Found {len(items)} items")
for item in items:
    print(f"  {item.id}  cloud_cover={item.properties.get('eo:cloud_cover', 'N/A')}%")
```

## 2. Open with the `eopf-zarr` xarray backend

The `xarray-eopf` package registers an `eopf-zarr` engine that:
- Reads the EOPF Zarr product structure
- Applies scale/offset automatically (returning physical reflectance values)
- Attaches proper CRS coordinates (`x`, `y` in projected metres + `spatial_ref`)
- Resamples all bands to the requested `resolution` on-the-fly

```python
import xarray as xr

# Pick the least-cloudy scene
item = sorted(items, key=lambda i: i.properties.get('eo:cloud_cover', 999))[0]
url = item.assets['product'].href
print(f"Opening: {item.id}")
print(f"URL: {url}")

# Open at 60m resolution for fast exploration — change to 10m resolution for full detail
ds = xr.open_dataset(url, engine='eopf-zarr', resolution=60, chunks={})
ds
```

```python
#!pip install xarray-eopf
```

Notice:
- **`x` / `y` coordinates** in UTM metres — no manual georeferencing needed
- **`spatial_ref`** variable carries the full CRS (EPSG:32632)
- **All bands** at a single consistent resolution via one `open_dataset` call
- Values are **float64 surface reflectance** (scale/offset already applied)

---

### `repo-ciri/planetary_computer_PAA.ipynb`

[View on GitHub](https://github.com/cirozugar/MAGICA-School-2026/blob/main/repo-ciri/planetary_computer_PAA.ipynb)

# Cloud Native Satellite Data Demo 1
* Use pystac_client to search the STAC API at Planetary Computer
* Use odc.stac to load the data
* Use hvplot to visualize the data

```python
import pystac_client
import planetary_computer
from rich.table import Table
import hvplot.xarray

# Connect to the Planetary Computer STAC API
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)
```

```python
collections = list(catalog.get_all_collections())
collections.sort(key=lambda c: c.id)
table = Table("ID", "Title", title="Planetary Computer collections")
for collection in collections:
    table.add_row(collection.id, collection.title)
table
```

```python
# Define our area of interest (AOI) - a rough box around Cape Cod
bbox = [-83.6997, 22.3963, -83.6987, 22.3575]
```

```python
# Let's search for Sentinel-2 Level-2A data from this past summer
# with minimal cloud cover.
search = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=bbox,
    datetime="2025-08-01/2025-09-30",
    query={"eo:cloud_cover": {"lt": 01}}, # less than 10% clouds
)

# Get the results and see what we found
items = search.item_collection()
print(f"Found {len(items)} scenes matching your criteria.")

# Let's inspect the assets of the first item to see the available data bands
if items:
    first_item = items[0]
    print("\nAssets for the first scene:")
    for asset_key, asset in first_item.assets.items():
        print(f"- {asset_key}: {asset.title}")
```

```python
# Assuming 'items' is the ItemCollection from the first step
from odc.stac import stac_load


# Load the data using odc-stac.
# Note that we specify the dask chunks as a dictionary.
data_ds = stac_load(
    items,
    bands=["B04", "B03", "B02", "B08"], # B04=Red, B03=Green, B02=Blue, B08=NIR
    bbox=bbox,
    resolution=10,
    chunks={"x": 2048, "y": 2048},
)

# Let's inspect our new xarray Dataset.
# Notice the "Data variables" section.
print(data_ds)
```

---

### `repo-ciri/spain-v1.ipynb`

[View on GitHub](https://github.com/cirozugar/MAGICA-School-2026/blob/main/repo-ciri/spain-v1.ipynb)

# Coastal Dynamics — Tarragona Coast, Spain
## A Cloud-Native Time Series Analysis of Shoreline Change

**Open Science in Practice FERS School, Bertinoro, April 2026**

---

### Scientific Question

> Has the coastline near Tarragona (NE Spain) changed in extent and morphology between 2017 and 2024,
> and if so, can those changes be linked to sea level anomaly or sea surface temperature?

### Study Site

The study area (bbox `[0.55, 40.55, 1.05, 40.90]`) covers the Tarragona coastal zone
in Catalonia, NE Spain — including the Costa Daurada beaches, the Ebro Delta margin,
and nearshore Mediterranean waters. The area is sensitive to storm surges, longshore
sediment transport, and seasonal wave climatology.

### Data sources (all open, cloud-native)
| Variable | Source | Access |
|---|---|---|
| Multispectral imagery | Sentinel-2 L2A | Planetary Computer STAC |
| Sea Level Anomaly (SLA) | CMEMS SEALEVEL_GLO | `copernicusmarine` Python client |
| Sea Surface Temperature | CMEMS SST GLO | `copernicusmarine` Python client |

### FAIR compliance
- All data accessed via public APIs — no local files
- Notebook fully reproducible on JupyterHub
- Outputs documented with standard metadata
- Results to be published via Zenodo + STAC catalog

### Section 0 — Environment Setup

```python
# Core scientific stack
import numpy as np
import xarray as xr
import rioxarray
import pandas as pd

# STAC access
import pystac_client
import planetary_computer

# CMEMS access
import copernicusmarine

# Visualization
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn

hv.extension('bokeh')
pn.extension()

print('All imports successful')
```

### Section 1 — Study Area Definition

```python
# Study area — Tarragona coast, NE Spain
BBOX   = [0.55, 40.55, 1.05, 40.90]   # [lon_min, lat_min, lon_max, lat_max]
YEARS  = list(range(2017, 2026))       # 2017–2025 (full Sentinel-2 archive)
CLOUD_COVER_MAX = 20                   # % — Mediterranean, full-year search

# Full-year windows — no dry-season restriction for Spain
DATE_WINDOWS = [(f'{y}-01-01', f'{y}-12-31') for y in YEARS]

print(f'Study area: {BBOX}')
print(f'Time windows: {DATE_WINDOWS[0]} → {DATE_WINDOWS[-1]}')
```

---

### `repo-ciri/spain-v2.ipynb`

[View on GitHub](https://github.com/cirozugar/MAGICA-School-2026/blob/main/repo-ciri/spain-v2.ipynb)

# Coastal Dynamics — Tarragona Coast, Spain
## A Cloud-Native Time Series Analysis of Shoreline Change

**Open Science in Practice — FERS School, Bertinoro, April 2026**

---

### Scientific Question

> Has the coastline near Tarragona (NE Spain) changed in extent and morphology between 2017 and 2024,
> and if so, can those changes be linked to sea level anomaly or sea surface temperature?
> Does the choice of water index (NDWI vs MNDWI) affect the detected change signal?

### Study Site

The study area (`bbox [0.55, 40.55, 1.05, 40.90]`) covers the Tarragona coastal zone
in Catalonia, NE Spain — including the Costa Daurada beaches, the Ebro Delta margin,
and nearshore Mediterranean waters. The area is sensitive to storm surges, longshore
sediment transport, and seasonal wave climatology.

### Methods: two water indices compared

| Index | Formula | Bands | Reference |
|---|---|---|---|
| **NDWI** | (Green − NIR) / (Green + NIR) | B03, B08 | McFeeters (1996) |
| **MNDWI** | (Green − SWIR) / (Green + SWIR) | B03, B11 | Xu (2006) |

MNDWI uses SWIR instead of NIR, which improves separation of water from built-up areas,
sand, and turbid coastal water — making it more reliable for Mediterranean shorelines.
Thresholds are computed automatically per year using Otsu's method instead of a fixed value.

### Data sources (all open, cloud-native)
| Variable | Source | Access |
|---|---|---|
| Multispectral imagery | Sentinel-2 L2A | Planetary Computer STAC |
| Sea Level Anomaly (SLA) | CMEMS SEALEVEL_GLO | `copernicusmarine` Python client |
| Significant Wave Height | CMEMS Wave reanalysis | `copernicusmarine` Python client |

### FAIR compliance
- ✅ All data accessed via public APIs — no local files
- ✅ Notebook fully reproducible on JupyterHub
- ✅ Outputs documented with standard metadata
- 🔄 Results to be published via Zenodo + STAC catalog

---
## Section 0 — Environment Setup

```python
# Core scientific stack
import numpy as np
import xarray as xr
import rioxarray
import pandas as pd
import geopandas as gpd
import warnings

# STAC access
import pystac_client
import planetary_computer

# CMEMS access
import copernicusmarine

# Raster utilities
from rioxarray.merge import merge_arrays
from rasterio.features import shapes
from shapely.geometry import shape
from scipy.ndimage import binary_opening
from skimage.filters import threshold_otsu

# Visualization
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn
import matplotlib.pyplot as plt
import folium

hv.extension('bokeh')
pn.extension()

print('✅ All imports successful')
```

---

### `repo-ciri/spain-v3.ipynb`

[View on GitHub](https://github.com/cirozugar/MAGICA-School-2026/blob/main/repo-ciri/spain-v3.ipynb)

# Coastal Dynamics — Tarragona Coast, Spain
## A Cloud-Native Time Series Analysis of Shoreline Change

**Open Science in Practice — FERS School, Bertinoro, April 2026**

---

### Scientific Question

> Has the coastline near Tarragona (NE Spain) changed in extent and morphology between 2017 and 2024,
> and if so, can those changes be linked to sea level anomaly or wave energy?
> Does the choice of water index (NDWI vs MNDWI) affect the detected change signal?

### Study Site

The study area (`bbox [0.55, 40.55, 1.05, 40.90]`) covers the Tarragona coastal zone
in Catalonia, NE Spain — including the Costa Daurada beaches, the Ebro Delta margin,
and nearshore Mediterranean waters. The area is sensitive to storm surges, longshore
sediment transport, and seasonal wave climatology.

### Methods: two water indices compared

| Index | Formula | Bands | Reference |
|---|---|---|---|
| **NDWI** | (Green − NIR) / (Green + NIR) | B03, B08 | McFeeters (1996) |
| **MNDWI** | (Green − SWIR) / (Green + SWIR) | B03, B11 | Xu (2006) |

Both indices range from −1 to +1. The water/land threshold is **fixed at 0** for both:
- index ≥ 0 → water
- index < 0 → land / beach / built-up

A fixed threshold of 0 is theoretically motivated (the original definition of both indices)
and empirically justified by inspecting the NDWI/MNDWI histograms of all three scenes
(Section 3b), which show the water–land transition consistently crossing zero.
An adaptive Otsu threshold was tested in v2 but produced a misplaced threshold for the
January 2021 scene due to a different illumination geometry, leading to overestimation
of land area. The fixed threshold is more robust across scenes from different months.

### Data sources (all open, cloud-native)
| Variable | Source | Access |
|---|---|---|
| Multispectral imagery | Sentinel-2 L2A | Planetary Computer STAC |
| Sea Level Anomaly (SLA) | CMEMS SEALEVEL_GLO | `copernicusmarine` Python client |
| Significant Wave Height | CMEMS Wave reanalysis | `copernicusmarine` Python client |

### FAIR compliance
- ✅ All data accessed via public APIs — no local files
- ✅ Notebook fully reproducible on JupyterHub
- ✅ Outputs documented with standard metadata
- 🔄 Results to be published via Zenodo + STAC catalog

---
## Section 0 — Environment Setup

```python
# Core scientific stack
import numpy as np
import xarray as xr
import rioxarray
import pandas as pd
import geopandas as gpd
import warnings

# STAC access
import pystac_client
import planetary_computer

# CMEMS access
import copernicusmarine

# Raster utilities
from rioxarray.merge import merge_arrays
from rasterio.features import shapes
from shapely.geometry import shape
from scipy.ndimage import binary_opening

# Visualization
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn
import matplotlib.pyplot as plt
import folium

hv.extension('bokeh')
pn.extension()

print('✅ All imports successful')
```

---

### `repo-ciri/spain_v5.ipynb`

[View on GitHub](https://github.com/cirozugar/MAGICA-School-2026/blob/main/repo-ciri/spain_v5.ipynb)

# Coastal Dynamics — Tarragona Coast, Spain
## A Cloud-Native Time Series Analysis of Shoreline Change

**Open Science in Practice — FERS School, Bertinoro, April 2026**

---

### Scientific Question

> Has the coastline near Tarragona (NE Spain) changed in extent and morphology between 2017 and 2024,
> and if so, can those changes be linked to sea level anomaly or wave energy?
> Does the choice of water index (NDWI vs MNDWI) affect the detected change signal?

### Study Site

The study area (`bbox [0.55, 40.55, 1.05, 40.90]`) covers the Tarragona coastal zone
in Catalonia, NE Spain — including the Costa Daurada beaches, the Ebro Delta margin,
and nearshore Mediterranean waters. The area is sensitive to storm surges, longshore
sediment transport, and seasonal wave climatology.

### Methods: two water indices compared

| Index | Formula | Bands | Reference |
|---|---|---|---|
| **NDWI** | (Green − NIR) / (Green + NIR) | B03, B08 | McFeeters (1996) |
| **MNDWI** | (Green − SWIR) / (Green + SWIR) | B03, B11 | Xu (2006) |

Both indices range from −1 to +1. The water/land threshold is **fixed at 0** for both:
- index ≥ 0 → water
- index < 0 → land / beach / built-up

A fixed threshold of 0 is theoretically motivated (the original definition of both indices)
and empirically justified by inspecting the NDWI/MNDWI histograms of all three scenes
(Section 3b), which show the water–land transition consistently crossing zero.
An adaptive Otsu threshold was tested in v2 but produced a misplaced threshold for the
January 2021 scene due to different illumination geometry, leading to overestimation
of land area. The fixed threshold is more robust across scenes from different months.

### Data sources (all open, cloud-native)
| Variable | Source | Access |
|---|---|---|
| Multispectral imagery | Sentinel-2 L2A | Planetary Computer STAC |
| Sea Level Anomaly (SLA) | CMEMS SEALEVEL_GLO | `copernicusmarine` Python client |
| Significant Wave Height | CMEMS Wave reanalysis | `copernicusmarine` Python client |
| Tidal level at acquisition | REDMAR Tarragona (3756N) | Puertos del Estado annual reports |

### FAIR compliance
- ✅ All data accessed via public APIs — no local files
- ✅ Notebook fully reproducible on JupyterHub
- ✅ Outputs documented with standard metadata
- 🔄 Results to be published via Zenodo + STAC catalog

---
## Section 0 — Environment Setup

```python
# Core scientific stack
import numpy as np
import xarray as xr
import rioxarray
import pandas as pd
import geopandas as gpd
import warnings

# STAC access
import pystac_client
import planetary_computer

# CMEMS access
import copernicusmarine

# Raster utilities
from rioxarray.merge import merge_arrays
from rasterio.features import shapes
from shapely.geometry import shape
from scipy.ndimage import binary_opening

# Visualization
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn
import matplotlib.pyplot as plt
import folium

hv.extension('bokeh')
pn.extension()

print('✅ All imports successful')
```

---

### `repo-ciri/tarragona_final.ipynb`

[View on GitHub](https://github.com/cirozugar/MAGICA-School-2026/blob/main/repo-ciri/tarragona_final.ipynb)

# Coastal Dynamics — Tarragona Coast, Spain
## A Cloud-Native Time Series Analysis of Shoreline Change

**Open Science in Practice — FERS School, Bertinoro, April 2026**

---

### Scientific Questions

>Has the coastline near Tarragona (NE Spain) changed in extent between 2017 and 2024?

>How have sea level anomaly and significant wave height behaved over the same period?

>Does the choice of water index (NDWI vs MNDWI) affect the detected change signal?

### Study Site

The study area (`bbox [0.55, 40.55, 1.05, 40.90]`) covers the Tarragona coastal zone
in Catalonia, NE Spain — including the Costa Daurada beaches, the Ebro Delta margin,
and nearshore Mediterranean waters. The area is sensitive to storm surges, longshore
sediment transport, and seasonal wave climatology.

### Methods: two water indices compared

| Index | Formula | Bands | Reference |
|---|---|---|---|
| **NDWI** | (Green − NIR) / (Green + NIR) | B03, B08 | McFeeters (1996) |
| **MNDWI** | (Green − SWIR) / (Green + SWIR) | B03, B11 | Xu (2006) |

Both indices range from −1 to +1. The water/land threshold is **fixed at 0** for both:
- index ≥ 0 → water
- index < 0 → land / beach / built-up

A fixed threshold of 0 is theoretically motivated (the original definition of both indices)
and empirically justified by inspecting the NDWI/MNDWI histograms of all three scenes
(Section 3b), which show the water–land transition consistently crossing zero.
An adaptive Otsu threshold was tested in v2 but produced a misplaced threshold for the
January 2021 scene due to different illumination geometry, leading to overestimation
of land area. The fixed threshold is more robust across scenes from different months.

### Data sources (all open, cloud-native)
| Variable | Source | Access |
|---|---|---|
| Multispectral imagery | Sentinel-2 L2A | Planetary Computer STAC |
| Sea Level Anomaly (SLA) | CMEMS SEALEVEL_GLO | `copernicusmarine` Python client |
| Significant Wave Height | CMEMS Wave reanalysis | `copernicusmarine` Python client |
| Tidal level at acquisition | REDMAR Tarragona (3756N) | Puertos del Estado annual reports |

---
## Section 0 — Environment Setup

```python
# Core scientific stack
import numpy as np
import xarray as xr
import rioxarray
import pandas as pd
import geopandas as gpd
import warnings

# STAC access
import pystac_client
import planetary_computer

# CMEMS access
import copernicusmarine

# Raster utilities
from rioxarray.merge import merge_arrays
from rasterio.features import shapes
from shapely.geometry import shape
from scipy.ndimage import binary_opening

# Visualization
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn
import matplotlib.pyplot as plt
import folium

hv.extension('bokeh')
pn.extension()

print('✅ All imports successful')
```

---

### `taranto-icechunk-append-ciri.ipynb`

[View on GitHub](https://github.com/cirozugar/MAGICA-School-2026/blob/main/taranto-icechunk-append-ciri.ipynb)

# Create or append Virtual Icechunk Store from SHYFEM forecast NetCDF files

* This notebook appends Taranto SHYFEM forecast data to Icechunk store using date-based set logic.
* If no repo exists, a new one will be created with references to all the existing NetCDF files. 

**Append Methodology:**
1. **`set_repo`**: extract all dates currently present in the Icechunk store's `time` coordinate
2. **`set_cloud`**: Scan the S3 bucket for all available NOS files and extract their dates.
3. **`new_dates`**: Calculate the difference (`set_cloud - set_repo`) to determine exactly which days need to be written for creation or appended.

```python
import warnings
import os
import pandas as pd
import fsspec
import xarray as xr
from dotenv import load_dotenv

warnings.filterwarnings("ignore", category=UserWarning)
```

```python
import icechunk
from obstore.store import from_url
from virtualizarr import open_virtual_dataset
from virtualizarr.parsers import HDFParser
from obspec_utils.registry import ObjectStoreRegistry
from obstore.store import S3Store
```

```python
# Load credentials
_ = load_dotenv(f'{os.environ["HOME"]}/dotenv/protocoast.env', override=True)

# Configuration
storage_endpoint = os.environ['ENDPOINT_URL']
source_bucket = 'protocoast-data'        # NetCDF source files
storage_bucket = 'ciri'                  # icechunk repo destination
storage_name = 'taranto-icechunk-tubitak-v1'
source_bucket_url = f"s3://{source_bucket}"
bucket_url = f"s3://{storage_bucket}"

# Setup Filesystem (reads from source bucket)
fs = fsspec.filesystem('s3', anon=False, endpoint_url=storage_endpoint,
                       skip_instance_cache=True, use_listings_cache=False)
```

```python
fs.ls(f'{source_bucket}/icechunk')
```

```python
# Define Icechunk Storage & Config
storage = icechunk.s3_storage(
    bucket=storage_bucket,
    prefix=f"icechunk/{storage_name}",
    from_env=True,
    endpoint_url=storage_endpoint,
    region='not-used',
    force_path_style=True,
)

config = icechunk.RepositoryConfig.default()
config.set_virtual_chunk_container(
    icechunk.VirtualChunkContainer(
        url_prefix=f"{source_bucket_url}/",
        store=icechunk.s3_store(region="not-used", anonymous=False, s3_compatible=True,
                                force_path_style=True, endpoint_url=storage_endpoint),
    ),
)

credentials = icechunk.containers_credentials({f"{source_bucket_url}/": icechunk.s3_credentials(anonymous=False)})

store_obj = S3Store(
    bucket=source_bucket,
    endpoint=storage_endpoint,
    region="not-used",
)

registry = ObjectStoreRegistry({source_bucket_url: store_obj})
parser = HDFParser()
```

---


## [RakeshSwain2604/MAGICA-School-2026](https://github.com/RakeshSwain2604/MAGICA-School-2026)

### `Project/copernicus_marine_model.ipynb`

[View on GitHub](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/copernicus_marine_model.ipynb)

# Copernicus Marine Service: Access Model Output
* Need to have an account on Copernicus Marine
* The ~/.netrc file contains the account credentials: your login and password for [Copernicus Marine](https://marine.copernicus.eu/)
```
$ more ~/.netrc
machine auth.marine.copernicus.eu
  login xxxxxxxxxx 
  password xxxxxxxxxxxxx
```

```python
import copernicusmarine
```

```python
%%time
ds = copernicusmarine.open_dataset(dataset_id='cmems_mod_ibi_wav_my_0.027deg-climatology_P1M-m')
```

```python
ds
```

```python
import hvplot.xarray
import panel as pn
pn.extension()
```

```python
#%%time
#da = ds['VHM0_mean'].sel(depth=0, time='2021-10-21 00:00', method='nearest').load()
```

```python
%%time

# 1. Select Direction (VMDR), remove depth, and slice the time range.
# Notice we DO NOT put .load() at the end!
da = ds['VHM0_mean'].sel(time=slice('2017-01-01', '2024-12-31'))

# 2. Plot the data. Because the 'time' dimension still exists in 'da', 
# hvplot will AUTOMATICALLY create an interactive time slider for you!
da.hvplot(
    x='longitude', 
    y='latitude', 
    rasterize=True, 
    geo=True, 
    frame_width=00, 
    cmap='twilight',   # 'twilight' is a cyclic colormap, perfect for 0-360 degree directions!
    tiles='OSM'
)
```

```python
"""
%%time

# 1. Select the exact location first using 'nearest'
da_point = ds['VHM0'].sel(
    latitude=40.7318, 
    longitude=0.8509, 
    method='nearest'
)

# 2. Then slice the time range (without the 'nearest' method)
da_point = da_point.sel(time=slice('2017-01-01', '2024-12-31'))

# 3. Load the 1D time series into memory
da_point = da_point.load()

# 4. Plot the interactive 2D line graph
da_point.hvplot.line(
    x='time', 
    y='VHM0', 
    title='Significant Wave Height Over Time',
    ylabel='Wave Height (meters)', 
    xlabel='Date',
    width=800, 
    height=400,
    color='blue'
)
```

---

### `Project/eopf_geozarr_demo.ipynb`

[View on GitHub](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/eopf_geozarr_demo.ipynb)

# EOPF GeoZarr Format — Interactive Exploration

This notebook demonstrates the usefulness of the **EOPF Zarr format** for cloud-native Sentinel-2 analysis:

- **No download required** — data streams directly from object storage
- **Analysis-ready** — the `eopf-zarr` xarray backend delivers properly georeferenced, scaled data
- **Resolution-aware** — request any resolution on open; bands are resampled on-the-fly
- **Interactive visualization** — explore imagery and indices with `hvplot`

We'll work over the **Ebro Delta, Spain** using Sentinel-2 L2A data.

## 1. Search the STAC catalog

```python
import warnings
warnings.filterwarnings('ignore')

import pystac_client

catalog = pystac_client.Client.open("https://stac.core.eopf.eodc.eu/")

# Ebro Delta, Spain
bbox = [0.55, 40.55, 1.05, 40.90]

search = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=bbox,
    datetime="2025-04-01/2025-06-01",
    query={"eo:cloud_cover": {"lt": 20}},
)

items = list(search.items())
print(f"Found {len(items)} items")
for item in items:
    print(f"  {item.id}  cloud_cover={item.properties.get('eo:cloud_cover', 'N/A')}%")
```

```python
! pip install xarray-eopf
```

## 2. Open with the `eopf-zarr` xarray backend

The `xarray-eopf` package registers an `eopf-zarr` engine that:
- Reads the EOPF Zarr product structure
- Applies scale/offset automatically (returning physical reflectance values)
- Attaches proper CRS coordinates (`x`, `y` in projected metres + `spatial_ref`)
- Resamples all bands to the requested `resolution` on-the-fly

```python
import xarray as xr

# Pick the least-cloudy scene
item = sorted(items, key=lambda i: i.properties.get('eo:cloud_cover', 999))[0]
url = item.assets['product'].href
print(f"Opening: {item.id}")
print(f"URL: {url}")

# Open at 60m resolution for fast exploration — change to 10m resolution for full detail
ds = xr.open_dataset(url, engine='eopf-zarr', resolution=60, chunks={})
ds
```

Notice:
- **`x` / `y` coordinates** in UTM metres — no manual georeferencing needed
- **`spatial_ref`** variable carries the full CRS (EPSG:32632)
- **All bands** at a single consistent resolution via one `open_dataset` call
- Values are **float64 surface reflectance** (scale/offset already applied)

---

### `Project/ibi_wave_icechunk.ipynb`

[View on GitHub](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/ibi_wave_icechunk.ipynb)

# IBI Wave Reanalysis 2017–2024 — IceChunk + Monthly/Half-yearly Visualisation

**Dataset**: [IBI_ANALYSISFORECAST_WAV_005_005](https://data.marine.copernicus.eu/product/IBI_ANALYSISFORECAST_WAV_005_005/services)  
**Strategy**:
1. Stream hourly data lazily from Copernicus Marine year-by-year
2. Write real zarr chunks into an IceChunk store on `protocoast-data` S3
3. Re-open the store, resample to monthly / half-yearly averages
4. Visualise with hvplot (maps, time-series, seasonal climatology)

## 1. Imports & Configuration

```python
import os
import warnings
import numpy as np
import pandas as pd
import xarray as xr
import icechunk
from icechunk.xarray import to_icechunk
import copernicusmarine
import hvplot.xarray
import panel as pn
import cartopy.crs as ccrs
from dotenv import load_dotenv

warnings.filterwarnings("ignore", category=UserWarning)
_ = load_dotenv(f'{os.environ["HOME"]}/dotenv/protocoast.env', override=True)
pn.extension()
```

```python
BUCKET       = "rswain"
STORE_NAME   = "ibi-wave-2017-2024-v1"
ENDPOINT_URL = os.environ["ENDPOINT_URL"]
DATASET_ID   = "cmems_mod_ibi_wav_anfc_0.027deg_PT1H-i"   # multi-year hindcast
YEARS        = range(2023, 2024)    # the time range
CHUNKS       = {"time": 168, "latitude": 50, "longitude": 50}  # 168h = 1 week

print(f"Endpoint : {ENDPOINT_URL}")
print(f"Store    : s3://{BUCKET}/icechunk/{STORE_NAME}")
```

## 2. Explore Dataset (run once to confirm dataset ID & variables)

If the dataset ID below is wrong, adjust `DATASET_ID` in cell 1 and re-run.

```python
copernicusmarine.describe(dataset_id=DATASET_ID)
```

# Dask deployment

```python
#cluster_type = 'Local'    
#cluster_type = 'Coiled'
cluster_type = 'Coiled'
#cluster_type = 'Coiled'
```

```python
if cluster_type == 'Coiled':
    import coiled

    # Forward S3 + Copernicus Marine credentials to every worker
    worker_env = {k: os.environ[k] for k in [
        "AWS_ACCESS_KEY_ID", "AWS_SECRET_ACCESS_KEY", "ENDPOINT_URL",
        "COPERNICUSMARINE_SERVICE_USERNAME", "COPERNICUSMARINE_SERVICE_PASSWORD",
    ] if k in os.environ}

    cluster = coiled.Cluster(
        region="eu-central-1",
        arm=True,
        worker_vm_types=["t4g.large"],  # 4 GB — avoids OOM on large chunks
        worker_options={'nthreads': 2},
        n_workers=30,
        wait_for_workers=False,
        compute_purchase_option="spot_with_fallback",
        software='protocoast-notebook-arm',
        workspace='esip-lab',
        name="rswain-large",
        timeout=180,
        environ=worker_env,
    )

    client = cluster.get_client()
```

---

### `Project/planetary_computer_1.ipynb`

[View on GitHub](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/planetary_computer_1.ipynb)

# Cloud Native Satellite Data Demo 1
* Use pystac_client to search the STAC API at Planetary Computer
* Use odc.stac to load the data
* Use hvplot to visualize the data

```python
import pystac_client
import planetary_computer
from rich.table import Table
import hvplot.xarray

# Connect to the Planetary Computer STAC API
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)
```

```python
collections = list(catalog.get_all_collections())
collections.sort(key=lambda c: c.id)
table = Table("ID", "Title", title="Planetary Computer collections")
for collection in collections:
    table.add_row(collection.id, collection.title)
table
```

```python
# Define our area of interest (AOI) - a rough box around spain
bbox = [0.55, 40.55, 1.05, 40.90]
```

```python
# Let's search for Sentinel-2 Level-2A data from this past summer
# with minimal cloud cover.
search = catalog.search(                                                                                                                                      
      collections=["sentinel-2-l2a"],                                                                                                                           
      bbox=bbox,                                                                                                                                                
      datetime="2024-11-01/2025-02-28",                                                                                                                         
      query={"eo:cloud_cover": {"lt": 10}},                                                                                                                     
  )   

# Get the results and see what we found
items = search.item_collection()
print(f"Found {len(items)} scenes matching your criteria.")

# Let's inspect the assets of the first item to see the available data bands
if items:
    first_item = items[0]
    print("\nAssets for the first scene:")
    for asset_key, asset in first_item.assets.items():
        print(f"- {asset_key}: {asset.title}")
```

```python
# Assuming 'items' is the ItemCollection from the first step
from odc.stac import stac_load


# Load the data using odc-stac.
# Note that we specify the dask chunks as a dictionary.
data_ds = stac_load(
    items,
    bands=["B04", "B03", "B02", "B08"], # B04=Red, B03=Green, B02=Blue, B08=NIR
    bbox=bbox,
    resolution=10,
    chunks={"x": 2048, "y": 2048},
)

# Let's inspect our new xarray Dataset.
# Notice the "Data variables" section.
print(data_ds)
```

---

### `Project/planetary_computer_2.ipynb`

[View on GitHub](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/planetary_computer_2.ipynb)

# Cloud Native Satellite Data Demo 2
* Generated by Gemini (Pro 2.5)
* Prompt said to use pystac_client, odc.stac, hvplot and panel

```python
import panel as pn
import pystac_client
import planetary_computer
import odc.stac
import rioxarray
import hvplot.xarray
import numpy as np
```

```python
# --- 1. Panel Setup and App Title ---
# Load the panel extension to make plots interactive
pn.extension()

# Create a title and description for our app
title = pn.pane.Markdown("## 🛰️ Cloud-Native Satellite Data Explorer", sizing_mode='stretch_width')
description = pn.pane.Markdown("""
Select a date from the dropdown below to view a true-color Sentinel-2 satellite image.
The map is fully interactive—you can pan and zoom. The data is loaded directly from the cloud!
""", sizing_mode='stretch_width')
```

```python
# --- 2. One-Time Data Loading ---
# This section runs only once when the app starts.
@pn.cache
def load_data():
    """
    Finds and loads satellite data using pystac and odc-stac.
    The pn.cache decorator prevents this from re-running on every interaction.
    """
    print("Searching for satellite imagery...")
    # Define our area of interest (AOI) - a rough box around Cape Cod
    bbox = [-71.0, 41.5, -69.8, 42.2]

    # Connect to the Planetary Computer STAC API
    catalog = pystac_client.Client.open(
        "https://planetarycomputer.microsoft.com/api/stac/v1",
        modifier=planetary_computer.sign_inplace,
    )

    # Search for Sentinel-2 Level-2A data with minimal cloud cover.
    # We use a recent, complete summer for good data (adjust if needed).
    search = catalog.search(
        collections=["sentinel-2-l2a"],
        bbox=bbox,
        datetime="2025-08-01/2025-10-30",
        query={"eo:cloud_cover": {"lt": 10}},
    )
    items = search.item_collection()
    if not items:
        return None, None # Handle case with no items found

    print(f"Found {len(items)} scenes. Loading data cube...")
    # Load the data using odc-stac
    data_ds = odc.stac.stac_load(
        items,
        bands=["B04", "B03", "B02"], # Red, Green, Blue
        bbox=bbox,
        resolution=50, # Can use a coarser resolution for faster app performance
        chunks={"x": 2048, "y": 2048},
    )

    # Attach the CRS information
    source_crs = 'EPSG:32619'
    data_ds = data_ds.rio.write_crs(source_crs)
    
    print("Data loading complete.")
    return data_ds, source_crs

# Execute the data loading function
data_cube, source_crs = load_data()
```

---

### `Project/spain-v1.ipynb`

[View on GitHub](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/Project/spain-v1.ipynb)

# Coastal Dynamics — Tarragona Coast, Spain
## A Cloud-Native Time Series Analysis of Shoreline Change

**Open Science in Practice FERS School, Bertinoro, April 2026**

---

### Scientific Question

> Has the coastline near Tarragona (NE Spain) changed in extent and morphology between 2017 and 2024,
> and if so, can those changes be linked to sea level anomaly or sea surface temperature?

### Study Site

The study area (bbox `[0.55, 40.55, 1.05, 40.90]`) covers the Tarragona coastal zone
in Catalonia, NE Spain — including the Costa Daurada beaches, the Ebro Delta margin,
and nearshore Mediterranean waters. The area is sensitive to storm surges, longshore
sediment transport, and seasonal wave climatology.

### Data sources (all open, cloud-native)
| Variable | Source | Access |
|---|---|---|
| Multispectral imagery | Sentinel-2 L2A | Planetary Computer STAC |
| Sea Level Anomaly (SLA) | CMEMS SEALEVEL_GLO | `copernicusmarine` Python client |
| Sea Surface Temperature | CMEMS SST GLO | `copernicusmarine` Python client |

### FAIR compliance
- All data accessed via public APIs — no local files
- Notebook fully reproducible on JupyterHub
- Outputs documented with standard metadata
- Results to be published via Zenodo + STAC catalog

### Section 0 — Environment Setup

```python
# Core scientific stack
import numpy as np
import xarray as xr
import rioxarray
import pandas as pd

# STAC access
import pystac_client
import planetary_computer

# CMEMS access
import copernicusmarine

# Visualization
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn

hv.extension('bokeh')
pn.extension()

print('All imports successful')
```

### Section 1 — Study Area Definition

```python
# Study area — Tarragona coast, NE Spain
BBOX   = [0.55, 40.55, 1.05, 40.90]   # [lon_min, lat_min, lon_max, lat_max]
YEARS  = list(range(2017, 2026))       # 2017–2025 (full Sentinel-2 archive)
CLOUD_COVER_MAX = 20                   # % — Mediterranean, full-year search

# Full-year windows — no dry-season restriction for Spain
DATE_WINDOWS = [(f'{y}-01-01', f'{y}-12-31') for y in YEARS]

print(f'Study area: {BBOX}')
print(f'Time windows: {DATE_WINDOWS[0]} → {DATE_WINDOWS[-1]}')
```

---

### `taranto-icechunk-append-rswain.ipynb`

[View on GitHub](https://github.com/RakeshSwain2604/MAGICA-School-2026/blob/main/taranto-icechunk-append-rswain.ipynb)

# Create or append Virtual Icechunk Store from SHYFEM forecast NetCDF files

* This notebook appends Taranto SHYFEM forecast data to an Icechunk store in the **rswain** bucket using date-based set logic.
* Source NetCDF files are read from `protocoast-data`; the Icechunk repo is written to `rswain`.
* If no repo exists, a new one will be created with references to all the existing NetCDF files. 

**Append Methodology:**
1. **`set_repo`**: extract all dates currently present in the Icechunk store's `time` coordinate
2. **`set_cloud`**: Scan the S3 bucket for all available NOS files and extract their dates.
3. **`new_dates`**: Calculate the difference (`set_cloud - set_repo`) to determine exactly which days need to be written for creation or appended.

```python
import warnings
import os
import pandas as pd
import fsspec
import xarray as xr
from dotenv import load_dotenv

warnings.filterwarnings("ignore", category=UserWarning)
```

```python
import icechunk
from obstore.store import from_url
from virtualizarr import open_virtual_dataset
from virtualizarr.parsers import HDFParser
from obspec_utils.registry import ObjectStoreRegistry
from obstore.store import S3Store
```

```python
# Load credentials
_ = load_dotenv(f'{os.environ["HOME"]}/dotenv/protocoast.env', override=True)

# Configuration
storage_endpoint = os.environ['ENDPOINT_URL']
icechunk_bucket = 'rswain'           # bucket where the Icechunk repo is stored
source_bucket = 'protocoast-data'    # bucket where source NetCDF files live
storage_name = 'taranto-icechunk-tubitak-v1'
icechunk_bucket_url = f"s3://{icechunk_bucket}"
source_bucket_url = f"s3://{source_bucket}"

# Setup Filesystem (for scanning source files)
fs = fsspec.filesystem('s3', anon=False, endpoint_url=storage_endpoint,
                       skip_instance_cache=True, use_listings_cache=False)
```

```python
# Define Icechunk Storage & Config
# Repo is written to the rswain bucket
storage = icechunk.s3_storage(
    bucket=icechunk_bucket,
    prefix=f"icechunk/{storage_name}",
    from_env=True,
    endpoint_url=storage_endpoint,
    region='not-used',
    force_path_style=True,
)

config = icechunk.RepositoryConfig.default()
# Virtual chunks point back to the source data in protocoast-data
config.set_virtual_chunk_container(
    icechunk.VirtualChunkContainer(
        url_prefix=f"{source_bucket_url}/",
        store=icechunk.s3_store(region="not-used", anonymous=False, s3_compatible=True,
                                force_path_style=True, endpoint_url=storage_endpoint),
    ),
)

credentials = icechunk.containers_credentials({f"{source_bucket_url}/": icechunk.s3_credentials(anonymous=False)})

store_obj = S3Store(
    bucket=source_bucket,
    endpoint=storage_endpoint,
    region="not-used",
)

registry = ObjectStoreRegistry({source_bucket_url: store_obj})
parser = HDFParser()
```

---


## [maexer1998/MAGICA-School-2026](https://github.com/maexer1998/MAGICA-School-2026)

### `groupproject/baltic_sea_sst_copernicusmarine.ipynb`

[View on GitHub](https://github.com/maexer1998/MAGICA-School-2026/blob/main/groupproject/baltic_sea_sst_copernicusmarine.ipynb)

# Baltic Sea SST Group Project
* Need to have an account on Copernicus Marine

```python
import copernicusmarine
import xarray as xr
import hvplot.xarray
import warnings
```

```python
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
dataset_id = 'DMI-BALTIC-SST-L4-NRT-OBS_FULL_TIME_SERIE'
```

```python
ds = copernicusmarine.open_dataset(dataset_id)
```

```python
ds = ds.chunk({'time': 10})
```

```python
%%time
da = ds['analysed_sst'].sel(time=slice('2026-03-20','2026-04-20')).load() - 273.15
```

```python
da.hvplot(x='longitude', y='latitude', rasterize=True, geo=True, cmap='turbo', tiles='OSM')
```

```python
del da
```

```python
%%time
da_baltic = ds['analysed_sst'].sel(time=slice('2026-03-20','2026-04-20'),latitude=slice(53.5, 65.9),longitude=slice(9.4, 30.0)).load() - 273.15
```

```python
lon = da_baltic.longitude
lat = da_baltic.latitude

# Cut Norwegian/Skagerrak waters (west of 16°E and north of 60°N)
no_norway = (lon >= 16.0) | (lat <= 60.0)
da_baltic = da_baltic.where(no_norway)

# Cut Lake Vänern (~58.3–59.5°N, 12.3–14.1°E) and Lake Vättern (~57.9–59.0°N, 14.3–15.0°E)
in_vanern  = (lat >= 58.3) & (lat <= 59.5) & (lon >= 12.3) & (lon <= 14.1)
in_vattern = (lat >= 57.7) & (lat <= 59.0) & (lon >= 14.0) & (lon <= 15.0)
da_baltic = da_baltic.where(~in_vanern & ~in_vattern)
```

```python
da_baltic.hvplot(x='longitude', y='latitude', rasterize=True, geo=True, cmap='turbo', tiles='OSM')
```

---

### `groupproject/baltic_sea_wave_copernicusmarine.ipynb`

[View on GitHub](https://github.com/maexer1998/MAGICA-School-2026/blob/main/groupproject/baltic_sea_wave_copernicusmarine.ipynb)

# Baltic Sea Wave Hindcast
* Need to have an account on Copernicus Marine

```python
import copernicusmarine
import xarray as xr
import hvplot.xarray
import warnings
```

```python
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
dataset_id = 'cmems_mod_bal_wav_my_PT1H-i'
```

```python
ds = copernicusmarine.open_dataset(dataset_id)
```

```python
ds
```

```python
%%time
da = ds['VMDR'].sel(time=slice('2023-10-19','2023-10-23')).load()
```

```python
da.hvplot(x='longitude', y='latitude', rasterize=True, geo=True, cmap='turbo', tiles='OSM')
```

## October 2023 Storm — German Baltic Coast

Storm **Babet** (20–21 Oct 2023) generated exceptional wave heights along the southern Baltic, with sea-level surges exceeding 1.5 m at Warnemünde.  
We examine five key wave parameters from the CMEMS Baltic Sea Wave Hindcast (1 nmi resolution, hourly):

| Variable | Description | Unit |
|---|---|---|
| `VHM0` | Significant spectral wave height | m |
| `VCMX` | Maximum individual wave height | m |
| `VHM0_WW` | Wind-sea significant wave height | m |
| `VTPK` | Peak spectral period | s |
| `VMDR` | Mean wave direction (met. convention) | ° |

```python
%%time
# Southern Baltic subset covering the German coast and the wider storm area
ds_storm = ds[['VHM0', 'VCMX', 'VHM0_WW', 'VTPK', 'VMDR']].sel(
    time=slice('2023-10-18', '2023-10-23'),
    latitude=slice(53.5, 58.0),
    longitude=slice(9.5, 22.0),
).load()
```

### Spatial map — max wave height with wave direction (time slider)

```python
import numpy as np
import holoviews as hv
import geoviews as gv
import cartopy.crs as ccrs
import panel as pn

pn.extension()

time_strs = [str(t)[:16] for t in ds_storm.time.values]
time_map = dict(zip(time_strs, ds_storm.time.values))

player = pn.widgets.Player(
    name="Time", options=time_strs,
    interval=200,        # ms between frames
    loop_policy="loop",
    width=700,
)

@pn.depends(player)
def combined_plot(time_str):
    snap = ds_storm.sel(time=time_map[time_str])

    bg = snap["VCMX"].hvplot(
        x="longitude", y="latitude", rasterize=True, geo=True,
        tiles="OSM", width=700, height=500, colorbar=True,
        clim=(0, 8), cmap="plasma",
        title=f"Max Wave Height (m) with Wave Direction — {time_str}",
    )

    stride = 8
    lons = snap.longitude.values[::stride]
    lats = snap.latitude.values[::stride]
    lon2d, lat2d = np.meshgrid(lons, lats)

    vmdr = snap["VMDR"].values[::stride, ::stride]
    vcmx_vals = snap["VCMX"].values[::stride, ::stride]

    angle_rad = np.deg2rad(270 - vmdr)
    magnitude = np.where(np.isfinite(vcmx_vals), vcmx_vals, 0)

    arrows = gv.VectorField(
        (lon2d.ravel(), lat2d.ravel(), angle_rad.ravel(), magnitude.ravel()),
        kdims=["Longitude", "Latitude"],
        vdims=["Angle", "Magnitude"],
        crs=ccrs.PlateCarree(),
    ).opts(scale=5, color="white", line_width=1.5)

    return bg * arrows

pn.Column(player, combined_plot)
```

---

### `groupproject/baltic_sst_yearly_comparison.ipynb`

[View on GitHub](https://github.com/maexer1998/MAGICA-School-2026/blob/main/groupproject/baltic_sst_yearly_comparison.ipynb)

# Baltic SST — Yearly Comparison

This notebook reads Baltic SST from Copernicus Marine, computes monthly means per year,
and produces four interactive plots:
1. **Spatial map** — monthly mean SST with a time slider
2. **Seasonal cycle** — area-mean SST lines overlaid for each year
3. **Point seasonal cycle** — monthly SST at Warnemünde (54.18 °N, 12.08 °E) per year
4. **Anomaly heatmap** — Warnemünde SST minus Baltic-wide area mean, month × year

`version 1.0.2` `23.04.2026`

```python
import copernicusmarine
import hvplot.xarray  # noqa
import warnings

warnings.filterwarnings('ignore', category=UserWarning)
warnings.filterwarnings('ignore', category=FutureWarning)
```

## Open dataset

```python
dataset_id = 'DMI-BALTIC-SST-L4-NRT-OBS_FULL_TIME_SERIE'
ds = copernicusmarine.open_dataset(dataset_id)
ds
```

## Spatial subset to the Baltic Sea

The Baltic proper sits roughly `lat 54–66 °N, lon 10–30 °E`.
Using `time=-1` in the chunk keeps all timesteps in a single chunk along
the time axis so that `resample().mean()` can scan each spatial tile in
one pass instead of assembling thousands of tiny cross-chunk tasks.

```python
ds
```

```python
# ds_baltic = (ds.sel(latitude=slice(54, 66), longitude=slice(8, 30)).chunk({'time': 100, 'latitude': 100, 'longitude': 100}))
ds_baltic = (ds.sel(latitude=slice(53.5, 66), longitude=slice(9.4, 30)))
ds_baltic
```

```python
lon = ds_baltic.longitude
lat = ds_baltic.latitude

# Cut Norwegian/Skagerrak waters (west of 16°E and north of 60°N)
no_norway = (lon >= 16.0) | (lat <= 60.0)
ds_baltic = ds_baltic.where(no_norway)

# Cut Lake Vänern (~58.3–59.5°N, 12.3–14.1°E) and Lake Vättern (~57.9–59.0°N, 14.3–15.0°E)
in_vanern  = (lat >= 58.3) & (lat <= 59.5) & (lon >= 12.3) & (lon <= 14.1)
in_vattern = (lat >= 57.7) & (lat <= 59.0) & (lon >= 14.0) & (lon <= 15.0)
ds_baltic = ds_baltic.where(~in_vanern & ~in_vattern)
```

```python
#cluster_type = 'Local'    
cluster_type = 'Coiled'
# cluster_type = 'Gateway'
#cluster_type = 'Coiled'
```

```python
if cluster_type == 'Coiled':
    import coiled
    region="eu-central-1"
    cluster = coiled.Cluster(
        region=region,
        arm=True,   # run on ARM to save energy & cost
        worker_vm_types=["t4g.large"],  # cheap, small ARM instances, 2cpus, 2GB RAM 
        # worker_options={'nthreads':2}, # Amazon does not charge for this
        n_workers=30,
        name=f'max_{region}',
        wait_for_workers=False,
        compute_purchase_option="spot_with_fallback",
        software='protocoast-notebook-arm',  # Conda environment name
        workspace='esip-lab',
        timeout=180   # leave cluster running for 3 min in case we want to use it again
    )

    client = cluster.get_client()
```

---

### `taranto-icechunk-append-max.ipynb`

[View on GitHub](https://github.com/maexer1998/MAGICA-School-2026/blob/main/taranto-icechunk-append-max.ipynb)

# Create or append Virtual Icechunk Store from SHYFEM forecast NetCDF files

* This notebook appends Taranto SHYFEM forecast data to Icechunk store using date-based set logic.
* Icechunk repo is written to bucket **`max`**; source NetCDF files are read from `protocoast-data`.
* If no repo exists, a new one will be created with references to all the existing NetCDF files. 

**Append Methodology:**
1. **`set_repo`**: extract all dates currently present in the Icechunk store's `time` coordinate
2. **`set_cloud`**: Scan the S3 bucket for all available NOS files and extract their dates.
3. **`new_dates`**: Calculate the difference (`set_cloud - set_repo`) to determine exactly which days need to be written for creation or appended.

```python
import warnings
import os
import pandas as pd
import fsspec
import xarray as xr
from dotenv import load_dotenv

warnings.filterwarnings("ignore", category=UserWarning)
```

```python
import icechunk
from obstore.store import from_url
from virtualizarr import open_virtual_dataset
from virtualizarr.parsers import HDFParser
from obspec_utils.registry import ObjectStoreRegistry
from obstore.store import S3Store
```

```python
# Load credentials
_ = load_dotenv(f'{os.environ["HOME"]}/dotenv/protocoast.env')

# Configuration
storage_endpoint = os.environ['ENDPOINT_URL']

# Source data bucket (NOS/OUS NetCDF files)
data_bucket = 'protocoast-data'
data_bucket_url = f"s3://{data_bucket}"

# Icechunk repo destination bucket
icechunk_bucket = 'max'
storage_name = 'taranto-icechunk-tubitak-v1'

# Setup Filesystem (for scanning source data)
fs = fsspec.filesystem('s3', anon=False, endpoint_url=storage_endpoint,
                       skip_instance_cache=True, use_listings_cache=False)
```

```python
# Define Icechunk Storage & Config
storage = icechunk.s3_storage(
    bucket=icechunk_bucket,
    prefix=f"icechunk/{storage_name}",
    from_env=True,
    endpoint_url=storage_endpoint,
    region='not-used',
    force_path_style=True,
)

config = icechunk.RepositoryConfig.default()
config.set_virtual_chunk_container(
    icechunk.VirtualChunkContainer(
        url_prefix=f"{data_bucket_url}/",
        store=icechunk.s3_store(region="not-used", anonymous=False, s3_compatible=True,
                                force_path_style=True, endpoint_url=storage_endpoint),
    ),
)

credentials = icechunk.containers_credentials({f"{data_bucket_url}/": icechunk.s3_credentials(anonymous=False)})

store_obj = S3Store(
    bucket=data_bucket,
    endpoint=storage_endpoint,
    region="not-used",
)

registry = ObjectStoreRegistry({data_bucket_url: store_obj})
parser = HDFParser()
```

---


## [MALEK313-sketch/MAGICA-School-2026](https://github.com/MALEK313-sketch/MAGICA-School-2026)

### `cmems_salinity_tunisia.ipynb`

[View on GitHub](https://github.com/MALEK313-sketch/MAGICA-School-2026/blob/main/cmems_salinity_tunisia.ipynb)

# Copernicus Marine — Sea Water Salinity, Gulf of Tunisia

**Product:** `MEDSEA_MULTIYEAR_PHY_006_004`  
**Dataset:** `cmems_mod_med_phy-sal_my_4.2km_P1D-m`  
**Variable:** `so` — sea water practical salinity (PSU) at 1 m depth  
**Period:** 2026-01-01 → 2026-03-31  
**Domain:** Gulf of Tunisia — lon 10°–10.9°E, lat 36.6°–37.2°N  
**Model:** Mediterranean Sea Physics Reanalysis (MFS E3R1I, 4.2 km, daily mean)

```python
%pip install copernicusmarine hvplot matplotlib xarray --quiet
```

```python
import copernicusmarine

# Saves credentials to ~/.copernicusmarine so you are not prompted again
copernicusmarine.login(
    username="mazzabi1",
    password="YOUR_PASSWORD_HERE",  # <-- replace with your Copernicus Marine password
    force_overwrite=True,
)
```

## 1. Open the salinity dataset (lazy, no download yet)

```python
import warnings
warnings.filterwarnings("ignore")

ds_sal = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-sal_my_4.2km_P1D-m",
    variables=["so"],
    minimum_longitude=10.0,
    maximum_longitude=10.9,
    minimum_latitude=36.6,
    maximum_latitude=37.2,
    start_datetime="2026-01-01T00:00:00",
    end_datetime="2026-03-31T00:00:00",
)
ds_sal
```

## 2. Inspect the dataset

```python
print("Variables :", list(ds_sal.data_vars))
print("Dimensions:", dict(ds_sal.sizes))
print("Time range:", str(ds_sal.time.values[0])[:10], "->", str(ds_sal.time.values[-1])[:10])
print("Depth levels:", ds_sal.depth.values)
```

## 3. Select surface level (1 m depth) and load into memory

```python
%%time
so = ds_sal["so"].isel(depth=0).load()

print(f"Salinity min : {float(so.min()):.2f} PSU")
print(f"Salinity max : {float(so.max()):.2f} PSU")
print(f"Salinity mean: {float(so.mean()):.2f} PSU")
```

## 4. Spatial map — salinity on the first day (2026-01-01)

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(11, 4.5))
so.isel(time=0).plot(ax=ax, cmap="Blues_r", cbar_kwargs={"label": "PSU"})
ax.set_title(f"Surface salinity (so, 1 m) — {str(so.time.values[0])[:10]}")
ax.set_xlabel("Longitude")
ax.set_ylabel("Latitude")
plt.tight_layout()
plt.show()
```

---

### `copernicus_marine-Tunisia.ipynb`

[View on GitHub](https://github.com/MALEK313-sketch/MAGICA-School-2026/blob/main/copernicus_marine-Tunisia.ipynb)

# Copernicus Marine Service: Access Earth Observation Data
* Need to have an account on Copernicus Marine
* The ~/.netrc file contains the account credentials: your login and password for [Copernicus Marine](https://marine.copernicus.eu/)
```
$ more ~/.netrc
machine auth.marine.copernicus.eu
  login xxxxxxxxxx 
  password xxxxxxxxxxxxx
```

```python
import copernicusmarine
import hvplot.xarray
```

```python
# turn of some annoying warnings
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
dataset_id = 'SST_MED_SST_L3S_NRT_OBSERVATIONS_010_012_b'
```

```python
ds = copernicusmarine.open_dataset(dataset_id)
```

```python
%%time
da = ds['adjusted_sea_surface_temperature'].sel(time='2025-10-01 16:00', method='nearest').load() - 273.15
```

```python
da.hvplot(x='longitude', y='latitude', rasterize=True, geo=True, cmap='turbo', tiles='OSM')
```

## Local CMEMS file: Mediterranean Physics (thetao & bottomT)

**Dataset**: `cmems_mod_med_phy-temp_my_4.2km_P1D-m` (Mediterranean Multiyear Physics, 4.2 km, daily).

Contains **90 days (Jan–Mar 2026)** of:
- `thetao` — potential temperature at 1 m depth
- `bottomT` — bottom temperature

```python
import xarray as xr
import numpy as np
import matplotlib.pyplot as plt
import hvplot.xarray  # noqa: F401

local_nc = 'cmems_mod_med_phy-temp_my_4.2km_P1D-m_1776862674750.nc'
ds_med = xr.open_dataset(local_nc)
ds_med
```

```python
# Inspect available variables, dimensions, and time range
print('Variables :', list(ds_med.data_vars))
print('Dimensions:', dict(ds_med.sizes))
print('Time range:', str(ds_med.time.values[0])[:10], '->', str(ds_med.time.values[-1])[:10])
```

### 1. Spatial map — surface temperature (`thetao` at 1 m) on the first day

```python
# Select first time step; squeeze depth if present
sst0 = ds_med['thetao'].isel(time=0)
if 'depth' in sst0.dims:
    sst0 = sst0.isel(depth=0)

fig, ax = plt.subplots(figsize=(11, 4.5))
im = sst0.plot(ax=ax, cmap='turbo', cbar_kwargs={'label': '°C'})
ax.set_title(f"Surface potential temperature (thetao, 1 m) — {str(sst0.time.values)[:10]}")
ax.set_xlabel('Longitude'); ax.set_ylabel('Latitude')
plt.tight_layout(); plt.show()
```

---

### `marine_heatwave_gulf_of_tunis.ipynb`

[View on GitHub](https://github.com/MALEK313-sketch/MAGICA-School-2026/blob/main/marine_heatwave_gulf_of_tunis.ipynb)

# Marine Heatwave Detection — Gulf of Tunis (2016–2025)

This notebook detects and analyses **marine heatwave (MHW) events** in the Gulf of Tunis over ten summer seasons (JAS: July–August–September, 2016–2025).

**Definition used (Hobday et al. 2016):** a pixel is classified as *in heatwave* on day *t* if its sea surface temperature has been ≥ the local **90th-percentile climatological threshold** for at least 5 consecutive days ending on day *t* (i.e., the 5-day trailing rolling minimum ≥ threshold).  The threshold is spatially varying — computed per pixel from the full JAS 2016–2025 record.  An *event* is a contiguous temporal period in which at least one pixel satisfies this condition.

| Step | Description |
|------|-------------|
| 1 | Load daily SST from CMEMS reprocessed L3S (Gulf of Tunis bbox) |
| 2 | Compute 90th-percentile threshold map per pixel |
| 3 | Detect heatwave pixels — rolling-minimum approach |
| 4 | Label and characterise events |
| 5 | Spatial map of MHW frequency |
| 6 | Interactive statistics plots (hvplot) |
| 7 | SST map of the largest event (peak day) |
| 8 | Chlorophyll-a map for the same date (Sentinel-3 OLCI via Planetary Computer) |

---
## Setup

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)

import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import copernicusmarine
import pystac_client
import planetary_computer
import hvplot.xarray
import hvplot.pandas
import panel as pn
pn.extension()

# Area of interest: Gulf of Tunis  [lon_min, lat_min, lon_max, lat_max]
bbox = [10.0, 36.6, 10.9, 37.2]

# Marine heatwave parameters (Hobday et al. 2016)
PERCENTILE = 90        # threshold percentile per pixel
MIN_DAYS   = 5         # consecutive days above threshold
MONTHS     = [7, 8, 9]              # JAS
YEARS      = list(range(2016, 2026))

print(f"Gulf of Tunis bbox : lon [{bbox[0]}, {bbox[2]}]  lat [{bbox[1]}, {bbox[3]}]")
print(f"MHW threshold      : {PERCENTILE}th percentile per pixel (Hobday et al. 2016)")
print(f"Min duration       : >= {MIN_DAYS} consecutive days")
print(f"Season / years     : JAS  {YEARS[0]}-{YEARS[-1]}")
```

---
## 1. Load CMEMS SST (JAS 2016–2025)

We use the **Mediterranean Sea SST L3S multi-year reprocessed** product, which provides daily gap-filled satellite-composite SST at ~0.01° resolution.

- **Dataset ID:** `cmems_SST_MED_PHY_L3S_MY_010_042_202211_P1D-m`
- **Variable:** `adjusted_sea_surface_temperature` (Kelvin)

> If the dataset ID raises a `ServiceNotFound` error, run `copernicusmarine.describe(contains=["SST_MED", "L3S", "MY"])` to find the current name.

```python
DATASET_ID = "SST_MED_SST_L3S_NRT_OBSERVATIONS_010_012_b"
VAR_NAME   = "adjusted_sea_surface_temperature"   # Kelvin

ds_sst = copernicusmarine.open_dataset(
    DATASET_ID,
    variables=[VAR_NAME],
    minimum_longitude=bbox[0],
    maximum_longitude=bbox[2],
    minimum_latitude=bbox[1],
    maximum_latitude=bbox[3],
    start_datetime="2016-07-01T00:00:00",
    end_datetime="2025-09-30T23:59:59",
)
print(f"Dataset dims : {dict(ds_sst.sizes)}")
print(f"Time range   : {str(ds_sst.time.values[0])[:10]} -> {str(ds_sst.time.values[-1])[:10]}")
```

---

### `planetary_computer_Tunis.ipynb`

[View on GitHub](https://github.com/MALEK313-sketch/MAGICA-School-2026/blob/main/planetary_computer_Tunis.ipynb)

# Cloud Native Satellite Data Demo 1
* Use pystac_client to search the STAC API at Planetary Computer
* Use odc.stac to load the data
* Use hvplot to visualize the data

```python
%pip install planetary-computer rich hvplot h5netcdf h5py
```

```python
import pystac_client
import planetary_computer
from rich.table import Table
import hvplot.xarray

# Connect to the Planetary Computer STAC API
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)
```

```python
collections = list(catalog.get_all_collections())
collections.sort(key=lambda c: c.id)
table = Table("ID", "Title", title="Planetary Computer collections")
for collection in collections:
    table.add_row(collection.id, collection.title)
#table
```

```python
# Define our area of interest (AOI) - a rough box around Cape Cod gulf of tunisia
bbox = [10.0, 36.6, 10.9, 37.2]
```

```python
# Let's search for Sentinel-3 SLSTR WST (Sea Surface Temperature) Level-2 data  NADIA
search = catalog.search(
    collections=["sentinel-3-slstr-wst-l2-netcdf"],
    bbox=bbox,
    datetime="2026-01-01/2026-04-21",
)

# Get the results and see what we found
items = search.item_collection()
print(f"Found {len(items)} scenes matching your criteria.")

# Let's inspect the assets of the first item to see the available data bands
if items:
    first_item = items[0]
    print("\nAssets for the first scene:")
    for asset_key, asset in first_item.assets.items():
        print(f"- {asset_key}: {asset.title}")
```

```python
import xarray as xr
import fsspec

# Prendre le premier item
item = items[0]

# Signer l'URL
asset = item.assets["l2p"]
signed_asset = planetary_computer.sign(asset)

print(f"URL : {signed_asset.href}")

# Ouvrir avec xarray
ds = xr.open_dataset(fsspec.open(signed_asset.href).open())
print(ds)
```

---

### `socib-wmop-Med.ipynb`

[View on GitHub](https://github.com/MALEK313-sketch/MAGICA-School-2026/blob/main/socib-wmop-Med.ipynb)

# Explore SOCIB WMOP Model — Full Western Mediterranean Domain
* Access data from the **SOCIB THREDDS Server using OPeNDAP** (same approach as `socib-wmop-best.ipynb`)
* Full WMOP domain: 5.8°W – 9.2°E, 34.9°N – 44.7°N
* Variables available: `temp`, `salt`, `u`, `v`, `ubar`, `vbar`, `zeta`
* Visualize and explore with **HoloViz** (interactive maps with time slider)

```python
import xarray as xr
import hvplot.xarray
import panel as pn
import pandas as pd
import warnings
warnings.filterwarnings('ignore', category=UserWarning)
warnings.filterwarnings('ignore', category=FutureWarning)
```

```python
# Rolling 5-day window ending now
stop_time  = pd.Timestamp.now()
start_time = stop_time - pd.DateOffset(days=5)

print(f"Start Time: {start_time}")
print(f"Stop Time:  {stop_time}")
```

## Open the WMOP "Best Time Series" via OPeNDAP

The SOCIB THREDDS server exposes the WMOP surface Best Time Series (most recent model run) at:
```
http://thredds.socib.es/thredds/dodsC/operational_models/oceanographical/hydrodynamics/
    model_run_aggregation/wmop_surface/wmop_surface_best.ncd
```
Opening with `xr.open_dataset()` is lazy — no data are downloaded until `.load()` or `.sel()` is called.

```python
dap_url = (
    'http://thredds.socib.es/thredds/dodsC/'
    'operational_models/oceanographical/hydrodynamics/'
    'model_run_aggregation/wmop_surface/wmop_surface_best.ncd'
)

ds = xr.open_dataset(dap_url)
ds
```

```python
# Inspect spatial extent and available variables
print('Variables:', list(ds.data_vars))
print('Lat range:', float(ds['lat_rho'].min()), '–', float(ds['lat_rho'].max()), '°N')
print('Lon range:', float(ds['lon_rho'].min()), '–', float(ds['lon_rho'].max()), '°E')
print('Time range:', str(ds['time'].values[0])[:16], '–', str(ds['time'].values[-1])[:16])
```

## Colour scale

```python
temp_clim = (13, 26)   # °C — full Western Med seasonal range
salt_clim = (36, 39)   # PSU
zeta_clim = (-0.3, 0.3)  # m — sea surface height anomaly
```

## Sea Surface Temperature — interactive map with time slider

```python
%%time
da_temp = ds['temp'].sel(time=slice(start_time, stop_time)).load()

temp_viz = da_temp.hvplot.quadmesh(
    x='lon_rho', y='lat_rho',
    rasterize=True,
    cmap='turbo',
    geo=True, tiles='OSM',
    clim=temp_clim,
    clabel='°C',
    title='WMOP Sea Surface Temperature (°C) — Full Western Mediterranean',
    frame_width=800,
    widgets={'time': pn.widgets.Select},
)
```

---


## [LauraF97/MAGICA-School-2026](https://github.com/LauraF97/MAGICA-School-2026)

### `LAR(Laura_Angela_Roberta)_Project/SSC+Streamlines_Analysis.ipynb`

[View on GitHub](https://github.com/LauraF97/MAGICA-School-2026/blob/main/LAR(Laura_Angela_Roberta)_Project/SSC+Streamlines_Analysis.ipynb)

(could not fetch content)

---

### `LAR(Laura_Angela_Roberta)_Project/SSC_Analysis.ipynb`

[View on GitHub](https://github.com/LauraF97/MAGICA-School-2026/blob/main/LAR(Laura_Angela_Roberta)_Project/SSC_Analysis.ipynb)

```python
import copernicusmarine
```

```python
%%time
ds = copernicusmarine.open_dataset(dataset_id='cmems_mod_med_phy-cur_my_4.2km_P1D-m')
```

```python
# Gulf of Lion bounding box
lon_min, lon_max = 2.0, 7.5
lat_min, lat_max = 41.0, 44.5

dates = ["2026-03-21", "2026-03-31"]
uvs = []
for date in dates:
    uv = (
        ds[["uo", "vo"]]
        .sel(time=date, method="nearest")
        .isel(depth=0)
        .sel(longitude=slice(lon_min, lon_max), latitude=slice(lat_min, lat_max))
    )
    uvs.append(uv)
```

```python
import numpy as np
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
import cartopy.feature as cfeature

ref_speed = 0.3
step = 3

# Compute global vmin/vmax across both dates for a consistent colorscale
speeds = [np.sqrt(uv["uo"]**2 + uv["vo"]**2) for uv in uvs]
vmin = float(min(np.nanmin(s) for s in speeds))
vmax = float(max(np.nanmax(s) for s in speeds))

fig, axes = plt.subplots(1, 2, figsize=(18, 7),
                         subplot_kw={"projection": ccrs.PlateCarree()})
fig.subplots_adjust(right=0.88, wspace=0.35)

for ax, uv, speed, date in zip(axes, uvs, speeds, dates):
    uo = uv["uo"]
    vo = uv["vo"]
    speed_min = float(np.nanmin(speed))
    speed_max = float(np.nanmax(speed))
    print(f"{date}  |  Speed min: {speed_min:.4f} m/s  max: {speed_max:.4f} m/s")

    im = ax.pcolormesh(
        uo.longitude, uo.latitude, speed,
        cmap="magma", transform=ccrs.PlateCarree(), vmin=vmin, vmax=vmax,
    )
    ax.quiver(
        uo.longitude.values[::step], uo.latitude.values[::step],
        uo.values[::step, ::step], vo.values[::step, ::step],
        transform=ccrs.PlateCarree(), scale=10, width=0.003, color="white", alpha=0.8,
    )

    # Reference arrow on land (top-right, French Riviera area)
    ref_lon, ref_lat = 5.8, 43.9
    ax.quiver(ref_lon, ref_lat, ref_speed, 0,
              transform=ccrs.PlateCarree(), scale=10, width=0.003,
              color="black", zorder=10)
    ax.text(ref_lon + 0.35, ref_lat, f"{ref_speed} m/s",
            transform=ccrs.PlateCarree(), fontsize=9,
            va="center", ha="left", color="black", zorder=11,
            bbox=dict(facecolor="white", alpha=0.5, edgecolor="none", pad=1))

    ax.add_feature(cfeature.LAND, zorder=5, facecolor="lightgray")
    ax.add_feature(cfeature.COASTLINE, zorder=6, linewidth=0.5)
    ax.gridlines(draw_labels=True, linewidth=0.4, linestyle="--", alpha=0.5)
    ax.set_extent([lon_min, lon_max, lat_min, lat_max], crs=ccrs.PlateCarree())
    ax.set_title(f"Surface current speed — Gulf of Lion — {date}\nmin={speed_min:.3f} m/s  max={speed_max:.3f} m/s")

cbar_ax = fig.add_axes([0.90, 0.15, 0.02, 0.7])
fig.colorbar(im, cax=cbar_ax, label="Current speed (m/s)")
plt.suptitle("Surface currents (uo, vo)", fontsize=13)
plt.show()
```

---

### `LAR(Laura_Angela_Roberta)_Project/SSC_socibmodel.ipynb`

[View on GitHub](https://github.com/LauraF97/MAGICA-School-2026/blob/main/LAR(Laura_Angela_Roberta)_Project/SSC_socibmodel.ipynb)

```python
import xarray as xr
import hvplot.xarray
from datetime import datetime
import panel as pn
```

```python
import pandas as pd

# Get the current date and time
stop_time= pd.Timestamp.now()

# Subtract 10 days using pd.DateOffset
start_time = stop_time - pd.DateOffset(days=3)

print(f"Start Time: {start_time}")
print(f"Stop Time: {stop_time}")
```

```python
dap_url = 'http://thredds.socib.es/thredds/dodsC/operational_models/oceanographical/hydrodynamics/model_run_aggregation/wmop_surface/wmop_surface_best.ncd'
```

```python
ds = xr.open_dataset(dap_url)
```

```python
# Variabili disponibili nel dataset
ds
```

```python
%%time
# Correnti superficiali 21-31 marzo 2026
u = ds['u'].sel(time=slice('2026-03-21', '2026-03-31')).load()
v = ds['v'].sel(time=slice('2026-03-21', '2026-03-31')).load()
print(u)
print(v)
```

# Mappe di Circolazione Superficiale - WMOP SOCIB\n## Media giornaliera per 21 marzo e 31 marzo 2026

```python
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import numpy as np
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import xarray as xr

# Carica il dataset se non già in memoria
try:
    u
    v
except NameError:
    dap_url = 'http://thredds.socib.es/thredds/dodsC/operational_models/oceanographical/hydrodynamics/model_run_aggregation/wmop_surface/wmop_surface_best.ncd'
    ds = xr.open_dataset(dap_url)
    u = ds['u'].sel(time=slice('2026-03-21', '2026-03-31')).load()
    v = ds['v'].sel(time=slice('2026-03-21', '2026-03-31')).load()

LON = slice(2.0, 6.5)
LAT = slice(41.0, 44.5)
DATES = ['2026-03-21', '2026-03-31']
TITLES = ['21 March 2026', '31 March 2026']

datasets = []
for date in DATES:
    u_day = u.sel(time=date).mean(dim='time').sel(lon_uv=LON, lat_uv=LAT)
    v_day = v.sel(time=date).mean(dim='time').sel(lon_uv=LON, lat_uv=LAT)
    spd   = np.sqrt(u_day**2 + v_day**2)
    datasets.append({'u': u_day, 'v': v_day, 'spd': spd})

vmin   = float(min(d['spd'].min() for d in datasets))
vmax   = float(max(d['spd'].max() for d in datasets))
levels = np.linspace(vmin, vmax, 70)

print(f"vmin={vmin:.4f}  vmax={vmax:.4f} m/s")
print(f"Griglia: {datasets[0]['u'].lon_uv.size} lon × {datasets[0]['u'].lat_uv.size} lat")
```

---

### `LAR(Laura_Angela_Roberta)_Project/Salinity_Analysis.ipynb`

[View on GitHub](https://github.com/LauraF97/MAGICA-School-2026/blob/main/LAR(Laura_Angela_Roberta)_Project/Salinity_Analysis.ipynb)

(could not fetch content)

---

### `LAR(Laura_Angela_Roberta)_Project/T-S.ipynb`

[View on GitHub](https://github.com/LauraF97/MAGICA-School-2026/blob/main/LAR(Laura_Angela_Roberta)_Project/T-S.ipynb)

(could not fetch content)

---

### `LAR(Laura_Angela_Roberta)_Project/Temperature_Analysis.ipynb`

[View on GitHub](https://github.com/LauraF97/MAGICA-School-2026/blob/main/LAR(Laura_Angela_Roberta)_Project/Temperature_Analysis.ipynb)

(could not fetch content)

---


## [pendaoriana47-svg/MAGICA-School-2026](https://github.com/pendaoriana47-svg/MAGICA-School-2026)

### `Project 2.ipynb`

[View on GitHub](https://github.com/pendaoriana47-svg/MAGICA-School-2026/blob/main/Project 2.ipynb)

```python
import warnings
import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import os
warnings.simplefilter('ignore') # filter some warning messages
xr.set_options(display_style="html")  #display dataset nicely
```

```python
%%time

import coiled

cluster_type = "Coiled"

if cluster_type == "Coiled":
    cluster = coiled.Cluster(
        region="eu-central-1",
        arm=True,  # run on ARM to save energy & cost
        worker_vm_types=["t4g.medium"],  # cheap ARM instances
        worker_options={"nthreads": 2},
        n_workers=200,
        name='Oriana',
        wait_for_workers=False,
        compute_purchase_option="spot_with_fallback",
        software="protocoast-notebook-arm",
        workspace="esip-lab",
        timeout=180  # keep cluster alive for 3 min
    )
```

```python
client = cluster.get_client()
client
```

```python
%%time

ds_sst = xr.open_zarr('https://mur-sst.s3.us-west-2.amazonaws.com/zarr-v1',consolidated=True)

ds_sst
```

```python
sst = ds_sst['analysed_sst']
sst_subset = sst.sel(
    lat=slice(-25, 25),   # reversed if lat is descending
    lon=slice(-180, -70)
)
```

```python
# MONTHLY MEAN 
sst_monthly = sst_subset.resample(time='1MS').mean('time', keep_attrs=True, skipna=False)

# MONTHLY CLIMATOLOGY ---
climatology_mean_monthly = sst_monthly.groupby('time.month').mean('time', keep_attrs=True, skipna=False)

# MONTHLY ANOMALY ---
sst_anomaly_monthly = (sst_monthly.groupby('time.month') - climatology_mean_monthly)

# OUTPUT 
sst_anomaly_monthly
```

---

### `Project.ipynb`

[View on GitHub](https://github.com/pendaoriana47-svg/MAGICA-School-2026/blob/main/Project.ipynb)

```python
import warnings
import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import os
warnings.simplefilter('ignore') # filter some warning messages
xr.set_options(display_style="html")  #display dataset nicely
```

```python
%%time

import coiled

cluster_type = "Coiled"

if cluster_type == "Coiled":
    cluster = coiled.Cluster(
        region="eu-central-1",
        arm=True,  # run on ARM to save energy & cost
        worker_vm_types=["t4g.medium"],  # cheap ARM instances
        worker_options={"nthreads": 2},
        n_workers=200,
        wait_for_workers=False,
        compute_purchase_option="spot_with_fallback",
        software="protocoast-notebook-arm",
        workspace="esip-lab",
        timeout=180  # keep cluster alive for 3 min
    )
```

```python
client = cluster.get_client()
client
```

```python
%%time

ds_sst = xr.open_zarr('https://mur-sst.s3.us-west-2.amazonaws.com/zarr-v1',consolidated=True)

ds_sst
```

```python
sst = ds_sst['analysed_sst']
sst_subset = sst.sel(
    lat=slice(-20, 20),   # reversed if lat is descending
    lon=slice(100, 160)
)
```

```python
sst_subset
```

```python
# MONTHLY MEAN 
sst_monthly = sst_subset.resample(time='1MS').mean('time', keep_attrs=True, skipna=False)

# MONTHLY CLIMATOLOGY ---
climatology_mean_monthly = sst_monthly.groupby('time.month').mean('time', keep_attrs=True, skipna=False)

# MONTHLY ANOMALY ---
sst_anomaly_monthly = (sst_monthly.groupby('time.month') - climatology_mean_monthly)

# OUTPUT 
sst_anomaly_monthly
```

---

### `Project_SST.ipynb`

[View on GitHub](https://github.com/pendaoriana47-svg/MAGICA-School-2026/blob/main/Project_SST.ipynb)

## Project El Nino effects

Import packages

```python
import warnings
import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import os
from dask_gateway import Gateway
import hvplot.xarray

warnings.simplefilter('ignore') # filter some warning messages
xr.set_options(display_style="html")  #display dataset nicely
```

## Set up Dask Cluster

```python
# 1. Initialize Gateway
gateway = Gateway()

# 2. Configure Cluster Options
options = gateway.cluster_options()
# Ensures workers use the same environment as your notebook
options.image = os.environ.get('JUPYTER_IMAGE', 'default') 

# 3. Create and Scale Cluster
cluster = gateway.new_cluster(options)

# We set a minimum of 4 workers to start immediately, 
# but allow scaling up to 30 for the heavy climatology math.
cluster.adapt(minimum=4, maximum=40)

# 4. Connect the Client
client = cluster.get_client()

# Display the dashboard link
print(f"Cluster is up! View progress here: {cluster.dashboard_link}")
client
```

```python
if cluster_type == 'Coiled':
    import coiled
    cluster = coiled.Cluster(
        region="us-west-2",
        arm=True,   # run on ARM to save energy & cost
        worker_vm_types=["t4g.small"],  # cheap, small ARM instances, 2cpus, 2GB RAM
        worker_options={'nthreads':2},
        n_workers=30,
        wait_for_workers=False,
        compute_purchase_option="spot_with_fallback",
        name='coawst',   # Dask cluster name
        software='protocoast-develop-arm',  # Conda environment name
        workspace='osc-aws',
        timeout=180   # leave cluster running for 3 min in case we want to use it again
    )

    client = cluster.get_client()
```

#Explore data

```python
%%time

ds_sst = xr.open_zarr('https://mur-sst.s3.us-west-2.amazonaws.com/zarr-v1',consolidated=True)

ds_sst
```

---

### `Untitled.ipynb`

[View on GitHub](https://github.com/pendaoriana47-svg/MAGICA-School-2026/blob/main/Untitled.ipynb)

(could not fetch content)

---

### `Untitled1.ipynb`

[View on GitHub](https://github.com/pendaoriana47-svg/MAGICA-School-2026/blob/main/Untitled1.ipynb)

```python
import warnings
import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import os
warnings.simplefilter('ignore') # filter some warning messages
xr.set_options(display_style="html")  #display dataset nicely
```

```python
%%time

import coiled

cluster_type = "Coiled"

if cluster_type == "Coiled":
    cluster = coiled.Cluster(
        region="eu-central-1",
        arm=True,  # run on ARM to save energy & cost
        worker_vm_types=["t4g.medium"],  # cheap ARM instances
        worker_options={"nthreads": 2},
        n_workers=200,
        name='Oriana',
        wait_for_workers=False,
        compute_purchase_option="spot_with_fallback",
        software="protocoast-notebook-arm",
        workspace="esip-lab",
        timeout=180  # keep cluster alive for 3 min
    )
```

```python
client = cluster.get_client()
client
```

```python
%%time

ds_sst = xr.open_zarr('https://mur-sst.s3.us-west-2.amazonaws.com/zarr-v1',consolidated=True)

ds_sst
```

```python
sst = ds_sst['analysed_sst']
sst_subset = sst.sel(
    lat=slice(-20, 20),   # reversed if lat is descending
    lon=slice(100, 160)
)
```

```python
# MONTHLY MEAN 
sst_monthly = sst_subset.resample(time='1MS').mean('time', keep_attrs=True, skipna=False)

# MONTHLY CLIMATOLOGY ---
climatology_mean_monthly = sst_monthly.groupby('time.month').mean('time', keep_attrs=True, skipna=False)

# MONTHLY ANOMALY ---
sst_anomaly_monthly = (sst_monthly.groupby('time.month') - climatology_mean_monthly)

# OUTPUT 
sst_anomaly_monthly
```

---

### `aws_mur_sst_tutorial_short.ipynb`

[View on GitHub](https://github.com/pendaoriana47-svg/MAGICA-School-2026/blob/main/aws_mur_sst_tutorial_short.ipynb)

# Tutorial for MUR SST on AWS  

- Funding: Interagency Implementation and Advanced Concepts Team [IMPACT](https://earthdata.nasa.gov/esds/impact) for the Earth Science Data Systems (ESDS) program and AWS Public Dataset Program

Credits: Tutorial development
* [Dr. Chelle Gentemann](mailto:gentemann@faralloninstitute.org) -  [Twitter](https://twitter.com/ChelleGentemann)   - Farallon Institute
* [Dr. Rich Signell](mailto:rsignell@usgs.gov) - [Twitter](https://twitter.com/rsignell) - USGS
* [Dr. Ryan Abernathey](mailto:rpa@ldeo.columbia.edu) - [Twitter](https://twitter.com/rabernat) - LDEO


Credits: Creating of the Zarr MUR SST dataset.  

* [Aimee Barciauskas](mailto:aimee@developmentseed.org) - [Twitter](https://twitter.com/_aimeeb) - Development Seed
* [Dr. Rich Signell](mailto:rsignell@usgs.gov) - [Twitter](https://twitter.com/rsignell) - USGS
* [Dr. Chelle Gentemann](mailto:gentemann@faralloninstitute.org)  -  [Twitter](https://twitter.com/ChelleGentemann) - Farallon Institute
* [Joseph Flasher](mailto:jflasher@amazon.com) [Twitter](https://twitter.com/joseph_flasher) - AWS

Credits: Tutorial review and comments.
* [Dr. Ed Armstrong](mailto:edward.m.armstrong@jpl.nasa.gov) - JPL PODAAC

-------------

## Please note that this is global, 1 km, daily data.  This is a very large dataset and the analyses below can take up to 5-10 minutes

## [MUR SST](https://podaac.jpl.nasa.gov/Multi-scale_Ultra-high_Resolution_MUR-SST) [AWS Public dataset program](https://registry.opendata.aws/mur/) 

### This Pangeo binder is faster when run on AWS.  

![image](https://podaac.jpl.nasa.gov/Podaac/thumbnails/MUR-JPL-L4-GLOB-v4.1.jpg)

This code shows how to read from a s3 bucket.  
Right now (2/16/2020) this takes ~1min on AWS and ~2 min on google cloud, there are couple issues here and we are working to solve both.  
1. Some shortcomings in the s3fs and zarr formats have been identified.  To work on these, git issues were raised to the developers [here](https://github.com/dask/s3fs/issues/285) and [here](https://github.com/zarr-developers/zarr-python/issues/536)

# To run this notebook

Code is in the cells that have <span style="color: blue;">In [  ]:</span> to the left of the cell and have a colored background

To run the code:
- option 1) click anywhere in the cell, then hold shift and press Enter
- option 2) click on the Run button at the top of the page in the dashboard

# Structure of this tutorial

1. Opening data
2. Data exploration

# 1. Opening data

-------------------

## Import python packages

It is nice to turn off warnings and set xarray display options.

```python
import warnings
import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import os
warnings.simplefilter('ignore') # filter some warning messages
xr.set_options(display_style="html")  #display dataset nicely
```

---

### `code for hvplot.ipynb`

[View on GitHub](https://github.com/pendaoriana47-svg/MAGICA-School-2026/blob/main/code for hvplot.ipynb)

```python
import warnings
import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import os
warnings.simplefilter('ignore') # filter some warning messages
xr.set_options(display_style="html")  #display dataset nicely
```

```python
%%time

import coiled

cluster_type = "Coiled"

if cluster_type == "Coiled":
    cluster = coiled.Cluster(
        region="eu-central-1",
        arm=True,  # run on ARM to save energy & cost
        worker_vm_types=["t4g.medium"],  # cheap ARM instances
        worker_options={"nthreads": 2},
        n_workers=200,
        name='Oriana',
        wait_for_workers=False,
        compute_purchase_option="spot_with_fallback",
        software="protocoast-notebook-arm",
        workspace="esip-lab",
        timeout=180  # keep cluster alive for 3 min
    )
```

```python
client = cluster.get_client()
client
```

```python
%%time

ds_sst = xr.open_zarr('https://mur-sst.s3.us-west-2.amazonaws.com/zarr-v1',consolidated=True)

ds_sst
```

```python
sst = ds_sst['analysed_sst']
sst_subset = sst.sel(
    lat=slice(-20, 20),   # reversed if lat is descending
    lon=slice(100, 160)
)
```

```python
# MONTHLY MEAN 
sst_monthly = sst_subset.resample(time='1MS').mean('time', keep_attrs=True, skipna=False)

# MONTHLY CLIMATOLOGY ---
climatology_mean_monthly = sst_monthly.groupby('time.month').mean('time', keep_attrs=True, skipna=False)

# MONTHLY ANOMALY ---
sst_anomaly_monthly = (sst_monthly.groupby('time.month') - climatology_mean_monthly)

# OUTPUT 
sst_anomaly_monthly
```

---

### `main.ipynb`

[View on GitHub](https://github.com/pendaoriana47-svg/MAGICA-School-2026/blob/main/main.ipynb)

# OpenScience Project: Equatorial Pacific SST Analysis and its Influence in Peruvian Coast.

High resolution MUR SST dataset at AWS available at: https://registry.opendata.aws/mur/

```python
import warnings
import numpy as np
import pandas as pd
import xarray as xr
import fsspec
import os
import coiled

warnings.simplefilter('ignore') # filter some warning messages
xr.set_options(display_style="html")  #display dataset nicely
```

## 1. Cluster: Dask

```python
%%time

cluster_type='Coiled'

if cluster_type == 'Coiled': 
    cluster=coiled.Cluster(
        region='us-west-2',
        arm=True,
        worker_vm_types=['t4g.large'],
        worker_options={"nthreads": 2},
        n_workers= 40,
        name='Vsazonova-us-west-2-large',
        wait_for_workers=False,
        compute_purchase_option='spot_with_fallback',
        software='protocoast-notebook-arm',
        workspace='esip-lab',
    )
```

```python
client = cluster.get_client()
client
```

## 2. Equatorial Pacific SST Analysis

### Initializing the data

```python
%%time

ds_sst = xr.open_zarr('https://mur-sst.s3.us-west-2.amazonaws.com/zarr-v1',consolidated=True)
ds_sst
```

```python
sst = ds_sst['analysed_sst']

cond = (ds_sst.mask==1) & ((ds_sst.sea_ice_fraction<.15) | np.isnan(ds_sst.sea_ice_fraction))
sst_masked = ds_sst['analysed_sst'].where(cond)
sst_masked
```

```python
sst_elnino = sst_masked.sel(lon=slice(-180,-70), lat=slice(-25,25)) - 273.15 # Slicing EP region for memory safety
```

### Diagnostics: monthly mean, anomalies

```python
%%time
#create a daily climatology and anomaly
climatology_mean = sst_elnino.groupby('time.dayofyear').mean('time',keep_attrs=True,skipna=False)
sst_anomaly = sst_elnino.groupby('time.dayofyear')-climatology_mean  #take out annual mean to remove trends

#create a monthly dataset, climatology, and anomaly
sst_monthly = sst_elnino.resample(time='1MS').mean('time',keep_attrs=True,skipna=False)
climatology_mean_monthly = sst_monthly.groupby('time.month').mean('time',keep_attrs=True,skipna=False)
sst_anomaly_monthly = sst_monthly.groupby('time.month')-climatology_mean_monthly  #take out annual mean to remove trends

sst_anomaly
```

---


## [virginiasofiacampanella-beep/MAGICA-School-2026](https://github.com/virginiasofiacampanella-beep/MAGICA-School-2026)

### `Project_SST.ipynb`

[View on GitHub](https://github.com/virginiasofiacampanella-beep/MAGICA-School-2026/blob/main/Project_SST.ipynb)

(could not fetch content)

---


## [majidniazkar/MAGICA-School-2026](https://github.com/majidniazkar/MAGICA-School-2026)

### `copernicus_salinity_anomaly_mediterranean.ipynb`

[View on GitHub](https://github.com/majidniazkar/MAGICA-School-2026/blob/main/copernicus_salinity_anomaly_mediterranean.ipynb)

# Mediterranean Sea Surface Salinity — Climatology, Anomalies & Extremes (1987–2025)

**Workflow**
1. Open the Mediterranean Sea Physics Reanalysis (`cmems_mod_med_phy-sal_my_4.2km_P1M-m`) — monthly means, lazy (dask).
2. Compute the **yearly mean salinity** at every pixel and explore it with a year slider.
3. Compute the **long-term climatological mean** at every pixel (mean over 1987–2025).
4. Compute the **monthly salinity anomaly** = monthly value − pixel climatological mean.
5. Visualise the monthly anomaly with an interactive time slider showing the current month.
6. At every pixel, find the **minimum** and **maximum** of the anomaly time-series.
7. Plot spatial maps of the min-anomaly and max-anomaly fields.

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import cmocean
import copernicusmarine
import hvplot.xarray
import numpy as np
import warnings

warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
# Mediterranean bounding box
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)

# The reanalysis starts January 1987; request through end of 2025
TIME_START = "1987-01-01"
TIME_END   = "2025-12-31"
```

## 1 — Open the salinity dataset (lazy)

```python
%%time
ds = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-sal_my_4.2km_P1M-m",
    start_datetime=TIME_START,
    end_datetime=TIME_END,
    **MED_BBOX,
)
ds
```

```python
# Surface salinity: select depth nearest to 0 m — result is (time, latitude, longitude), still lazy
sal = ds["so"].sel(depth=0, method="nearest")
print(f"Salinity shape: {sal.dims} = {dict(zip(sal.dims, sal.shape))}")
sal
```

## 2 — Yearly mean salinity with time slider

Resample monthly → annual, compute (≈ 39 frames × 380 × 1016 ≈ 60 MB), then display with a year scrubber.

```python
%%time
sal_yearly = sal.resample(time="YE").mean().compute()

# Replace the datetime coordinate with plain integer years for a cleaner slider
years = sal_yearly.time.dt.year.values
sal_yearly = (
    sal_yearly
    .assign_coords(year=("time", years))
    .swap_dims({"time": "year"})
    .drop_vars("time")
)
sal_yearly.attrs["long_name"] = "Annual Mean Salinity"
sal_yearly.attrs["units"] = "PSU"
print(f"Yearly salinity shape: {sal_yearly.dims} = {dict(zip(sal_yearly.dims, sal_yearly.shape))}")
sal_yearly
```

---

### `copernicus_salinity_anomaly_mediterranean2.ipynb`

[View on GitHub](https://github.com/majidniazkar/MAGICA-School-2026/blob/main/copernicus_salinity_anomaly_mediterranean2.ipynb)

# Mediterranean Sea Surface Salinity — Climatology, Anomalies & Extremes (1987–2025)

**Workflow**
1. Open the Mediterranean Sea Physics Reanalysis (`cmems_mod_med_phy-sal_my_4.2km_P1M-m`) — monthly means, lazy (dask).
2. Compute the **yearly mean salinity** at every pixel and explore it with a year slider.
3. Compute the **long-term climatological mean** at every pixel (mean over 1987–2025).
4. Compute the **monthly salinity anomaly** = monthly value − pixel climatological mean.
5. Visualise the monthly anomaly with an interactive time slider showing the current month.
6. At every pixel, find the **minimum** and **maximum** of the anomaly time-series.
7. Plot spatial maps of the min-anomaly and max-anomaly fields.

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import cmocean
import copernicusmarine
import hvplot.xarray
import numpy as np
import warnings

warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
# Mediterranean bounding box
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)

# The reanalysis starts January 1987; request through end of 2025
TIME_START = "1987-01-01"
TIME_END   = "2025-12-31"
```

## 1 — Open the salinity dataset (lazy)

```python
%%time
ds = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-sal_my_4.2km_P1M-m",
    start_datetime=TIME_START,
    end_datetime=TIME_END,
    **MED_BBOX,
)
ds
```

```python
# Surface salinity: select depth nearest to 0 m — result is (time, latitude, longitude), still lazy
sal = ds["so"].sel(depth=0, method="nearest")
print(f"Salinity shape: {sal.dims} = {dict(zip(sal.dims, sal.shape))}")
sal
```

## 2 — Yearly mean salinity with time slider

Resample monthly → annual, compute (≈ 39 frames × 380 × 1016 ≈ 60 MB), then display with a year scrubber.

```python
%%time
sal_yearly = sal.resample(time="YE").mean().compute()

# Replace the datetime coordinate with plain integer years for a cleaner slider
years = sal_yearly.time.dt.year.values
sal_yearly = (
    sal_yearly
    .assign_coords(year=("time", years))
    .swap_dims({"time": "year"})
    .drop_vars("time")
)
sal_yearly.attrs["long_name"] = "Annual Mean Salinity"
sal_yearly.attrs["units"] = "PSU"
print(f"Yearly salinity shape: {sal_yearly.dims} = {dict(zip(sal_yearly.dims, sal_yearly.shape))}")
sal_yearly
```

---

### `copernicus_sst_anomaly_mediterranean-cluster.ipynb`

[View on GitHub](https://github.com/majidniazkar/MAGICA-School-2026/blob/main/copernicus_sst_anomaly_mediterranean-cluster.ipynb)

# Mediterranean SST Climatology, Anomalies & Extremes (1987–2025)

**Workflow**
1. Open the Mediterranean Sea Physics Reanalysis (`cmems_mod_med_phy-temp_my_4.2km_P1M-m`) — monthly means, lazy (dask).
2. Compute the **yearly mean SST** at every pixel and explore it with a year slider.
3. Compute the **long-term climatological mean** at every pixel (mean over 1987–2025).
4. Compute the **monthly SST anomaly** = monthly value − pixel climatological mean.
5. At every pixel, find the **minimum** and **maximum** of the anomaly time-series.
6. Plot spatial maps of the min-anomaly and max-anomaly fields.

> **Note**: This notebook uses the *monthly* product (`P1M`). To work with daily data substitute `P1M-m` → `P1D-m` in the dataset ID; the rest of the code is identical, but dask chunking becomes important for memory management.

```python
import copernicusmarine
import hvplot.xarray
import numpy as np
import warnings

warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
# Mediterranean bounding box
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)

# The reanalysis starts January 1987; request through end of 2025
TIME_START = "1987-01-01"
TIME_END   = "2025-12-31"
```

## 1 — Open the SST dataset (lazy)

```python
%%time
ds = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-temp_my_4.2km_P1M-m",
    start_datetime=TIME_START,
    end_datetime=TIME_END,
    **MED_BBOX,
)
ds
```

```python
# Surface SST: select depth nearest to 0 m — result is (time, latitude, longitude), still lazy
sst = ds["thetao"].sel(depth=0, method="nearest")
print(f"SST shape: {sst.dims} = {dict(zip(sst.dims, sst.shape))}")
sst
```

## 2 — Yearly mean SST with time slider

Resample monthly → annual, compute (≈ 39 frames × 380 × 1016 ≈ 60 MB), then display with a year scrubber.

```python
%%time
sst_yearly = sst.resample(time="YE").mean().compute()

# Replace the datetime coordinate with plain integer years for a cleaner slider
years = sst_yearly.time.dt.year.values
sst_yearly = (
    sst_yearly
    .assign_coords(year=("time", years))
    .swap_dims({"time": "year"})
    .drop_vars("time")
)
sst_yearly.attrs["long_name"] = "Annual Mean SST"
sst_yearly.attrs["units"] = "°C"
print(f"Yearly SST shape: {sst_yearly.dims} = {dict(zip(sst_yearly.dims, sst_yearly.shape))}")
sst_yearly
```

---

### `copernicus_sst_anomaly_mediterranean.ipynb`

[View on GitHub](https://github.com/majidniazkar/MAGICA-School-2026/blob/main/copernicus_sst_anomaly_mediterranean.ipynb)

# Mediterranean SST Climatology, Anomalies & Extremes (1987–2025)

**Workflow**
1. Open the Mediterranean Sea Physics Reanalysis (`cmems_mod_med_phy-temp_my_4.2km_P1M-m`) — monthly means, lazy (dask).
2. Compute the **yearly mean SST** at every pixel and explore it with a year slider.
3. Compute the **long-term climatological mean** at every pixel (mean over 1987–2025).
4. Compute the **monthly SST anomaly** = monthly value − pixel climatological mean.
5. At every pixel, find the **minimum** and **maximum** of the anomaly time-series.
6. Plot spatial maps of the min-anomaly and max-anomaly fields.

> **Note**: This notebook uses the *monthly* product (`P1M`). To work with daily data substitute `P1M-m` → `P1D-m` in the dataset ID; the rest of the code is identical, but dask chunking becomes important for memory management.

```python
import copernicusmarine
import hvplot.xarray
import numpy as np
import warnings

warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
# Mediterranean bounding box
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)

# The reanalysis starts January 1987; request through end of 2025
TIME_START = "1987-01-01"
TIME_END   = "2025-12-31"
```

## 1 — Open the SST dataset (lazy)

```python
%%time
ds = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-temp_my_4.2km_P1M-m",
    start_datetime=TIME_START,
    end_datetime=TIME_END,
    **MED_BBOX,
)
ds
```

```python
# Surface SST: select depth nearest to 0 m — result is (time, latitude, longitude), still lazy
sst = ds["thetao"].sel(depth=0, method="nearest")
print(f"SST shape: {sst.dims} = {dict(zip(sst.dims, sst.shape))}")
sst
```

## 2 — Yearly mean SST with time slider

Resample monthly → annual, compute (≈ 39 frames × 380 × 1016 ≈ 60 MB), then display with a year scrubber.

```python
%%time
sst_yearly = sst.resample(time="YE").mean().compute()

# Replace the datetime coordinate with plain integer years for a cleaner slider
years = sst_yearly.time.dt.year.values
sst_yearly = (
    sst_yearly
    .assign_coords(year=("time", years))
    .swap_dims({"time": "year"})
    .drop_vars("time")
)
sst_yearly.attrs["long_name"] = "Annual Mean SST"
sst_yearly.attrs["units"] = "°C"
print(f"Yearly SST shape: {sst_yearly.dims} = {dict(zip(sst_yearly.dims, sst_yearly.shape))}")
sst_yearly
```

---

### `copernicus_sst_chlorophyll.ipynb`

[View on GitHub](https://github.com/majidniazkar/MAGICA-School-2026/blob/main/copernicus_sst_chlorophyll.ipynb)

# Copernicus Marine: Sea Surface Temperature & Chlorophyll — Mediterranean Sea
* Need to have an account on Copernicus Marine
* The ~/.netrc file contains the account credentials: your login and password for [Copernicus Marine](https://marine.copernicus.eu/)
```
$ more ~/.netrc
machine auth.marine.copernicus.eu
  login xxxxxxxxxx 
  password xxxxxxxxxxxxx
```
* Uses the **Mediterranean Sea Physics Reanalysis** (`MEDSEA_MULTIYEAR_PHY_006_004`) for SST
* Uses the **Mediterranean Sea Biogeochemistry Reanalysis** (`MEDSEA_MULTIYEAR_BGC_006_008`) for chlorophyll
* Monthly mean fields for August 2020

```python
import copernicusmarine
import hvplot.xarray
import warnings

warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)
```

```python
# Mediterranean bounding box
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)
YEAR_MONTH = "2020-08"
```

## Sea Surface Temperature
Dataset: `cmems_mod_med_phy-temp_my_4.2km_P1M-m` (monthly means, Mediterranean Physics Reanalysis)  
Variable: `thetao` — potential temperature (°C) at the surface (depth = 0)

```python
%%time
ds_sst = copernicusmarine.open_dataset(
    dataset_id="cmems_mod_med_phy-temp_my_4.2km_P1M-m",
    **MED_BBOX,
)
ds_sst
```

```python
%%time
# Select surface (nearest to 0 m) and the target month
sst = (
    ds_sst["thetao"]
    .sel(depth=0, method="nearest")
    .sel(time=YEAR_MONTH, method="nearest")
    .load()
)
print(f"SST range: {float(sst.min()):.1f} – {float(sst.max()):.1f} °C")
sst
```

```python
sst.hvplot(
    x="longitude",
    y="latitude",
    rasterize=True,
    geo=True,
    cmap="turbo",
    clabel="SST (°C)",
    tiles="OSM",
    frame_width=800,
    frame_height=450,
    title=f"Mediterranean Sea Surface Temperature — {YEAR_MONTH}",
    fontscale=1.2,
)
```

---

### `copernicus_sst_timeseries_pixels.ipynb`

[View on GitHub](https://github.com/majidniazkar/MAGICA-School-2026/blob/main/copernicus_sst_timeseries_pixels.ipynb)

# Mediterranean SST — Monthly Time Series at Two Pixels (1987–2025)

This notebook follows the same data loading and anomaly approach as
`copernicus_sst_anomaly_mediterranean.ipynb` and then extracts pixel-level
time series at two representative locations:

| Label | Location | Approx. coordinates |
|---|---|---|
| North Adriatic (Venice) | Northern Adriatic Sea, offshore Venice | 45.3 °N, 12.5 °E |
| Gulf of Lion (near Italy) | Eastern Gulf of Lion, near French-Italian Riviera | 43.3 °N, 7.5 °E |

**Plots produced**
1. Map showing the actual model grid-points selected.
2. Raw SST monthly time series (1987–2025) for both pixels.
3. SST anomaly monthly time series for both pixels (with ±1 σ envelope).
4. Monthly climatology (mean seasonal cycle) for both pixels.

> **Credentials**: requires a `~/.netrc` entry for `auth.marine.copernicus.eu`.

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)

import numpy as np
import pandas as pd
import xarray as xr
import copernicusmarine
import hvplot.xarray
import hvplot.pandas
import holoviews as hv
import panel as pn
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

pn.extension()
```

```python
# Mediterranean bounding box and time range — identical to the SST anomaly notebook
MED_BBOX = dict(
    minimum_longitude=-6,
    maximum_longitude=42,
    minimum_latitude=30,
    maximum_latitude=47,
)
TIME_START = "1987-01-01"
TIME_END   = "2025-12-31"

# Pixel definitions — (lat, lon) chosen to lie in open-water model cells
PIXELS = {
    "North Adriatic (Venice)": {"lat": 45.3, "lon": 12.5},
    "Gulf of Lion (near Italy)": {"lat": 43.3, "lon": 7.5},
}
COLORS = {
    "North Adriatic (Venice)": "#1f77b4",   # blue
    "Gulf of Lion (near Italy)": "#d62728",  # red
}
```

## 1 — Open the SST dataset (lazy)

Same dataset and subsetting as `copernicus_sst_anomaly_mediterranean.ipynb`.

---

### `era5_wind_anomaly_mediterranean.ipynb`

[View on GitHub](https://github.com/majidniazkar/MAGICA-School-2026/blob/main/era5_wind_anomaly_mediterranean.ipynb)

# Mediterranean 10-Metre Wind Speed — Climatology, Anomalies & Extremes (1979–2022)

**Workflow**
1. Open ERA5 10-metre U and V wind components from the `era5-pds` collection on Microsoft Planetary Computer — monthly means, lazy (dask).
2. Compute the **yearly mean wind speed** at every pixel and explore it with a year slider.
3. Compute the **long-term climatological mean** wind speed at every pixel (mean over all years).
4. Compute the **monthly wind speed anomaly** = monthly value − pixel climatological mean.
5. Visualise the monthly anomaly with an interactive time slider.
6. At every pixel, find the **minimum** and **maximum** of the anomaly time-series.
7. Plot spatial maps of the min-anomaly and max-anomaly fields.

> **Access**: ERA5-PDS on Planetary Computer is publicly accessible — no credentials needed.

```python
!pip install adlfs
```

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
warnings.filterwarnings("ignore", category=FutureWarning)

import numpy as np
import pandas as pd
import xarray as xr
import dask
import holoviews as hv
import hvplot.xarray
import panel as pn
import pystac_client
import planetary_computer

pn.extension()
hv.config.image_rtol = 0.01
```

```python
# Mediterranean bounding box (ERA5 uses 0–360 longitude)
LAT_MIN, LAT_MAX     = 30.0, 47.0
LON_MIN_W, LON_MAX_W = 354.0, 360.0   # -6° to 0°E
LON_MIN_E, LON_MAX_E =   0.0,  42.0   #  0° to 42°E

# ERA5-PDS on Planetary Computer spans 1979-01 through 2022-12
TIME_START = "1979-01"
TIME_END   = "2022-12"

catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)
```

## 1 — Open ERA5 wind datasets (lazy)

Each month of ERA5-PDS is a separate STAC item.  We open every month's Zarr store lazily and concatenate along time.

```python
def open_era5_asset(signed_item, asset_key):
    """Open one ERA5-PDS Zarr asset as an xarray Dataset."""
    asset = signed_item.assets[asset_key]
    open_kwargs = asset.extra_fields["xarray:open_kwargs"].copy()
    storage_options = open_kwargs.pop("storage_options")
    open_kwargs.pop("engine", None)
    return xr.open_zarr(asset.href, storage_options=storage_options, **open_kwargs)


def subset_med(da):
    """Subset to Mediterranean and convert lon to -180..180."""
    da_lat = da.sel(lat=slice(LAT_MAX, LAT_MIN))
    west   = da_lat.sel(lon=slice(LON_MIN_W, LON_MAX_W))
    east   = da_lat.sel(lon=slice(LON_MIN_E, LON_MAX_E))
    da_med = xr.concat([west, east], dim="lon").sortby("lon")
    new_lon = ((da_med.lon.values + 180) % 360 - 180)
    return da_med.assign_coords(lon=new_lon).sortby("lon")
```

---

### `projectmei.ipynb`

[View on GitHub](https://github.com/majidniazkar/MAGICA-School-2026/blob/main/projectmei.ipynb)

# ERA5 Sea Surface Temperature — Mediterranean Sea
* Use `pystac_client` to search the ERA5-PDS collection on Planetary Computer
* Open the Zarr store directly via `xarray`
* Subset to the Mediterranean domain and compute a monthly mean
* Visualise with `hvplot` (same approach as the Sentinel-2 notebook)

```python
import pystac_client
import planetary_computer
import xarray as xr
import hvplot.xarray
import numpy as np

# Connect to the Planetary Computer STAC API
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)
```

```python
# Search for the ERA5 analysis item for a specific month
# 'an' (analysis) items contain sea_surface_temperature; 'fc' (forecast) items do not
YEAR_MONTH = "2020-08"

search = catalog.search(collections=["era5-pds"], datetime=YEAR_MONTH)
items = search.item_collection()
print(f"Found {len(items)} items for {YEAR_MONTH}")
for item in items:
    print(f"  {item.id}  (kind={item.properties['era5:kind']})")
```

```python
# Select the analysis item and sign it to get a time-limited access credential
an_item = next(i for i in items if i.properties["era5:kind"] == "an")
signed_item = planetary_computer.sign(an_item)

# Grab the sea_surface_temperature asset and its recommended open kwargs
asset = signed_item.assets["sea_surface_temperature"]
open_kwargs = asset.extra_fields["xarray:open_kwargs"].copy()
storage_options = open_kwargs.pop("storage_options")
open_kwargs.pop("engine", None)  # engine kwarg is not accepted by open_zarr

print("Opening:", asset.href)
ds = xr.open_zarr(asset.href, storage_options=storage_options, **open_kwargs)
ds
```

```python
# Mediterranean bounding box
# Latitude:  30 N – 47 N
# Longitude: ERA5 uses 0–360, so the Med spans both -6°→42° E
#            which maps to 354–360 (western part) and 0–42 (eastern part)
LAT_MIN, LAT_MAX = 30.0, 47.0
LON_MIN_W, LON_MAX_W = 354.0, 360.0   # -6° to 0°
LON_MIN_E, LON_MAX_E = 0.0,   42.0    #  0° to 42°

sst = ds["sea_surface_temperature"]

# Subset latitude (lat is stored descending: 90 → -90)
sst_lat = sst.sel(lat=slice(LAT_MAX, LAT_MIN))

# Combine western and eastern longitude slices, then sort by lon
sst_west = sst_lat.sel(lon=slice(LON_MIN_W, LON_MAX_W))
sst_east = sst_lat.sel(lon=slice(LON_MIN_E, LON_MAX_E))
sst_med  = xr.concat([sst_west, sst_east], dim="lon").sortby("lon")

# Convert longitude from 0–360 to -180–180 for a nicer map
# Correct formula: (lon + 180) % 360 - 180  maps 354→-6, 360/0→0, 42→42
sst_med = sst_med.assign_coords(lon=((sst_med.lon.values + 180) % 360 - 180))
sst_med = sst_med.sortby("lon")

print("Mediterranean SST shape:", sst_med.dims)
sst_med
```

---

### `wind_components.ipynb`

[View on GitHub](https://github.com/majidniazkar/MAGICA-School-2026/blob/main/wind_components.ipynb)

# ERA5 10-Metre Wind Components — Mediterranean Sea
* Use `pystac_client` to search the ERA5-PDS collection on Planetary Computer
* Open the U and V wind Zarr stores directly via `xarray`
* Subset to the Mediterranean domain and compute a monthly mean wind speed
* Visualise with `hvplot` (same approach as the SST notebook)

```python
import pystac_client
import planetary_computer
import xarray as xr
import numpy as np
import hvplot.xarray

# Connect to the Planetary Computer STAC API
catalog = pystac_client.Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,
)
```

```python
# Search for the ERA5 item for a specific month
# U/V wind components are analysis ('an') variables
YEAR_MONTH = "2020-08"

search = catalog.search(collections=["era5-pds"], datetime=YEAR_MONTH)
items = search.item_collection()
print(f"Found {len(items)} items for {YEAR_MONTH}")
for item in items:
    print(f"  {item.id}  (kind={item.properties['era5:kind']})")
```

```python
# Select the analysis item — wind components live here
an_item = next(i for i in items if i.properties["era5:kind"] == "an")
signed_item = planetary_computer.sign(an_item)

# Inspect available wind-related assets
wind_assets = [k for k in sorted(signed_item.assets) if "wind" in k]
print("Wind assets:", wind_assets)
```

```python
def open_era5_asset(signed_item, asset_key):
    asset = signed_item.assets[asset_key]
    open_kwargs = asset.extra_fields["xarray:open_kwargs"].copy()
    storage_options = open_kwargs.pop("storage_options")
    open_kwargs.pop("engine", None)
    print(f"Opening: {asset.href}")
    return xr.open_zarr(asset.href, storage_options=storage_options, **open_kwargs)

# Actual asset keys in the 'an' item for 10-metre winds
ds_u = open_era5_asset(signed_item, "eastward_wind_at_10_metres")
ds_v = open_era5_asset(signed_item, "northward_wind_at_10_metres")

print(ds_u)
print(ds_v)
```

```python
# Mediterranean bounding box
# Latitude:  30 N – 47 N
# Longitude: ERA5 uses 0–360; Med spans -6°→42° E → 354–360 + 0–42
LAT_MIN, LAT_MAX     = 30.0, 47.0
LON_MIN_W, LON_MAX_W = 354.0, 360.0   # -6° to 0°
LON_MIN_E, LON_MAX_E =   0.0,  42.0   #  0° to 42°

def subset_med(da):
    da_lat = da.sel(lat=slice(LAT_MAX, LAT_MIN))
    west   = da_lat.sel(lon=slice(LON_MIN_W, LON_MAX_W))
    east   = da_lat.sel(lon=slice(LON_MIN_E, LON_MAX_E))
    da_med = xr.concat([west, east], dim="lon").sortby("lon")
    # Convert longitude from 0-360 to -180-180
    da_med = da_med.assign_coords(lon=((da_med.lon.values + 180) % 360 - 180))
    return da_med.sortby("lon")

u10_med = subset_med(ds_u["eastward_wind_at_10_metres"])
v10_med = subset_med(ds_v["northward_wind_at_10_metres"])

print("Mediterranean U10 shape:", dict(u10_med.sizes))
print("Mediterranean V10 shape:", dict(v10_med.sizes))
```

---


## [valsazonova/MAGICA-School-2026](https://github.com/valsazonova/MAGICA-School-2026)

### `main.ipynb`

[View on GitHub](https://github.com/valsazonova/MAGICA-School-2026/blob/main/main.ipynb)

(could not fetch content)

---


## [capellaraquelgn-hue/MAGICA-School-2026](https://github.com/capellaraquelgn-hue/MAGICA-School-2026)

### `myanmar_ocean_response.ipynb`

[View on GitHub](https://github.com/capellaraquelgn-hue/MAGICA-School-2026/blob/main/myanmar_ocean_response.ipynb)

# Ocean Response to Cyclone Mocha — Bay of Bengal, May 2023

Tropical Cyclone Mocha made landfall near **Sittwe, Myanmar** on **14 May 2023** as one of
the strongest storms ever recorded in the Bay of Bengal (Category 5 equivalent).

This notebook analyses how key physical and biogeochemical ocean variables changed across
the **Bay of Bengal and Andaman Sea** during May 2023 by following the MAGICA course workflow:

1. **Lazy metadata-first access** — open Copernicus Marine datasets without downloading data
2. **Inspect metadata** — confirm shape, coordinates and chunk layout before any compute
3. **Subset** spatially (AOI) and temporally (May 2023)
4. **Write to Icechunk** — versioned zarr store in personal S3 bucket
5. **Read back lazily** — re-open from Icechunk without loading data
6. **Analyse** — load area-averaged timeseries into memory for plotting
7. **Visualise** — spatial maps before, during, and after landfall

## Variables monitored

| Variable | CMEMS product |
|---|---|
| Sea surface temperature (`thetao`) | `cmems_mod_glo_phy_my_0.083deg_P1D-m` |
| Salinity (`so`) | `cmems_mod_glo_phy_my_0.083deg_P1D-m` |
| Significant wave height (`VHM0`) | `cmems_mod_glo_wav_my_0.2deg_PT3H-i` |
| Chlorophyll (`chl`) | `cmems_mod_glo_bgc_my_0.25deg_P1D-m` |
| Net primary production (`nppv`) | `cmems_mod_glo_bgc_my_0.25deg_P1D-m` |
| Nitrate (`no3`) | `cmems_mod_glo_bgc_my_0.25deg_P1D-m` |
| Phosphate (`po4`) | `cmems_mod_glo_bgc_my_0.25deg_P1D-m` |

All products are **reanalysis** (measurements-based), covering 1993–present.

## 0. Setup

```python
import warnings
warnings.filterwarnings("ignore")

import os
import xarray as xr
import icechunk
import copernicusmarine
import hvplot.xarray
import panel as pn
from dotenv import load_dotenv

pn.extension()
print("All imports OK")
```

```python
# Credentials
_ = load_dotenv(f'{os.environ["HOME"]}/dotenv/protocoast.env', override=True)
storage_endpoint = os.environ['ENDPOINT_URL']
BUCKET = 'raquelcapella'

# Area of Interest: Bay of Bengal + Andaman Sea (Cyclone Mocha track)
#bbox = [85.0, 10.0, 100.0, 25.0]  # [lon_min, lat_min, lon_max, lat_max]
bbox= [90.9, 17.5, 94.6, 21.3]

# Full month of May 2023 — captures pre-storm baseline, cyclone peak, and recovery
TIME_START = "2023-05-01"
TIME_END   = "2023-05-31"
LANDFALL   = "2023-05-14"

print(f"AOI: lon [{bbox[0]}, {bbox[2]}]  lat [{bbox[1]}, {bbox[3]}]")
print(f"Period: {TIME_START} → {TIME_END}")
print(f"Cyclone Mocha landfall: {LANDFALL}")
```

---


## [TheJeran/MAGICA-School-2026](https://github.com/TheJeran/MAGICA-School-2026)

### `Arctic /placeholder.md`

[View on GitHub](https://github.com/TheJeran/MAGICA-School-2026/blob/main/Arctic /placeholder.md)



---

### `Arctic/Processing.ipynb`

[View on GitHub](https://github.com/TheJeran/MAGICA-School-2026/blob/main/Arctic/Processing.ipynb)

(could not fetch content)

---

### `Arctic/ToFloat16.ipynb`

[View on GitHub](https://github.com/TheJeran/MAGICA-School-2026/blob/main/Arctic/ToFloat16.ipynb)

```python
from openexr_numpy import imread, imwrite
from glob import glob
import numpy as np
import matplotlib.pyplot as plt
files = glob('2023_float16/*.exr')
```

```python
global_arr = np.full((6000,12000), np.nan)
for file in files:
    filename = file.split('/')[-1]
    array = imread(file, channel_names="R")
    global_arr[-array.shape[0]:] = array
    imwrite(f'global_2023/{filename}', global_arr.astype('float16'), channel_names="R")
```

---

### `change.ipynb`

[View on GitHub](https://github.com/TheJeran/MAGICA-School-2026/blob/main/change.ipynb)

(could not parse notebook JSON)

---
