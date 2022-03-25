
EII Sample Apps
===============

There are three sample apps for EII:


#. `cpp_sample_app <https://github.com/open-edge-insights/eii-samples/blob/master/cpp_sample_app/README.md>`_
#. `go_sample_app <https://github.com/open-edge-insights/eii-samples/blob/master/go_sample_app/README.md>`_
#. `python_sample_app <https://github.com/open-edge-insights/eii-samples/blob/master/python_sample_app/README.md>`_

Steps to run EII samples apps
-----------------------------


#. Each sample app expects a set of config, interfaces & public private keys to be present in ETCD as a pre-requisite.
   To achieve this, please ensure an entry for a publisher-subscriber pair with its relative path from `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory is set in any of the .yml files present in the `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory. An example has been provided below:

.. code-block:: yaml

       AppContexts:
       - Samples/cpp_sample_app/publisher
       - Samples/cpp_sample_app/subscriber
       - Samples/go_sample_app/publisher
       - Samples/go_sample_app/subscriber
       - Samples/python_sample_app/publisher
       - Samples/python_sample_app/subscriber


#. 
   With the above pre-requisite done, please run the below command:

   .. code-block:: sh

      $ cd [WORKDIR]/IEdgeInsights/build
      $ python3 builder.py -f ./usecases/<yml file used>

#. Refer `README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_ to provision, build and run the Sample Apps
