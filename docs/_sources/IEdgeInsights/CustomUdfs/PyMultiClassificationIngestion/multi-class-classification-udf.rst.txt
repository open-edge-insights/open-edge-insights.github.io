
Multi-Class Classification UDF
------------------------------

This UDF accepts the frame, and classifies object in frame into different cataegories. Additionally it shows probability of other classes too with its confidence value. This classification doesn't need any specialized image preprocessing UDF.

.. note:: \ : For a successful execution user can stream a sample video file
   `classification_vid.avi <https://github.com/open-edge-insights/video-custom-udfs/blob/master/PyMultiClassificationIngestion/classification_vid.avi>`_.
   For using camera classification will work correctly if the model has been trained for the object earlier. It is currently trained with some subset of imageNet database. The labels for which it is trained already trained can be found in following `label file <https://github.com/open-edge-insights/eii-core/blob/master/sample_classification/ref/squeezenet1.1.labels>`_


   ``UDF config``\ :

.. code-block:: javascript

     {
         "name": "sample_classification.multi_class_classifier",
         "type": "python",
         "device": "CPU",
         "labels_file_path": "./sample_classification/ref/squeezenet1.1.labels",
         "model_xml": "./sample_classification/ref/squeezenet1.1_FP32.xml",
         "model_bin": "./sample_classification/ref/squeezenet1.1_FP32.bin"
     }

----

  **NOTE**\ :
  The above config works for both "CPU" and "GPU" devices after setting appropriate ``device`` value.
  If the device in the above config is "HDDL" or "MYRIAD", please use the below config where the
  model_xml and model_bin files are different and should be of FP16 based.
  Please set the "device" value appropriately based on the device used for inferencing.

.. code-block:: javascript

     {
         "name": "sample_classification.multi_class_classifier",
         "type": "python",
         "device": "HDDL",
         "labels_file_path": "./sample_classification/ref/squeezenet1.1.labels",
         "model_xml": "./sample_classification/ref/squeezenet1.1_FP16.xml",
         "model_bin": "./sample_classification/ref/squeezenet1.1_FP16.bin",
     }
