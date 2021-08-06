## GVA Safety Gear Ingestion

This container is based out of VideoIngestion container. Since GVA elements are used for analytics there is no need for using any udf.

 * `Video File - Gstreamer ingestor with GVA elements`

      ```javascript
      {
        "type": "gstreamer",
        "pipeline": "multifilesrc loop=TRUE stop-index=0 location=./test_videos/Safety_Full_Hat_and_Vest.avi ! h264parse ! decodebin ! videoconvert ! video/x-raw,format=BGR ! gvadetect model=models/ref/frozen_inference_graph.xml ! appsink"
      }


 * `Generic Plugin - Gstreamer ingestor with GVA elements`

    ```javascript
     {
       "type": "gstreamer",
       "pipeline": "gencamsrc serial=<DEVICE_SERIAL_NUMBER> pixel-format=<PIXEL_FORMAT> ! vaapipostproc format=bgrx ! videoconvert !  video/x-raw,format=BGR ! gvadetect model=models/ref/frozen_inference_graph.xml ! appsink"
     }
    ```

    **Below is the sample pipeline for using monochrome/mono8 basler camera:**

    ```javascript
    {
     "type": "gstreamer",
     "pipeline" : "gencamsrc serial=<DEVICE_SERIAL_NUMBER> pixel-format=mono8 ! videoconvert ! video/x-raw,format=BGR ! gvadetect model=models/ref/frozen_inference_graph.xml ! appsink"
    }
    ```

 * `RTSP camera - Gstreamer ingestor with GVA elements`

      ```javascript
      {
        "type": "gstreamer",
        "pipeline": "rtspsrc location=\"rtsp://<USERNAME>:<PASSWORD>@<RTSP_CAMERA_IP>:<PORT>/<FEED>\" latency=100 ! rtph264depay ! h264parse ! vaapih264dec ! vaapipostproc format=bgrx ! gvadetect model=models/ref/frozen_inference_graph.xml ! videoconvert ! video/x-raw,format=BGR ! appsink"
      }
      ```

 * `USB camera - Gstreamer ingestor with GVA elements`

      ```javascript
      {
        "type": "gstreamer",
        "pipeline": "v4l2src ! decodebin ! videoconvert ! video/x-raw,format=BGR ! gvadetect model=models/ref/frozen_inference_graph.xml ! appsink"
      }
      ```

 * `RTSP simulated - Gstreamer ingestor with GVA elements`

      ```javascript
      {
        "type": "gstreamer",
        "pipeline": "rtspsrc location=\"rtsp://<SOURCE_IP>:<PORT>/<FEED>\" latency=100 ! rtph264depay ! h264parse ! vaapih264dec ! vaapipostproc format=bgrx ! gvadetect model=models/ref/frozen_inference_graph.xml ! videoconvert ! video/x-raw,format=BGR ! appsink"
      }
      ```

**Refer [GVA-README](../../VideoIngestion/docs/gva_doc.md) for more information on GVA.**

**Refer [VideoIngestion-README](../../VideoIngestion/README.md) for more information on ingestor configurations.**
