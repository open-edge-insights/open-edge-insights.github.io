
Edge Video Analytics Microservice for OEI
=========================================

The Edge Video Analytics Microservice combines the video ingestion and analytics capabilities provided by the Open Edge Insights' (OEI) visual ingestion and analytics modules. In the following sections, you will find information related to setting up the Edge Video Analytics Microservice and its configuration details.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as OEI. This is due to the product name change of EII as OEI.


Get the OEI source code
-----------------------

To set up the Edge Video Analytics Microservice, get the OEI source code. The Edge Video Analytics Microservice depends on the OEI provisioning, so you need to build it in the OEI context. Run the following commands to get the OEI source code:

.. code-block:: sh

   repo init -u "https://github.com/open-edge-insights/eii-manifests.git"
   repo sync

.. note::  To learn more about OEI manifests, refer to `eii-manifests <https://github.com/open-edge-insights/eii-manifests>`_.


Run Edge Video Analytics Microservice
-------------------------------------

You can download the required model file to use it for the pipeline that is mentioned in the `Readme <https://github.com/open-edge-insights/eii-core/blob/master/home/rkamal/eii/github/applications.industrial.edge-insights.open-edge-insights-github-io/eii_repo/IEdgeInsights/README.md#running-the-image>`_ file. For more information, refer to the `Readme <https://github.com/open-edge-insights/eii-core/blob/master/home/rkamal/eii/github/applications.industrial.edge-insights.open-edge-insights-github-io/eii_repo/IEdgeInsights/README.md#running-the-image>`_.

After provisioning the OEI stack, complete the following steps to run the Video Analytics Microservice in OEI context. For more information on OEI provisioning, refer to the `Readme <https://github.com/open-edge-insights/eii-core/blob/master/README.md#provision>`_.

.. code-block:: sh

   cd [WORK_DIR]/IEdgeInsights/build

   # Execute the builder.py script
   python3 builder.py -f usecases/evas.yml

   # Create the necessary items for the service
   sudo mkdir -p /opt/intel/eii/models/

   # Copy the downloaded model files to /opt/intel/eii
   sudo cp -r models /opt/intel/eii/

   # Build the docker containers
   docker-compose -f docker-compose-build.yml build

   # Start the docker containers
   docker-compose up -d ia_configmgr_agent # workaround for now, we are exploring on getting this and the sleep avoided
   sleep 30 # If any failures like configmgr data store client certs or msgbus certs failures, please increase this time to a higher value
   docker-compose up -d

.. note:: 
   If the prebuilt container image for the Edge `Video Analytics Microservice <https://hub.docker.com/r/intel/edge_video_analytics_microservice>`_ is not available on the host system, then you can run the ``docker-compose up -d`` command to download the image.
   This repository provides the Deep Learning Streamer (DL Streamer) pipelines that are required to use a URI source and send the ingested frames using the OEI MsgBus Publisher. The commands mentioned above that creates the ``models/`` directory and the ``pipelines/``\ , installs the required directories for the Edge Video Analytics Microservice to run the DL Streamer pipelines.


Configuration
-------------

Configure the Edge Video Analytics Microservice
-----------------------------------------------

Refer to the `config <https://github.com/open-edge-insights/eii-core/blob/master/home/rkamal/eii/github/applications.industrial.edge-insights.open-edge-insights-github-io/eii_repo/IEdgeInsights/video-analytics-serving/config.json>`_ file for the configuration of the Edge Video Analytics Microservice. The default configuration will start the ``object_detection`` demo for OEI. The following table describes the configuration attributes that are currently supported.

.. list-table::
   :header-rows: 1

   * - Parameter
     - Description
   * - ``source``
     - The source of the frames. This must be ``"gstreamer"`` or ``"msgbus"``.
   * - ``source_parameters``
     - The parameters for the source element. The provided object is the typical parameters.
   * - ``pipeline``
     - The name of the DL Streamer pipeline to use. This should correspond to a directory in the ``pipelines`` directory.
   * - ``pipeline_version``
     - The version of the pipeline to use. This is a sub-directory of a pipeline in the ``pipelines`` directory.
   * - ``publish_frame``
     - Boolean flag for whether to publish the metadata and the analyzed frame, or only the metadata.
   * - ``model_parameters``
     - Provides the parameters for the model used for inference.


Configure the interfaces
^^^^^^^^^^^^^^^^^^^^^^^^

In OEI mode, currently the Edge Video Analytics Microservice only supports launching a single pipeline and publishing on a single topic. This means, in the configuration file (\ ``config.json``\ ),
the single JSON object in the ``Publisher`` list is where the configuration resides for the published data. See the OEI documentation for more details on the structure of this.

Configure OEI subscriber and publisher
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Edge Video Analytics Microservice also supports subscribing and publishing messages or frames using the MessageBus. Specify the following details in the ``config.json`` file:


* In the **Subscribers** section, specify the endpoint details for the OEI service that you want to subscribe from.
* In the **Publishers** section, specify the endpoint details where you want to publish.

Make the following changes to enable the injection of frames into the GStreamer pipeline obtained from the MessageBus


* 
  Set the source parameter in the ``config.json`` file to ``msgbus`` as follows:

  .. code-block:: javascript

       "config": {
           "source": "msgbus"
       }

* 
  Set the template of respective pipeline to ``appsrc`` as source instead of ``uridecodebin`` as follows:

.. code-block:: javascript

       {
           "type": "GStreamer",
           "template": ["appsrc name=source",
                        " ! rawvideoparse",
                        " ! appsink name=destination"
                       ]
       }
