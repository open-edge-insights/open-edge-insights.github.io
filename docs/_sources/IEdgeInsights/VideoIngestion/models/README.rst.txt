
Contents
========


* `Contents <#contents>`__

  * `Using GVA elements in VideoIngestion <#using-gva-elements-in-videoingestion>`__

Using GVA elements in VideoIngestion
------------------------------------

For creating a GVA-based ingestion custom udf container, refer to the `custom-udfs-gva Readme <https://github.com/open-edge-insights/video-custom-udfs/blob/master/README.md>`_.

For running the worker safety GVA sample, refer to the `GVASafetyGearIngestion Readme <https://github.com/open-edge-insights/video-custom-udfs/blob/master/GVASafetyGearIngestion/README.md>`_.

To run the GVA pipeline in the VideoIngestion container, perform the following:


#. Copy the IR model files to the ``[WORKDIR]/IEdgeInsights/VideoIngestion/models`` directory.
#. Refer the `docs/gva_doc.md <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/gva_doc.md>`_ for the GVA configuration with the supported camera.
#. Provision, build, and run Open EII. Refer to the `Readme <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_.
