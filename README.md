# georeader

Read data from rasters: very few dependencies, compatible with cloud platforms and lazy loading.

## Install

```bash
# Install with minimal requirements (only rasterio, numpy as shapely)
pip install git+https://github.com/spaceml-org/georeader#egg=georeader

# Install with Google dependencies (to read objects from Google Cloud Storage)
pip install git+https://github.com/spaceml-org/georeader#egg=georeader[google]

# Install with Planetary Computer requirements
pip install git+https://github.com/spaceml-org/georeader#egg=georeader[microsoftplanetary]
```

This package is work in progress. The API might change without notice. Use it with caution.

## Getting started

```python
# This snipper requires Google requirements
# pip install git+https://github.com/spaceml-org/georeader#egg=georeader[google]
import os

os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "/path/to/requester/pays/credentials.json"
os.environ["GS_USER_PROJECT"] = "project-name-to-bill"

from georeader.readers import S2_SAFE_reader
from georeader import read

bounds_read = (113.86632280340005, 35.56034107313706, 114.31452615868524, 35.751739315423876)
crs_bounds = "EPSG:4326"
s2obj = S2_SAFE_reader.s2loader("gs://gcp-public-data-sentinel-2/tiles/13/S/ER/S2B_MSIL1C_20191008T173219_N0208_R055_T13SER_20191008T204555.SAFE", 
                                out_res=10, bands=["B04","B03","B02"])

data = read.read_from_bounds(s2obj, bounds_read, crs_bounds=crs_bounds, 
                             pad_add=(20,20))

data_memory = data.load() # this triggers downloading the data

data_memory # GeoTensor object
```
```
>>  Transform: | 10.00, 0.00, 759560.00|
               | 0.00,-10.00, 3960420.00|
               | 0.00, 0.00, 1.00|
    Shape: (3, 2040, 4040)
    Resolution: (10.0, 10.0)
    Bounds: (759560.0, 3940020.0, 799960.0, 3960420.0)
    CRS: EPSG:32649
    fill_value_default: 0
```

In the `.values` attribute we have the plain numpy array that we can plot with `show`:

```python
from rasterio.plot import  show
show(data_memory.values/3500, transform=data_memory.transform)

```
![S2image](notebooks/images/sample_read.jpeg)

Saving the `GeoTensor` as a COG GeoTIFF: 

```python
from georeader.save_cog import save_cog

# Supports writing in bucket location (e.g. gs://bucket-name/s2_crop.tif)
save_cog(data_memory, "s2_crop.tif", descriptions=s2obj.bands)
```

## Tutorials

* [Tutorial to read overlapping tiles from a GeoTIFF and a Sentinel-2 image](https://github.com/spaceml-org/georeader/blob/main/notebooks/reading_overlapping_sentinel2_aviris.ipynb)
* [Example of reading a Proba-V image overlapping with Sentinel-2 forcing same resolution](https://github.com/spaceml-org/georeader/blob/main/notebooks/read_overlapping_probav_and_sentinel2.ipynb)
* [Reading Sentinel-2 images from the public Google bucket](https://github.com/spaceml-org/georeader/blob/main/notebooks/read_S2_SAFE_from_bucket.ipynb)
* [Query Sentinel-2 images over a location and time span, mosaic and plot them](https://github.com/spaceml-org/georeader/blob/main/notebooks/Sentinel-2/query_mosaic_s2_images.ipynb)

