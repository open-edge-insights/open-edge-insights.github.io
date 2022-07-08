
Contents
========


* `Contents <#contents>`__

  * `VideoAnalytics Module <#videoanalytics-module>`__

    * `Configuration <#configuration>`__
    * `Update Security Context of VideoIngestion Helm Charts <#update-security-context-of-videoingestion-helm-charts>`__

VideoAnalytics Module
---------------------

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for file names, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights for Industrial (Open EII). This is due to the product name change of EII as Open EII.


The VideoAnalytics module is mainly responsibly for running the classifier UDFs and doing the required inferencing on the chosen Intel(R) Hardware (CPU, GPU, VPU, HDDL) using openVINO.

The high-level logical flow of VideoAnalytics pipeline is as follows:


#. App reads the application configuration via Open EII Configuration Manager which has details of ``encoding`` and ``udfs``.
#. App gets the msgbus endpoint configuration from system environment.
#. Based on above two configurations, app subscribes to the published topic/stream coming from VideoIngestion module.
#. The frames received in the subscriber are passed onto one or more chained native/python UDFs for running inferencing and doing any post-processing as required. For more information, refer to `UDFs Readme <https://github.com/open-edge-insights/video-common/blob/master/udfs/README.md>`_.
#. The frames coming out of chained UDFs are published on the different topic/stream on Message Bus.

Configuration
^^^^^^^^^^^^^


#. `UDFs Configuration <https://github.com/open-edge-insights/video-common/blob/master/udfs/README.md>`_
#. `Etcd Secrets Configuration <https://github.com/open-edge-insights/eii-core/blob/master/Etcd_Secrets_Configuration.md>`_ and
#. `MessageBus Configuration <https://github.com/open-edge-insights/eii-core/blob/master/common/libs/ConfigMgr/README.md#interfaces>`_ respectively.
#. `JSON schema <https://github.com/open-edge-insights/video-analytics/blob/master/schema.json>`_

.. note:: 



* The ``max_workers`` and ``udfs`` are configuration keys related to UDFs.
  For more details on UDF configuration, visit `../common/video/udfs/README.md <https://github.com/open-edge-insights/video-common/blob/master/udfs/README.md>`_
* For details on Etcd and MessageBus endpoint configuration, visit `Etcd_Secrets_Configuration <https://github.com/open-edge-insights/eii-core/blob/master/Etcd_Secrets_Configuration.md>`_.
* If the VideoAnalytics container consumes a lot of memory, then one of the suspects could be that Algo processing is slower than the frame ingestion rate. Hence a lot of frames are occupying RAM waiting to be processed. In that case user can reduce the high watermark value to acceptable lower number so that RAM consumption will be under control and stay stabilzed. The exact config parameter is called **ZMQ_RECV_HWM** present in `docker-compose.yml <https://github.com/open-edge-insights/video-analytics/blob/master/docker-compose.yml>`_. This config is also present in other types of container, hence user can tune them to control the memory bloat if applicable. The config snippet is pasted below:

.. code-block:: bash

         ZMQ_RECV_HWM: "1000"

All the app module configuration are added into distributed key-value data store under ``AppName`` env as mentioned in the environment section of this app's service definition in docker-compose.

Developer mode related overrides go to `docker-compose-dev.override.yml <https://github.com/open-edge-insights/video-analytics/blob/master/docker-compose-dev.override.yml>`_.

If ``AppName`` is ``VideoAnalytics``\ , then the app's config would be fetched from ``/VideoAnalytics/config`` key via Open EII Configuration Manager.

**Note:**


* For ``jpeg`` encoding type, ``level`` is the quality from ``0 to 100`` (the higher is the better).
* For ``png`` encoding type, ``level`` is the compression level from ``0 to 9``. A higher value means a smaller size and longer compression time.

Use the `JSON validator tool <https://www.jsonschemavalidator.net/>`_ for validating the app configuration against the above schema.

Update Security Context of VideoIngestion Helm Charts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Complete the following steps to update Helm charts to enable K8s environment to access or detect Basler cameras and NCS2 devices


#. Open the ``EII_HOME_DIR/IEdgeInsights/VideoAnalytics/helm/templates/video-analytics.yaml`` file.
#. 
   Update the following security context snippet

   .. code-block:: yml

           securityContext:
             privileged: true

   in the yml file as:

   .. code-block:: yaml

       ...
       ...
       ...
           imagePullPolicy: {{ $global.Values.imagePullPolicy }}
           securityContext:
             privileged: true
           volumeMounts:
           - name: dev
             mountPath: /dev
       ...

#. 
   Rerun ``builder.py`` to apply these changes to your deployment helm charts.
