
.. _prep:

==============
Preparing data
==============

---------------------
Choosing your dataset
---------------------

Free geospatial data is widely available, so today’s challenges is not
finding it but deciding which dataset is best for the project at hand.
There are a number of different topographic datasets freely available
online, you will need to decide which is most appropriate for your
project. LIDAR is preferable for detailed sudies with resolutions of
approx 10m. Digital elevation models of 30 m (1 arc second) to 90 m (3
arc second) resolution are often used for regional to continental-scale
drainage inversions. Computational power will be one consideration when
deciding on the areal extent of your project. 

Checkout `Terrain Data on Github <https://github.com/openterrain/openterrain/wiki/Terrain-Data>`_ for an extensive list of digital elevation
models available. Here are the main high resolution datasets 
with global coverage:

**Space Shuttle Radar Topography Mission (SRTM):** SRTM has global 30m
and 90m datasets available on `NASA’s Earthdata portal <https://search.earthdata.nasa.gov/search>`_. NASA gathered the data using
interferometric synthetic arpeture radar (inSAR) on an 11 day mission
back in 2000 aboard its Space Shuttle Endeavour. Keep a look out for
NASADEM which will be a fully reprocessed SRTM data using
state-of-the-art interferometric processing techniques. The final
version has not yet been released.

**ASTER Global Digital Elevation Model:** ASTER GDEM-2 is also available
for download at NASA Earthdata. ASTER GDEM is a collaboration between
NASA and Japan’s Advanced Spaceborne Thermal Emission and Reflection
Radiometer (ASTER). The digital elevation model was created using 
stereoscopic pairs and photogrammetry to measure
elevation from two images at different angles. Cloud cover is the main
problem with accuracy of the ASTER GDEM products but on the other hand it
is considered to be a more accurate representation of rugged mountainous terrain than SRTM.

**JAXA’s Global ALOS 3D World:** Tiles can be downloaded from `JAXA’s
portal <https://www.eorc.jaxa.jp/ALOS/en/aw3d30/>`_ on registration. This
30m digital surface model was collected aboard the Advanced Land
Observing Satellite "ALOS" by the Panachromatic Remote-sensing
Instrument for Stereo Mapping (PRISM), with the latest version released
in 2019.

**LIDAR:** The following `website <https://arheologijaslovenija.blogspot.com/p/blog-page_81.html?spref=tw>`_ has a nice compilation of some of the
global LIDAR data available for use.

.. _downloading:

-------------------------
Downloading SRTM Granules
-------------------------

Earthdata website provides a ready-made script to download your chosen
granules directly onto your computer. Opt for direct download and click
on *"Download Access Script"*. This will take you to a page containing
instructions on how to use the download script. Click the *"Download
Script File"* button to download the script.

If you are using Ubuntu, make the following change to the Download script:

:math:`\#`!/bin/sh :math:`\rightarrow` :math:`\#`!/bin/bash 

as *bin/sh* no longer links to bash, instead pointing to another shell called 
*dash*. Now execute and run the script from the command line in your Downloads
directory. You will be prompted for your Earthdata username and password,
be aware that nothing will show up when you type your password. The
grids will now be automatically downloaded to the Downloads directory, where you
can unzip files to you chosen destination directory.

.. code:: bash

   #Change to Downloads directory
   $ cd Downloads
   #Execute script
   $ chmod 777 download.sh 
   #Run program to download STRM granules from Earthdata website
   $ ./download.sh 
   #Unzip files to chosen directory
   $ unzip "*.hgt" destination-directory

.. _merging-raster:

---------------
Merging rasters
---------------

Merge individual grids into a DEM using *gdal_merge.py*. By far the simplest and most time efficient way when compared to ArcGIS’s *“Mosaic to New
Raster"* tool. To give an example, 176 SRTM grids at 30m resolution covering the northern Arabian Peninsula was stitched together in less than a minute. 

The following code example uses an output format of Gtiff but any format can be
specified using the *-of* flag. 

.. code:: bash

   $ ls *.hgt > DEMs.txt
   $ gdal_merge.py -of Gtiff -n -32768 -a_nodata -32768 \
   	    -o merged_dem.tif --optfile DEMs.txt

**No data values.** No_data values can be lost in
translation so specify a no_data value to maintain NoData cells in the
merged dem. This will be a pre-requisite if using programs like RichDem to hydrologically process the DEM. The *-n* flag ignores any
pixels in the original grids containing this pixel value, while
*-a_nodata* assigns a specified no_data value to the output grid.
You can check no_data values in the original SRTM grids using
*gdalinfo*. This tool will list information about your raster dataset,
including the raster’s coordinate system, resolution (i.e. pixel size)
and regional extent.

**Example**

.. code:: bash
   
   $ gdalinfo ~/openev/utm.tif

::

   Driver: GTiff/GeoTIFF
   Size is 512, 512
   Coordinate System is:
   PROJCS["NAD27 / UTM zone 11N",
       GEOGCS["NAD27",
           DATUM["North_American_Datum_1927",
               SPHEROID["Clarke 1866",6378206.4,294.978698213901]],
           PRIMEM["Greenwich",0],
           UNIT["degree",0.0174532925199433]],
       PROJECTION["Transverse_Mercator"],
       PARAMETER["latitude_of_origin",0],
       PARAMETER["central_meridian",-117],
       PARAMETER["scale_factor",0.9996],
       PARAMETER["false_easting",500000],
       PARAMETER["false_northing",0],
       UNIT["metre",1]]
   Origin = (440720.000000,3751320.000000)
   Pixel Size = (60.000000,-60.000000)
   Corner Coordinates:
   Upper Left  (  440720.000, 3751320.000) (117d38'28.21"W, 33d54'8.47"N)
   Lower Left  (  440720.000, 3720600.000) (117d38'20.79"W, 33d37'31.04"N)
   Upper Right (  471440.000, 3751320.000) (117d18'32.07"W, 33d54'13.08"N)
   Lower Right (  471440.000, 3720600.000) (117d18'28.50"W, 33d37'35.61"N)
   Center      (  456080.000, 3735960.000) (117d28'27.39"W, 33d45'52.46"N)
   Band 1 Block=512x16 Type=Byte, ColorInterp=Gray

.. _projecting:

---------------
Projecting data
---------------

Use *gdalwarp* to project the DEM into your preferred coordinate system
before importing into GRASS. SRTM raster
datasets uses a geographic coordinate system based on a spherical
surface. This can be problematic when measuring distances in angular
units as it is highly dependant on where you are on the Earth’s surface.
Extracting rivers relies on accurately knowing river distances measured
in length, so use an equal areas projected coordinate system. For small
study areas, Universal Transvere Mercator (UTM) system is widely used
while Albers Equal Areas or equivalent projections may be used for
regional or continent-wide analyses. Note that you will also need to consider 
the appropriate ellipsoid to use for your projection.

.. code:: bash

   $ proj='+proj=lcc +lat_1=17.0 +lat_2=33.0 +lat_0=25.08951 \
   	    +lon_0=48.0 +ellps=intl +units=m +no_defs'

   $ gdalwarp -t_srs $proj merged_dem.tif projected_dem.tif

You can use *-te <xmin ymin xmax ymax>* to clip the dem to a specific
extent if you want.

.. _holes:

-------------
Filling Voids
-------------

The primary objective of the NASA MeaSUREs project (Making Earth System
Data Records for Use in Research Environments) Program was to remove
voids (no data holes) in the NASA SRTM DEM. In theory data should be
seamless and no processing is required to fill voids.

There is a tool in GRASS to check and fill no data voids in the dem. It
is also possible to use the flag in gdal *dstnodata -9999* to fix any
issues although this isn’t always guaranteed to work.
