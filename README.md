# xarray-sentinel

**WARNING: this is v0.1 "technology preview", needs xarray >= 0.18.0**

Xarray backend to explore and load Copernicus Sentinel-1 satellite data products.

## Install

```
    git clone https://github.com/bopen/xarray-sentinel.git
    cd xarray-sentinel
    conda install -c conda-forge rioxarray xarray xmlschema
    pip install --no-deps https://github.com/pydata/xarray/archive/master.zip  # for xarray version>= 0.17.0
    pip install -e .
```

## Product support status:

- Sentinel-1 SLC IW (Interferometric Wide Swath): **work-in-progress**
- Sentinel-1 SLC EW (Extended Wide Swath): in roadmap
- Sentinel-1 SLC SM (Stripmap): in roadmap
- Sentinel-1 GRD SM/IW/EW: in roadmap
- Sentinel-2 L1C/L2A: in roadmap


## Sentinel-1 SLC IW

### Data

Currently, xarray-sentinel provides access as Xarray datsets to the following data:

- burst data
- gcp
- orbit
- attitude

using `azimuth_time` and `slant_range_time` dimensions.


## Examples:

### Open root dataset

```python
>>> import xarray as xr
>>> product_path = "tests/data/S1B_IW_SLC__1SDV_20210401T052622_20210401T052650_026269_032297_EFA4.SAFE"
>>> xr.open_dataset(product_path, engine="sentinel-1")
<xarray.Dataset>
Dimensions:  ()
Data variables:
    *empty*
Attributes: (12/15)
    constellation:              sentinel-1
    platform:                   sentinel-1b
    instrument:                 ['c-sar']
    sat:orbit_state:            descending
    sat:absolute_orbit:         26269
    sat:relative_orbit:         168
    ...                         ...
    sar:polarizations:          ['VV', 'VH']
    sar:product_type:           SLC
    xs:instrument_mode_swaths:  ['IW1', 'IW2', 'IW3']
    groups:                     ['IW1', 'IW1/gcp', 'IW1/attitude', 'IW1/orbit...
    Conventions:                CF-1.7
    history:                    created by xarray_sentinel-...

```

the attributes `groups` shows the avaible groups to be loaded. The key `group`
shall be used to select the dataset to be loaded.

### Open gcp dataset
To load the gcp relative to the first swath use the key `group="IW1/gcp"`:
```python
>>> xr.open_dataset(product_path, engine="sentinel-1", group="IW1/gcp")
<xarray.Dataset>
Dimensions:           (azimuth_time: 10, slant_range_time: 21)
Coordinates:
  * azimuth_time      (azimuth_time) datetime64[ns] 2021-04-01T05:26:24.20973...
  * slant_range_time  (slant_range_time) float64 0.005343 0.00536 ... 0.005679
Data variables:
    latitude          (azimuth_time, slant_range_time) float64 ...
    longitude         (azimuth_time, slant_range_time) float64 ...
    height            (azimuth_time, slant_range_time) float64 ...
    incidenceAngle    (azimuth_time, slant_range_time) float64 ...
    elevationAngle    (azimuth_time, slant_range_time) float64 ...
Attributes:
    Conventions:  CF-1.7
    title:        Geolocation grid
    comment:      The dataset contains geolocation grid point entries for eac...
    history:      created by xarray_sentinel-...

```

### Open attitude dataset

```python
>>> xr.open_dataset(product_path, engine="sentinel-1", group="IW1/attitude")
<xarray.Dataset>
Dimensions:  (time: 25)
Coordinates:
  * time     (time) datetime64[ns] 2021-04-01T05:26:24.750001 ... 2021-04-01T...
Data variables:
    q0       (time) float64 ...
    q1       (time) float64 ...
    q2       (time) float64 ...
    wx       (time) float64 ...
    wy       (time) float64 ...
    wz       (time) float64 ...
    pitch    (time) float64 ...
    roll     (time) float64 ...
    yaw      (time) float64 ...
Attributes:
    Conventions:  CF-1.7
    title:        Attitude information used by the IPF during processing
    comment:      The dataset contains a sets of attitude data records that a...
    history:      created by xarray_sentinel-...

```

### Open orbit dataset

```python
>>> xr.open_dataset(product_path, engine="sentinel-1", group="IW1/orbit")
<xarray.Dataset>
Dimensions:  (time: 17)
Coordinates:
  * time     (time) datetime64[ns] 2021-04-01T05:25:19 ... 2021-04-01T05:27:59
Data variables:
    x        (time) float64 ...
    y        (time) float64 ...
    z        (time) float64 ...
    vx       (time) float64 ...
    vy       (time) float64 ...
    vz       (time) float64 ...
Attributes:
    reference_system:  Earth Fixed
    Conventions:       CF-1.7
    title:             Orbit information used by the IPF during processing
    comment:           The dataset contains a sets of orbit state vectors tha...
    history:           created by xarray_sentinel-...

```

### Open a single burst

```python
>>> xr.open_dataset(product_path, engine="sentinel-1", group="IW1/R168-N459-E0115")
<xarray.Dataset>
Dimensions:           (azimuth_time: 1501, slant_range_time: 21632)
Coordinates:
  * azimuth_time      (azimuth_time) datetime64[ns] 2021-04-01T05:26:43.51577...
  * slant_range_time  (slant_range_time) float64 0.005343 0.005343 ... 0.005679
Data variables:
    VH                (azimuth_time, slant_range_time) complex128 ...
    VV                (azimuth_time, slant_range_time) complex128 ...
Attributes: (12/14)
    constellation:              sentinel-1
    platform:                   sentinel-1b
    instrument:                 ['c-sar']
    sat:orbit_state:            descending
    sat:absolute_orbit:         26269
    sat:relative_orbit:         168
    ...                         ...
    sar:instrument_mode:        IW
    sar:polarizations:          ['VV', 'VH']
    sar:product_type:           SLC
    xs:instrument_mode_swaths:  ['IW1', 'IW2', 'IW3']
    Conventions:                CF-1.7
    history:                    created by xarray_sentinel-...

```

# License

    Copyright 2021, B-Open Solutions srl and the xarray-sentinel authors.
    
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
    
        http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
