
DiscoverHistory tool
====================

You can get history metadata and images from the InfluxDB and ImageStore containers using the DiscoverHistory tool.

Build and run the DiscoverHistory tool
--------------------------------------

This section provides information for building and running DiscoverHistory tool in various modes such as the PROD mode and the DEV mode. To run the DiscoverHistory tool base images should be on the same node. Ensure that on the node where the DiscoverHistory tool is running, the ``ia_common`` and ``ia_eiibase`` base images are also available. For scenario, where EII and DiscoverHistory tool are not running on the same node, then you must build the base images, ``ia_common`` and ``ia_eiibase``.

Prerequisites
^^^^^^^^^^^^^

As a prerequisite to run the DiscoverHistory tool, a set of config, interfaces, public, and private keys should be present in etcd. To meet the prerequisite, ensure that an entry for the DiscoverHistory tool with its relative path from the ``[WORK_DIR]/IEdgeInsights]`` directory is set in the ``video-streaming-storage.yml`` file in the ``[WORK_DIR]/IEdgeInsights/build/usecases/`` directory. For more information, see the following example:

.. code-block:: sh

       AppContexts:
       - VideoIngestion
       - VideoAnalytics
       - Visualizer
       - WebVisualizer
       - tools/DiscoverHistory
       - ImageStore
       - InfluxDBConnector

Run the DiscoverHistory tool in the PROD mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After completing the prerequisites, perform the following steps to run the DiscoverHistory tool in the PROD mode:


#. Open the ``config.json`` file.
#. Enter the query for InfluxDB.
#. 
   Run the following command to generate the new ``docker-compose.yml`` that includes DiscoverHistory:

   .. code-block:: sh

          python3 builder.py -f usecases/video-streaming-storage.yml

#. 
   Provision, build, and run the DiscoverHistory tool along with the EII video-streaming-storage recipe or stack. For more information, refer to the `EII README <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_.

#. Check if the ``imagestore`` and ``influxdbconnector`` services are running.
#. Locate the ``data`` and the ``frames`` directories from the following path:
   ``/opt/intel/eii/tools_output``.
   ..

      Note: The ``frames`` directory will be created only if ``img_handle`` is part of the select statement.


#. Use the ETCDUI to change the query in the configuration.
#. 
   Run the following command to start container with new configuration:

   .. code-block:: sh

         docker restart ia_discover_history

Run the DiscoverHistory tool in the DEV mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After completing the prerequisites, perform the following steps to run the DiscoverHistory tool in the DEV mode:


#. Open the [.env] file from the ``[WORK_DIR]/IEdgeInsights/build`` directory.
#. Set the ``DEV_MODE`` variable as ``true``.

Run the DiscoverHistory tool in the zmq_ipc mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After completing the prerequisites, to run the DiscoverHistory tool in the zmq_ipc mode, modify the interface section of the ``config.json`` file as follows:

.. code-block:: json

       {
           "type": "zmq_ipc",
           "EndPoint": "/EII/sockets"
       }

Sample select queries
---------------------

The following table shows the samples for the select queries and its details:

.. list-table::
   :header-rows: 1

   * - Select Query
     - Details
   * - "select * from camera1_stream_results order by desc limit 10"
     - This query will return latest 10 records.
   * - "select height,img_handle from camera1_stream_results order by desc limit 10"
     - 
   * - "select * from camera1_stream_results where time>='2019-08-30T07:25:39Z' AND time<='2019-08-30T07:29:00Z'"
     - This query will return all the records between the given time frame, which is (time>='2019-08-30T07:25:39Z' and time<='2019-08-30T07:29:00Z')
   * - "select * from camera1_stream_results where time>=now()-1h"
     - This query will return all the records from the current time, going back upto last 1 hour.
   * - 


..

   Note
   Include the following parameters in the query to get the good and the bad frames:


   * *img_handle
   * *defects
   * *encoding_level
   * *encoding_type
   * *height
   * *width
   * *channel

   The following examples shows how to include the parameters:


   * "select img_handle, defects, encoding_level, encoding_type, height, width, channel from camera1_stream_results order by desc limit 10"
   * "select * from camera1_stream_results order by desc limit 10"


Multi-instance feature support for the Builder script with the DiscoverHistory tool
-----------------------------------------------------------------------------------

The multi-instance feature support of Builder works only for the video pipeline (\ ``[WORK_DIR]/IEdgeInsights/build/usecase/video-streaming.yml``\ ). For more details, refer to the `EII core readme <https://github.com/open-edge-insights/eii-core/blob/master/README.md#running-builder-to-generate-multi-instance-configs>`_

In the following example you can view how to change the configuration to use the builder.py script -v 2 feature with 2 instances of the DiscoverHistory tool enabled:

.. image:: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/DiscoverHistory/img/discoverHistoryTool-conf-change1.png
   :target: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/DiscoverHistory/img/discoverHistoryTool-conf-change1.png
   :alt: DiscoverHistory instance 1 interfaces


.. image:: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/DiscoverHistory/img/discoverHistoryTool-conf-change2.png
   :target: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/DiscoverHistory/img/discoverHistoryTool-conf-change2.png
   :alt: DiscoverHistory instance 2 interfaces

