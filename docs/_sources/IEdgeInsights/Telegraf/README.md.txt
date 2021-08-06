# Telegraf
# Contents:

1. [Telegraf in brief](#Telegraf-in-brief)

2. [Telegraf's default configuration](#Telegraf's-default-configuration)

3. [MQTT sample configuration and tool to test it](#MQTT-sample-configuration-and-tool-to-test-it)

4. [Enabling EII message bus input plugin in Telegraf](#Enabling-EII-message-bus-input-plugin-in-Telegraf)

5. [Advanced: Multiple plugin sections of EII message bus input plugin](#Advanced:-Multiple-plugin-sections-of-EII-message-bus-input-plugin)

6. [Enabling EII message bus Output plugin in Telegraf](#Enabling-EII-message-bus-output-plugin-in-Telegraf)

7. [Advanced: Multiple plugin sections of EII message bus output plugin](#Advanced:-Multiple-plugin-sections-of-EII-message-bus-output-plugin)

8. [Optional: Adding multiple telegraf instances](#Optional:-Adding-multiple-telegraf-instances)

## Telegraf in brief
-------------------
Telegraf is a part of TICK stack (https://www.influxdata.com/time-series-platform/). This is a plugin based agent.  Telegraf has many input and output plugins. In EII's basic configuration, it's being used for data ingestion. However EII framework does not restrict Telegraf's any of the features. In EII basic configuration uses Telegraf for data ingestion and sending it to influxdb. 

## Telegraf's default configuration
1. Telegraf starts with the default configuration which is present at [config/Telegraf/Telegraf.conf](./config/Telegraf/Telegraf.conf) (for the dev mode the name is 'Telegraf_devmode.conf'). By default the below plugins are enabled.
	- 	MQTT input plugin (**[[inputs.mqtt_consumer]]**)
	- 	EII message bus input plugin (**[[inputs.eii_msgbus]]**)
	- 	Influxdb output plugin (**[[outputs.influxdb]]**)

Telegraf will be started using script 'telegraf_start.py. This script will get the configuration from ETCD first and then it will start the  Telegraf service by picking the right configuration depending on the developer/production mode. By default only single instance of Telegraf container runs (named 'ia_telegraf').


## MQTT sample configuration and tool to test it.

* To test with MQTT publisher in k8s helm environment, Please update 'MQTT_BROKER_HOST' Environment Variables in [values.yaml](./helm/values.yaml) with HOST IP address of the system where MQTT Broker is running.

* To test with remote mqtt broker in docker environment, Please update 'MQTT_BROKER_HOST' Environment Variables in [docker-compose.yml](docker-compose.yml) with HOST IP address of the system where MQTT Broker is running.

```
  ia_telegraf:
    environment:
      ...
      MQTT_BROKER_HOST: '<HOST IP address of the system where MQTT Broker is running>'
```

* Telegraf Instance can be configured with pressure point data ingestion. In the following example, the MQTT input plugin of Telegraf is configured to read pressure point data and stores into ‘point_pressure_data’ measurement.

	```
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

	```

* To start the mqtt-publisher with pressure data,
   ```sh
   $ cd ../tools/mqtt/publisher/
   ```
   change the command option in [docker-compose.yml](../tools/mqtt/publisher/docker-compose.yml) to:
   ```sh
   ["--pressure", "10:30"]
   ```
   Build and Run mqtt publisher:
   ```sh
   $ docker-compose up --build -d
   ```
please refer [tools/mqtt/README.md](../tools/mqtt/README.md)

## Enabling EII message bus input plugin in Telegraf

**Purpose**
Receiving the data from EII message bus and storing it into influx db, at scale.

**Overview**

The plugin subscribes to configured topic / topic prefixes. Plugin has component
called subscriber which receives the data from eii message bus.
After receiving the data, depending on configuration, the plugin process
the data, either synchronously or asynchronously. 
- In synchronous processing, the receiver thread (thread which receives the data from EII message bus) is also resposible for the processing of the data (json parsing). After processing the previous data only, the receiver thread process next data available on the EII message bus. 
- In asynchronous processing the  receiver thread  receives the data and put it into the queue. There will be pool of threads which will be dequeing the data from the queue and processing it.

**Guidelines for choosing the data processing options are**
- Synchronous option: When the ingestion rate is consistent
- Asynchronous options: There are two options
1.  Topic specific queue+threadpool : Frequent spike in ingestion rate for a specific topic
2.  Global queue+threadpool : Sometimes spike in ingestion rate for a specific topic

**Configuration of the plugin**
Configuration of the plugin is divided into two parts
1. ETCD configuration
2. Configuration in Telegraf.conf file [config/Telegraf/Telegraf.conf](./config/Telegraf/Telegraf.conf)

**ETCD configuration**
Since this is eii message bus plugin and so it's part of EII framework,
message bus configuration and plugin's topic specific configuration is kept into etcd.
Below is the sample configuration

```json
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
```
**Brief description of the configuration**.
Like any other EII service Telegraf has 'config' and 'interfaces'  sections.  "interfaces" are the eii interface details. Let's have more information of "config" section.

config :  Contains the configuration of the influxdb (**"influxdb"**) and  eii messagebus input plugin (**"default"**). In the above sample configuration, the **"default"** is an instance name. This instance name is referenced from the Telegraf's configuration file [config/Telegraf/Telegraf.conf](./config/Telegraf/Telegraf.conf)

- topics_info : This is an array of topic prefix configuration, where user specifies, how the data from topic-prefix should be processed. Below is the  way every line in the topic information should be interpreted.
	1. "topic-pfx1:temperature:10:2" : Process data from topic prefix 'topic-pfx1' asynchronously using the dedicated queue of length 10 and dedicated thread pool of size 2. And the processed data will be stored at measurement named 'temperature' in influxdb.
	2. "topic-pfx2:pressure::" : Process data from topic prefix 'topic-pfx2' asynchronously using the global queue and global thread pool. And the processed data will be stored at measurement named 'pressure' in influxdb.
	3. "topic-pfx3:humidity" : Process data synchronously. And the processed data will be stored at measurement named 'humidity' in influxdb.

    *Note : In case, topic specific configuration is not mentioned, then by default, data  gets processed synchronously and measurement name would be, same as topic name.*

- queue_len : Global queue length.
- num_worker : Global thread pool size.
- profiling : This is to enable profiling mode of this plugin (value can be either "true" or "false"). In profiling mode every point will get the following information and will be kept into same measurement as that of point.

	1. Total time spent in plugin (time in ns)
	2. Time spent in queue (in case of asynchronous processing only and time in ns)
	3. Time spend in json processing (time in ns)
	4. The name of the thread pool and the thread id which processed the point.

	*Note: The name of the global thread pool is "GLOBAL". For a topic specific thread pool, the name is "for-$topic-name".*
                                                                                                                                  |

**Configuration at Telegraf.conf file**

The plugin instance name is an additional key, kept into plugin configuration section. This key is used to fetch the configuration from ETCD. Below is the minimmum, sample configuration with single plugin instance.

    [[inputs.eii_msgbus]]
    **instance_name = "default"**
    data_format = "json"
    json_strict = true

Here, the value **'default'**  acts as a key in the file **[config.json](./config.json)**. For this key, there is configuration in the **'interfaces' and 'config'** sections of the file **[config.json](./config.json)**. So the value of** 'instance_name'** acts as a connect/glue between the Telegraf configuration **[config/Telegraf/Telegraf.conf](./config/Telegraf/Telegraf.conf)** and the **ETCD configuration [config.json](./config.json)**

*Note: 
Since it’s telegraf input plugin, the telegraf’s parser configuration
has to be in Telegraf.conf file. The more information of the telegraf json parser plugin can be be found at https://github.com/influxdata/telegraf/tree/master/plugins/parsers/json.*
In case if there are multiple telegraf instances, then the location of the Telegraf configuration files would be different. For more details please refer the section [Optional: Adding multiple telegraf instance](#Optional: Adding multiple telegraf instance)

## Advanced: Multiple plugin sections of EII message bus input plugin
Like any other Telegraf plugin user can keep multiple configuration sections of the EII message bus input plugin in the **[config/Telegraf/Telegraf.conf](./config/Telegraf/Telegraf.conf)** file.

Let's have an example for the same. Let's assume there are two EII apps, one with the AppName "EII_APP1" and another with the AppName "EII_APP2", which are publishing the data to eii message bus.
*The Telegraf's ETCD configuration for the same is*
```json
{
   "config":{
      "publisher1":{
         "topics_info":[
            "topic-pfx1:temperature:10:2",
            "topic-pfx2:pressure::",
            "topic-pfx3:humidity"
         ],
         "queue_len":100,
         "num_worker":5,
         "profiling":"true"
      },
      "publisher2":{
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
            "Name":"publisher1",
            "EndPoint":"EII_APP1_container_name:5569",
            "Topics":[
               "*"
            ],
            "Type":"zmq_tcp",
			"PublisherAppName": "EII_APP1"
         },
         {
            "Name":"publisher2",
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
```
*The Telegraf.conf configuration sections for the same is*

    [[inputs.eii_msgbus]]
    instance_name = "publisher1"
    data_format = "json"
    json_strict = true
    
    [[inputs.eii_msgbus]]
    instance_name = "publisher2"
    data_format = "json"
    json_strict = true
    
## Enabling EII message bus Output plugin in Telegraf

**Purpose**
Receiving the data from Telegraf Input Plugin and publish data to eii msgbus.

**Configuration of the plugin**
Configuration of the plugin is divided into two parts
1. ETCD configuration
2. Configuration in Telegraf.conf file [config/Telegraf/Telegraf.conf](./config/Telegraf/Telegraf.conf)

**ETCD configuration**
Since this is eii message bus plugin and so it’s part of EII framework, message bus configuration and plugin’s topic specific configuration is kept into etcd.
Below is the sample configuration

```json
{
    "config": {
        "publisher1": {
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
            }
        ]
    }
}
```

**Brief description of the configuration**.
Like any other EII service Telegraf has 'config' and 'interfaces'  sections.  "interfaces" are the eii interface details. Let's have more information of "config" section.

config :  Contains eii messagebus output plugin (**"publisher1"**). In the above sample configuration, the **"publisher1"** is an instance name. This instance name is referenced from the Telegraf's configuration file [config/Telegraf/Telegraf.conf](./config/Telegraf/Telegraf.conf)

- measurements : This is an array of measurements configuration, where user specifies, which measurement data should be published in msgbus.
- profiling : This is to enable profiling mode of this plugin (value can be either "true" or "false").


**Configuration at Telegraf.conf file**

The plugin instance name is an additional key, kept into plugin configuration section. This key is used to fetch the configuration from ETCD. Below is the minimmum, sample configuration with single plugin instance.

    [[outputs.eii_msgbus]]
    **instance_name = "publisher1"**


Here, the value **'publisher1'**  acts as a key in the file **[config.json](./config.json)**. For this key, there is configuration in the **'interfaces' and 'config'** sections of the file **[config.json](./config.json)**. So the value of** 'instance_name'** acts as a connect/glue between the Telegraf configuration **[config/Telegraf/Telegraf.conf](./config/Telegraf/Telegraf.conf)** and the **ETCD configuration [config.json](./config.json)**

## Advanced: Multiple plugin sections of EII message bus output plugin
Like any other Telegraf plugin user can keep multiple configuration sections of the EII message bus output plugin in the **[config/Telegraf/Telegraf.conf](./config/Telegraf/Telegraf.conf)** file.

*The Telegraf's ETCD configuration for the same is*
```json
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
```
*The Telegraf.conf configuration sections for the same is*

    [[outputs.eii_msgbus]]
    instance_name = "publisher1"
    
    [[outputs.eii_msgbus]]
    instance_name = "publisher2"

## Run Telegraf Input output plugin in IPC mode
- User needs to modify interfaces section of **[config.json](./config.json)** to run in IPC mode as following:

```
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
                "Name": "publisher1",
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

```
## Optional: Adding multiple telegraf instances
- User can add multiple instances of Telegarf. For that user needs to add additional environment variable named 'ConfigInstance' in docker-compose.yml file. For  every additional telegraf instance, there has to be additional compose section in the docker-compose.yml file. 
- The configuration for every instance has to be in the telegraf image. The standard to be followed is described as below. 

    For instance named $ConfigInstance the telegraf configuration has to be kept in the repository at 
    [config](./config)/$ConfigInstance/$ConfigInstance.conf (for production mode) and 
    [config](./config)/$ConfigInstance/$ConfigInstance_devmode.conf (for developer mode).

    The same files will be available inside the respective container at
    '/etc/Telegraf/$ConfigInstance/$ConfigInstance.conf' (for production mode)
    '/etc/Telegraf/$ConfigInstance/$ConfigInstance_devmode.conf' (for developer mode)

Let's have some of the examples.

Example1:
For $ConfigInstance = 'Telegraf1'

- The location of the Telegraf configuration would be
[config](./config)/Telegraf1/Telegraf1.conf (for production mode) and [config](./config)/Telegraf1/Telegraf1_devmode.conf (for developer mode)
- The additional docker compose section which has to be manually added in the file 'docker-compose.yml' would be
	```
	  ia_telegraf1:
	    depends_on:
	      - ia_common
	    build:
	      context: $PWD/../Telegraf
	      dockerfile: $PWD/../Telegraf/Dockerfile
	      args:
	        EII_VERSION: ${EII_VERSION}
	        EII_UID: ${EII_UID}
	        DOCKER_REGISTRY: ${DOCKER_REGISTRY}
	        TELEGRAF_SOURCE_TAG: ${TELEGRAF_SOURCE_TAG}
	        TELEGRAF_GO_VERSION: ${TELEGRAF_GO_VERSION}
	    container_name: ia_telegraf1
	    hostname: ia_telegraf1
	    image: ${DOCKER_REGISTRY}openedgeinsights/ia_telegraf:${EII_VERSION}
	    restart: unless-stopped
	    ipc: "none"
	    read_only: true
	    environment:
	      AppName: "Telegraf"
	      ConfigInstance: "Telegraf1"
	      CertType: "pem,zmq"
	      DEV_MODE: ${DEV_MODE}
	      no_proxy: "${ETCD_HOST},ia_influxdbconnector"
	      NO_PROXY: "${ETCD_HOST},ia_influxdbconnector"
	      ETCD_HOST: ${ETCD_HOST}
	      MQTT_BROKER_HOST: $MQTT_BROKER_HOST
	      INFLUX_SERVER: 'ia_influxdbconnector'
	      INFLUXDB_PORT: $INFLUXDB_PORT
	      ETCD_PREFIX: ${ETCD_PREFIX}
	    user: ${EII_UID}
            networks:
              - eii
	    volumes:
	      - "vol_temp_telegraf:/tmp/"
	      - "vol_eii_socket:${SOCKET_DIR}"
	    secrets:
	      - ca_etcd
	      - etcd_Telegraf_cert
	      - etcd_Telegraf_key
	```

**Note**: It's been practice followed by many users, to keep the configuration in a modular way. One way to achieve the same could be keeping the additional configuration inside 'Telegraf/config/$ConfigInstance/telegraf.d. For example:

create a directory 'telegraf.d' inside 'Telegraf/config/config/$ConfigInstance' :
   ```
   $ mkdir config/$ConfigInstance/telegraf.d
   $ cd config/$ConfigInstance/telegraf.d
  ```
   keep additional configuration files inside that directory and pass the whole command to start the Telegraf in docker-compose.yml file as following:
	
```
	command: ["telegraf -config=/etc/Telegraf/$ConfigInstance/$ConfigInstance.conf -config-directory=/etc/Telegraf/$ConfigInstance/telegraf.d"]
```

