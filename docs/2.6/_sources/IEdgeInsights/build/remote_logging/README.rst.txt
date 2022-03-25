
OEI distributed services centralized logging using ELK
======================================================

"ELK" is the acronym for three open source projects: Elasticsearch, Logstash, and Kibana. Elasticsearch is a search and analytics engine. Logstash is a server‑side data processing pipeline that ingests data from multiple sources simultaneously, transforms it, and then sends it to a "stash" like Elasticsearch. Kibana lets users visualize data with charts and graphs in Elasticsearch.

The guide below demonstrates how the logs of OEI services running in distributed environment can be centralized via ELK stack. It allows you to search all the logs in a single place.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights (OEI). This is due to the product name change of EII as OEI.

Prerequisites
-------------


#. 
   OEI centralized logging using ELK expects a set of config, interfaces & public private keys to be present in ETCD as a prerequisite.
    To achieve this, please ensure an entry for ``remote_logging`` with its relative path from `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory is set in the use-case yml file present in `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory. An example has been provided below:

   .. code-block:: sh

           AppContexts:
           - Grafana
           - InfluxDBConnector
           - Kapacitor
           - Telegraf
           - build/remote_logging

#. 
   With the above pre-requisite done, please run the below command:

   .. code-block:: sh

           python3 builder.py -f ./usecases/<use-case>.yml

#. 
   Generate the certificates required to run the Kibana Server using the following command

   .. code-block::

       ./generate_kibana_certs.sh test-server-ip

#. 
   The OEI centralized logging architecture can be visualized as eii-containers--->rsyslog--->logstash--->elasticsearch--->kibana

#. 
   The eii-containers logs has to be forwarded to the rsyslog, running onto the local system.
   For that, the eii-container's logging driver need to be syslog.
   Anyone wants to forward the eii-container's log to the local rsyslog has to modify the file named '/etc/docker/daemon.json'.
   The sample 'daemon.json' is availale in the directory.
   After the modification of 'daemon.json' docker daemon has to be restarted using the below command

   .. code-block:: sh

      sudo systemctl restart docker

   In case of sending the logs over a secure channel from docker daemon to rsyslog, please refer the 'daemon.json.secure'.
   In this sample configuration, please replace
   CA_CERT_PATH : CA certificate path
   DOCKER_CERT_PATH : Docker certificate path
   DOCKER_KEY_PATH : Docker key path

#. 
   The rsyslog has to forward the received logs from containers to logstash.
   The file named 'eii.conf' has to be copied into the directory name '/etc/rsyslog.d/'
   After copying this file, install rsyslog module using the below command(The rsyslog module
   installation is one time activity).

   .. code-block:: sh

      sudo apt-get install rsyslog-gnutls

   Then rsyslog service need to restarted using the below command

   .. code-block:: sh

      sudo systemctl restart rsyslog

   For more information on the rsyslog,please refer `https://www.rsyslog.com/plugins/ <https://www.rsyslog.com/plugins/>`_

   In case of receiving the logs from docker over the secure channel, please refer the sample config file 'eii.conf.secure'
   In this sample configuration, please replace
   CA_CERT_PATH : CA certificate path
   DOCKER_CERT_PATH : Docker certificate path
   DOCKER_KEY_PATH : Docker key path
   If case of docker daemon sending logs to remote rsyslog agent, then replace '127.0.0.1' (address="127.0.0.1") with the
   actual IP address of the machine where rsyslog is running.

#. 
   To start ELK containers Please follow below commands

   .. code-block:: sh

      sudo sysctl -w vm.max_map_count=262144

#. 
   Refer `README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_ to provision and run the services

   .. code-block:: sh

      docker-compose -f docker-compose-build.yml build 
      docker-compose -f docker-compose.yml up -d # this will work in both DEV or PROD mode automatically

   Please visit `https://host-ip:5601 <https://host-ip:5601>`_ for viewing the logs in KIBANA UI.
   Create new index pattern with the elasticsearch indices\ ``logstash`` pattern

   ..

      **NOTE**\ : The certificate attached to kibana is self signed and has to be accepted in
      browser as an exception. The attached certificates are sample certificates only and need to
      be replaced for production environment.
      host-ip : Actual IP address of the machine where kibana is running


#. 
   After above 4 steps, please start/restart the EII services.
   Note:

   ..

      **NOTE**\ : In case of logs are not showing in the KIBANA UI, restart the rsyslog service
      using the command ``sudo systemctl restart rsyslog``

