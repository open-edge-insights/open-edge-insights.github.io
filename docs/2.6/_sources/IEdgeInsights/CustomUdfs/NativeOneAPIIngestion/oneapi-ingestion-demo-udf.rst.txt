
OneAPI Ingestion Demo UDF
=========================

This User defined function (UDF) accepts the frames, and shows how to use a DPC++ kernel with the OneAPI library to blur a simple frame content. The UDF config is shown as follows:

``UDF config``\ :

.. code-block:: json

   {
   "name": "dpcpp_blur_udf",
   "type": "native",
   "height" : 50,
   "width" : 50
   }

.. note::  In the config, the ``height`` and ``width`` are used as the intensity of blurring each pixels in each row and column.


For more information on configs of other inbuilt UDFs like dummy, fps, and resize udfs, refer to the `UDFs-README <https://github.com/open-edge-insights/video-common/blob/master/udfs/README.md>`_.

OEI's OneAPI usecase demonstrates the usage of `oneAPI Base Toolkit image <https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit-download.html?operatingsystem=linux&distributions=aptpackagemanager>`_\ , which includes all the required features of oneAPI. Due to this, the docker build time is approximately 2 hours and the docker image size goes upto nearly 20GB or more.

To use smaller oneAPI toolkit for the respective usecase, you can use different oneAPI toolkits available at https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html#gs.p5b1kr instead of the ``oneAPI Base Toolkit``.
