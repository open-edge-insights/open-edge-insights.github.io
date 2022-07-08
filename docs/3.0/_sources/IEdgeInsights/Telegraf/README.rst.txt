.. role:: raw-html-m2r(raw)
   :format: html


Contents
========


* `Contents <#contents>`__

  * `Plugins Overview <#plugins-overview>`__
  * `Telegraf's Default Configuration <#telegrafs-default-configuration>`__
  * `MQTT Sample Configuration and Testing Tool <#mqtt-sample-configuration-and-testing-tool>`__
  * `Enabling Message Bus Input Plugin in Telegraf <#enabling-message-bus-input-plugin-in-telegraf>`__

    * `Plugin Configuration <#plugin-configuration>`__

      * `ETCD Configuration <#etcd-configuration>`__
      * `Configuration at Telegraf.conf File <#configuration-at-telegrafconf-file>`__

    * `Advanced: Multiple Plugin Sections of Open EII Message Bus Input Plugin <#advanced-multiple-plugin-sections-of-open-eii-message-bus-input-plugin>`__

  * `Using Input Plugin <#using-input-plugin>`__
  * `Enable Message Bus Output Plugin in Telegraf <#enable-message-bus-output-plugin-in-telegraf>`__
  * `Advanced: Multiple plugin sections of message bus output plugin <#advanced-multiple-plugin-sections-of-message-bus-output-plugin>`__
  * `Run Telegraf Input Output Plugin in IPC Mode <#run-telegraf-input-output-plugin-in-ipc-mode>`__
  * `Optional: Adding Multiple Telegraf Instances <#optional-adding-multiple-telegraf-instances>`__

Telegraf is a part of the `TICK stack <https://www.influxdata.com/time-series-platform>`_. This is a plugin-based agent. Telegraf has many input and output plugins. In Open EII's basic configuration, it's being used for data ingestion. However, Open EII framework does not restrict Telegraf's any of the features. In Open EII, basic configuration uses Telegraf for data ingestion and sending it to InfluxDB.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for file names, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights for Industrial (Open EII). This is due to the product name change of EII as Open EII.


Plugins Overview
----------------

The plugin subscribes to configured topic or topic prefixes. Plugin has component
called subscriber which receives the data from eii message bus.
After receiving the data, depending on configuration, the plugin process
the data, either synchronously or asynchronously.


* In **synchronous** processing**, the receiver thread (thread which receives the data from message bus) is also resposible for the processing of the data (json parsing). After processing the previous data only, the receiver thread process next data available on the message bus.
* In **asynchronous** processing the  receiver thread  receives the data and put it into the queue. There will be pool of threads which will be dequeing the data from the queue and processing it.

Guidelines for choosing the data processing options are as follows:


* Synchronous option: When the ingestion rate is consistent
* Asynchronous options: There are two options

  * Topic specific queue+threadpool : Frequent spike in ingestion rate for a specific topic
  * Global queue+threadpool : Sometimes spike in ingestion rate for a specific topic

Telegraf's Default Configuration
--------------------------------


#. Telegraf starts with the default configuration which is present at `config/Telegraf/Telegraf.conf <https://github.com/open-edge-insights/ts-telegraf/blob/master/config/Telegraf/Telegraf.conf>`_ (for the dev mode the name is 'Telegraf_devmode.conf'). By default the below plugins are enabled.


* MQTT input plugin (\ **[[inputs.mqtt_consumer]]**\ )
* Open EII message bus input plugin (\ **[[inputs.eii_msgbus]]**\ )
* Influxdb output plugin (\ **[[outputs.influxdb]]**\ )

Telegraf will be started using script 'telegraf_start.py. This script will get the configuration from ETCD first and then it will start the Telegraf service by picking the right configuration depending on the developer/production mode. By default only single instance of Telegraf container runs (named 'ia_telegraf').

MQTT Sample Configuration and Testing Tool
------------------------------------------


* 
  To test with MQTT publisher in k8s helm environment, Please update 'MQTT_BROKER_HOST' Environment Variables in `values.yaml <https://github.com/open-edge-insights/ts-telegraf/blob/master/helm/values.yaml>`_ with HOST IP address of the system where MQTT Broker is running.

* 
  To test with remote mqtt broker in docker environment, Please update 'MQTT_BROKER_HOST' Environment Variables in `docker-compose.yml <https://github.com/open-edge-insights/ts-telegraf/blob/master/docker-compose.yml>`_ with HOST IP address of the system where MQTT Broker is running.

.. code-block:: sh

     ia_telegraf:
       environment:
         ...
         MQTT_BROKER_HOST: '<HOST IP address of the system where MQTT Broker is running>'


* 
  Telegraf Instance can be configured with pressure point data ingestion. In the following example, the MQTT input plugin of Telegraf is configured to read pressure point data and stores into ‘point_pressure_data’ measurement.

  .. code-block:: sh

     # # Read metrics from MQTT topic(s)
     [[inputs.mqtt_consumer]]
     #   ## MQTT broker URLs to be used. The format should be scheme://host:port,
     #   ## schema can be tcp, ssl, or ws.
     servers = ["tcp://localhost:1883"]
     #
     #   ## MQTT QoS, must be 0, 1, or 2
     #   qos = 0
     #   ## Connection timeout for initial connection in seconds
     #   connection_timeout = "30s"
     #
     #   ## Topics to subscribe to
     topics = [
     "pressure/simulated/0",
     ]
     name_override = "point_pressure_data"
     data_format = "json"
     #
     #   # if true, messages that can't be delivered while the subscriber is offline
     #   # will be delivered when it comes back (such as on service restart).
     #   # NOTE: if true, client_id MUST be set
     persistent_session = false
     #   # If empty, a random client ID will be generated.
     client_id = ""
     #
     #   ## username and password to connect MQTT server.
     username = ""
     password = ""

* 
  To start the mqtt-publisher with pressure data,

  .. code-block:: sh

      cd ../tools/mqtt/publisher/

  change the command option in `docker-compose.yml <https://github.com/open-edge-insights/eii-tools/blob/master/mqtt/publisher/docker-compose.yml>`_ to:

  .. code-block:: sh

      ["--pressure", "10:30"]

  Build and Run mqtt publisher:

  .. code-block:: sh

      docker-compose up --build -d

Refer to the `tools/mqtt/publisher/README.md <https://github.com/open-edge-insights/eii-tools/blob/master/mqtt/README.md>`_

Enabling Message Bus Input Plugin in Telegraf
---------------------------------------------

The purpose of this enablement is to allow telegraf received data from message bus and storing it into influxdb that able to be scalable.

Plugin Configuration
^^^^^^^^^^^^^^^^^^^^

Configuration of the plugin is divided as follows:


* ETCD configuration
* Configuration in Telegraf.conf file `config/Telegraf/Telegraf.conf <https://github.com/open-edge-insights/ts-telegraf/blob/master/config/Telegraf/Telegraf.conf>`_

ETCD Configuration
~~~~~~~~~~~~~~~~~~

Since this is an Open EII message bus plugin and so it's part of the Open EII framework, message bus configuration and plugin's topic specific configuration is kept into etcd. The following is the sample configuration:

.. code-block:: json

   {
       "config": {
           "influxdb": {
               "username": "admin",
               "password": "admin123",
               "dbname": "datain"
           },
           "default": {
               "topics_info": [
                   "topic-pfx1:temperature:10:2",
                   "topic-pfx2:pressure::",
                   "topic-pfx3:humidity"
               ],
               "queue_len": 10,
               "num_worker": 2,
               "profiling": "false"
           }
       },
       "interfaces": {
           "Subscribers": [
               {
                   "Name": "default",
                   "Type": "zmq_tcp",
                   "EndPoint": "ia_zmq_broker:60515",
                   "Topics": [
                       "*"
                   ],
                   "PublisherAppName": "ZmqBroker"
               }
           ]
       }
   }

**Brief description of the configuration**.

Similar to other Open EII services, Telegraf has 'config' and 'interfaces' sections. "interfaces" are the eii interface details. Let's have more information of "config" section.

config : Contains the configuration of the influxdb (\ **"influxdb"**\ ) and Open EII messagebus input plugin (\ **"default"**\ ). In the above sample configuration, the **"default"** is an instance name. This instance name is referenced from the Telegraf's configuration file `config/Telegraf/Telegraf.conf <https://github.com/open-edge-insights/ts-telegraf/blob/master/config/Telegraf/Telegraf.conf>`_.


* 
  topics_info : This is an array of topic prefix configuration, where user specifies, how the data from topic-prefix should be processed. Below is the way every line in the topic information should be interpreted.


  #. "topic-pfx1:temperature:10:2" : Process data from topic prefix 'topic-pfx1' asynchronously using the dedicated queue of length 10 and dedicated thread pool of size 2. And the processed data will be stored at measurement named 'temperature' in influxdb.
  #. "topic-pfx2:pressure::" : Process data from topic prefix 'topic-pfx2' asynchronously using the global queue and global thread pool. And the processed data will be stored at measurement named 'pressure' in influxdb.
  #. "topic-pfx3:humidity" : Process data synchronously. And the processed data will be stored at measurement named 'humidity' in influxdb.

  ..

     **Note**\ : If topic specific configuration is not mentioned, then by default, data gets processed synchronously and measurement name would be same as topic name.


* 
  queue_len : Global queue length.

* num_worker : Global thread pool size.
* 
  profiling : This is to enable profiling mode of this plugin (value can be either "true" or "false"). In profiling mode every point will get the following information and will be kept into same measurement as that of point.


  #. Total time spent in plugin (time in ns)
  #. Time spent in queue (in case of asynchronous processing only and time in ns)
  #. Time spend in json processing (time in ns)
  #. The name of the thread pool and the thread id which processed the point.

.. note:: \ : The name of the global thread pool is "GLOBAL". For a topic specific thread pool, the name is "for-$topic-name".*


Configuration at Telegraf.conf File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The plugin instance name is an additional key, kept into plugin configuration section. This key is used to fetch the configuration from ETCD. The following is the minimmum sample configuration with single plugin instance.

.. code-block::

   [[inputs.eii_msgbus]]
   **instance_name = "default"**
   data_format = "json"
   json_strict = true


Here, the value ``default`` acts as a key in the file `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_. For this key, there is configuration in the ``interfaces`` and ``config`` sections of the file `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_. So the value of ``instance_name`` acts as a connect/glue between the Telegraf configuration `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_ and the ETCD configuration `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_.

.. note:: \ : Since it’s Telegraf input plugin, the Telegraf’s parser configuration has to be in Telegraf.conf file. The more information of the telegraf json parser plugin can be be found at https://github.com/influxdata/telegraf/tree/master/plugins/parsers/json.
   If there are multiple telegraf instances, then the location of the Telegraf configuration files would be different. For more details, refer to the `Optional: Adding multiple telegraf instance <#optional-adding-multiple-telegraf-instances>`__ section.


Advanced: Multiple Plugin Sections of Open EII Message Bus Input Plugin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Like any other Telegraf plugin, user can keep multiple configuration sections of the message bus input plugin in the **\ `config/Telegraf/Telegraf.conf <https://github.com/open-edge-insights/ts-telegraf/blob/master/config/Telegraf/Telegraf.conf>`_\ ** file.

Let's have an example for the same. Let's assume there are two Open EII apps, one with the AppName "EII_APP1" and another with the AppName "EII_APP2", which are publishing the data to Open EII message bus.
*The Telegraf's ETCD configuration for the same is*

.. code-block:: json

   {
      "config":{
         "subscriber1":{
            "topics_info":[
               "topic-pfx1:temperature:10:2",
               "topic-pfx2:pressure::",
               "topic-pfx3:humidity"
            ],
            "queue_len":100,
            "num_worker":5,
            "profiling":"true"
         },
         "subscriber2":{
            "topics_info":[
               "topic-pfx21:temperature2:10:2",
               "topic-pfx22:pressure2::",
               "topic-pfx23:humidity2"
            ],
            "queue_len":100,
            "num_worker":10,
            "profiling":"true"
         }
      },
      "interfaces":{
         "Subscribers":[
            {
               "Name":"subscriber1",
               "EndPoint":"EII_APP1_container_name:5569",
               "Topics":[
                  "*"
               ],
               "Type":"zmq_tcp",
               "PublisherAppName": "EII_APP1"
            },
            {
               "Name":"subscriber2",
               "EndPoint":"EII_APP2_container_name:5570",
               "Topics":[
                  "topic-pfx21",
                  "topic-pfx22",
                  "topic-pfx23"
               ],
               "Type":"zmq_tcp",
               "PublisherAppName": "EII_APP2"
            }
         ]
      }
   }

*The Telegraf.conf configuration sections for the same is*

.. code-block::

   [[inputs.eii_msgbus]]
   instance_name = "subscriber1"
   data_format = "json"
   json_strict = true

   [[inputs.eii_msgbus]]
   instance_name = "subscriber2"
   data_format = "json"
   json_strict = true


Using Input Plugin
------------------

By default, the message bus input plugin is disabled. To configure the Open EII input plugin, uncomment the following lines in **\ `config/Telegraf/Telegraf_devmode.conf <https://github.com/open-edge-insights/ts-telegraf/blob/master/config/Telegraf/Telegraf_devmode.conf>`_\ ** and **\ `config/Telegraf/Telegraf_devmode.conf <https://github.com/open-edge-insights/ts-telegraf/blob/master/config/Telegraf/Telegraf_devmode.conf>`_\ **

.. code-block::

   ```sh
   [[inputs.eii_msgbus]]
   instance_name = "default"
   data_format = "json"
   json_strict = true
   tag_keys = [
     "my_tag_1",
     "my_tag_2"
   ]
   json_string_fields = [
     "field1",
     "field2"
   ]
   json_name_key = ""
   ```



* 
  Edit `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_ to add message bus input plugin.

  .. code-block:: sh

     {
         "config": {
             ...
             "default": {
                 "topics_info": [
                     "topic-pfx1:temperature:10:2",
                     "topic-pfx2:pressure::",
                     "topic-pfx3:humidity"
                 ],
                 "queue_len": 10,
                 "num_worker": 2,
                 "profiling": "false"
             },
             ...
         },
         "interfaces": {
             "Subscribers": [
                 {
                     "Name": "default",
                     "Type": "zmq_tcp",
                     "EndPoint": "ia_zmq_broker:60515",
                     "Topics": [
                         "*"
                     ],
                     "PublisherAppName": "ZmqBroker"
                 }
             ],
             ...
         }
     }

Enable Message Bus Output Plugin in Telegraf
--------------------------------------------

**Purpose**
Receiving the data from Telegraf Input Plugin and publish data to Open EII msgbus.

**Configuration of the plugin**
Configuration of the plugin is divided into two parts


* ETCD configuration
* Configuration in Telegraf.conf file `config/Telegraf/Telegraf.conf <https://github.com/open-edge-insights/ts-telegraf/blob/master/config/Telegraf/Telegraf.conf>`_

**ETCD configuration**
Since this is eii message bus plugin and so it’s part of Open EII framework, message bus configuration and plugin’s topic specific configuration is kept into etcd.
Below is the sample configuration

.. code-block:: json

   {
       "config": {
           "publisher": {
               "measurements": ["*"],
               "profiling": "false"
           }
       },
       "interfaces": {
           "Publishers": [
               {
                   "Name": "publisher",
                   "Type": "zmq_tcp",
                   "EndPoint": "0.0.0.0:65077",
                   "Topics": [
                       "*"
                   ],
                   "AllowedClients": [
                       "*"
                   ]
               }
           ]
       }
   }

**Brief description of the configuration**.
Like any other Open EII service Telegraf has 'config' and 'interfaces' sections. "interfaces" are the Open EII interface details. Let's have more information of "config" section.

config : Contains OPen EII messagebus output plugin (\ **"publisher"**\ ). In the above sample configuration, the **"publisher"** is an instance name. This instance name is referenced from the Telegraf's configuration file `config/Telegraf/Telegraf.conf <https://github.com/open-edge-insights/ts-telegraf/blob/master/config/Telegraf/Telegraf.conf>`_


* measurements : This is an array of measurements configuration, where user specifies, which measurement data should be published in msgbus.
* profiling : This is to enable profiling mode of this plugin (value can be either "true" or "false").

**Configuration at Telegraf.conf file**

The plugin instance name is an additional key, kept into plugin configuration section. This key is used to fetch the configuration from ETCD. Below is the minimmum, sample configuration with single plugin instance.

.. code-block::

   [[outputs.eii_msgbus]]
   instance_name = "publisher"


Here, the value **'publisher'**  acts as a key in the file **\ `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_\ **. For this key, there is configuration in the **'interfaces' and 'config'** sections of the file **\ `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_\ **. So the value of\ **'instance_name'** acts as a connect/glue between the Telegraf configuration **\ `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_\ ** and the **ETCD configuration `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_\ **

Advanced: Multiple plugin sections of message bus output plugin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Like any other Telegraf plugin user can keep multiple configuration sections of the message bus output plugin in the **\ `config/Telegraf/Telegraf.conf <https://github.com/open-edge-insights/ts-telegraf/blob/master/config/Telegraf/Telegraf.conf>`_\ ** file.

*The Telegraf's ETCD configuration for the same is*

.. code-block:: json

   {
       "config": {
           "publisher1": {
               "measurements": ["*"],
               "profiling": "false"
           },
           "publisher2": {
               "measurements": ["*"],
               "profiling": "false"
           }
       },
       "interfaces": {
           "Publishers": [
               {
                   "Name": "publisher1",
                   "Type": "zmq_tcp",
                   "EndPoint": "0.0.0.0:65077",
                   "Topics": [
                       "*"
                   ],
                   "AllowedClients": [
                       "*"
                   ]
               },
               {
                   "Name": "publisher2",
                   "Type": "zmq_tcp",
                   "EndPoint": "0.0.0.0:65078",
                   "Topics": [
                       "*"
                   ],
                   "AllowedClients": [
                       "*"
                   ]
               }
           ]
       }
   }

*The Telegraf.conf configuration sections for the same is*

.. code-block::

   [[outputs.eii_msgbus]]
   instance_name = "publisher1"

   [[outputs.eii_msgbus]]
   instance_name = "publisher2"


Run Telegraf Input Output Plugin in IPC Mode
--------------------------------------------


* User needs to modify interfaces section of **\ `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_\ ** to run in IPC mode as following:

.. code-block:: sh

       "interfaces": {
           "Subscribers": [
               {
                   "Name": "default",
                   "Type": "zmq_ipc",
                   "EndPoint": {
                     "SocketDir": "/EII/sockets",
                     "SocketFile": "backend-socket"
                   },

                   "Topics": [
                       "*"
                   ],
                   "PublisherAppName": "ZmqBroker"
               }
           ],
           "Publishers": [
               {
                   "Name": "publisher",
                   "Type": "zmq_ipc",
                   "EndPoint": {
                     "SocketDir": "/EII/sockets",
                     "SocketFile": "telegraf-out"
                   },
                   "Topics": [
                       "*"
                   ],
                   "AllowedClients": [
                       "*"
                   ]
               }
           ]
       }

Optional: Adding Multiple Telegraf Instances
--------------------------------------------


* User can add multiple instances of Telegarf. For that user needs to add additional environment variable named 'ConfigInstance' in docker-compose.yml file. For  every additional telegraf instance, there has to be additional compose section in the docker-compose.yml file.
* 
  The configuration for every instance has to be in the telegraf image. The standard to be followed is described as below.

    For instance named $ConfigInstance the telegraf configuration has to be kept in the repository at
    `config <https://github.com/open-edge-insights/ts-telegraf/blob/master/config>`_\ /$ConfigInstance/$ConfigInstance.conf (for production mode) and
    `config <https://github.com/open-edge-insights/ts-telegraf/blob/master/config>`_\ /$ConfigInstance/$ConfigInstance_devmode.conf (for developer mode).

    The same files will be available inside the respective container at
    '/etc/Telegraf/$ConfigInstance/$ConfigInstance.conf' (for production mode)
    '/etc/Telegraf/$ConfigInstance/$ConfigInstance_devmode.conf' (for developer mode)

Let's have some of the examples.

Example1:
For $ConfigInstance = 'Telegraf1'


* The location of the Telegraf configuration would be
  `config <https://github.com/open-edge-insights/ts-telegraf/blob/master/config>`_\ /Telegraf1/Telegraf1.conf (for production mode) and `config <https://github.com/open-edge-insights/ts-telegraf/blob/master/config>`_\ /Telegraf1/Telegraf1_devmode.conf (for developer mode)
* The additional docker compose section which has to be manually added in the file 'docker-compose.yml' would be

.. code-block:: sh

     ia_telegraf1:
       depends_on:
         - ia_common
       build:
         context: $PWD/../Telegraf
         dockerfile: $PWD/../Telegraf/Dockerfile
         args:
           EII_VERSION: ${EII_VERSION}
           EII_UID: ${EII_UID}
           EII_USER_NAME: ${EII_USER_NAME}
           TELEGRAF_SOURCE_TAG: ${TELEGRAF_SOURCE_TAG}
           TELEGRAF_GO_VERSION: ${TELEGRAF_GO_VERSION}
           UBUNTU_IMAGE_VERSION: ${UBUNTU_IMAGE_VERSION}
           CMAKE_INSTALL_PREFIX: ${EII_INSTALL_PATH}
       container_name: ia_telegraf1
       hostname: ia_telegraf1
       image: ${DOCKER_REGISTRY}openedgeinsights/ia_telegraf:${EII_VERSION}
       restart: unless-stopped
       ipc: "none"
       security_opt:
       - no-new-privileges
       read_only: true
       healthcheck:
         test: ["CMD-SHELL", "exit", "0"]
         interval: 5m
       environment:
         AppName: "Telegraf"
         ConfigInstance: "Telegraf1"
         CertType: "pem,zmq"
         DEV_MODE: ${DEV_MODE}
         no_proxy: "${ETCD_HOST},ia_influxdbconnector"
         NO_PROXY: "${ETCD_HOST},ia_influxdbconnector"
         ETCD_HOST: ${ETCD_HOST}
         ETCD_CLIENT_PORT: ${ETCD_CLIENT_PORT}
         MQTT_BROKER_HOST: ${HOST_IP}
         INFLUX_SERVER: ${HOST_IP}
         INFLUXDB_PORT: $INFLUXDB_PORT
         ETCD_PREFIX: ${ETCD_PREFIX}
       ports:
         - 65078:65078
       networks:
         - eii
       volumes:
         - "vol_temp_telegraf:/tmp/"
         - "vol_eii_socket:${SOCKET_DIR}"
         - ./Certificates/Telegraf:/run/secrets/Telegraf
         - ./Certificates/rootca:/run/secrets/rootca

.. note:: \ : If user wants to add telegraf output plugin in telegraf instance, modify `docker-compose.yml <https://github.com/open-edge-insights/ts-telegraf/blob/master/docker-compose.yml>`_\ , `docker-compose.yml <https://github.com/open-edge-insights/ts-telegraf/blob/master/docker-compose.yml>`_ and telegraf configuration(.conf) files.


   #. Add publisher configuration in `config.json <https://github.com/open-edge-insights/ts-telegraf/blob/master/config.json>`_\ :

   .. code-block:: sh

        {
            "config": {

                ...,
                "<output plugin instance_name>": {
                    "measurements": ["*"],
                    "profiling": "true"
                }
            },
            "interfaces": {
                ...,
                "Publishers": [
                    ...,
                    {
                        "Name": "<output plugin instance_name>",
                        "Type": "zmq_tcp",
                        "EndPoint": "0.0.0.0:<publisher port>",
                        "Topics": [
                            "*"
                        ],
                        "AllowedClients": [
                            "*"
                        ]
                    }

                ]
            }
        }

   Example:

   .. code-block:: sh

        {
            "config": {

                ...,
                "publisher1": {
                    "measurements": ["*"],
                    "profiling": "true"
                }
            },
            "interfaces": {
                ...,
                "Publishers": [
                    ...,
                    {
                        "Name": "publisher1",
                        "Type": "zmq_tcp",
                        "EndPoint": "0.0.0.0:65078",
                        "Topics": [
                            "*"
                        ],
                        "AllowedClients": [
                            "*"
                        ]
                    }

                ]
            }
        }


   #. 
      Expose "publisher port" in `docker-compose.yml <https://github.com/open-edge-insights/ts-telegraf/blob/master/docker-compose.yml>`_ file:

      .. code-block:: sh

         ia_telegraf<ConfigInstance number>:
          ...
          ports:
            - <publisher port>:<publisher port>

   Example:

   .. code-block:: sh

          ia_telegraf<ConfigInstance number>:
            ...
            ports:
              - 65078:65078


   #. 
      Add eii_msgbus output plugin in Telegraf instance config file `config <https://github.com/open-edge-insights/ts-telegraf/blob/master/config>`_\ /$ConfigInstance/$ConfigInstance.conf (for production mode) and
      `config <https://github.com/open-edge-insights/ts-telegraf/blob/master/config>`_\ /$ConfigInstance/$ConfigInstance_devmode.conf (for developer mode).

      [[outputs.eii_msgbus]]
      instance_name = "\ :raw-html-m2r:`<publisher instance>`\ "

   Example:
   For $ConfigInstance = 'Telegraf1'

   User needs to add following section in `config <https://github.com/open-edge-insights/ts-telegraf/blob/master/config>`_\ /Telegraf1/Telegraf1.conf (for production mode) and `config <https://github.com/open-edge-insights/ts-telegraf/blob/master/config>`_\ /Telegraf1/Telegraf1_devmode.conf (for developer mode)

   .. code-block::

      [[outputs.eii_msgbus]]
      instance_name = "publisher1"



* 
  After that, user will need to run the **builder.py** script command to allow the changes of the docker file take place.

  .. code-block:: sh

       cd [WORK_DIR]/IEdgeInsights/build
       python3 builder.py

* 
  User will need to provision, build, and bring up all the containers again by using the following command.

  .. code-block:: sh

       cd [WORK_DIR]/IEdgeInsights/build/
       docker-compose -f docker-compose-build.yml build
       docker-compose up -d ia_configmgr_agent
       # Check `$ docker logs -f ia_configmgr_agent` if Provision is Done
       docker-compose up -d

* 
  Based on above example, user can check the telegraf service will have multiple  container as by using docker ps command.

**Note:** It's been practice followed by many users, to keep the configuration in a modular way. One way to achieve the same could be keeping the additional configuration inside ``Telegraf/config/$ConfigInstance/telegraf.d``. For example, create a directory ``telegraf.d`` in ``Telegraf/config/config/$ConfigInstance``\ :

.. code-block:: sh

      mkdir config/$ConfigInstance/telegraf.d
      cd config/$ConfigInstance/telegraf.d

Keep additional configuration files inside that directory and pass the whole command to start the Telegraf in docker-compose.yml file as following:

.. code-block:: sh

    command: ["telegraf -config=/etc/Telegraf/$ConfigInstance/$ConfigInstance.conf -config-directory=/etc/Telegraf/$ConfigInstance/telegraf.d"]
