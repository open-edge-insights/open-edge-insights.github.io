.. role:: raw-html-m2r(raw)
   :format: html


Contents
========


* `Contents <#contents>`__

  * `VideoIngestion Module <#videoingestion-module>`__

    * `Configuration <#configuration>`__

      * `Ingestor Config <#ingestor-config>`__

    * `VideoIngestion Features <#videoingestion-features>`__

      * `Image Ingestion <#image-ingestion>`__
      * `UDF Configurations <#udf-configurations>`__
      * `Updating Security Context of VideoIngestion Helm Charts <#updating-security-context-of-videoingestion-helm-charts>`__
      * `Camera Configurations <#camera-configurations>`__

        * `GenICam GigE or USB3 Cameras <#genicam-gige-or-usb3-cameras>`__

          * `Prerequisites for Working with the GenICam Compliant Cameras <#prerequisites-for-working-with-the-genicam-compliant-cameras>`__

      * `RTSP Cameras <#rtsp-cameras>`__
      * `USB v4l2 Cameras <#usb-v4l2-cameras>`__
      * `RealSense Depth Cameras <#realsense-depth-cameras>`__

VideoIngestion Module
---------------------

The VideoIngestion (VI) module ingests the video frames from video sources in the Open Edge Insights for Industrial (Open EII) stack for processing. The example of video sources include video files or cameras such as Basler, RTSP, or USB cameras. The VI module can also perform video analytics, when it runs with the classifier and post-processing user defined functions (UDFs).


.. image:: https://raw.githubusercontent.com/open-edge-insights/video-ingestion/master/img/fig_9_1.png
   :target: https://raw.githubusercontent.com/open-edge-insights/video-ingestion/master/img/fig_9_1.png
   :alt: Video ingestion diagram


The high-level logical flow of the VideoIngestion pipeline is as follows:


#. The app reads the application configuration via the Configuration Manager which has details of the ``ingestor``\ , ``encoding``\ , and ``UDFs``.
#. Based on the ingestor configuration, the app reads the video frames from a video file or camera.
#. [\ ``Optional``\ ] The read frames are passed to one or more chained native or python UDFs for doing any pre-processing. Passing through UDFs is optional and it is not required to perform any pre-processing on the ingested frames. With chaining of UDFs supported, you also have classifier UDFs and any post-processing UDFs like resize etc., configured in the ``udfs`` key to get the classified results. For more details, refer to the `../common/video/udfs/README.md <https://github.com/open-edge-insights/video-common/blob/master/udfs/README.md>`_.
#. App gets the msgbus endpoint configuration from system environment. Based on the configuration, the app publishes data on the mentioned topic on the MessageBus.

.. note:: 

   The following use cases are suitable for a single node deployment, where the overhead of the VideoAnalytics (VA) service can be avoided:


   * The VA service is not required, when the VI service is configured with a UDF that does the classification. The VI service uses multiple UDFs for pre-processing, classification, and post-processing.
   * The VA service is not required, when the VI service uses the Gstreamer Video Analytics (GVA) elements. Pre-processing, classification, and post-processing (using the vappi gstreamer elements) can be done in the gstreamer pipeline. If required, you can configure post-processing by having multiple UDFs in the VI service.


Configuration
^^^^^^^^^^^^^

For configuration details refer to the following topics:


* `UDFs configuration <https://github.com/open-edge-insights/video-common/blob/master/udfs/README.md>`_
* `Etcd secrets configuration <https://github.com/open-edge-insights/eii-core/blob/master/Etcd_Secrets_Configuration.md>`_ and
* `MessageBus configuration <https://github.com/open-edge-insights/eii-core/blob/master/common/libs/ConfigMgr/README.md#interfaces>`_ respectively.
* `JSON schema <https://github.com/open-edge-insights/video-ingestion/blob/master/schema.json>`_

All the app module configurations are added to the distributed key-value store under the ``AppName`` env, as mentioned in the environment section of the app's service definition in the ``docker-compose``. If the ``AppName`` is ``VideoIngestion``\ , then the app's config is taken from the ``/VideoIngestion/config`` key via the Open EII Configuration Manager.

.. note:: 


   * The Developer mode-related overrides go in the ``docker-compose-dev.override.yml`` file.
   * For the ``jpeg`` encoding type, ``level`` is the quality from ``0 to 100``. A higher value means better quality.
   * For the ``png`` encoding type, ``level`` is the compression level from ``0 to 9``. A higher value means a smaller size and longer compression time.
   * Use the `JSON validator tool <https://www.jsonschemavalidator.net/>`_ for validating the app configuration for the schema.


Ingestor Config
~~~~~~~~~~~~~~~

Open EII supports the following type of ingestors:


* `OpenCV <https://opencv.org/>`_
* `GStreamer <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/gstreamer_ingestor_doc.md>`_
* `RealSense <https://www.intelrealsense.com/>`_

For more information on the Intel RealSense SDK, refer to `librealsense <https://github.com/IntelRealSense/librealsense>`_.

VideoIngestion Features
^^^^^^^^^^^^^^^^^^^^^^^

Refer the following to learn more about the VideoIngestion features and supported camera:


* `Generic server <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/generic_server_doc.md>`_
* `Gstreamer video analytics <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/gva_doc.md>`_
* `Run GVA elements in VI or VA <https://github.com/open-edge-insights/video-ingestion/blob/master/models/README.md>`_
* `GenICam GigE/USB3.0 camera support <#genicam-gige-or-usb3-cameras>`__
* `RTSP camera support <#rtsp-cameras>`__
* `USB v4l2 compatible camera support <#usb-v4l2-cameras>`__

Image Ingestion
~~~~~~~~~~~~~~~

The Image ingestion feature is responsible for ingesting the images coming from a directory into the Open EII stack for further processing. The ``OpenCV Ingestor`` supports image ingestion.
Image ingestion supports the following image formats:


* Jpg
* Jpeg
* Jpe
* Bmp
* Png

Refer the following snippet for configuring the `config.json <https://github.com/open-edge-insights/video-ingestion/blob/master/config.json>`_ file for enabling the image ingestion feature.


* 
  OpenCV Ingestor

  .. code-block:: javascript

     {
      //
       "ingestor": {
       "type": "opencv",
       "pipeline": "/app/img_dir/",
       "poll_interval": 2,
       "loop_video": true,
       "image_ingestion": true
     },
       "sw_trigger": {
           "init_state": "running"
       },
       "max_workers":1,
         //
     }

The description of the keys used in the ``config.json`` file is as follows:


* pipeline ??? Provides the path to the images directory that is volume mounted.
* poll_interval ??? Refers to the pull rate of image in seconds. Configure the ``poll_interval`` value as required.
* loop_video ??? Would loop through the images directory.
* 
  image_ingestion ??? Optional boolean key. It is required to enable the image ingestion feature.

  ..

     **Note:**


     * The image_ingestion key in the ``config.json`` needs to be set true for enabling the image ingestion feature.
     * Set the ``max_workers`` value to 1 as ``"max_workers":1`` in the ``config.json`` files for `VideoIngestion/config.json <https://github.com/open-edge-insights/video-ingestion/blob/master/config.json>`_ and `VideoIngestion/config.json <https://github.com/open-edge-insights/video-ingestion/blob/master/config.json>`_. This is to ensure that the images sequence is maintained. If the ``max_workers`` is set more than 1, then more likely the images would be out of order due to those many multiple threads operating asynchronously.
     * If the resolution of the image is greater than ``1920??1200``\ , then the image will be resized to ``width = 1920`` and ``height = 1200``. The image is resized to reduce the loading time of the image in the Web Visualizer and the native Visualizer.


Volume mount the image directory present on the host system. To do this, provide the absolute path of the images directory in the ``docker-compose file``.
Refer the following snippet of the ``ia_video_ingestion`` service to add the required changes in the `docker-compose.yml <https://github.com/open-edge-insights/video-ingestion/blob/master/docker-compose.yml>`_ file. After making the changes, ensure that the `docker-compose.yml <https://github.com/open-edge-insights/video-ingestion/blob/master/docker-compose.yml>`_ is executed before you build and run the services.

.. code-block:: yaml

   ia_video_ingestion:
     ...
     volume:
       - "vol_eii_socket:${SOCKET_DIR}"
       - "/var/tmp:/var/tmp"
       # Add volume
       # Please provide the absolute path to the images directory present in the host system for volume mounting the directory. Eg: -"home/directory_1/images_directory:/app/img_dir"
       - "<path_to_images_directory>:/app/img_dir"
       ...

UDF Configurations
~~~~~~~~~~~~~~~~~~

Ensure that you are using the appropriate UDF configuration for all the video and camera streams. If the UDF is not compatible with the video source, then you may not get the expected output in the Visualizer or the Web Visualizer screen. Use the ``dummy`` UDF, if you are not sure about the compatibility of the UDF and a video source. The dummy UDF will not do any analytics on the video, and it will not filter any of the video frames. You will see the video streamed by the camera, as it is displayed on the video output screen in the Visualizer or Web Visualizer.

..

   Refer the following configuration for the ``dummy`` UDF:


.. code-block:: javascript

       "udfs": [{
           "name": "dummy",
           "type": "python"
       }]

>

..

   Apply the same changes in the VideoAnalytics configuration if it is subscribing to VideoIngestion.


Updating Security Context of VideoIngestion Helm Charts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Complete the following steps to update Helm charts to enable K8s environment to access or detect Basler Camera, USB devices, and NCS2 Devices:


#. Open the ``EII_HOME_DIR/IEdgeInsights/VideoIngestion/helm/templates/video-ingestion.yaml`` file
#. 
   Update the following security context snippet

   .. code-block:: yml

           securityContext:
             privileged: true

   in the yaml file as:

   .. code-block:: yaml

       ...
       ...
       ...
           imagePullPolicy: {{ $global.Values.imagePullPolicy }}
           securityContext:
             privileged: true
           volumeMounts:
           - name: dev
             mountPath: /dev
       ...

#. 
   Rerun the ``builder.py`` to apply these changes to your deployment Helm charts.

Camera Configurations
~~~~~~~~~~~~~~~~~~~~~

The camera configurations for the VI module are as follows:


* 
  For a video file:


  * OpenCV Ingestor

  .. code-block:: javascript

     {
       "type": "opencv",
       "pipeline": "./test_videos/pcb_d2000.avi",
       "poll_interval": 0.2
       "loop_video": true
     }


  * Gstreamer Ingestor

  .. code-block:: javascript

     {
       "type": "gstreamer",
       "pipeline": "multifilesrc loop=TRUE stop-index=0 location=./test_videos/pcb_d2000.avi ! h264parse ! decodebin ! videoconvert ! video/x-raw,format=BGR ! appsink"
     }

For more information or configuration details for the ``multifilesrc`` element, refer to the `docs/multifilesrc_doc.md <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/multifilesrc_doc.md>`_.

GenICam GigE or USB3 Cameras
""""""""""""""""""""""""""""

For more information or configuration details for the GenICam GigE or the USB3 camera support, refer to the `GenICam GigE/USB3.0 Camera Support <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/generic_plugin_doc.md>`_.

Prerequisites for Working with the GenICam Compliant Cameras
############################################################

The following are the prerequisites for working with the GeniCam compliant cameras.

.. note:: 

   For other cameras such as RealSense, RSTP, and USB (v4l2 driver compliant) revert the changes that are mentioned in this section. Refer to the following snip of the ``ia_video_ingestion`` service, to add the required changes in the `docker-compose.yml <https://github.com/open-edge-insights/video-ingestion/blob/master/docker-compose.yml>`_ file of the respective ingestion service (including custom UDF services). After making the changes, before you build and run the services, ensure to run the `docker-compose.yml <https://github.com/open-edge-insights/video-ingestion/blob/master/docker-compose.yml>`_.



* For GenICam GigE cameras:

.. code-block:: yaml

   ia_video_ingestion:
     # Add root user
     user: root
     # Add network mode host
     network_mode: host
     # Please make sure that the above commands are not added under the environment section and also take care about the indentations in the compose file.
     ...
     environment:
     ...
       # Add HOST_IP to no_proxy and ETCD_HOST
       no_proxy: "${RTSP_CAMERA_IP},<HOST_IP>"
       ETCD_HOST: "<HOST_IP>"
     ...
     # Comment networks section as below as it will throw an error when network mode host is used.
     # networks:
       # - eii
     # Comment ports section as below
     # ports:
     # - 64013:64013

For GenIcam USB3.0 cameras:

.. code-block:: yaml

   ia_video_ingestion:
     # Add root user
     user: root
     ...
     environment:
       # Refer [GenICam GigE/USB3.0 Camera Support](https://github.com/open-edge-insights/video-ingestion/blob/master/docs/generic_plugin_doc.md) to install the respective camera SDK
       # Setting GENICAM value to the respective camera/GenTL producer which needs to be used
       GENICAM: "<CAMERA/GenTL>"
     ...

.. note:: 


   * If the GenICam cameras do not get initialized during the runtime, then on the host system, run the ``docker system prune`` command. After that, remove the GenICam specific semaphore files from the ``/dev/shm/`` path of the host system. The ``docker system prune`` command will remove all the stopped containers, networks that are not used (by at least one container), any dangling images, and build cache which could prevent the plugin from accessing the device.
   * If you get the ``Feature not writable`` message while working with the GenICam cameras, then reset the device using the camera software or using the reset property of the Generic Plugin. For more information, refer the `README <https://github.com/open-edge-insights/video-ingestion/blob/master/src-gst-gencamsrc/README.md>`_.
   * In the ``IPC`` mode, if the Ingestion service is running with the ``root`` privilege, then the ``ia_video_analytics`` and ``ia_visualizer`` service subscribing to it must also run with the ``root`` privileges.
   * In a multi-node scenario, replace :raw-html-m2r:`<HOST_IP>` in "no_proxy" with the leader node's IP address.
   * In the TCP mode of communication, the msgbus subscribers and clients of VideoIngestion are required to configure the ``Endpoint`` in the ``config.json`` file with the host IP and port under the ``Subscribers`` or ``Clients`` interfaces section.



* 
  Gstreamer Ingestor


  * 
    GenICam GigE/USB3.0 cameras

    .. code-block:: javascript

         {
           "type": "gstreamer",
           "pipeline": "gencamsrc serial=<DEVICE_SERIAL_NUMBER> pixel-format=<PIXEL_FORMAT> exposure-time=5000 exposure-mode=timed exposure-auto=off throughput-limit=300000000 ! videoconvert ! video/x-raw,format=BGR ! appsink"
         }

    ..

       **Note:**


       * Generic Plugin can work only with GenICam compliant cameras and only with gstreamer ingestor.
       * The above gstreamer pipeline was tested with Basler and IDS GigE cameras.
       * If ``serial`` is not provided then the first connected camera in the device list will be used.
       * If ``pixel-format`` is not provided then the default ``mono8`` pixel format will be used.
       * If ``width`` and ``height`` properies are not set then gencamsrc plugin will set the maximum resolution supported by the camera.
       * By default ``exposure-auto`` property is set to on. If the camera is not placed under sufficient light then with auto exposure, ``exposure-time`` can be set to very large value which will increase the time taken to grab frame. This can lead to ``No frame received error``. Hence it is recommended to manually set exposure as in the below sample pipline when the camera is not placed under good lighting conditions.
       * ``throughput-limit`` is the bandwidth limit for streaming out data from the camera(in bytes per second).


  * 
    Hardware trigger based ingestion with Gstreamer ingestor

    .. code-block:: javascript

         {
         "type": "gstreamer",
         "pipeline": "gencamsrc serial=<DEVICE_SERIAL_NUMBER> pixel-format=<PIXEL_FORMAT> trigger-selector=FrameStart trigger-source=Line1 trigger-activation=RisingEdge hw-trigger-timeout=100 acquisition-mode=singleframe exposure-time=5000 exposure-mode=timed exposure-auto=off throughput-limit=300000000 ! videoconvert ! video/x-raw,format=BGR ! appsink"
         }

    ..

       **Note:**


       * For PCB use case, use the ``width`` and ``height`` properties of gencamsrc to set the resolution to ``1920x1200`` and make sure it is pointing to the rotating PCB boards, as seen in the ``pcb_d2000.avi`` video file for the PCB filter to work.


    Refer the following example pipeline:

    .. code-block:: javascript

         {
           "type": "gstreamer",
           "pipeline": "gencamsrc serial=<DEVICE_SERIAL_NUMBER> pixel-format=ycbcr422_8 width=1920 height=1200 exposure-time=5000 exposure-mode=timed exposure-auto=off throughput-limit=300000000 ! videoconvert ! video/x-raw,format=BGR ! appsink"
         }

    Refer to the `docs/basler_doc.md <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/basler_doc.md>`_ for more information/configuration on Basler camera.

RTSP Cameras
~~~~~~~~~~~~

  Refer to the `docs/rtsp_doc.md <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/rtsp_doc.md>`_ for information/configuration on RTSP camera.**


* 
  OpenCV Ingestor

  .. code-block:: javascript

       {
         "type": "opencv",
         "pipeline": "rtsp://<USERNAME>:<PASSWORD>@<RTSP_CAMERA_IP>:<PORT>/<FEED>"
       }

  ..

     **Note**\ : OpenCV for RTSP will use software decoders.


* 
  Gstreamer Ingestor

  .. code-block:: javascript

      {
        "type": "gstreamer",
        "pipeline": "rtspsrc location=\"rtsp://<USERNAME>:<PASSWORD>@<RTSP_CAMERA_IP>:<PORT>/<FEED>\" latency=100 ! rtph264depay ! h264parse ! vaapih264dec ! vaapipostproc format=bgrx ! videoconvert ! video/x-raw,format=BGR ! appsink"
      }

  ..

     **Note:** The RTSP URI of the physical camera depends on how it is configured using the camera software. You can use VLC Network Stream to verify the RTSP URI to confirm the RTSP source.


* 
  For RTSP simulated camera using cvlc

* 
  OpenCV Ingestor

  .. code-block:: javascript

         {
           "type": "opencv",
           "pipeline": "rtsp://<SOURCE_IP>:<PORT>/<FEED>"
         }

* 
  Gstreamer Ingestor

  .. code-block:: javascript

         {
           "type": "gstreamer",
           "pipeline": "rtspsrc location=\"rtsp://<SOURCE_IP>:<PORT>/<FEED>\" latency=100 ! rtph264depay ! h264parse ! vaapih264dec ! vaapipostproc format=bgrx ! videoconvert ! video/x-raw,format=BGR ! appsink"
         }

    **Refer `docs/rtsp_doc.md <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/rtsp_doc.md>`_ for more information/configuration on rtsp simulated camera.**

USB v4l2 Cameras
~~~~~~~~~~~~~~~~

For information or configurations details on the USB cameras, refer to `docs/usb_doc.md <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/usb_doc.md>`_.


* 
  OpenCV Ingestor

  .. code-block:: javascript

         {
           "type": "opencv",
           "pipeline": "/dev/<DEVICE_VIDEO_NODE>"
         }

* 
  Gstreamer Ingestor

  .. code-block:: javascript

         {
           "type": "gstreamer",
           "pipeline": "v4l2src device=/dev/<DEVICE_VIDEO_NODE> ! video/x-raw,format=YUY2 ! videoconvert ! video/x-raw,format=BGR ! appsink"
         }

RealSense Depth Cameras
~~~~~~~~~~~~~~~~~~~~~~~


* 
  ``RealSense Ingestor``

  .. code-block:: javascript

          "ingestor": {
               "type": "realsense",
               "serial": "<DEVICE_SERIAL_NUMBER>",
               "framerate": <FRAMERATE>,
               "imu_on": true
           },

  ..

     **Note**


     * RealSense Ingestor was tested with the Intel RealSense Depth Camera D435i.
     * RealSense Ingestor does not support poll_interval. If required, use framerate to reduce the ingestion frames per second (FPS).
     * If the ``serial`` config is not provided then the first RealSense camera in the device list will be connected.
     * If the ``framerate`` config is not provided then the default framerate of ``30`` will be applied. Ensure that the framerate provided is compatible with both the color and depth sensor of the RealSense camera. With the D435i camera only framerate 6,15,30, and 60 is supported and tested.
     * The IMU stream will work only if the RealSense camera model supports the IMU feature. The default value for ``imu_on`` is set to ``false``.

