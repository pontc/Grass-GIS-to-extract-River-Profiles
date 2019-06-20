=================
Stream Extraction
=================

------------------------------------------
Extracting Individual Streams for Plotting
------------------------------------------

.. figure:: images/grass_riv65.png
   :alt: river_profile

   Right: River long profile and drainage area 
   Left: Plan view of river profile.

To extract individual stream channels we first need to connect the 
stream network and find linking segments using their unique *cat* identifiers.
In the example below, replace stream_300 with the name given for your vectorised
 stream network.

..code:: bash

   v.db.addcolumn stream_300 columns="x1 double,x2 double,y1 double,y2 double,tostream int"
   v.to.db map=stream_300 option=start columns=x1,y1
   v.to.db map=stream_300 option=end columns=x2,y2

   #Indexing sqllite database
   db.execute sql="create index stream_300_i1 on stream_300(x2,y2)"
   db.execute sql="create index stream_300_i2 on stream_300(x1,y1)"

   #Finding cat numbers to linking stream segments
   db.execute sql="update stream_300 set tostream=(select s2.cat from stream_300 s2 where stream_300.x2=s2.x1 and stream_300.y2=s2.y1 and stream_300.cat<>s2.cat)"

Now we can loop through connecting river segments, to extract points along an
individual stream. The output  vector map has 2 layers - layer 2 stores each point
as a unique category  together with the distance from the line’s start stored as
*’along’*. Use *v.what.rast* to retrieve values for elevation from the dem and 
drainage area pixels from the flow accumulation raster and add those columns 
to the points file. 

The following example uses a random selection of start_nodes of the 
vectorised stream network for stream extraction. 

.. code:: bash

   #Setting parameters

   #Determining cell resolution to calculate drainage area (km^2)
   GRASS :~ > eval `g.region -g`
   GRASS :~ > SQ_M=$( echo "${ewres}*${nsres}" | bc -l )
   GRASS :~ > res=$( echo "${nsres}" )
   GRASS :~ > echo "Using resolution: $res and cell area: $SQ_M m^2"

   #Extract individual streams

   #Set maximum number of rivers to extract
   lim = 50

   i=0
   time for start_node in `db.select -c sql='select cat from stream_300 where stream_type="start" order by random() limit '${lim}''`; do
   i=$(( ${i} + 1 ))
       echo "$start_node"
      #Extract linking stream segments
      v.extract -d input=stream_300 output=line_${start_node} where="cat in (with recursive cats(cat) as (values(${start_node}) union all select tostream from stream_300 s, cats where s.cat=cats.cat) select cat from cats where cat is not null)" new=0
      #Convert to polyline (to remove duplicate vertices)
      v.build.polylines input=line_${start_node} output=cpath_${start_node}
      #Extract X, Y coordinates and distance along channel
      v.to.points input=cpath_${start_node} output=pnt_${start_node} use=vertex layer=-1 
      #Remove superfluous nodes from v.build.polylines extraction
      v.extract input=pnt_${start_node} where="along > 0" output=cpathpnt_${start_node} layer=2 
      #Extract elevation at each point
      v.what.rast cpathpnt_${start_node} raster=dem column="elevation_m" layer=2 
      #Extract flow accumulation at each point
      v.what.rast cpathpnt_${start_node} raster=facc column=accum_pixels layer=2 
      v.db.addcolumn cpathpnt_${start_node} columns="drainage_area_m2 double" layer=2
      #Convert flow accumulation pixels to m^2
      v.db.update cpathpnt_${start_node} column=drainage_area_m2 query_col="accum_pixels*${SQ_M}" layer=2
      #Drop column with flow accumulation pixels
      v.db.dropcolumn cpathpnt_${start_node} columns="accum_pixels" layer=2 
      v.db.renamecolumn cpathpnt_${start_node} column="along,distance_m" layer=2 
      #Output each stream channel in a separate ascii file
      v.out.ascii -c input=cpathpnt_${start_node} layer=2 columns="elevation_m,distance_m,drainage_area_m2" separator=',' output=riv${start_node}.dat 
      echo "----------------------------"
      echo " River $i of ${lim} created "
      echo "----------------------------"
      echo ""
   done

Alternatively, you can use your own pre-determined set of X Y coordinates and 
feed them into the loop above. 

**Example ASCII file output**

::

   east,north,cat,elevation_m,distance_m,drainage_area_m2
   287177.44940819,-1014567.43537917,4,1372.00000000,27.92307279,279911.57985879
   287149.52633541,-1014539.51230638,5,1368.00000000,67.41226103,285369.46581704
   287177.44940819,-1014511.58923359,6,1368.00000000,106.90144927,287708.55979915
   287177.44940819,-1014483.6661608,7,1359.00000000,134.82452206,288488.25779319
   287177.44940819,-1014455.74308801,8,1352.00000000,162.74759485,356321.98327429
   287177.44940819,-1014427.82001522,9,1342.00000000,190.67066764,368017.45318482
   287149.52633541,-1014399.89694243,10,1335.00000000,230.15985588,372695.64114904
   287121.60326262,-1014371.97386965,11,1331.00000000,269.64904412,391408.39300589
   287093.68018983,-1014371.97386965,12,1328.00000000,297.57211691,772680.71208932
   287065.75711704,-1014371.97386965,13,1323.00000000,325.49518970,854549.00146306
   287037.83404425,-1014344.05079686,14,1315.00000000,364.98437794,873261.75331992
   287009.91097146,-1014344.05079686,15,1313.00000000,392.90745072,890415.10918870
   286981.98789867,-1014371.97386965,16,1313.00000000,432.39663897,895093.29715291
   286954.06482589,-1014371.97386965,17,1311.00000000,460.31971175,902110.57909923
   286926.1417531,-1014371.97386965,18,1307.00000000,488.24278454,902890.27709327
   286898.21868031,-1014371.97386965,19,1304.00000000,516.16585733,909907.55903959
   286870.29560752,-1014399.89694243,20,1300.00000000,555.65504557,958248.83466980
   286842.37253473,-1014371.97386965,21,1292.00000000,595.14423381,959808.23065787
   286814.44946194,-1014344.05079686,22,1281.00000000,634.63342205,973842.79455051
   286786.52638915,-1014316.12772407,23,1280.00000000,674.12261030,976961.58652666
   286758.60331637,-1014316.12772407,24,1278.00000000,702.04568308,981639.77449087
   286730.68024358,-1014288.20465128,25,1270.00000000,741.53487132,983199.17047894


You are now ready to plot and analyse your river profiles!