
EII VA Serving Microservice
===========================

The EII VA Serving (EVAS) Microservice wraps the Intel Video Analytics Serving
library to provide an EII visual ingestion and analytics module utilizing the
features and functionality of the library.

Building
--------

The Docker container for this service cannot be built by itself. It must be
built in the context of the EII multi-repo project. Follow the instructions
`here <https://github.com/open-edge-insights/eii-manifests>`_.

Since the model files are large in size, they are not part of the repo.
Download the required model files to be used for the pipeline mentioned in 
`config <https://github.com/open-edge-insights/video-analytics/blob/master/Serving/config.json>`_ by referring https://github.com/intel/video-analytics-serving/tree/master/tools/model_downloader
and have the downloaded model files placed in "/opt/intel/eii/models" 
directory.

.. code-block:: sh

   # Execute the builder.py script
   $ python3 buidler.py -f evas.yml

   # Create some necessary items for the service
   $ sudo mkdir -p /opt/intel/eii/models/

   # Build the docker containers
   $ docker-compose build

   # Start the docker containers
   $ docker-compose up -d

This repository provides the VA Serving pipeline needed for using a URI source
and sending the ingested frames using the EII MsgBus Publisher. The commands above
which create the, "models/", directory and the, "pipelines/", install needed
directories for the service to run and install the VA Serving pipeline.

Configuration
-------------

Please see the ``config.json`` file for the configuration of the service. The
default configuration will start the object_detection demo for EII.

The table below describes each of the configuration attributes currently
supported.

..

   **TODO:** Need to document these parameters much more.


.. list-table::
   :header-rows: 1

   * - Parameter
     - Description
   * - ``source``
     - Source of the frames, must be ``"gstreamer"`` or ``"msgbus"``.
   * - ``source_parameters``
     - The parameters for the source element. The provided object is the typical parameters.
   * - ``pipeline``
     - The name of the VA Serving pipeline to use (should correspond to a directory in the pipelines directory).
   * - ``pipeline_version``
     - The version of the pipeline to use. This typically is a subdirectory of a pipeline in the pipelines directory.
   * - ``publish_frame``
     - Boolean flag for whether to publish the meta-data and the analyzed frame, or just the meta-data.
   * - ``model_parameters``
     - Provides the parameters for the model if DL streamer is being used in the VA Serving pipeline.


Configuring the Interfaces
^^^^^^^^^^^^^^^^^^^^^^^^^^

The service only supports launching a single pipeline and publishing on a
single topic currently. This means, in the configuration file ("config.json"),
the single JSON object in the ``Publisher`` list is where the configuration
resides for the published data. See the EII documentation for more details on
the structure of this.

Configuring EII subscriber and publisher
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This service also supports subscribing and publishing messages/frames using the
EIIMessageBus.
The endpoint details for the EII service you need to subscribe from is to be
provided in the **Subscribers** section in `config <https://github.com/open-edge-insights/video-analytics/blob/master/Serving/config.json>`_ and the endpoints
where you need to publish to are to be provided in **Publishers** section in
`config <https://github.com/open-edge-insights/video-analytics/blob/master/Serving/config.json>`_.

For enabling injection of frames into the GStreamer pipeline obtained from
EIIMessageBus, please ensure the following changes are done:

.. code-block::

   * The source parameter in [config](https://github.com/open-edge-insights/video-analytics/blob/master/Serving/config.json) is set to msgbus
   ```javascript
      "config": {
          "source": "msgbus"
       }
   ```

   * The template of respective pipeline is set to appsrc as source instead of uridecodebin
   ```javascript
      {
           "type": "GStreamer",
           "template": ["appsrc name=source",
                        " ! rawvideoparse",
                        " ! appsink name=destination"
                       ]
       }
   ```
