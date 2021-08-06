## Safety Gear Demo UDF

  Acceps the frame, detects safety gear such as safety helmet, safety jacket in
  the frame and any violations occuring.

  > **NOTE**: This works well with only the
  > [safe gear video file](../../VideoIngestion/test_videos/Safety_Full_Hat_and_Vest.avi).
  > For camera usecase, proper tuning needs to be done to have the proper model
  > built and used for inference.

   `UDF config`:

  ```json
  {
      "name": "safety_gear.safety_classifier",
      "type": "python",
      "device": "CPU",
      "model_xml": "./safety_gear/ref/frozen_inference_graph.xml",
      "model_bin": "./safety_gear/ref/frozen_inference_graph.bin"
  }
  ```

  ----
  **NOTE**:
  * The above config works for both "CPU" and "GPU" devices after setting
  appropriate `device` value. If the device in the above config is "HDDL" or
  "MYRIAD", please use the below config where the model_xml and model_bin files
  are different.Please set the "device" value appropriately based on the device
  used for inferencing.

        ```json
        {
            "name": "safety_gear.safety_classifier",
            "type": "python",
            "device": "HDDL",
            "model_xml": "./safety_gear/ref/frozen_inference_graph_fp16.xml",
            "model_bin": "./safety_gear/ref/frozen_inference_graph_fp16.bin"
        }
        ```

* The UDFs are placed under [CustomUDFs](../../CustomUdfs) for the sake of maintaining modularity of code structure so that all sample UDFs can be clubbed at one place.
User can move these directory to anwhere in the host system and compile the code. For example following steps can make this UDF to run from home directory.

    1. The [docker-compose.yml](./docker-compose.yml) file of the UDF can be altered to update the path for **build-context** and **Dockerfile** as shown below.

    From:

	```yml
	---snip---
    build:
        context: $PWD/../CustomUdfs/PySafetyGearAnalytics
        dockerfile: $PWD/../CustomUdfs/PySafetyGearAnalytics/Dockerfile
	---snip---
    ```
    To

	```yml
    -----snip-----
    build:
        context: $HOME/PySafetyGearAnalytics
        dockerfile: $HOME/PySafetyGearAnalytics/Dockerfile
	```

	2. Change the [video-streaming-all-udfs.yml](../../build/video-streaming-all-udfs.yml) file to define the service with full path instead of relative path from IEdgeInsights as follows:

    From:

    ```yml
    CustomUdfs/PySafetyGearAnalytics
    ```
    To

    ```yml
    /home/<user_name>/PySafetyGearAnalytics   <<< Make sure the full path should start with "/">>>
    ```
