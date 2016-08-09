# GIS SRTM Heightmap Tiles Pt


## Motivation

There's great open data for GIS stuff, but I find it hard to master and relate to.
I wanted simple, developer-friendly resources to explore de altitude data I've known
I had available via the SRTM initiative. These tiles are my contribution for that cause.


## What is this about?

This is a set of 256x256 RGBA slippy tiles encoding the altitude of Portugal's mainland, Azores and Madeira islands.

You can get a glimpse of the data by visiting
[this leaflet map](https://josepedrodias.github.io/gis-srtm-heightmap-tiles-pt/demos/explore_map.html)
with the tile in use.

You can
[pick pixels in an example tile](https://josepedrodias.github.io/gis-srtm-heightmap-tiles-pt/demos/read_altitude.html)
to get how the data is encoded.


## Where does the information come from?

I started up with [SRTM](https://lta.cr.usgs.gov/SRTM) information.
For convenience I grabbed 5 tiles via [SRTM Tile Grabber](http://dwtkns.com/srtm/).
These are [GeoTIFF files](https://en.wikipedia.org/wiki/GeoTIFF) and I grabbed:

* srtm_30_05.tif
* srtm_31_05.tif
* srtm_33_06.tif
* srtm_35_04.tif
* srtm_35_05.tif


## How did I generate it?

Disclaimer: I'm by no means a GIS expert. I have some experience in manipulating and exploring
OSM and GMaps information and I dig 2D raster formats so bear with me if my methodology
wasn't top notch.

I ended up using mostly [GDAL](http://www.gdal.org/), installed via homebrew and did most scripting
tasks in JS.

Most scripts can be found in [GIS REPOS TODO](#). Removed empty PNGs by hand and merged the seams
between 35_04 and 35_05 by using the highest pixels value in the transition.

The executive summary for what I did was:
* found a reasonable zoom factor to encode the data (zoom 10)
* identified the tiles relevant for my quest (PT, Azores and Madeira)
* computed the lat,lon bounds for each tile
* used GDAL tools to get the altitude info, using the following workflow:
  * gdalwarp using the relevant src geoTiff, passing it lat,lon bounds
  * gdal_translate to resize the data to 256x256
  * gdal_translate to get back the XYZ data format, which is very verbose but easy to parse (for every pixel, a line with lat, lon, altitude)
  * process the XYZ file into altitude readings and stored them as a PNG, storing the integer altitude encded in RG RGBA channels.
* automated the generation of all elected tiles and did a bit of manual tweaking (namely in the transition between 35_04 - 35_05)


## How is the information encoded?

I'm not exposing the lack of data in any way (original data had it marked).

All readings below 0 meters are being capped to 0. Data is stored in integer meters using the first
two bytes (red and green channels) and stored in lossless PNGs. The results aren't great for humans
to interpret (since it looks like green gradients all the way, for every 256 meters)
but isn't terrible either and is simple for computers to get back to altitude data.

The following formulas map altitude from RGBA and back again:

    h = R + G * 256
    <=>
    R = floor(h / 256),
    G = h % 256


## Community-related

Feel free to report back any errors and/or borrow/improve the process for other parts of the globe.
Just keep in mind I'm not maintaining this set professionally.


## Further work

There's some more manipulations I would like to have but have no urgency or simple way to make available just yet.
These are:
* compute the image pyramids for lower zoom factors (not too hard to do, resizes and merges);
* generate hillshade tiles from the gathered data (that's mostly running [GDAL on those](http://blog.thematicmapping.org/2012/06/creating-color-relief-and-slope-shading.html), computing hill shade and color relief and multiply both)
