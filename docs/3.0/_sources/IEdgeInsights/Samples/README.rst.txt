
Sample Apps
===========

This section provides more information about the Open Edge Insights for Industrial (Open EII) sample apps and how to use the core libraries packages like Utils, Message Bus, and ConfigManager in various flavors of Linux such as Ubuntu and Alpine operating systems or docker images for programming languages such as C++, Go, and Python.

The following table shows the details for the supported flavors of Linux operating systems or docker images and programming languages that support sample apps:

.. list-table::
   :header-rows: 1

   * - Linux Flavor
     - Language
   * - ``Ubuntu``
     - ``C++, Go, Python``
   * - ``Alpine``
     - ``C++, Go``


The sample apps are classified as ``publisher`` and ``subscriber`` apps. For more information, refer to the following:


* `Publisher <https://github.com/open-edge-insights/eii-samples/blob/master/publisher/README.md>`_
* `Subscriber <https://github.com/open-edge-insights/eii-samples/blob/master/subscriber/README.md>`_

Run the Samples Apps
--------------------

For default scenario, the sample custom UDF containers are not the mandatory containers to run. The ``builder.py`` script runs the ``sample-apps.yml`` from the `build/usecases <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases>`_ directory and adds all the sample apps containers. Refer to the following list to view the details of the sample apps containers:

.. code-block:: yml

       AppContexts:
       # CPP sample apps for Ubuntu and Alpine operating systems or docker images
       - Samples/publisher/cpp/ubuntu
       - Samples/publisher/cpp/alpine
       - Samples/subscriber/cpp/ubuntu
       - Samples/subscriber/cpp/alpine

       # Python sample apps for Ubuntu operating systems or docker images
       - Samples/publisher/python/ubuntu
       - Samples/subscriber/python/ubuntu

       # Go sample apps for Ubuntu and Alpine operating systems or docker images
       - Samples/publisher/go/ubuntu
       - Samples/publisher/go/alpine
       - Samples/subscriber/go/ubuntu
       - Samples/subscriber/go/alpine


#. 
   To run the ``sample-apps.yml`` file, run the following command:

   .. code-block:: sh

       cd [WORKDIR]/IEdgeInsights/build
       python3 builder.py -f ./usecases/sample-apps.yml file used>

#. 
   Refer to the `\ ``Build Open EII stack`` <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_ and the `\ ``Run Open EII service`` <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_ sections to build and run the sample apps.
