
**Contents**


* `\ ``InfluxDBConnector Module`` <https://open-edge-insights.github.io/IEdgeInsights/InfluxDBConnector/#influxdbconnector-module>`_

  * `\ ``Configuration`` <https://open-edge-insights.github.io/IEdgeInsights/InfluxDBConnector/#configuration>`_

InfluxDBConnector
=================

InfluxDBConnector service stores the timeseries/video metadata received over the EII messagebus to InfluxDB database.

The high level logical flow of InfluxDBConnector service is as below:


#. InfluxDBConnector service will subscribe to the InfluxDB and start the EII messagebus
   publisher, EII messagebus subscriber threads, and EII messagebus request reply thread
   based on PubTopics, SubTopics and QueryTopics configuration.
#. EII messagebus subscriber thread connects to the PUB socket of messagebus on which
   the data is published by VideoAnalytics and push it to the InfluxDB
#. EII messagebus publisher thread will publish the point data ingested by the telegraf
   and the classifier result coming out of the point data analytics.
#. EII messagebus reply request service will receive the InfluxDB select query and
   response with the historical data.

Configuration
-------------

All the InfluxDBConnector module configuration are added into etcd (distributed
key-value data store) under ``AppName`` as mentioned in the
environment section of this app's service definition in docker-compose.

If ``AppName`` is ``InfluxDBConnector``\ , then the app's config would look like as below
 for ``/InfluxDBConnector/config`` key in Etcd:

.. code-block::

       "influxdb": {
               "retention": "1h30m5s",
               "username": "admin",
               "password": "admin123",
               "dbname": "datain",
               "ssl": "True",
               "verifySsl": "False",
               "port": "8086"
           }

In case of nested json data, by default InfluxDBConnector will flatten the nested json and push
the flat data to InfluxDB, In order to avoid the flattening of any particular nested key please mention the
tag key in the **\ `config.json <https://github.com/open-edge-insights/eii-influxdb-connector/blob/master/config.json>`_\ ** file. Currently "defects" key is ignored from flattening. Every key to be ignored has to be in newline.

 for example,

.. code-block::

      ignore_keys = [ "Key1", "Key2", "Key3" ]

By default, all the keys in the data schema will be pushed to InfluxDB as fields. In case if tags are present in data schema,
it can be mentioned in the **\ `config.json <https://github.com/open-edge-insights/eii-influxdb-connector/blob/master/config.json>`_\ ** file then the data pushed to InfluxDB, will have fields and tags both.
Currently, no tags are present in the data scheme and tag_keys is kept blank in the config file.

for example,

.. code-block::

     tag_keys = [ "Tag1", "Tag2" ]

For more details on Etcd secrets and messagebus endpoint configuration, visit `Etcd_Secrets_Configuration.md <https://github.com/open-edge-insights/eii-core/blob/master/Etcd_Secrets_Configuration.md>`_ and
`MessageBus Configuration <https://github.com/open-edge-insights/eii-core/blob/master/common/libs/ConfigMgr/README.md#interfaces>`_ respectively.
