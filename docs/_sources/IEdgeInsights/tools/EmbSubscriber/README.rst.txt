
**Contents**


* `EmbSubscriber <https://open-edge-insights.github.io/IEdgeInsights/tools/EmbSubscriber/#embsubscriber>`_

  * `EII pre-requisites <https://open-edge-insights.github.io/IEdgeInsights/tools/EmbSubscriber/#eii-pre-requisites>`_
  * `Running EmbSubscriber <https://open-edge-insights.github.io/IEdgeInsights/tools/EmbSubscriber/#running-embsubscriber>`_
  * `Running EmbSubscriber in IPC mode <https://open-edge-insights.github.io/IEdgeInsights/tools/EmbSubscriber/#running-embsubscriber-in-ipc-mode>`_

EmbSubscriber
=============

EmbSubscriber subscribes message coming from a publisher.It subscribes to messagebus topic to get the data.

EII pre-requisites
------------------


#. 
   EmbSubscriber expects a set of config, interfaces & public private keys to be present in ETCD as a pre-requisite.
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

         $ cd [WORKDIR]/IEdgeInsights/build
         $ python3 builder.py -f usecases/time-series.yml

Running EmbSubscriber
---------------------


#. Refer `../README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_ to provision, build and run the tool along with the EII time-series recipe/stack.

Running EmbSubscriber in IPC mode
---------------------------------

User needs to modify interface section of **\ `config.json <https://github.com/open-edge-insights/eii-tools/blob/master/EmbSubscriber/config.json>`_\ ** to run in IPC mode as following

.. code-block::

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
