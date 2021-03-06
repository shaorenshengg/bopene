# xarray-sentinel

Xarray backend to explore and read Copernicus Sentinel-1 SAR data products.

This Open Source project is sponsored by B-Open - https://www.bopen.eu

## Features

- read SAR imagery data: full image, individual swaths, individual SLC bursts - **Alpha**
- read metadata: orbit, attitude, GCPs, calibration, dc_estimate, azimuth_fm_rate - **Alpha**
- supported products:
  - Sentinel-1 SLC IW (Interferometric Wide Swath): **Alpha**
  - Sentinel-1 SLC EW (Extended Wide Swath): **Alpha**
  - Sentinel-1 SLC SM (Stripmap): **Alpha**
  - Sentinel-1 GRD SM/IW/EW: **technology preview**
- read uncompressed and compressed SAFE products locally or on a network via *fsspec*

## Install

The easiest way to install *xarray-sentinel* is via *conda*.
You may create a new environment, activate it, install the package and its dependencies
with the following commands:

```shell
    conda create -n XARRAY-SENTINEL
    conda activate XARRAY-SENTINEL
    conda install -c conda-forge rioxarray xarray xmlschema
    pip install xarray-sentinel
```

## Sentinel-1 SLC IW

### Data

Currently, xarray-sentinel provides access as Xarray datasets to the following data:

- full image
- individual swaths
- individual SLC bursts
- gcp
- orbit
- attitude
- calibration
- dc_estimate
- azimuth_fm_rate

using `azimuth_time` and `slant_range_time` dimensions when it make sense.

## Examples:

### Open the root dataset

```python-repl
>>> from xarray_sentinel import sentinel1
>>> product_path = "tests/data/S1B_IW_SLC__1SDV_20210401T052622_20210401T052650_026269_032297_EFA4.SAFE"
>>> sentinel1.open_dataset(product_path)
<xarray.Dataset>
Dimensions:  ()
Data variables:
    *empty*
Attributes: ...
    constellation:              sentinel-1
    platform:                   sentinel-1b
    instrument:                 ['c-sar']
    sat:orbit_state:            descending
    sat:absolute_orbit:         26269
    sat:relative_orbit:         168
    ...                         ...
    sar:product_type:           SLC
    xs:instrument_mode_swaths:  ['IW1', 'IW2', 'IW3']
    group:                      /
    subgroups:                  ['IW1', 'IW1/VH', 'IW1/VH/gcp', 'IW1/VH/orbit...
    Conventions:                CF-1.8
    history:                    created by xarray_sentinel-...

```

The attribute `subgroups` shows the available groups to be loaded. The keyword `group`
shall be used to select the dataset to be loaded.

The groups present in a typical SLC SAFE package are:

```
/
?????? IW1
???  ?????? VH
???  ???  ?????? gcp
???  ???  ?????? orbit
???  ???  ?????? attitude
???  ???  ?????? dc_estimate
???  ???  ?????? azimuth_fm_rate
???  ???  ?????? calibration
???  ?????? VV
???     ?????? gcp
???     ?????? orbit
???     ?????? attitude
???     ?????? dc_estimate
???     ?????? azimuth_fm_rate
???     ?????? calibration
?????? IW2
???  ?????? VH
???  ???  ?????? gcp
...
???  ?????? VV
???     ?????? gcp
...

```

### Open the gcp dataset

To load the gcp relative to the VV polarisation of first swath use the key `group="IW1/VV/gcp"`:

```python-repl
>>> sentinel1.open_dataset(product_path, group="IW1/VV/gcp")
<xarray.Dataset>
Dimensions:           (azimuth_time: 10, slant_range_time: 21)
Coordinates:
  * azimuth_time      (azimuth_time) datetime64[ns] 2021-04-01T05:26:24.20973...
  * slant_range_time  (slant_range_time) float64 0.005343 0.00536 ... 0.005679
    line              (azimuth_time) int64 0 1501 3002 ... 10507 12008 13508
    pixel             (slant_range_time) int64 0 1082 2164 ... 19476 20558 21631
Data variables:
    latitude          (azimuth_time, slant_range_time) float64 ...
    longitude         (azimuth_time, slant_range_time) float64 ...
    height            (azimuth_time, slant_range_time) float64 ...
    incidenceAngle    (azimuth_time, slant_range_time) float64 ...
    elevationAngle    (azimuth_time, slant_range_time) float64 ...
Attributes: ...
    constellation:              sentinel-1
    platform:                   sentinel-1b
    instrument:                 ['c-sar']
    sat:orbit_state:            descending
    sat:absolute_orbit:         26269
    sat:relative_orbit:         168
    ...                         ...
    xs:instrument_mode_swaths:  ['IW1', 'IW2', 'IW3']
    group:                      /IW1/VV/gcp
    Conventions:                CF-1.8
    title:                      Geolocation grid
    comment:                    The dataset contains geolocation grid point e...
    history:                    created by xarray_sentinel-...

```

### Open the orbit dataset

Similarly for orbit data use `group="IW1/VV/attitude"`:

```python-repl
>>> sentinel1.open_dataset(product_path, group="IW1/VV/orbit")
<xarray.Dataset>
Dimensions:       (axis: 3, azimuth_time: 17)
Coordinates:
  * azimuth_time  (azimuth_time) datetime64[ns] 2021-04-01T05:25:19 ... 2021-...
  * axis          (axis) int64 0 1 2
Data variables:
    position      (axis, azimuth_time) ...
    velocity      (axis, azimuth_time) ...
Attributes: ...
    reference_system:           Earth Fixed
    constellation:              sentinel-1
    platform:                   sentinel-1b
    instrument:                 ['c-sar']
    sat:orbit_state:            descending
    sat:absolute_orbit:         26269
    ...                         ...
    xs:instrument_mode_swaths:  ['IW1', 'IW2', 'IW3']
    group:                      /IW1/VV/orbit
    Conventions:                CF-1.8
    title:                      Orbit information used by the IPF during proc...
    comment:                    The dataset contains a sets of orbit state ve...
    history:                    created by xarray_sentinel-...

```

### Attitude and calibration datasets

For attitude data use `group="IW1/VV/attitude"`:

```python-repl
>>> sentinel1.open_dataset(product_path, group="IW1/VV/attitude")
<xarray.Dataset>
Dimensions:       (azimuth_time: 25)
Coordinates:
  * azimuth_time  (azimuth_time) datetime64[ns] 2021-04-01T05:26:24.750001 .....
Data variables:
    q0            (azimuth_time) float64 ...
    q1            (azimuth_time) float64 ...
    q2            (azimuth_time) float64 ...
    q3            (azimuth_time) float64 ...
    wx            (azimuth_time) float64 ...
    wy            (azimuth_time) float64 ...
    wz            (azimuth_time) float64 ...
    pitch         (azimuth_time) float64 ...
    roll          (azimuth_time) float64 ...
    yaw           (azimuth_time) float64 ...
Attributes: ...
    constellation:              sentinel-1
    platform:                   sentinel-1b
    instrument:                 ['c-sar']
    sat:orbit_state:            descending
    sat:absolute_orbit:         26269
    sat:relative_orbit:         168
    ...                         ...
    xs:instrument_mode_swaths:  ['IW1', 'IW2', 'IW3']
    group:                      /IW1/VV/attitude
    Conventions:                CF-1.8
    title:                      Attitude information used by the IPF during p...
    comment:                    The dataset contains a sets of attitude data ...
    history:                    created by xarray_sentinel-...

```

and for calibration data use `group="IW1/VV/calibration"`:

```python-repl
>>> sentinel1.open_dataset(product_path, group="IW1/VV/calibration")
<xarray.Dataset>
Dimensions:       (line: 30, pixel: 542)
Coordinates:
  * line          (line) int64 -1042 -556 91 577 ... 13042 13688 14175 14661
  * pixel         (pixel) int64 0 40 80 120 160 ... 21520 21560 21600 21631
Data variables:
    azimuth_time  (line) datetime64[ns] ...
    sigmaNought   (line, pixel) float64 ...
    betaNought    (line, pixel) float64 ...
    gamma         (line, pixel) float64 ...
    dn            (line, pixel) float64 ...
Attributes: ...
    constellation:              sentinel-1
    platform:                   sentinel-1b
    instrument:                 ['c-sar']
    sat:orbit_state:            descending
    sat:absolute_orbit:         26269
    sat:relative_orbit:         168
    ...                         ...
    xs:instrument_mode_swaths:  ['IW1', 'IW2', 'IW3']
    group:                      /IW1/VV/calibration
    Conventions:                CF-1.8
    title:                      Calibration coefficients
    comment:                    The dataset contains calibration information ...
    history:                    created by xarray_sentinel-...

```

### Open a single swath / polarisation dataset

Finally the measurement data is found in a swath / polarisation groups accessed for
example as `group="IW1/VV"` for the VV polarisation of the first IW swath:

```python-repl
>>> swath_polarisation_ds = sentinel1.open_dataset(product_path, group="IW1/VV")
>>> swath_polarisation_ds
<xarray.Dataset>
Dimensions:           (pixel: 21632, line: 13509)
Coordinates:
  * pixel             (pixel) int64 0 1 2 ... 21629 21630 21631
  * line              (line) int64 0 1 2 ... 13506 13507 13508
    slant_range_time  (pixel) float64 0.005343 0.005343 ... 0.005679 0.005679
    azimuth_time      (line) datetime64[ns] 2021-04-01T05:26:24.209990 ...
Data variables:
    measurement       (line, pixel) complex64 ...
Attributes: ...
    azimuth_steering_rate:      1.590368784
    sar:center_frequency:       5.40500045433435
    number_of_bursts:           9
    lines_per_burst:            1501
    constellation:              sentinel-1
    platform:                   sentinel-1b
    ...                         ...
    sar:product_type:           SLC
    xs:instrument_mode_swaths:  ['IW1', 'IW2', 'IW3']
    group:                      /IW1/VV
    subgroups:                  ['gcp', 'orbit', 'attitude', 'dc_estimate', '...
    Conventions:                CF-1.8
    history:                    created by xarray_sentinel-...

```

### Crop single SLC burst dataset

A single burst can be cropped out of the swath data. *xarray_sentinel* offer an helper function
that also performs additional changes like swapping the dimenstions:

```python-repl
>>> sentinel1.crop_burst_dataset(swath_polarisation_ds, burst_index=8)
<xarray.Dataset>
Dimensions:           (slant_range_time: 21632, azimuth_time: 1501)
Coordinates:
    pixel             (slant_range_time) int64 0 1 2 3 ... 21629 21630 21631
    line              (azimuth_time) int64 12008 12009 12010 ... 13507 13508
  * slant_range_time  (slant_range_time) float64 0.005343 0.005343 ... 0.005679
  * azimuth_time      (azimuth_time) datetime64[ns] 2021-04-01T05:26:46.27227...
Data variables:
    measurement       (azimuth_time, slant_range_time) complex64 ...
Attributes: ...
    azimuth_steering_rate:      1.590368784
    sar:center_frequency:       5.40500045433435
    number_of_bursts:           9
    lines_per_burst:            1501
    constellation:              sentinel-1
    platform:                   sentinel-1b
    ...                         ...
    xs:instrument_mode_swaths:  ['IW1', 'IW2', 'IW3']
    group:                      /IW1/VV
    subgroups:                  ['gcp', 'orbit', 'attitude', 'dc_estimate', '...
    Conventions:                CF-1.8
    history:                    created by xarray_sentinel-...
    burst_index:                8

```

## Xarray integration

Starting with Xarray v0.18.0, xarray-sentinel will be automatically available as
an Xarray backend:

```python-repl
>>> import xarray as xr
>>> ds = xr.open_dataset(product_path, engine="sentinel-1")

```

## Project badges

[![on-push](https://github.com/bopen/xarray-sentinel/actions/workflows/on-push.yml/badge.svg)](https://github.com/bopen/xarray-sentinel/actions/workflows/on-push.yml)
[![codecov](https://codecov.io/gh/bopen/xarray-sentinel/branch/main/graph/badge.svg?token=OLw9it0i18)](https://codecov.io/gh/bopen/xarray-sentinel)

## Contributing

The main repository is hosted on GitHub,
testing, bug reports and contributions are highly welcomed and appreciated:

https://github.com/bopen/xarray-sentinel

Lead developer:

- [Aureliana Barghini](https://github.com/aurghs) - [B-Open](https://www.bopen.eu/)

Main contributors:

- [Alessandro Amici](https://github.com/alexamici) - [B-Open](https://www.bopen.eu/)
- [Corrado Avolio](https://github.com/corrado9999) - [e-GEOS](https://www.e-geos.it/)

See also the list of [contributors](https://github.com/bopen/xarray-sentinel/contributors) who participated in this project.

## License

```
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
```
