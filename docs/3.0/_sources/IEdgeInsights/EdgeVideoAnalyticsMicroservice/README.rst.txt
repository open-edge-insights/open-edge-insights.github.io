
Contents
========


* `Contents <#contents>`__

  * `Edge Video Analytics Microservice <#edge-video-analytics-microservice>`__

    * `Build the base image <#build-the-base-image>`__
    * `Run the base image <#run-the-base-image>`__
    * `Run EVAM in OEI mode <#run-evam-in-oei-mode>`__

Edge Video Analytics Microservice
---------------------------------

This repository contains the source code for Edge Video Analytics Microservice (EVAM) used for the `Video Analytics Use Case <https://www.intel.com/content/www/us/en/developer/articles/technical/video-analytics-service.html>`_. For information on how to build the use case, refer to the `Get Started <https://www.intel.com/content/www/us/en/developer/articles/technical/video-analytics-service.html#inpage-nav-3>`_ guide.

Build the base image
^^^^^^^^^^^^^^^^^^^^

Complete the following steps to build the base image:


#. 
   Run the following command:

   .. code-block:: sh

        `docker-compose -f  docker-compose-build.yml  build`

#. 
   If required, download the pre-built container image for Edge Video Analytics Microservice from `Docker Hub <https://hub.docker.com/r/intel/edge_video_analytics_microservice>`_.

Run the base image
^^^^^^^^^^^^^^^^^^

Complete the following steps to run the base image:


#. Clone this `repo <https://github.com/intel/edge-video-analytics-microservice>`_.
#. 
   Run the following command to make the following files executable:

   .. code-block:: sh

       chmod +x tools/model_downloader/model_downloader.sh docker/run.sh

#. 
   Download the required models. From the cloned repo, run the following command:

   .. code-block:: sh

      ./tools/model_downloader/model_downloader.sh  --model-list <Path to model-list.yml>

#. 
   After downloading the models, you will have the ``models`` directory in the base folder. Refer to the following:

   .. code-block:: json

       models/
       ├── action_recognition
       ├── audio_detection
       ├── emotion_recognition
       ├── face_detection_retail
       ├── object_classification
       └── object_detection

#. 
   Add the following lines in the ``docker-compose.yml`` environment if you are behind a proxy.

   .. code-block:: sh

       - HTTP_PROXY=<IP>:<Port>/
       - HTTPS_PROXY=<IP>:<Port>/
       - NO_PROXY=localhost,127.0.0.1

#. 
   Run the ``sudo docker-compose up`` command.

.. note::  For more details, refer to `Run the Edge Video Analytics Microservice <https://www.intel.com/content/www/us/en/developer/articles/technical/video-analytics-service.html#inpage-nav-3-1>`_.


Run EVAM in OEI mode
^^^^^^^^^^^^^^^^^^^^

To run EVAM in the OEI mode, refer to the `README <https://github.com/open-edge-insights/edge-video-analytics-microservice/blob/master/eii/README.md>`_.
