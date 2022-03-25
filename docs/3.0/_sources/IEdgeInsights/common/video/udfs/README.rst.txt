.. role:: raw-html-m2r(raw)
   :format: html


Contents
========


* `Contents <#contents>`__

  * `OEI Sample UDFs <#oei-sample-udfs>`__
  * `User Defined Function (UDF) <#user-defined-function-udf>`__

    * `UDF Configuration <#udf-configuration>`__
    * `UDF Writing Guide <#udf-writing-guide>`__
    * `Sample UDFs <#sample-udfs>`__

      * `Native UDFs <#native-udfs>`__
      * `Python UDFs <#python-udfs>`__

    * `Construction of Metadata in UDF <#construction-of-metadata-in-udf>`__
    * `Chaining of UDFs <#chaining-of-udfs>`__

      * `Combination of UDFs with ingestors <#combination-of-udfs-with-ingestors>`__

OEI Sample UDFs
---------------

Open Edge Insights (OEI) supports loading and executing of native(c++) and python UDFs. In here,
one can find the sample native and python UDFs(User Defined Functions) to be used with OEI components
like VideoIngestion and VideoAnalytics. The UDFs can modify the frame, drop the frame and generate meta-data from the frame.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as OEI. This is due to the product name change of EII as OEI.


User Defined Function (UDF)
---------------------------

An UDF is a chunk of user code that acts as a filter, preprocessor, or classifier for a given data input coming from the OEI. The User Defined Function (UDF) Loader Library provides a common API for loading C++ and Python UDFs.

The library itself is written in C++ and provides an abstraction layer for loading and calling UDFs. Additionally, the library defines a common interface inheritable by all UDFs (whether written in C++ or Python).

The overall block diagram for the library is shown in the following figure.


.. image:: https://raw.githubusercontent.com/open-edge-insights/open-edge-insights.github.io/main/docs/_images/fig_9_4.png
   :target: https://raw.githubusercontent.com/open-edge-insights/open-edge-insights.github.io/main/docs/_images/fig_9_4.png
   :alt: User-Defined Function Loader Library Block Design


In this case, the VideoIngestion component is also able to execute the video data classifier algorithm by including the classifier UDF into the VideoIngestion configuration. By defining the Classifier UDF in the VideoIngestion component, the VideoAnalytics component become optional

UDF Configuration
^^^^^^^^^^^^^^^^^

Below is the JSON schema for UDF json object configuration:

.. code-block:: javascript

   {
     "type": "object",
     "additionalProperties": false,
     "properties": {
       "max_jobs": {
         "description": "Number of queued UDF jobs",
         "type": "integer",
         "default": 20
       },
       "max_workers": {
         "description": "Number of threads acting on queued jobs",
         "type": "integer",
         "default": 4
       },
       "udfs": {
         "description": "Array of UDF config objects",
         "type": "array",
         "items": [
           {
             "description": "UDF config object",
             "type": "object",
             "properties": {
               "type": {
                 "description": "UDF type",
                 "type": "string",
                 "enum": [
                   "native",
                   "python",
                   "raw_native"
                 ]
               },
               "name": {
                 "description": "Unique UDF name",
                 "type": "string"
               },
               "device": {
                 "description": "Device on which inference occurs",
                 "type": "string",
                 "enum": [
                   "CPU",
                   "GPU",
                   "HDDL",
                   "MYRIAD"
                 ]
               }
             },
             "additionalProperties": true,
             "required": [
               "type",
               "name"
             ]
           }
         ]
       }
     }
   }

One can use `JSON validator tool <https://www.jsonschemavalidator.net/>`_ for
validating the UDF configuration object against the above schema.

Example UDF configuration:

.. code-block:: javascript

   {
     "max_jobs": 20,
     "max_workers": 4,
     "udfs": [ {
         "type": "native",
         "name": "dummy"
       },
       {
         "type": "python",
         "name": "pcb.pcb_filter"
       }]
   }

UDF Writing Guide
^^^^^^^^^^^^^^^^^

User can refer to `UDF Writing HOW-TO GUIDE <https://github.com/open-edge-insights/video-common/blob/master/udfs/HOWTO_GUIDE_FOR_WRITING_UDF.md>`_ for an detailed explanation of process to write an custom UDF.

Sample UDFs
^^^^^^^^^^^

.. note::  The UDF config of these go as json objects in the ``udfs`` key in
   the overall UDF configuration object


Native UDFs
~~~~~~~~~~~


* 
  **Dummy UDF**

  Accepts the frame and forwards the same without doing any processing. It's a
  do-nothing UDF.

  ``UDF config``\ :

  .. code-block:: javascript

     {
         "name": "dummy",
         "type": "native"
     }

* 
  **Raw Dummy UDF**

  Accepts the Frame object and forwards the same without doing any processing. It's a
  do-nothing UDF for working with multi-frame support.

  ``UDF config``\ :

  .. code-block:: javascript

     {
         "name": "raw_dummy",
         "type": "raw_native"
     }

**Note:** ``raw_native`` udf type has been added to support multi-frame ingestion support.RealSense usecase requires multi-frame ingestion for color and depth frames.


* 
  **Resize UDF**

  Accepts the frame, resizes it based on the ``width`` and ``height`` params.

  ``UDF config``\ :

  .. code-block:: javascript

     {
         "name": "resize",
         "type": "native",
         "width": 600,
         "height": 600
     }

* 
  **FPS UDF**

  FPS udf can be used to measure the total number of frames received every second. It can be used in VideoIngestion and VideoAnalytics
  application by adding the below configuration in the udf configuration. It can also be chained with other udfs in which case the FPS result will be affected depending on the other udfs used.

  ``UDF config``\ :

  .. code-block:: javascript

     {
         "name": "fps",
         "type": "native"
     }

  ``Config for chaining fps udf with other udfs``\ :

  .. code-block:: javascript

     "udfs": [{
          "name": "dummy",
          "type": "native"
      },
      {
          "name": "fps",
          "type": "native"
      }]

  ..

     **Note** The fps results will be logged in ``DEBUG`` LOG_LEVEL, added to the metadata with the AppName as the key and will be
     displayed in the visualizer.


* 
  **Sample Realsense UDF**

  Accepts the color and depth frame, converts to rs2::frame type by using rs2::software_device simulation, enables a color filter on the depth frame using rs2::colorizer.

  ``UDF config``\ :

  .. code-block:: javascript

     {
         "name": "sample_realsense",
         "type": "raw_native",
     }

  ----

Python UDFs
~~~~~~~~~~~

.. note::  Additional properties/keys other than ``name`` and ``type`` in the UDF
   config are the parameters of the python UDF constructor



* 
  **Dummy UDF**

  Accepts the frame and forwards the same without doing any processing. It's a
  do-nothing UDF.

  ``UDF config``\ :

  .. code-block:: javascript

     {
         "name": "dummy",
         "type": "python"
     }

* 
  **Multi Frame Dummy UDF**

  Accepts the Frame object which is a list of frames and forwards the same without doing any processing. It's a
  do-nothing UDF for working with multi-frame support.

  ``UDF config``\ :

  .. code-block:: javascript

     {
         "name": "multi_frame_dummy",
         "type": "python"
     }

**Note:** When multi frame ingestion is used then the Frame object is a list of numpy frames else it is a single numpy frame. The udf type remains ``python`` for multi frame ingestion and single frame ingestion.


* 
  **Jupyter Connector UDF**

  Accepts the frame and publishes it to the OEI JupyterNotebook service which processes the frame and publishes it back to the jupyter_connector UDF.

  ``UDF config``\ :

  .. code-block:: javascript

     {
         "name": "jupyter_connector",
         "type": "python"
     }

* 
  **PCB Filter UDF**

  Accepts the frame and based on if ``pcb`` board is at the center in the frame or not,
  it forwards or drops the frame. It basically sends out only the key frames forward
  for further processing and not all frames it receives.

  ``UDF config``\ :

  .. code-block:: javascript

     {
         "name": "pcb.pcb_filter",
         "type": "python",
         "training_mode": "false",
         "scale_ratio": 4,
         "n_total_px": 300000,
         "n_left_px": 1000,
         "n_right_px": 1000
     }

  Refer `python/pcb/README.md <https://github.com/open-edge-insights/video-common/blob/master/udfs/python/pcb/README.md>`_ for more information.

* 
  **PCB Classifier UDF**

  Accepts the frame, uses openvino inference engine APIs to determine whether it's
  a ``good`` pcb with no defects or ``bad`` pcb with defects. Metadata associated with
  the frame is populated accordingly.

  ``UDF config``\ :

  .. code-block:: javascript

     {
         "name": "pcb.pcb_classifier",
         "type": "python",
         "device": "CPU",
         "ref_img": "common/video/udfs/python/pcb/ref/ref.png",
         "ref_config_roi": "common/video/udfs/python/pcb/ref/roi_2.json",
         "model_xml": "common/video/udfs/python/pcb/ref/model_2.xml",
         "model_bin": "common/video/udfs/python/pcb/ref/model_2.bin"
     }

  Refer `python/pcb/README.md <https://github.com/open-edge-insights/video-common/blob/master/udfs/python/pcb/README.md>`_ for more information.

  ..

     **Note:** The above config works for both "CPU" and "GPU" devices after setting appropriate ``device`` value.
     Please set the "device" value appropriately based on the device used for inferencing.


* 
  **Sample ONNX UDF**

  This UDF mainly demonstrates the model deployment on edge devices via AzureBridge service only.

  Please follow the below steps:


  * Configure the sample ONNX UDF by following `Sample ONNX UDF configuration guide <https://github.com/open-edge-insights/eii-azure-bridge/blob/master/README.md#sample-eii-onnx-8>`_
  * Follow `Single-Node Azure IOT Edge Deployment <https://github.com/open-edge-insights/eii-azure-bridge/blob/master/README.md#single-node-azure-iot-edge-deployment>`_ to deploy the required modules

  For more details on AzureBridge setup, please refer `AzureBridge README.md <https://github.com/open-edge-insights/eii-azure-bridge/blob/master/README.md>`_

----

Construction of Metadata in UDF
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If OEI Visualizer/WebVisualizer clients are used for visualizing the classified frames, then please follow the metadata guidelines mentioned in **\ ``Metadata Structure``\ ** in `Visualizer <https://github.com/open-edge-insights/video-native-visualizer/blob/master/README.md>`_ / `WebVisualizer <https://github.com/open-edge-insights/video-web-visualizer/blob/master/README.md>`_ README respectively.

**Note**\ : User has to make sure that the data with in meta data should be of type list, tuple, dict or primitive data types (int, float, string or bool). Also, data with in list, tuple, dict
must contain only primitive data types.
Eg: Any data is of type "numpy.float" or "numpy.int" should be type-casted to float and int respectively.

Chaining of UDFs
^^^^^^^^^^^^^^^^

One can chain multiple native/python UDFs in the ``udfs`` key. The way chaining
works here is the output of the UDF listed first would send the modified frame
and metadata to the subsequent UDF and so on. One such classic example is having
``pcb.pcb_filter`` and ``pcb.pcb_classifier`` in VideoIngestion service config to
do both the pre-processing and the classification logic without the need of
VideoAnalytics service.

Combination of UDFs with ingestors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1

   * - Ingestor
     - Chaining UDFs for pcb demo usecase
     - Chaining UDFs for worker safety gear usecase
   * - opencv/gstreamer
     - :raw-html-m2r:`<br>`\ :raw-html-m2r:`<br>`\ Combination of resize (native), pcb filter (python) and\ :raw-html-m2r:`<br>`\ pcb classifier (python) can be used as :raw-html-m2r:`<br>`\ per the need.
     - :raw-html-m2r:`<br>`\ :raw-html-m2r:`<br>`\ :raw-html-m2r:`<br>`\ Combination of resize (native) and worker safety gear classifier (native/python) :raw-html-m2r:`<br>` can be used as per the need.
   * - gstreamer with GVA(Gstreamer Video Analytics) elements
     - Not Applicable
     - :raw-html-m2r:`<br>`\ Any post-processing UDFs can be used as all\ :raw-html-m2r:`<br>`\ the pre-processing and classification is :raw-html-m2r:`<br>`\ happening in the gstreamer pipeline itself


.. note:: 
   Dummy UDF can also be used for above use cases for testing chaining UDFs
   feature but as such there is no value add as it's a do-nothing UDF.
   In DEV Mode python udfs changes can be tested by restarting containers, no need to rebuild.

