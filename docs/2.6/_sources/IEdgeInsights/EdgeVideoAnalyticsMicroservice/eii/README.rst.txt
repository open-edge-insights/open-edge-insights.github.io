
Edge Video Analytics Microservice for EII
=========================================

The Edge Video Analytics Microservice combines the video ingestion and analytics
capabilities provided by EII visual ingestion and analytics modules.
This directory provides the Intel® Deep Learning Streamer(Intel® DL Streamer) pipelines
to perform object detection on an input URI source and send the ingested frames and
inference results using the EII MsgBus Publisher. It also provides a Docker compose
and config file to use Edge Video Analytics Microservice with Edge Insights for Industrial
software stack.

Pre-requisites
--------------

Fetch the EII source code by following the steps mentioned below:

.. code-block:: sh

   repo init -u "https://github.com/open-edge-insights/eii-manifests.git"

   repo sync

For more details, refer `here <https://github.com/open-edge-insights/eii-manifests>`_.

Complete the pre-requisite of provisioning the EII stack by referring to
`README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md#provision>`_ 

Since the model files are large in size, they are not part of the repo.
Download the required model files to be used for the pipeline mentioned in `config <https://github.com/open-edge-insights/edge-video-analytics-microservice/blob/master/eii/config.json>`_
by following the first four steps mentioned in `README <https://github.com/open-edge-insights/edge-video-analytics-microservice/blob/master/README.md#running-the-image>`_.

Run the following commands to set the environment, build ia_configmgr_agent container
and copy models to the required directory:

.. code-block:: sh

   cd [WORK_DIR]/IEdgeInsights/build

   # Execute the builder.py script
   python3 builder.py -f usecases/evas.yml

   # Create some necessary items for the service
   sudo mkdir -p /opt/intel/eii/models/

   # Copy the downloaded model files to /opt/intel/eii
   sudo cp -r models /opt/intel/eii/

Running
-------

Run the following command to pull the prebuilt EII container images and Edge Video Analytics Microservice
from Docker Hub and run the containers in the detached mode:

.. code-block:: sh

   # Start the docker containers
   docker-compose up -d

.. note:: 

   The pre-built container image for Edge Video Analytics Microservice (https://hub.docker.com/r/intel/edge_video_analytics_microservice)
   gets downloaded when one runs the docker-compose up -d if the image is not already present on the host system.


Configuration
-------------

Please see the `edge-video-analytics-microservice/eii/config.json <https://github.com/open-edge-insights/edge-video-analytics-microservice/blob/master/eii/config.json>`_ file for the configuration of the
Edge Video Analytics Microservice. The default configuration will start the
object_detection demo for EII.

The config file is divided in two sections listed below:

Config
^^^^^^

The table below describes each of the attributes currently
supported in config section.

.. list-table::
   :header-rows: 1

   * - Parameter
     - Description
   * - ``cert_type``
     - Type of EII certs to be created, must be ``"zmq"`` or ``"pem"``.
   * - ``source``
     - Source of the frames, must be ``"gstreamer"`` or ``"msgbus"``.
   * - ``source_parameters``
     - The parameters for the source element. The provided object is the typical parameters.
   * - ``pipeline``
     - The name of the DL Streamer pipeline to use (should correspond to a directory in the pipelines directory).
   * - ``pipeline_version``
     - The version of the pipeline to use. This typically is a subdirectory of a pipeline in the pipelines directory.
   * - ``publish_frame``
     - Boolean flag for whether to publish the meta-data and the analyzed frame, or just the meta-data.
   * - ``model_parameters``
     - Provides the parameters for the model used for inference.


Interfaces
^^^^^^^^^^

In EII mode, the microservice only supports launching a single pipeline and publishing on a
single topic currently. This means, in the configuration file ("config.json"),
the single JSON object in the ``Publisher`` list is where the configuration
resides for the published data. See the `EII documentation <https://github.com/open-edge-insights/eii-core/blob/master/README.md#adding-new-eii-service-so-it-gets-picked-up-by-builder>`_ for more details on
the structure of this.

The Edge Video Analytics Microservice also supports subscribing and publishing messages/frames
using the EII_Message_Bus.
The endpoint details for the EII service you need to subscribe from is to be
provided in the **Subscribers** section in `config <https://github.com/open-edge-insights/edge-video-analytics-microservice/blob/master/eii/config.json>`_ and the endpoints
where you need to publish to are to be provided in **Publishers** section in
`config <https://github.com/open-edge-insights/edge-video-analytics-microservice/blob/master/eii/config.json>`_.

For enabling injection of frames into the GStreamer pipeline obtained from
EIIMessageBus, please ensure the following changes are made:


* 
  The source parameter in `config <https://github.com/open-edge-insights/edge-video-analytics-microservice/blob/master/eii/config.json>`_ is set to msgbus

  .. code-block:: javascript

       "config": {
           "source": "msgbus"
       }

* 
  The template of respective pipeline is set to appsrc as source instead of uridecodebin

.. code-block:: javascript

       {
           "type": "GStreamer",
           "template": ["appsrc name=source",
                        " ! rawvideoparse",
                        " ! appsink name=destination"
                       ]
       }
