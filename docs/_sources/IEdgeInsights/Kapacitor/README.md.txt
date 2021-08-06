# Point-data (Time-series data) analytics introduction

Any integral value that gets generated over time, we can say it is a point data.
The examples can be :
* Temperature at a different time in a day.
* Number of oil barrels processed per minute.

By doing the analytics over point data, the factory can have an anomaly detection mechanism.
That's where the PointDataAnalytics come into the picture.

IEdgeInsights uses the [TICK stack](https://www.influxdata.com/time-series-platform/)
to do point data analytics.

IEdgeInsights has a temperature anomaly detection example for demonstrating the time-series data analytics flow.

The high-level flow of the data, in the example, can be seen as MQTT-temp-sensor-->Telegraf-->Influx-->Kapacitor-->Influx.

MQTT-temp-sensor simulator sends the data to the Telegraf. Telegraf just sends the same data to the
Influx and Influx send it to Kapacitor. Kapacitor does anomaly detection and publishes the results back to
Influx.

Here,
Telegraf is the TICK stack component and supporting the number of input plug-ins for data ingestion.
Influx is a time-series database.
Kapacitor is an analytics engine where users can write custom analytics plug-ins (TICK scripts).

## Starting the example

1. To start the mqtt-temp-sensor, please refer [tools/mqtt-publisher/README.md](../tools/mqtt-publisher/README.md).

2. In case, if SI wants to use the IEdgeInsights only for Point Data Analytics,
   then comment Video use case containers ia_video_ingestion and ia_video_analytics in [build/docker-compose.yml](../build/docker-compose.yml)

3. Starting the EII.
   To start the EII in production mode, provisioning is required. For more information on provisioning
   please refer to [README](../README.md#provision).
   After provisioning, please follow the below commands
   ```
   cd build
   docker-compose build
   docker-compose up -d
   ```

   To start the EII in developer mode, please refer to the [README](../README.md#provision).

4. To verify the output please check the output of below command
   ```
   docker logs -f ia_influxdbconnector
   ```

   Below is the snapshot of sample output of the ia_influxdbconnector command.
   ```
   I0822 09:03:01.705940       1 pubManager.go:111] Published message: map[data:point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 temperature=19.29358085726703,ts=1566464581.6201317 1566464581621377117]
   I0822 09:03:01.927094       1 pubManager.go:111] Published message: map[data:point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 temperature=19.29358085726703,ts=1566464581.6201317 1566464581621377117]
   I0822 09:03:02.704000       1 pubManager.go:111] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 ts=1566464582.6218634,temperature=27.353740759929877 1566464582622771952]
   ```

   The data can be visualized using the Grafana dashboard, to know more refer [Grafana/README.md](../Grafana/README.md)

## Purpose of Telegraf
Telegraf is one of the data entry points for IEdgeInsights. It supports many input plugins, which can be used for
point data ingestion. In the above example, the MQTT input plugin of Telegraf is used. And below is the configuration
of the plugin.

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
        "temperature/simulated/0",
        ]
        name_override = "point_data"
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

The production mode Telegraf configuration file is
[Telegraf/config/telegraf.conf](../Telegraf/config/telegraf.conf) and in developer mode,
the configuration file is
[Telegraf/config/telegraf_devmode.conf](../Telegraf/config/telegraf_devmode.conf).

For more information on the supported input and output plugins please refer
[https://docs.influxdata.com/telegraf/v1.10/plugins/](https://docs.influxdata.com/telegraf/v1.10/plugins/)

## Purpose of Kapacitor

  About Kapacitor and UDF
  * User can write the custom anomaly detection algorithm in PYTHON/GOLANG. And these algorithms will be called as
    UDF (user-defined function). These algorithms have to follow certain API standards so that the Kapacitor will be able to
    call these UDFs at run time.

  * IEdgeInsights has come up with the sample UDF written in GOLANG. Kapacitor is subscribed to the InfluxDB, and
    gets the temperature data. After getting this data, Kapacitor calls these UDF, which detects the anomaly in the temperature
    and sends back the results to Influx.

  * The sample Go UDF is at [go_classifier.go](udf/go_point_classifier.go) and
    the tick script  is at [go_point_classifier.tick](TICK_script/go_point_classifier.tick)

  * The sample Python UDF is at [py_classifier.py](udf/py_point_classifier.py) and
    the tick script  is at [py_point_classifier.tick](TICK_script/py_point_classifier.tick)

    For more details, on Kapacitor and UDF, please refer below links
    i)  Writing a sample UDF at [anomaly detection](https://docs.influxdata.com/kapacitor/v1.5/guides/anomaly_detection/)
    ii) UDF and kapacitor interaction [here](https://docs.influxdata.com/kapacitor/v1.5/guides/socket_udf/)

  * In production mode the Kapacitor config file is
    [Kapacitor/config/kapacitor.conf](./config/kapacitor.conf)
    and in developer mode the config file would be
    [Kapacitor/config/kapacitor_devmode.conf](./config/kapacitor_devmode.conf)

## Custom UDFs available in the [udfs](udfs) directory

  * UNIX Socket based UDFs

    1. go_classifier.go: Filter the points based on temperature (data > 20 and < 25 filtered out).

    2. py_classifier.py: Filter the points based on temperature (data > 20 and < 25 filtered out).

    3. profiling_udf.go: Add the profiling (time taken to process the data) data in the points.

    4. temperature_classifier.go: Filter the points based on temperature (data < 25 filtered out).

    5. humidity_classifier.py: Filter the points based on humidity (data < 25 filtered out).

  * Process based UDFs

    1. rfc_classifier.py: Random Forest Classification algo sample. This UDF can be used as profiling udf as well.

## Steps to configure the UDFs in kapacitor.

  * Keep the custom UDFs in the [udfs](udfs) directory and the TICK script in the [tick_scripts](tick_scripts) directory.

  * Keep the training data set (if any) required for the custom UDFs in the [training_data_sets](training_data_sets) directory.

  * For python UDFs, if any external python package dependency needs to be installed. To install the python package using pip, it can be added in the [requirements.txt](requirements.txt) file and to install the python package using conda, it can be added in the [conda_requirements.txt](conda_requirements.txt) file.

  * Modify the udf section in the [kapacitor.conf](config/kapacitor.conf) and in the [kapacitor_devmode.conf](config/kapacitor_devmode.conf).
    Mention the custom UDF in the conf
    for example
    ```
    [udf.functions.customUDF]
      socket = "/tmp/socket_file"
      timeout = "20s"
    ```

  * In case of go/python based UDF, update the values of keys named "type", "name", "tick_script", "task_name", in the
    [config.json](config.json)file.
    for example
    ```
    "task": [{
         "tick_script": "py_point_classifier.tick",
         "task_name": "py_point_classifier",
         "udfs": [{
             "type": "python",
             "name": "py_classifier"
         }]
    }]
    ```

  * In case of, tick only UDF, update the values of keys named "tick_script", "task_name", in the [config.json](config.json)file.
    for example
    ```
    "task": [{
         "tick_script": "simple_logging.tick",
         "task_name": "simple_logging"
         }]
    ```

    ### Note:
    1. By default, go_classifier and rfc_classifier is configured.

    2. Mention the TICK script udf function same as configured in the Kapacitor config file.
       For example, UDF Node in the TICK script
       ```
       @py_point_classifier()
       ```
       should be same as
       ```
       [udf.functions.py_point_classifier]
          socket = "/tmp/socket_file"
          timeout = "20s"
       ```

    3. go/python based UDF should listen on the same socket file as mentioned in the the udf section in the
       [kapacitor.conf](config/kapacitor.conf) and in the [kapacitor_devmode.conf](config/kapacitor_devmode.conf).
       For example
       ```
       [udf.functions.customUDF]
         socket = "/tmp/socket_file"
         timeout = "20s"
       ```

    4. In case of process based UDFs, provide the correct path of the code within the container
       in the [kapacitor.conf](config/kapacitor.conf) and in the [kapacitor_devmode.conf](config/kapacitor_devmode.conf).
       By default, the files and directories will be copied into the container under "/EII" director. It is recommended to keep the custom UDFs
       in the [udfs](udfs) directory, the path of the custom UDF will be "/EII/udfs/customUDF_name" as shown in below example.
       If the UDF is kept in different path please modify the path in the args accordingly.

       The PYTHONPATH of the Kapacitor agent directory is "/EII/go/src/github.com/influxdata/kapacitor/udf/agent/py/". How to pass
       it is shown in the below example.

       For example
       ```
       [udf.functions.customUDF]
          prog = "python3.7"
          args = ["-u", "/EII/udfs/customUDF"]
          timeout = "60s"
          [udf.functions.customUDF.env]
             PYTHONPATH = "/go/src/github.com/influxdata/kapacitor/udf/agent/py/"
       ```

  * Do the [provisioning](../README.md#provision) and run the EII stack.

## Step to run the samples of multiple UDFs in a single task and mulitple tasks using single UDF

  * please refer to the [samples/README](samples/README.md)

# Kapacitor input and output plugins

## Purpose of plugins

  The plugins allow Kapacitor to interact directly with EII message bus. They use message bus publisher/subscriber
  interface. Using these plugins Kapacitor can now receive data from various EII publishers and send data 
  to various EII subscribers.
  Hence, it's possible to have a time-series use case without InfluxDB and Kapacitor can act as
  an independent analytical engine.

  A simple use case flow can be:

        MQTT-temp-sensor-->Telegraf-->Kapacitor-->TimeseriesProfiler

## Using input plugin

  * Configure the EII input plugin in [config/kapacitor.conf](config/kapacitor.conf) and [config/kapacitor_devmode.conf](config/kapacitor_devmode.conf)
    For example:
    ```
    [eii]
      enabled = true
    ```
  * Edit [config.json](config.json) to add a subscriber under interfaces.

    For example, to receive data published by Telegraf:

    **TCP mode**
    ```
        "Subscribers": [
            {
                "Name": "telegraf_sub",
                "Type": "zmq_tcp",
                "EndPoint": "ia_telegraf:65077",
                "PublisherAppName": "Telegraf",
                "Topics": [
                    "*"
                ]
            }
        ]
    ```

    **IPC mode**
    ```
        "Subscribers": [
            {
                "Name": "telegraf_sub",
                "Type": "zmq_ipc",
                "EndPoint": {
                    "SocketDir": "/EII/sockets",
                    "SocketFile": "telegraf-out"
                },
                "PublisherAppName": "Telegraf",
                "Topics": [
                    "*"
                ]
            }
        ]
    ```
    **Note: For IPC mode, we need to specify the 'EndPoint' as a dict of 'SocketDir' and 'SocketFile' in case
    where 'Topics' is [*] (as in the above example).
    In case of single topic the 'EndPoint' can be defined as below (as in the example of Kapacitor o/p plugin):**
    ```
   	"EndPoint": "/EII/sockets"
    ```

    The received data will be available in the 'eii' storage for the tick scripts to use.
  * Create/modify a tick script to process the data and configure the same in [config.json](config.json).
    For example, use the stock [tick_scripts/eii_input_plugin_logging.tick](tick_scripts/eii_input_plugin_logging.tick) which logs the data received from 'eii'
    storage onto the kapacitor log file (residing in the container at /tmp/log/kapacitor/kapacitor.log).
    ```
        "task": [
           {
             "tick_script": "eii_input_plugin_logging.tick",
             "task_name": "eii_input_plugin_logging"
           }
        ]
    ```
  * Do the [provisioning](../README.md#provision) and run the EII stack.

    The subscribed data will now be available in the above logs file which can be viewed with the 
    command below:
    ```
    docker exec ia_kapacitor tail -f /tmp/log/kapacitor/kapacitor.log
    ```

## Using output plugin
  * Create/modify a tick script to use 'eiiOut' node to send the data using publisher interface
    For example, you may modify the profiling UDF as below:
    ```
    dbrp "eii"."autogen"

    var data0 = stream
        |from()
                .database('eii')
                .retentionPolicy('autogen')
                .measurement('point_data')
        @profiling_udf()
        |eiiOut()
                .pubname('sample_publisher')
                .topic('sample_topic')

    ```
  * Add a publisher interface added to [config.json](config.json) with the same publisher name and topic 
    i.e. 'sample_publisher' and 'sample_topic' respectively as in the above example.
    For example:
   
    **TCP mode**
    ```
        "Publishers": [
            {
                "Name": "sample_publisher",
                "Type": "zmq_tcp",
                "EndPoint": "0.0.0.0:65034",
                "Topics": [
                    "sample_topic"
                ],
                "AllowedClients": [
                    "TimeSeriesProfiler"
                ]
            }
        ]

    ```

    **IPC mode**
    ```
        "Publishers": [
            {
                "Name": "sample_publisher",
                "Type": "zmq_ipc",
                "EndPoint": "/EII/sockets",
                "Topics": [
                    "sample_topic"
                ],
                "AllowedClients": [
                    "TimeSeriesProfiler"
                ]
            }
        ]

    ```
  * Do the [provisioning](../README.md#provision) and run the EII stack.

## Using input/output plugin with RFC udf
  * Add the RFC task to [config.json](config.json):
    ```
        "task": [
           {
             "tick_script": "rfc_task.tick",
             "task_name": "random_forest_sample",
             "udfs": [{
                 "type": "python",
                 "name": "rfc_classifier"
             }]
           }
        ]
    ```
  * Modify the [rfc_task.tick](tick_scripts/rfc_task.tick) as below, for example:
    ```
    dbrp "eii"."autogen"

    var data0 = stream
            |from()
                    .database('eii')
                    .retentionPolicy('autogen')
                    .measurement('ts_data')
            |window()
            .period(3s)
            .every(4s)

    data0
            @rfc()
            |eiiOut()
                    .pubname('sample_publisher')
                    .topic('sample_topic')
    ```
  * Add a publisher interface added to [config.json](config.json) with the same publisher name and topic 
    i.e. 'sample_publisher' and 'sample_topic' respectively as in the above example.
    For example:


    **TCP mode**
    ```
        "Publishers": [
            {
                "Name": "sample_publisher",
                "Type": "zmq_tcp",
                "EndPoint": "0.0.0.0:65034",
                "Topics": [
                    "sample_topic"
                ],
                "AllowedClients": [
                    "TimeSeriesProfiler",
                    "EmbSubscriber",
                    "GoSubscriber"
                ]
            }
        ]

    ```

    **IPC mode**
    ```
        "Publishers": [
            {
                "Name": "sample_publisher",
                "Type": "zmq_ipc",
                "EndPoint": "/EII/sockets",
                "Topics": [
                    "sample_topic"
                ],
                "AllowedClients": [
                    "TimeSeriesProfiler",
                    "EmbSubscriber",
                    "GoSubscriber"
                ]
            }
        ]

    ```

  * Do the [provisioning](../README.md#provision) and run the EII stack.
