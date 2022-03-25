
Contents
========


* `Contents <#contents>`__

  * `Introduction <#introduction>`__
  * `UDF Container Directory Layout <#udf-container-directory-layout>`__
  * `Deploy Process <#deploy-process>`__
  * `Sample UDFs Directory <#sample-udfs-directory>`__
  * `GVASafetyGearIngestion <#gvasafetygearingestion>`__
  * `NativeOneAPIIngestion <#nativeoneapiingestion>`__
  * `NativePclIngestion <#nativepclingestion>`__

Introduction
------------

This document describes the new approach of creating User Defined Functions (UDFs) or algorithms, and how to use UDFs inside the Open Edge Insights (OEI) framework. This document describes the workflow of a custom UDF. For more information on the coding aspects (callbacks) of the UDFs, refer to the `UDF Writing Guide <https://github.com/open-edge-insights/video-common/blob/master/udfs/HOWTO_GUIDE_FOR_WRITING_UDF.md>`_.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as OEI. This is due to the product name change of EII as OEI.


For information on the usage of the OpenVINO Inference Engine APIs, refer to the following:


* `OpenVINO-Documentation <https://docs.openvino.ai/2021.4/documentation.html>`_
* `OpenVINO-API-Reference <https://docs.openvino.ai/2021.4/api/api_reference.html>`_.

Select the suitable APIs depending on your requirement. For example, to start inference, select either the synchronous or the asynchronous method of the ``InferRequest`` class. The following sample video custom UDFs leverages the OpenVINO Inference Engine APIs:

..

   * `NativeSafetyGearAnalytics <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeSafetyGearAnalytics>`_
   * `PySafetyGearAnalytics <https://github.com/open-edge-insights/video-custom-udfs/blob/master/PySafetyGearAnalytics>`_
   * `PyMultiClassificationIngestion <https://github.com/open-edge-insights/video-custom-udfs/blob/master/PyMultiClassificationIngestion>`_


Create UDFs in the `udfs-path <https://github.com/open-edge-insights/video-common/tree/master/udfs>`_ of the OEI build environment. This allows UDFs to be compiled in the VI (Video Ingestion) & the VA (Video Analytics) containers.

In addition to the approach mentioned earlier, you can build each UDF as an independent container, based out of the VI or the VA container image. This provides the following benefits:


* With an increased number of sample UDFs, VI and VA need not grow large because of the bloated Algo artifacts.
* Any update to the UDF's Algo or its logic will compile and build only the intended UDF specific code, instead of rebuilding every UDFs.
* Any change to base containers because of some unrelated changes in the Ubuntu packages triggers a complete container build again. This adds to the build time of a UDF.
* Every UDF can be versioned independently. The UDFs are represented by their own containers.
* A reduction in the size of the UDF container will reduce the network overhaul while transferring the images from REGISTRY to the target system.

As per this approach, a UDF or a chain of UDFs should be compiled and run as a separate OEI container. A video streaming pipeline contains two important components—video ingestion and video analytics. In OEI, you add UDFs as a pre-processing, post-processing, or analytics algo. Hence, these UDF containers need to be inherited from the VI and VA container.

UDF Container Directory Layout
------------------------------


#. A native C++ and Python UDF container source base is shown as follows:
   ..

      **Note:** Based on a use case, the native C++ and Python container source base can look different.


.. code-block:: bash

   NativeSafetyGearAnalytics
   ├── config.json
   ├── docker-compose.yml
   ├── Dockerfile
   └── safety_gear_demo
       ├── CMakeLists.txt
       ├── ref
       │   ├── frozen_inference_graph.bin
       │   ├── frozen_inference_graph_fp16.bin
       │   ├── frozen_inference_graph_fp16.xml
       │   └── frozen_inference_graph.xml
       ├── safety_gear_demo.cpp
       └── safety_gear_demo.h

The Python containers typically look as follows:

.. code-block:: bash

   PyMultiClassificationIngestion
   ├── config.json
   ├── docker-compose.yml
   ├── Dockerfile
   └── sample_classification
       ├── __init__.py
       ├── multi_class_classifier.py
       └── ref
           ├── squeezenet1.1_FP16.bin
           ├── squeezenet1.1_FP16.xml
           ├── squeezenet1.1_FP32.bin
           ├── squeezenet1.1_FP32.xml
           └── squeezenet1.1.labels

The top-level directories **"NativeSafetyGearAnalytics"** and **"PyMultiClassificationIngestion"** hosts the container's build ingredients. The Algo or pre-processing logics are placed under the examples **"safety_gear_demo"** and **sample_classification** to showcase a grouping of logically related entities. This is not a mandatory directory layout.


* ## *config.json*

This file defines UDF specific configuration and other generic configs such as queue-depth, number-of-worker-thread etc. These generic configs can be added to overwrite any default of setting of VI and VA container.

For more information about schema of defining the configs and its permissible values, refer to the `UDF-README <https://github.com/open-edge-insights/video-common/blob/master/udfs/README.md>`_ file.

For ingestor related configs, refer to the `VideoIngestion-README <https://github.com/open-edge-insights/video-ingestion/blob/master/README.md>`_.

An example snippet of the configuration is as follows:

.. code-block:: bash

       {
           "encoding": {
               "level": 95,
               "type": "jpeg"
           },
           "max_jobs": 20,
           "max_workers": 4,
           "queue_size": 10,
           "udfs": [
               {
                   "name": "safety_gear_demo",
                   "type": "native",
                   "device": "CPU",
                   "model_xml": "./safety_gear_demo/ref/frozen_inference_graph.xml",
                   "model_bin": "./safety_gear_demo/ref/frozen_inference_graph.bin"
               }
           ]
       }


* ## *Dockerfile*

This file defines the container build process. All the build time and runtime artifacts should be copied to the container. For the native (C++) UDF, describe the destination path to copy the code to the native UDF and compilation instruction of the same. For Python, copy the UDF defining artifacts to a proper destination location. Some code comments are provided for describing the important key values.

An example of **Dockerfile** for C++ based UDF is as follows:

.. code-block:: dockerfile

   ARG EII_VERSION                                          <<<<This is to use latest version of VI & VA automatically instead of hardcoding a version
   ARG DOCKER_REGISTRY
   ARG ARTIFACTS="/artifacts"
   FROM ia_video_common:$EII_VERSION as video_common
   FROM ia_openvino_base:$EII_VERSION as openvino_base
   FROM ${DOCKER_REGISTRY}openedgeinsights/ia_video_analytics:$EII_VERSION as video_analytics
   FROM ia_eiibase:$EII_VERSION as builder
   LABEL description="C++ based Safety Gear UDF Image"
   WORKDIR /app
   ARG ARTIFACTS
   RUN mkdir $ARTIFACTS \
             $ARTIFACTS/safety_gear_demo \
             $ARTIFACTS/lib
   ARG CMAKE_INSTALL_PREFIX
   COPY --from=video_common ${CMAKE_INSTALL_PREFIX}/include ${CMAKE_INSTALL_PREFIX}/include
   COPY --from=video_common ${CMAKE_INSTALL_PREFIX}/lib ${CMAKE_INSTALL_PREFIX}/lib
   COPY --from=openvino_base /opt/intel /opt/intel

   # Copy more than one UDFs here
   # Both C++ & Python are allowed in a container.
   COPY ./safety_gear_demo/ ./safety_gear_demo
   RUN cp -r ./safety_gear_demo $ARTIFACTS/safety_gear_demo

   # Build native UDF samples
   RUN /bin/bash -c "source /opt/intel/openvino/bin/setupvars.sh && \
       cd ./safety_gear_demo && \
       rm -rf build && \
       mkdir build && \
       cd build && \
       cmake -DCMAKE_INSTALL_INCLUDEDIR=${CMAKE_INSTALL_PREFIX}/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} .. && \
       make"

   RUN cp ./safety_gear_demo/build/libsafety_gear_demo.so $ARTIFACTS/lib

   FROM video_analytics as runtime   <<<<<This container is based VA container
   HEALTHCHECK NONE
   WORKDIR /app
   ARG ARTIFACTS
   ARG CMAKE_INSTALL_PREFIX
   COPY --from=builder $ARTIFACTS/lib ${CMAKE_INSTALL_PREFIX}/lib
   COPY --from=builder $ARTIFACTS/safety_gear_demo .

An Example of Python based UDF's **Dockerfile** will look as follows:

.. code-block:: dockerfile

   ARG EII_VERSION
   ARG DOCKER_REGISTRY
   FROM ${DOCKER_REGISTRY}openedgeinsights/ia_video_ingestion:$EII_VERSION
   LABEL description="Multi-class clasifcation UDF Image"
   WORKDIR /app

   # Added /app to python path as copying the UDF code block under /app
   # else user can add the path to which it copies its udf code.
   ENV PYTHONPATH ${PYTHONPATH}:/app
   # User can mention about all UDF directories here, both py and C++.
   COPY ./sample_classification ./sample_classification  <<<<< ./sample_classification is the destination here for UdfLoader to pick it at runtime. Additionally we have set the python path forUdfLoader to identify the algo artifacts properly.


* 
  ## *docker-compose.yml*

  The ``docker-compose.yml`` makes the custom UDF container to be managed by the OEI infrastructure. The content for this file also defines how the UDF container will communicate with other containers via the messagebus. Some of the important key-value pairs of this ``docker-compose.yml`` file is described in the following code snippet. If required, you can change the value of certain keys. The following keys should be changed for a different UDF container. Import key-value combinations are described in the code comments with prescript of "<<<<<".

  .. code-block:: yml

       services:
         python_multi_classification:      <<<<< Define the name of the container
           build:
             context: $PWD/../CustomUdfs/PyMultiClassificationIngestion                 <<<<< This path should be the relative path of container artifact
             dockerfile: $PWD/../CustomUdfs/PyMultiClassificationIngestion/Dockerfile   <<<<< Path to Dockerfile of the container
       -----snip-----
           image: ${DOCKER_REGISTRY}python_multi_classification:${EII_VERSION} <<<<< Image Name
           container_name: python_multi_classification                         <<<<< container Name
           hostname: python_multi_classification                               <<<<< Docker Network Namespace defined container name
       -----snip----
           environment:
             AppName: "PyMultiClassificationIngestion"                                  <<<<< Application name which will be used by configuration server & for security purpose.
       ----snip------
             PubTopics: "py_analytic_result_strm"                              <<<<< Publisher topic of this container And sub topic can be added based on need
             py_analytic_result_strm_cfg: "zmq_ipc,${SOCKET_DIR}/"
             Server: "zmq_tcp,127.0.0.1:66018"
       -----snip------
           privileged: true
           devices:
             - "/dev/:/dev/"
           volumes:                                                             <<<<< In the PROD mode the "volumes" section need to be defined as described.
             - ./Certificates/PyMultiClassificationIngestion:/run/secrets/PyMultiClassificationIngestion:ro
             - ./Certificates/rootca/cacert.pem:/run/secrets/rootca/cacert.pem:ro

* 
  ## *UDF core-logic Directory*

This directory need to have the Algo/pre-processing implementation which defines the necessary callbacks, IR files, and other configurational files, as per need. You can place them directly without having another directory level too. In that case, the **Dockerfile** and **docker-compose.yml** should update the path accordingly.

Deploy Process
--------------

For deploying the Custom UDFs, complete the following steps:


#. Configure Visualizer and WebVisualizer services ``config.json`` to connect to one or more streams coming out of the following CustomUdf services. For more information, refer to the following:

   * `../Visualizer/README.md <https://github.com/open-edge-insights/video-native-visualizer/blob/master/README.md>`_
   * `../WebVisualizer/README.md <https://github.com/open-edge-insights/video-web-visualizer/blob/master/README.md>`_

#. 
   Enable the required CustomUDF services. As per the OEI default scenario, the sample custom UDF containers are not mandatory containers to run, hence the ``builder.py`` should run the ``video-streaming-all-udfs.yml`` use case. The following code snippet shows all the sample UDF containers:

   .. code-block:: yml

       AppName:
       - Visualizer
       - WebVisualizer
       - CustomUdfs/NativeSafetyGearAnalytics   <<<<< All lines from here added are customUDFs, User can define his own container directory here
       - CustomUdfs/NativeSafetyGearIngestion
       - CustomUdfs/PyMultiClassificationIngestion
       - CustomUdfs/PySafetyGearAnalytics
       - CustomUdfs/PySafetyGearIngestion
       - CustomUdfs/GVASafetyGearIngestion
       - CustomUdfs/NativeOneAPIIngestion

#. 
   Run the following commands:

   .. code-block:: bash

       cd [WORKDIR]/IEdgeInsights/build/
       python3 builder.py -f usecases/video-streaming-all-udfs.yml

   ..

      **Note:** It is not mandatory to keep the custom UDFs in the ``CustomUdfs`` directory, but you must change the ``video-streaming-all-udfs.yml`` file to point the right path. Additionally, if the   custom UDFs are placed under the ``IEdgeInsights`` directory then, the builder.py file automatically picks it to generate a consolidated ``IEdgeInsights/build/eii_config.json`` and ``IEdgeInsights/  build/docker-compose.yml`` files.


#. 
   After generating the consolidated **eii_config.json** and **docker-compose.yml** file, run the following commands to run the use case. As a precautionary measure, you can check the previously mentioned file to check the sanity of the UDF specific config and service details.

#. 
   Run the the following commands:

   .. code-block:: bash

       cd [WORKDIR]/IEdgeInsights/build/
       # Build base images (needed for buidling native custom udf services)
       docker-compose -f docker-compose-build.yml build ia_eiibase ia_common ia_video_common ia_openvino_base ia_configmgr_agent
       # Build custom udf services based on the use case selected
       docker-compose -f docker-compose-build.yml build ia_gva_safety_gear_ingestion ia_native_safety_gear_analytics ia_native_safety_gear_ingestion   ia_python_multi_classificationia_python_safety_gear_analytics ia_python_safety_gear_ingestion ia_native_oneapi_ingestion
       ./eii_start.sh

Sample UDFs Directory
---------------------

In the ``CustomUdfs`` directory, there are 5 sample UDfs implemented and shown as follows. These samples are created to showcase different use case.

.. code-block:: bash

   .
   ├── NativeSafetyGearAnalytics
   ├── NativeSafetyGearIngestion

   ├── PyMultiClassificationIngestion

   ├── PySafetyGearAnalytics
   ├── PySafetyGearIngestion

   ├── GVASafetyGearIngestion
   ├── NativeOneAPIIngestion

   └── README.md

The `docker-compose.yml <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeSafetyGearAnalytics/docker-compose.yml>`_ container has used the `docker-compose.yml <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeSafetyGearAnalytics/docker-compose.yml>`_ UDF which is defined in the Video Ingestion container. You can define its own preprocessing UDF and add to the ``config.json`` file to modify it. The results are posted to a eii-msgbus topic which is subscribed by the `docker-compose.yml <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeSafetyGearAnalytics/docker-compose.yml>`_ containers. The configs can be seen in the `docker-compose.yml <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeSafetyGearAnalytics/docker-compose.yml>`_ file. For more information on the UDF configs, refer to the following:


* `NativeSafetyGearIngestion-README <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeSafetyGearIngestion/README.md>`_
* `NativeSafetyGearAnalytics-README <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeSafetyGearAnalytics/README.md>`_

The `PySafetyGearAnalytics <https://github.com/open-edge-insights/video-custom-udfs/blob/master/PySafetyGearAnalytics>`_ and the `PySafetyGearAnalytics <https://github.com/open-edge-insights/video-custom-udfs/blob/master/PySafetyGearAnalytics>`_ are the same use case but they showcase the Python variant of ingestion and analytics UDF containers. For more information on the UDF configs, refer to the following:


* `PySafetyGearIngestion-README <https://github.com/open-edge-insights/video-custom-udfs/blob/master/PySafetyGearIngestion/README.md>`_
* `PySafetyGearAnalytics-README <https://github.com/open-edge-insights/video-custom-udfs/blob/master/PySafetyGearAnalytics/README.md>`_

The `PyMultiClassificationIngestion-README <https://github.com/open-edge-insights/video-custom-udfs/blob/master/PyMultiClassificationIngestion/README.md>`_ showcases a UDF where everything is performed inside the VideoIngestion containers. You will execute this UDF you don't want to involve the eii-msgbus for data transfer between containers. This allows faster processing of the pipeline. For more information on the UDF configs, refer to `PyMultiClassificationIngestion-README <https://github.com/open-edge-insights/video-custom-udfs/blob/master/PyMultiClassificationIngestion/README.md>`_.

..

   **Notes:**



* 
  It is not mandatory to have Ingestion containers for every analytics UDF. The UDFs are connected to each other via the MSGBUS topics. Hence we can always use stock VideoIngestion container as long as the Custom Analytic UDF container can read and churn the data it receives.

* 
  You shouldn't remove the VI & VA containers before first time build of custom UDF as it will fail to build custom UDFs. After the UDFs are functional, you can always remove the running VI & VA containers. While removing the VI and VA containers make necessary changes in the ``../build/docker-compose.yml`` files based on the way it is written. For example, you may need to remove **depends_on** keyword, if custom container has it for the VI and VA containers.

GVASafetyGearIngestion
----------------------

Following the steps mentioned in this document, the `GVASafetyGearIngestion-README.md <https://github.com/open-edge-insights/video-custom-udfs/blob/master/GVASafetyGearIngestion/README.md>`_ UDF container is added. It is a GVA-based UDF container based out of the VI container. For more information on the UDF configs, refer to `GVASafetyGearIngestion-README.md <https://github.com/open-edge-insights/video-custom-udfs/blob/master/GVASafetyGearIngestion/README.md>`_.

..

   **Notes:**


   * Analytics operation is performed using the constructed gstreamer pipeline using the GVA plugin elements. It is not mandatory to have a UDF parameter in the config and have an analytics container. The classified results can be directly subscribed from the GVA ingestion container.
   * The model files which are provided to the GVA plugin elements should be copied in the container to the ``./models`` directory path.


  **Example:** Refer to the `GVASafetyGearIngestion-Dockerfile <https://github.com/open-edge-insights/video-custom-udfs/blob/master/GVASafetyGearIngestion/Dockerfile>`_ snippet below:

.. code-block:: Dockerfile


     ----snippet----
     COPY ./ref ./models/ref

  Here the models files placed under the ``ref`` directory on the host system is copied to the ``./models/ref`` path in the container.

NativeOneAPIIngestion
---------------------

The `NativeOneAPIIngestion-README.md <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeOneAPIIngestion/README.md>`_ is a DPCPP-based UDF container based out of VideoIngestion. For more information on the UDF configs, refer to the `NativeOneAPIIngestion-README.md <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativeOneAPIIngestion/README.md>`_.

NativePclIngestion
------------------

The `NativePclIngestion-README.md <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativePclIngestion/README.md>`_ is a PCL-based UDF container based out of VideoIngestion. For more information on the UDF configs, refer to the `NativePclIngestion-README.md <https://github.com/open-edge-insights/video-custom-udfs/blob/master/NativePclIngestion/README.md>`_.
