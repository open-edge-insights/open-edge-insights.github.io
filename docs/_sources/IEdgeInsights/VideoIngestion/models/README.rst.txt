
**Contents**


* `Using GVA elements in VideoIngestion <#using-gva-elements-in-videoingestion>`__

Using GVA elements in VideoIngestion
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* 
  For creating a GVA based ingestion custom udf container, refer `custom-udfs-gva <https://github.com/open-edge-insights/video-custom-udfs/blob/master/README.md>`_.

* 
  Refer `GVASafetyGearIngestion <https://github.com/open-edge-insights/video-custom-udfs/blob/master/GVASafetyGearIngestion/README.md>`_ for running the worker safety GVA sample.

* 
  In case one needs to run the GVA pipeline in VideoIngestion container, follow the below steps:


  #. 
     Copy the IR model files to ``[WORKDIR]/IEdgeInsights/VideoIngestion/models`` directory.

  #. 
     Refer `docs/gva_doc.md <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/gva_doc.md>`_ for the GVA configuration with the supported camera.

  #. 
     Provision, Build and Run EII by refering `main-README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_.
