
GVA Safety Gear Ingestion
-------------------------

This container is based out of VideoIngestion container. Since GVA elements are used for analytics there is no need for using any udf.


* 
  ``Video File - Gstreamer ingestor with GVA elements``

  .. code-block::

     ```javascript
     {
       "type": "gstreamer",
       "pipeline": "multifilesrc loop=TRUE stop-index=0 location=./test_videos/Safety_Full_Hat_and_Vest.avi ! h264parse ! decodebin ! videoconvert ! video/x-raw,format=BGR ! gvadetect model=models/ref/frozen_inference_graph.xml ! appsink"
     }
     ```

* 
  ``Generic Plugin - Gstreamer ingestor with GVA elements``

  .. code-block:: javascript

        {
          "type": "gstreamer",
          "pipeline": "gencamsrc serial=<DEVICE_SERIAL_NUMBER> pixel-format=<PIXEL_FORMAT> exposure-time=5000 exposure-mode=timed exposure-auto=off throughput-limit=300000000 ! vaapipostproc format=bgrx ! videoconvert !  video/x-raw,format=BGR ! gvadetect model=models/ref/frozen_inference_graph.xml ! appsink"
        }

    **Below is the sample pipeline for using monochrome/mono8 basler camera:**

  .. code-block:: javascript

       {
        "type": "gstreamer",
        "pipeline" : "gencamsrc serial=<DEVICE_SERIAL_NUMBER> pixel-format=mono8 exposure-time=5000 exposure-mode=timed exposure-auto=off throughput-limit=300000000 ! videoconvert ! video/x-raw,format=BGR ! gvadetect model=models/ref/frozen_inference_graph.xml ! appsink"
       }

* 
  ``RTSP camera - Gstreamer ingestor with GVA elements``

  .. code-block::

     ```javascript
     {
       "type": "gstreamer",
       "pipeline": "rtspsrc location=\"rtsp://<USERNAME>:<PASSWORD>@<RTSP_CAMERA_IP>:<PORT>/<FEED>\" latency=100 ! rtph264depay ! h264parse ! vaapih264dec ! vaapipostproc format=bgrx ! gvadetect model=models/ref/frozen_inference_graph.xml ! videoconvert ! video/x-raw,format=BGR ! appsink"
     }
     ```

* 
  ``USB camera - Gstreamer ingestor with GVA elements``

  .. code-block::

     ```javascript
     {
       "type": "gstreamer",
       "pipeline": "v4l2src ! decodebin ! videoconvert ! video/x-raw,format=BGR ! gvadetect model=models/ref/frozen_inference_graph.xml ! appsink"
     }
     ```

* 
  ``RTSP simulated - Gstreamer ingestor with GVA elements``

  .. code-block::

     ```javascript
     {
       "type": "gstreamer",
       "pipeline": "rtspsrc location=\"rtsp://<SOURCE_IP>:<PORT>/<FEED>\" latency=100 ! rtph264depay ! h264parse ! vaapih264dec ! vaapipostproc format=bgrx ! gvadetect model=models/ref/frozen_inference_graph.xml ! videoconvert ! video/x-raw,format=BGR ! appsink"
     }
     ```

**Refer `GVA-README <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/gva_doc.md>`_ for more information on GVA.**

**Refer `VideoIngestion-README <https://github.com/open-edge-insights/video-ingestion/blob/master/README.md>`_ for more information on ingestor configurations.**
