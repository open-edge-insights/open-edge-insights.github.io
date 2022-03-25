
Edge Video Analytics Microservice
=================================

This repository contains the source code for Edge Video Analytics Microservice used for `Video Analytics Use Case <https://www.intel.com/content/www/us/en/developer/articles/technical/video-analytics-service.html>`_. For information on how to build the use case, please refer to the `Get Started <https://www.intel.com/content/www/us/en/developer/articles/technical/video-analytics-service.html#inpage-nav-3>`_ guide.

Building the image
------------------


* 
  To build the base image, run the following command
     ``docker-compose -f  docker-compose-build.yml  build``

* 
  You can download the pre-built container image for Edge Video Analytics Microservice from `Docker Hub <https://hub.docker.com/r/intel/edge_video_analytics_microservice>`_

Running the image
-----------------


* 
  Clone this repo.

* 
  Make the following files executable

    ``chmod +x tools/model_downloader/model_downloader.sh docker/run.sh``

* 
  Download the required models. From the cloned repo, run the following commands. 

    ``./tools/model_downloader/model_downloader.sh  --model-list <Path to model-list.yml>``

* 
  After downloading the models, you will have models directory in the base folder.

.. code-block::

   models/
   ├── action_recognition
   ├── audio_detection
   ├── emotion_recognition
   ├── face_detection_retail
   ├── object_classification
   └── object_detection


* Add the following lines in the docker-compose.yml environment if you are behind a proxy.

.. code-block::

       - HTTP_PROXY=<IP>:<Port>/
       - HTTPS_PROXY=<IP>:<Port>/
       - NO_PROXY=localhost,127.0.0.1


* 
  Run ``sudo docker-compose up``.

* 
  Please refer to `Run the Edge Video Analytics Microservice <https://www.intel.com/content/www/us/en/developer/articles/technical/video-analytics-service.html#inpage-nav-3-1>`_ for more details.

Running in EII mode
-------------------


* Please refer `README <https://github.com/open-edge-insights/edge-video-analytics-microservice/blob/master/eii/README.md>`_ for running Edge Video Analytics Microservice in EII mode
