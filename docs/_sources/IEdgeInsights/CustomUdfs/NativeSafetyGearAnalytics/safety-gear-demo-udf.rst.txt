
Safety Gear Demo UDF
--------------------

This udf accepts the frames, detects safety gear such as safety helmet, safety jacket in
the frame and any violations occuring.

.. note:: \ : This works well with only the
   `safe gear video file <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeSafetyGearIngestion/Safety_Full_Hat_and_Vest.avi>`_.
   For camera usecase, proper tuning needs to be done to have the proper model
   built and used for inference.


``UDF config``\ :

.. code-block:: json

   {
   "name": "safety_gear_demo",
   "type": "native",
   "device": "CPU",
   "model_xml": "./safety_gear_demo/ref/frozen_inference_graph.xml",
   "model_bin": "./safety_gear_demo/ref/frozen_inference_graph.bin"
   }

----

**NOTE**\ :


* 
  The above config works for both "CPU" and "GPU" devices after setting
  appropriate ``device`` value. If the device in the above config is "HDDL" or
  "MYRIAD", please use the below config where the model_xml and model_bin
  files are different. Please set the "device" value appropriately based on
  the device used for inferencing.

  .. code-block:: json

       {
       "name": "safety_gear_demo",
       "type": "native",
       "device": "HDDL",
       "model_xml": "./safety_gear_demo/ref/frozen_inference_graph_fp16.xml",
       "model_bin": "./safety_gear_demo/ref/frozen_inference_graph_fp16.bin"
       }

* 
  The UDFs are placed under `CustomUDFs <https://github.com/open-edge-insights/video-custom-udfs/blob/master/>`_ for the sake of maintaining modularity of code structure so that all sample UDFs can be clubbed at one place.
  User can move these directory to anwhere in the host system and compile the code. For example following steps can make this UDF to run from home directory.


  #. 
     The `docker-compose.yml <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeSafetyGearAnalytics/docker-compose.yml>`_ file of the UDF can be altered to update the path for **build-context** and **Dockerfile** as shown below.

     From:

  .. code-block:: yml

     ---snip---
       build:
           context: $PWD/../CustomUdfs/NativeSafetyGearAnalytics
           dockerfile: $PWD/../CustomUdfs/NativeSafetyGearAnalytics/Dockerfile
     ---snip---

    To

  .. code-block:: yml

       -----snip-----
       build:
           context: $HOME/NativeSafetyGearAnalytics
           dockerfile: $HOME/NativeSafetyGearAnalytics/Dockerfile


  #. 
     Change the `video-streaming-all-udfs.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/video-streaming-all-udfs.yml>`_ file to define the service with full path instead of relative path from IEdgeInsights as follows:

     From:

     .. code-block:: yml

        CustomUdfs/NativeSafetyGearAnalytics

     To

     .. code-block:: yml

        /home/<user_name>/NativeSafetyGearAnalytics   <<< Make sure the full path should start with "/">>>
