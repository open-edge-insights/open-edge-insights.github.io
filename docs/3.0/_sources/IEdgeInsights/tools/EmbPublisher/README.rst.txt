
Contents
========


* `Contents <#contents>`__

  * `EmbPublisher <#embpublisher>`__

    * `How to integrate this tool with video/timeseries use case <#how-to-integrate-this-tool-with-videotimeseries-use-case>`__
    * `Configuration of the tool <#configuration-of-the-tool>`__
    * `Running EmbPublisher in IPC mode <#running-embpublisher-in-ipc-mode>`__

EmbPublisher
------------

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights (OEI). This is due to the product name change of EII as OEI.



* This tool acts as a brokered publisher of message bus.
* Telegaf's message bus input plugin acts as a subscriber to the OEI broker.

How to integrate this tool with video/timeseries use case
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* In 'time-series.yml'/'video-streaming.yml' file, please add 'ZmqBroker' and 'tools/EmbPublisher' components.
* Use the modified 'time-series.yml'/'video-streaming.yml' file as an argument while generating the docker-compose.yml file using the 'builder.py' utility.
* Follow usual provisioning and starting process.

Configuration of the tool
^^^^^^^^^^^^^^^^^^^^^^^^^

Let us look at the sample configuration

.. code-block:: sh

   {
     "config": {
       "pub_name": "TestPub",
       "msg_file": "data1k.json",
       "iteration": 10,
       "interval": "5ms"
     },
     "interfaces": {
       "Publishers": [
         {
           "Name": "TestPub",
           "Type": "zmq_tcp",
           "AllowedClients": [
             "*"
           ],
           "EndPoint": "ia_zmq_broker:60514",
           "Topics": [
             "topic-pfx1",
             "topic-pfx2",
             "topic-pfx3",
             "topic-pfx4"
           ],
           "BrokerAppName" : "ZmqBroker",
           "brokered": true
         }
       ]
     }
   }


* -pub_name : The name of the publisher in the interface.
* -topics: The name of the topics seperated by comma, for which publisher need to be started.
* -msg_file : The file containing the JSON data, which represents the single data point (files should be kept into directory named 'datafiles').
* -num_itr : The number of iterations
* -int_btw_itr: The interval between any two iterations

Running EmbPublisher in IPC mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

User needs to modify interface section of **\ `config.json <https://github.com/open-edge-insights/eii-tools/blob/master/EmbPublisher/config.json>`_\ ** to run in IPC mode as following:

.. code-block:: sh

     "interfaces": {
       "Publishers": [
         {
           "Name": "TestPub",
           "Type": "zmq_ipc",
           "AllowedClients": [
             "*"
           ],
           "EndPoint": {
                   "SocketDir": "/EII/sockets",
                   "SocketFile": "frontend-socket"
               },
           "Topics": [
             "topic-pfx1",
             "topic-pfx2",
             "topic-pfx3",
             "topic-pfx4"
           ],
           "BrokerAppName" : "ZmqBroker",
           "brokered": true
         }
       ]
     }
