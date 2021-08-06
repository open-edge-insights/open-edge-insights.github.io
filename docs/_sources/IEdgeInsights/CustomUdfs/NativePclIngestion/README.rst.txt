
PCL Demo UDF
------------

This udf accepts the realsense color and depth frames, converts them to rs2 frame type and generates pcl point cloud.
The code for generating PCD file has been commented intentionally. Interested user need to uncomment it in **rs2pcl.cpp** file by following instruction in line number 78. The PCD file is created in **/var/tmp** directory.

``UDF config``\ :

.. code-block:: json

   {
       "name": "rs2pcl",
       "type": "raw_native"
   }

Refer `udfs-README <https://github.com/open-edge-insights/video-common/blob/master/udfs/README.md>`_ for more information on configs of other in-built udfs like dummy, fps and resize udfs.
