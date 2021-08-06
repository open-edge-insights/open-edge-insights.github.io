
DiscoverHistory
===============

DiscoverHistory tool helps in pulling the history meta-data and images from InfluxDB and ImageStore containers respectively

Steps to build and run DiscoverHistory service
----------------------------------------------


* 
  **Running in PROD mode**


  #. 
     DiscoverHistory expects a set of config, interfaces & public private keys to be present in ETCD as a pre-requisite.
     To achieve this, please ensure an entry for DiscoverHistory with its relative path from `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory is set in the video-streaming-storage.yml file present in `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory. An example has been provided below:

     .. code-block:: sh

           AppContexts:
           - VideoIngestion
           - VideoAnalytics
           - Visualizer
           - WebVisualizer
           - tools/DiscoverHistory
           - ImageStore
           - InfluxDBConnector

  #. 
     Open "config.json"

  #. Provide the required query to be passed on to InfluxDB.
  #. With the above pre-requisite done, please run the below to command:
     .. code-block:: sh

           python3 builder.py -f usecases/video-streaming-storage.yml

  #. Refer `main EII README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_ to provision, build and run the tool along with the EII video-streaming-storage recipe/stack.
  #. Check imagestore and influxdbconnector services are running.
  #. You will find data & frames directories at "/opt/intel/eii/tools_output".
     (Note: if img_handle is part of select statement , then only frames
     directory will be created)
  #. Use ETCDUI to change the query in configuration. Please run below command to start container with new configuration:
     .. code-block:: sh

           docker restart ia_discover_history

* 
  **Running in DEV mode**


  #. Open `.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_
  #. Set the DEV_MODE variable as "true".
     .. code-block::

        DEV_MODE=false
     to
     .. code-block::

        DEV_MODE=true

.. note:: \ :


   * Building the base images like ia_common, ia_eiibase are must in cases if this tool isn't run on the same node
     where EII is running.
   * Please ensure that the base images i.e. ia_common and ia_eiibase are present on the node where this tool is run.


List of sample select queries
-----------------------------


#. 
   "select * from camera1_stream_results order by desc limit 10"
   This query will return latest 10 records.

#. 
   "select height,img_handle from camera1_stream_results order by desc limit 10"

#. 
   "select * from camera1_stream_results where time>='2019-08-30T07:25:39Z' AND time<='2019-08-30T07:29:00Z'"
    This query will return all the records between the given time frame i.e. (time>='2019-08-30T07:25:39Z' AND time<='2019-08-30T07:29:00Z')

#. 
   "select * from camera1_stream_results where time>=now()-1h"
    This query will return all the records from the current time, going back upto last 1 hour.

To run the tool in zmq_ipc mode
-------------------------------

User needs to modify interface section of **\ `config.json <https://github.com/open-edge-insights/eii-tools/blob/master/DiscoverHistory/config.json>`_\ ** as following

.. code-block::

       {
           "type": "zmq_ipc",
           "EndPoint": "/EII/sockets"
       }

----

**NOTE**\ :
If you want good and bad frames then the query must contain the following parameters:

.. code-block::

   *img_handle
   *defects
   *encoding_level
   *encoding_type
   *height
   *width
   *channel

   Example:
    "select img_handle, defects, encoding_level, encoding_type,  height, width, channel from camera1_stream_results order by desc limit 10"

    or

    "select * from camera1_stream_results order by desc limit 10"

----
