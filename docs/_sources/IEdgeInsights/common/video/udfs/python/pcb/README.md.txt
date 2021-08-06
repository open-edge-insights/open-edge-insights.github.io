**Note**

* This works well with only the  [pcb demo video file](../../VideoIngestion/test_videos/pcb_d2000.avi).
* The PCB filter will not send the frame unless it is key frame i.e. the pcb filter expects the key frame to have the pcb board appear correctly like the way it does in the `pcb_d2000.avi` video file. One might notice issues in frame getting published if PCB filter is used with a physical camera.
* Hence for camera usecase, proper tuning needs to be done to have the proper model built and used for inference.
* pcb_filter and  pcb_classifier udfs expects the frame resolution to be `1920x1200`.
