# Changes from upstream

Upstream: https://github.com/ASFHyP3/hyp3-OPERA-RTC, tag `v0.1.9` at commit a6608d44b651420241abf9d42e33d0f927562b22 (2025-09-04), by the Alaska Satellite Facility, BSD 3-Clause License.

This copy starts from upstream's file tree at a6608d4 without its git history. It is unchanged except for the items below. Left out: `.github/` (upstream's CI workflows).

## 2026-10-24: DEM read from OPERA DEM v1.2

`src/hyp3_opera_rtc/dem.py` read the OPERA DEM mosaic from `https://nisar.asf.earthdatacloud.nasa.gov/STATIC/DEM/v1.1/EPSG4326/EPSG4326.vrt`. That path now returns HTTP 404, so every job stopped while fetching the DEM. Upstream's `main` branch still has the same path. The DEM is now read from `.../STATIC/DEM/v1.2/EPSG4326/EPSG4326.vrt`, the current release of the same Copernicus-based OPERA DEM on the same host.

Results: the RTC output still matches the official OPERA product (see Check below). Small differences against products made with DEM v1.1 are possible where the two DEM releases differ.

## 2026-10-24: DEM version in the PGE run config

`src/hyp3_opera_rtc/templates/pge.yml.j2` described the DEM as "version 1.1 (v1.1)". It now says "version 1.2 (v1.2)", so the product metadata names the DEM that was actually used. This changes only that text.

## Check

On 2026-10-24, upstream v0.1.9 with the DEM path change above was run on the README example burst `S1_146160_IW1_20241029T095958_VV_592B-BURST` (the full 2.45 GB SLC and 1.9 GB of DEM are downloaded by the job). It ran in 8 min 18 s with 2.7 GB peak memory. The output matches the official OPERA RTC-S1 product `T069-146160-IW1` for 2024-10-29 (made with the same RTC 1.0.2) on the identical grid: mean absolute difference 0.0045 dB, 99.997% of 1.94 million pixels within 0.1 dB, r = 0.999996, and the mask agrees on 99.995% of pixels. Plugin tests: 15 passed, 1 skipped; PGE tests: 13 passed.

One practical note from that run: the DEM download only succeeded with `VRT_NUM_THREADS=1 GDAL_NUM_THREADS=1` set in the environment, because parallel tile reads were refused by Earthdata Login. The cause (server throttling or the local network setup) is not known, so the code was left as it is.
