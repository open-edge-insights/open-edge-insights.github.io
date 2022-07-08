
Contents
========


* `Contents <#contents>`__

  * `EmbSubscriber <#embsubscriber>`__

    * `Prerequisites <#prerequisites>`__
    * `Running EmbSubscriber <#running-embsubscriber>`__
    * `Run EmbSubscriber in IPC mode <#run-embsubscriber-in-ipc-mode>`__

EmbSubscriber
-------------

EmbSubscriber subscribes message coming from a publisher.It subscribes to messagebus topic to get the data.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for file names, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights for Industrial (Open EII). This is due to the product name change of EII as Open EII.


Prerequisites
^^^^^^^^^^^^^


#. 
   EmbSubscriber expects a set of config, interfaces & public private keys to be present in ETCD as a prerequisite.
    To achieve this, please ensure an entry for EmbSubscriber with its relative path from `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory is set in the time-series.yml file present in `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory. An example has been provided below:

   .. code-block:: sh

           AppName:
           - Grafana
           - InfluxDBConnector
           - Kapacitor
           - Telegraf
           - tools/EmbSubscriber

#. 
   With the above pre-requisite done, please run the below command:

   .. code-block:: sh

         cd [WORKDIR]/IEdgeInsights/build
         python3 builder.py -f usecases/time-series.yml

Running EmbSubscriber
^^^^^^^^^^^^^^^^^^^^^


#. Refer to the `../README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_ to provision, build and run the tool along with the Open EII Time Series recipe or stack.

Run EmbSubscriber in IPC mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To run EmbSubscriber in the IPC mode, modify the interfaces section of the `config.json <https://github.com/open-edge-insights/eii-tools/blob/master/EmbSubscriber/config.json>`_ file as follows:

.. code-block:: sh

   {
     "config": {},
     "interfaces": {
       "Subscribers": [
         {
           "Name": "TestSub",
           "PublisherAppName": "Telegraf",
           "Type": "zmq_ipc",
           "EndPoint": {
                     "SocketDir": "/EII/sockets",
                     "SocketFile": "telegraf-out"
            },
           "Topics": [
             "*"
           ]
         }
       ]
     }
   }
