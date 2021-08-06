
EII distributed services centralized logging using ELK


#. 
   The EII centralized logging architecture can be visualized as eii-containers--->rsyslog--->logstash--->elasticsearch--->kibana

#. 
   The eii-containers logs has to be forwarded to the rsyslog, running onto the local system. 
   For that, the eii-container's logging driver need to be syslog. 
   Anyone wants to forward the eii-container's log to the local rsyslog has to modify the file named '/etc/docker/daemon.json'.
   The sample 'daemon.json' is availale in the directory.
   After the modification of 'daemon.json' docker daemon has to be restarted using the below command

   .. code-block:: sh

      $ sudo systemctl restart docker

   In case of sending the logs over a secure channel from docker daemon to rsyslog, please refer the 'daemon.json.secure'.
   In this sample configuration, please replace 
   CA_CERT_PATH : CA certificate path
   DOCKER_CERT_PATH : Docker certificate path
   DOCKER_KEY_PATH : Docker key path

#. 
   The rsyslog has to forward the received logs from containers to logstash.
   The file named 'eii.conf' has to be copyied into the directory name '/etc/rsyslog.d/'
   After copying this file, install rsyslog module using the below command(The rsyslog module
   installation is one time activity).

   .. code-block:: sh

      $ sudo apt-get install rsyslog-gnutls

   Then rsyslog service need to restarted using the below command

   .. code-block:: sh

      $ sudo systemctl restart rsyslog

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

      $ sudo sysctl -w vm.max_map_count=262144

   To start in dev mode, elk.yml can be used.

   .. code-block:: sh

      $ docker-compose -f elk.yml up -d

   To start in prod mode, elk_prod.yml can be used.

   .. code-block:: sh

      $ docker-compose -f elk_prod.yml up -d

   Pleas visit `https://localhost:5601 <https://localhost:5601>`_ for viewing the logs in KIBANA.
   Note:

   ..

      **NOTE**\ : The certificate attached to kibana is self signed and has to be accepted in
      browser as an exception. The attached certificates are sample certificates only and need to
      be replaced for production environment.

      To access Kibana UI in external machines, replace 'localhost' in `.env <https://github.com/open-edge-insights/eii-core/blob/master/env>`_ with the
      actual IP address of the machine where kibana is running.


#. 
   After above 4 steps, please start/restart the EII services.
