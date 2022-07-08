
Contents
========


* `Contents <#contents>`__

  * `Open EII Video Profiler <#open-eii-video-profiler>`__

    * `Open EII Video Profiler Prerequisites <#open-eii-video-profiler-prerequisites>`__
    * `Open EII Video Profiler Modes <#open-eii-video-profiler-modes>`__
    * `Open EII Video Profiler Configurations <#open-eii-video-profiler-configurations>`__
    * `Run Video Profiler <#run-video-profiler>`__
    * `Run VideoProfiler in Helm Use Case <#run-videoprofiler-in-helm-use-case>`__
    * `Optimize Open EII Video Pipeline by Analysing Video Profiler Results <#optimize-open-eii-video-pipeline-by-analysing-video-profiler-results>`__
    * `Benchmarking with Multi-instance Config <#benchmarking-with-multi-instance-config>`__

Open EII Video Profiler
-----------------------

This tool can be used to determine the complete metrics involved in the entire Video pipeline by measuring the time difference between every component of the pipeline and checking for Queue blockages at every component thereby determining the fast or slow components of the whole pipeline. It can also be used to calculate the FPS of any Open EII modules based on the stream published by that respective module.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for file names, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights for Industrial (Open EII). This is due to the product name change of EII as Open EII.


Open EII Video Profiler Prerequisites
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


#. 
   VideoProfiler expects a set of config, interfaces and public private keys to be present in ETCD as a prerequisite.
    To achieve this, ensure an entry for VideoProfiler with its relative path from `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory is set in any of the .yml files present in `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory. An example has been provided below:

   .. code-block:: sh

           AppContexts:
           - VideoIngestion
           - VideoAnalytics
           - tools/VideoProfiler

#. 
   With the above prerequisite done, run the below to command:

   .. code-block:: sh

           python3 builder.py -f usecases/video-streaming.yml

Open EII Video Profiler Modes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default, the Open EII Video Profiler supports the ``FPS`` and the ``Monitor`` mode. The following are details for these modes:


* 
  FPS mode
  Enabled by setting the 'mode' key in `config <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/config.json>`_ to 'fps', this mode calculates the frames per second of any Open EII module by subscribing to that module's respective stream.

  .. code-block:: sh

        "mode": "fps"

  ..

     **Note:** For running Video Profiler in the FPS mode, it is recommended to keep PROFILING_MODE set to false in `.env <https://github.com/open-edge-insights/eii-core/tree/master/build/.env>`_ for better performance.


* 
  Monitor mode
  Enabled by setting the 'mode' key in `config <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/config.json>`_ to 'monitor', this mode calculates average & per frame stats for every frame while identifying if the frame was blocked at any queue of any module across the video pipeline thereby determining the fastest/slowest components in the pipeline. To be performant in profiling scenarios, VideoProfiler is enabled to work when subscribing only to a single topic in monitor mode.

  User must ensure that ``ingestion_appname`` and ``analytics_appname`` fields of the ``monitor_mode_settings`` need to be set accordingly for monitor mode.

  Refer the following example config where VideoProfiler is used in monitor mode to subscribe PySafetyGearAnalytics CustomUDF results.

  .. code-block:: javascript

           "config": {
           "mode": "monitor",
           "monitor_mode_settings": {
                                       "ingestion_appname": "PySafetyGearIngestion",
                                       "analytics_appname": "PySafetyGearAnalytics",
                                       "display_metadata": false,
                                       "per_frame_stats":false,
                                       "avg_stats": true
                                   },
           "total_number_of_frames" : 5,
           "export_to_csv" : false
       }

  .. code-block:: sh

           "mode": "monitor"

The stats to be displayed by the tool in ``monitor_mode`` can be set in the ``monitor_mode_settings`` key of `config.json <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/config.json>`_.


* 'display_metadata': Displays the raw meta-data with timestamps associated with every frame.
* 'per_frame_stats': Continously displays the per frame metrics of every frame.
* 'avg_stats': Continously displays the average metrics of every frame.

.. note:: 


   * As a prerequisite for running in profiling or monitor mode, VI/VA should be running with the PROFILING_MODE set to **true** in `.env <https://github.com/open-edge-insights/eii-core/tree/master/build/.env>`_
   * It is mandatory to have a UDF for running in monitor mode. For instance `GVASafetyGearIngestion <https://github.com/open-edge-insights/video-custom-udfs/blob/master/GVASafetyGearIngestion/README.md>`_ does not have any udf(since it uses GVA elements) so it will not be supported in monitor mode. The workaround to use GVASafetyGearIngestion in monitor mode is to add `dummy-udf <https://github.com/open-edge-insights/video-common/blob/master/udfs/README.md>`_ in `GVASafetyGearIngestion-config <https://github.com/open-edge-insights/video-custom-udfs/blob/master/GVASafetyGearIngestion/config.json>`_.


Open EII Video Profiler Configurations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


#. 
   dev_mode

   Setting this to false enables secure communication with the Open EII stack. User must ensure this switch is in sync with DEV_MODE in `.env <https://github.com/open-edge-insights/eii-core/tree/master/build/.env>`_
   With PROD mode enabled, the path for the certs mentioned in `config <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/config.json>`_ can be changed by the user to point to the required certs.

#. 
   total_number_of_frames

   If mode is set to 'fps', the average FPS is calculated for the number of frames set by this variable.
   If mode is set to 'monitor', the average stats is calculated for the number of frames set by this variable.
   Setting it to (-1) will run the profiler forever unless terminated by signal interrupts('Ctrl+C').
   total_number_of_frames should never be set as (-1) for 'fps' mode.

#. 
   export_to_csv

   Setting this switch to **true** exports csv files for the results obtained in VideoProfiler. For monitor_mode, runtime stats printed in the csv
   are based on the the following precdence: avg_stats, per_frame_stats, display_metadata.

Run Video Profiler
^^^^^^^^^^^^^^^^^^


#. 
   Set environment variables accordingly in `config.json <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/config.json>`_.

#. 
   Set the required output stream/streams and appropriate stream config in `config.json <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/config.json>`_ file.

#. 
   If VideoProfiler is subscribing to multiple streams, ensure the **AppName** of VideoProfiler is added in the Clients list of all the publishers.

#. 
   If using Video Profiler in IPC mode, make sure to set required permissions to socket file created in SOCKET_DIR in `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_.

   .. code-block:: sh

           sudo chmod -R 777 /opt/intel/eii/sockets

   ..

      **Note:**


      * This step is required everytime publisher is restarted in the IPC mode.
      * **Caution:** This step will make the streams insecure. Do not do it on a production machine.
      * Refer the below VideoProfiler interface example to subscribe to PyMultiClassificationIngestion CustomUDF results in the FPS mode.


   .. code-block:: javascript

        "/VideoProfiler/interfaces": {
               "Subscribers": [
                   {
                       "EndPoint": "/EII/sockets",
                       "Name": "default",
                       "PublisherAppName": "PyMultiClassificationIngestion",
                       "Topics": [
                           "py_multi_classification_results_stream"
                       ],
                       "Type": "zmq_ipc"
                   }
               ]
           },

#. 
   If running VideoProfiler with helm usecase or trying to subscribe to any external publishers outside the Open EII network, ensure the correct IP of publisher has been provided in the interfaces section in `config <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/config.json>`_ and correct ETCD host & port are set in environment for **ETCD_ENDPOINT** & **ETCD_HOST**.


   * 
     For example, for helm use case, since the ETCD_HOST and ETCD_PORT are different, run the commands mentioned below wit the required HOST IP:

     .. code-block:: sh

          export ETCD_HOST="<HOST IP>"
          export ETCD_ENDPOINT="<HOST IP>:32379"

#. 
   Refer `provision/README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md#provision>`_ to provision, build and run the tool along with the Open EII video-streaming recipe/stack.

#. 
   Run the following command to see the logs:

   .. code-block:: sh

           docker logs -f ia_video_profiler

#. 
   The runtime stats of Video Profiler if enabled with export_to_csv switch can be found at `video_profiler_runtime_stats <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/video_profiler_runtime_stats.csv>`_

   ..

      **Note:**


      * ``poll_interval`` option in the VideoIngestion `config <https://github.com/open-edge-insights/video-ingestion/blob/master/config.json>`_ sets the delay(in seconds)
         to be induced after every consecutive frame is read by the opencv ingestor.
         Not setting it will ingest frames without any delay.
      * ``videorate`` element in the VideoIngestion `config <https://github.com/open-edge-insights/video-ingestion/blob/master/config.json>`_ can be used to modify the
         ingestion rate for gstreamer ingestor.
         More info available at `README <https://github.com/open-edge-insights/video-ingestion/blob/master/README.md>`_.
      * ``ZMQ_RECV_HWM`` option shall set the high water mark for inbound messages on the subscriber socket.
         The high water is a hard limit on the maximum number of outstanding messages ZeroMQ shall queue in memory for
         any single peer that the specified socket is communicating with.
         If this limit has been reached, the socket shall enter an exeptional state and drop incoming messages.
      * In case of running Video Profiler for GVA use case we do not display the stats of the algo running with GVA since no
         UDFs are used.
      * The rate at which the UDFs process the frames can be measured using the FPS UDF and ingestion rate can be monitored accordingly.
         In case multiple UDFs are used, the FPS UDF is required to be added as the last UDF.
      * In case running this tool with VI and VA in two different nodes, same time needs to be set in both the nodes.


Run VideoProfiler in Helm Use Case
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For running VideoProfiler in helm use case to subscribe to either VideoIngestion/VideoAnalytics or any other Open EII service, the etcd endpoint, volume mount for helm certs and service endpoints are to be updated.

For connecting to the etcd server running in helm environment, the endpoint and required volume mounts should be modified in the following manner in environment and volumes section of `docker-compose.yml <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/docker-compose.yml>`_\ :

.. code-block:: yaml

     ia_video_profiler:
       depends_on:
         - ia_common
       ...
       environment:
       ...
         ETCD_HOST: ${ETCD_HOST}
         ETCD_CLIENT_PORT: ${ETCD_CLIENT_PORT}
         # Update this variable referring
         # for helm use case
         ETCD_ENDPOINT: <HOST_IP>:32379
         CONFIGMGR_CERT: "/run/secrets/VideoProfiler/VideoProfiler_client_certificate.pem"
         CONFIGMGR_KEY: "/run/secrets/VideoProfiler/VideoProfiler_client_key.pem"
         CONFIGMGR_CACERT: "/run/secrets/rootca/cacert.pem"
       ...
       volumes:
         - "${EII_INSTALL_PATH}/tools_output:/app/out"
         - "${EII_INSTALL_PATH}/sockets:${SOCKET_DIR}"
         - ./helm-eii/eii-deploy/Certificates/rootca:/run/secrets/rootca
         - ./helm-eii/eii-deploy/Certificates/VideoProfiler:/run/secrets/VideoProfiler

For connecting to any service running in helm usecase, the container IP associated with the specific service should be updated in the Endpoint section in VideoProfiler `config <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/config.json>`_\ :

The IP associated with the service container can be obtained by checking the container pod IP using docker inspect. Assuming we are connecting to VideoAnalytics service, the command to be run would be:

.. code-block:: sh

    docker inspect <VIDEOANALYTICS CONTAINER ID> | grep VIDEOANALYTICS

The output of the above command consists the IP of the VideoAnalytics container that can be updated in VideoProfiler config using EtcdUI:

.. code-block:: sh

    "VIDEOANALYTICS_SERVICE_HOST=10.99.204.80"

The config can be updated with the obtained container IP in the following way:

.. code-block:: javascript

       {
           "interfaces": {
               "Subscribers": [
                   {
                       "Name": "default",
                       "Type": "zmq_tcp",
                       "EndPoint": "10.99.204.80:65013",
                       "PublisherAppName": "VideoAnalytics",
                       "Topics": [
                           "camera1_stream_results"
                       ]
                   }
               ]
           }
       }

Optimize Open EII Video Pipeline by Analysing Video Profiler Results
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


#. 
   VI ingestor/UDF input queue is blocked, consider reducing ingestion rate.

   If this log is displayed by the Video Profiler tool, it indicates that the ingestion rate is too high or the VideoIngestion
   UDFs are slow and causing latency throughout the pipeline.
   As per the log suggests, the user can increase the poll_interval to a optimum value to reduce the blockage of VideoIngestion
   ingestor queue thereby optimizing the video pipeline in case using the opencv ingestor. if Gstreamer ingestor is used, the videorate option can be optimized by following the `README <https://github.com/open-edge-insights/video-ingestion/blob/master/README.md>`_.

#. 
   VA subs/UDF input queue is blocked, consider reducing ZMQ_RECV_HWM value or reducing ingestion rate.

   If this log is displayed by the Video Profiler tool, it indicates that the VideoAnalytics UDFs are slow and causing latency
   throughout the pipeline.
   As per the log suggests, the user can consider reducing ZMQ_RECV_HWM to an optimum value to free the VideoAnalytics UDF input/subscriber
   queue by dropping incoming frames or reducing the ingestion rate to a required value.

#. 
   UDF VI output queue blocked.

   If this log is displayed by the Video Profiler tool, it indicates that the VI to VA messagebus transfer is delayed.


   #. User can consider reducing the ingestion rate to a required value.
   #. User can increase ZMQ_RECV_HWM to an optimum value so as to not drop
      the frames when the queue is full or switching to IPC mode of communication.

#. 
   UDF VA output queue blocked.

   If this log is displayed by the Video Profiler tool, it indicates that the VA to VideoProfiler messagebus transfer is delayed.


   #. User can consider reducing the ingestion rate to a required value.
   #. User can increase ZMQ_RECV_HWM to an optimum value so as to not drop
      the frames when the queue is full or switching to IPC mode of communication.

Benchmarking with Multi-instance Config
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


#. 
   Open EII supports multi-instance config generation for benchmarking purposes. This can be acheived by running the `builder.py <https://github.com/open-edge-insights/eii-core/blob/master/build/builder.py>`_ with certain parameters, please refer to the **Multi-instance Config Generation** section of **Open EII Preeequisites** in the `README <https://github.com/open-edge-insights/eii-core/blob/master/README.md#oei-prerequisites-installation>`_ for more details.

#. 
   For running VideoProfiler for multiple streams, run the builder with the **-v** flag provided the pre-requisites mentioned above are done. The following is an example for generating **6** streams config:

   .. code-block:: sh

           python3 builder.py -f usecases/video-streaming.yml -v 6

.. note:: 


   * For multi-instance monitor mode use case, ensure only **VideoIngestion** & **VideoAnalytics** are used as **AppName** for Publishers.
   * Running **VideoProfiler** with **CustomUDFs** for monitor mode is supported for single stream only. If required for multiple streams, ensure **VideoIngestion** & **VideoAnalytics** are used as **AppName**.
   * In IPC mode, for accelerators: ``MYRIAD``\ , ``GPU``\ , and USB 3.0 Vision cameras, add ``user: root`` in `VideoProfiler-docker-compose.yml <https://github.com/open-edge-insights/eii-core/blob/master/docker-compose.yml>`_ as the subscriber needs to run as ``root`` if the publisher is running as ``root``.

