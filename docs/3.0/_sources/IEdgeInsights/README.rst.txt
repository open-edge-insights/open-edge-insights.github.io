
Contents
========


* `Contents <#contents>`__

  * `About Open Edge Insights <#about-open-edge-insights>`__
  * `Minimum system requirements <#minimum-system-requirements>`__
  * `Install Open Edge Insights from GitHub <#install-open-edge-insights-from-github>`__

    * `Task 1: Get OEI codebase from GitHub <#task-1-get-oei-codebase-from-github>`__
    * `Task 2: Install prerequisites <#task-2-install-prerequisites>`__

      * `Run the pre-requisite script <#run-the-pre-requisite-script>`__
      * `Optional steps <#optional-steps>`__

        * `Method 1 <#method-1>`__
        * `Method 2 <#method-2>`__

    * `Task 3: Generate deployment and configuration files <#task-3-generate-deployment-and-configuration-files>`__

      * `Use the Builder script <#use-the-builder-script>`__
      * `Generate consolidated files for all applicable OEI services <#generate-consolidated-files-for-all-applicable-oei-services>`__
      * `Generate consolidated files for a subset of OEI services <#generate-consolidated-files-for-a-subset-of-oei-services>`__
      * `Generate multi-instance configs using the Builder <#generate-multi-instance-configs-using-the-builder>`__
      * `Generate benchmarking configs using Builder <#generate-benchmarking-configs-using-builder>`__
      * `Add OEI services <#add-oei-services>`__
      * `Distribute the OEI container images <#distribute-the-oei-container-images>`__
      * `List of OEI services <#list-of-oei-services>`__

    * `Task 4: Build and run the OEI video and time series use cases <#task-4-build-and-run-the-oei-video-and-time-series-use-cases>`__

      * `Build the OEI stack <#build-the-oei-stack>`__
      * `Run OEI services <#run-oei-services>`__

        * `OEI Provisioning <#oei-provisioning>`__

          * `Start OEI in Dev mode <#start-oei-in-dev-mode>`__
          * `Start OEI in Profiling mode <#start-oei-in-profiling-mode>`__
          * `Run OEI provisioning service and rest of the OEI stack services <#run-oei-provisioning-service-and-rest-of-the-oei-stack-services>`__

      * `Push the required OEI images to docker registry <#push-the-required-oei-images-to-docker-registry>`__

  * `Video pipeline analytics <#video-pipeline-analytics>`__

    * `Enable camera-based video ingestion <#enable-camera-based-video-ingestion>`__
    * `Use video accelerators in ingestion and analytics containers <#use-video-accelerators-in-ingestion-and-analytics-containers>`__

      * `To run on USB devices <#to-run-on-usb-devices>`__
      * `To run on MYRIAD devices <#to-run-on-myriad-devices>`__

        * `Troubleshooting issues for MYRIAD (NCS2) devices <#troubleshooting-issues-for-myriad-ncs2-devices>`__

      * `To run on HDDL devices <#to-run-on-hddl-devices>`__

        * `Troubleshooting issues for HDDL devices <#troubleshooting-issues-for-hddl-devices>`__

      * `To run on Intel(R) Processor Graphics (GPU/iGPU) <#to-run-on-intelr-processor-graphics-gpuigpu>`__

    * `Custom User Defined Functions <#custom-user-defined-functions>`__

  * `Time series analytics <#time-series-analytics>`__
  * `OEI multi-node cluster deployment <#oei-multi-node-cluster-deployment>`__

    * `With k8s orchestrator <#with-k8s-orchestrator>`__

  * `OEI tools <#oei-tools>`__
  * `OEI Uninstaller <#oei-uninstaller>`__
  * `Debugging options <#debugging-options>`__
  * `Web Deployment Tool <#web-deployment-tool>`__
  * `Troubleshooting guide <#troubleshooting-guide>`__

About Open Edge Insights
------------------------

Open Edge Insights (OEI) is a set of pre-validated ingredients for integrating video and time series data analytics on edge compute nodes. OEI includes modules to enable data collection, storage, and analytics for both time series and video data.

.. note:: 
   In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as OEI. This is due to the product name change of EII as OEI.


Minimum system requirements
---------------------------

The following are the minimum system requirements to run OEI:

.. list-table::
   :header-rows: 1

   * - System Requirement
     - Details
   * - Processor
     - 8th generation Intel® CoreTM processor onwards with Intel® HD Graphics or Intel® Xeon® processor
   * - RAM
     - Minimum 16 GB
   * - Hard drive
     - Minimum 256 GB
   * - Operating system
     - Ubuntu 18.04 or Ubuntu 20.04


.. note:: 


   * To use OEI, ensure that you are connected to the internet.
   * The recommended RAM capacity for video analytics pipeline is 16 GB. The recommended RAM for time series analytics pipeline is 4 GB with Intel® Atom processors.
   * OEI is validated on Ubuntu 18.04 and Ubuntu 20.04 but you can install OEI stack on other Linux distributions with support for docker-ce and docker-compose tools.


Install Open Edge Insights from GitHub
--------------------------------------

You can download and install OEI from the `Open Edge Insights GitHub repository <https://github.com/open-edge-insights/>`_. For more information on manifest files, refer `Readme <https://github.com/open-edge-insights/eii-manifests/blob/master/README.md>`_.

To install OEI, perform the tasks in the following order:


* `Task 1: Get OEI codebase from GitHub <#task-1-get-oei-codebase-from-github>`__
* `Task 2: Install prerequisites <#task-2-install-prerequisites>`__
* `Task 3: Generate deployment and configuration files <#task-3-generate-deployment-and-configuration-files>`__
* `Task 4: Build and run the OEI video and time series use cases <#task-4-build-and-run-the-oei-video-and-time-series-use-cases>`__

Task 1: Get OEI codebase from GitHub
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To get the OEI codebase, complete the following steps:


#. 
   To install the repo tool, run the following commands:

   .. code-block:: sh

       curl https://storage.googleapis.com/git-repo-downloads/repo > repo
       sudo mv repo /bin/repo
       sudo chmod a+x /bin/repo

#. 
   To create a working directory, run the following command:

   .. code-block:: sh

       mkdir -p <work-dir>

#. 
   To initialize the working directory using the repo tool, run the following commands:

   .. code-block:: sh

       cd <work-dir>
       repo init -u "https://github.com/open-edge-insights/eii-manifests.git"

#. 
   To pull all the projects mentioned in the manifest xml file, with the default or specific revision mentioned for each project, run the following command:

   .. code-block:: sh

       repo sync

Task 2: Install prerequisites
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``pre_requisites.sh`` script automates the installation and configuration of all the prerequisites required for building and running the OEI stack. The prerequisites are as follows:


* docker daemon
* docker client
* docker-compose
* Python packages

The ``pre-requisites.sh`` file performs the following:


* Checks if docker and docker-compose is installed in the system. If required, it uninstalls the older version and installs the correct version of docker and docker-compose.
* Configures the proxy settings for the docker client and docker daemon to connect to the internet.
* Configures the proxy settings system-wide (/etc/environment) and for docker. If a system is running behind a proxy, then the script prompts users to enter the proxy address to configure the proxy settings.
* Configures proxy setting for /etc/apt/apt.conf to enable apt updates and installations.

.. note:: 


   * The recommended version of the docker-compose is ``1.29.0``. In versions older than 1.29.0, the video use case docker-compose.yml files and the device_cgroup_rules command may not work.
   * To use versions older than docker-compose 1.29.0, in the ``ia_video_ingestion`` and ``ia_video_analytics`` services, comment out the ``device_cgroup_rules`` command.
   * 
     You can comment out the ``device_cgroup_rules`` command in the ``ia_video_ingestion`` and ``ia_video_analytics`` services to use versions older than 1.29.0 of docker-compose. This can result in limited inference and device support. The following code sample shows how the ``device_cgroup_rules`` commands are commented out:

     .. code-block:: sh

         ia_video_ingestion:
          ...
           #device_cgroup_rules:
              #- 'c 189:* rmw'
              #- 'c 209:* rmw'

   After modifying the ``docker-compose.yml`` file, refer to the ``Using the Builder script`` section. Before running the services using the ``docker-compose up`` command, rerun the ``builder.py`` script.


Run the pre-requisite script
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To run the pre-requisite script, execute the following commands:

.. code-block:: sh

       cd [WORKDIR]/IEdgeInsights/build
       sudo -E ./pre_requisites.sh --help
         Usage :: sudo -E ./pre_requisites.sh [OPTION...]
         List of available options...
         --proxy         proxies, required when the gateway/edge node running OEI (or any of OEI profile) is connected behind proxy
         --help / -h         display this help and exit

.. note:: 

   If the --proxy option is not provided, then script will run without proxy. Different use cases are as follows:


   * 
     Runs without proxy

     .. code-block:: sh

           sudo -E ./pre_requisites.sh

   * 
     Runs with proxy

     .. code-block:: sh

          sudo -E ./pre_requisites.sh --proxy="proxy.intel.com:891"


Optional steps
~~~~~~~~~~~~~~


* If required, you can enable full security for production deployments. Ensure that the host machine and docker daemon are configured per the security recommendation. For more info, see `build/docker_security_recommendation.md <https://github.com/open-edge-insights/eii-core/blob/master/build/docker_security_recommendation.md>`_.
* If required, you can enable log rotation for docker containers using any of the following methods:

Method 1
""""""""

Set the logging driver as part of the docker daemon. This applies to all the docker containers by default.


#. 
   Configure the json-file driver as the default logging driver. For more info, see `JSON File logging driver <https://docs.docker.com/config/containers/logging/json-file/>`_. The sample json-driver configuration which can be copied to ``/etc/docker/daemon.json`` is as follows:

   .. code-block:: json

               {
                 "log-driver": "json-file",
                 "log-opts": {
                 "max-size": "10m",
                 "max-file": "5"
                 }
             }

#. 
   Run the following command to reload the docker daemon:

   .. code-block:: sh

       sudo systemctl daemon-reload

#. 
   Run the following command to restart docker:

   .. code-block:: sh

       sudo systemctl restart docker

Method 2
""""""""

Set logging driver as part of docker compose which is container specific. This overwrites the 1st option (i.e /etc/docker/daemon.json). The following example shows how to enable the logging driver only for the video_ingestion service:

.. code-block:: json

       ia_video_ingestion:
         ...
         ...
         logging:
           driver: json-file
           options:
           max-size: 10m
     max-file: 5

Task 3: Generate deployment and configuration files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After downloading OEI from the release package or Git, run the commands mentioned in this section from the ``[WORKDIR]/IEdgeInsights/build/`` directory.

Use the Builder script
~~~~~~~~~~~~~~~~~~~~~~

To use the Builder script, run the following command:

.. code-block:: sh

       python3 builder.py -h
       usage: builder.py [-h] [-f YML_FILE] [-v VIDEO_PIPELINE_INSTANCES]
                           [-d OVERRIDE_DIRECTORY]
       optional arguments:
           -h, --help            show this help message and exit
           -f YML_FILE, --yml_file YML_FILE
                               Optional config file for list of services to include.
                               Eg: python3 builder.py -f video-streaming.yml
                               (default: None)
           -v VIDEO_PIPELINE_INSTANCES, --video_pipeline_instances VIDEO_PIPELINE_INSTANCES
                               Optional number of video pipeline instances to be
                               created. Eg: python3 builder.py -v 6 (default:
                               1)
           -d OVERRIDE_DIRECTORY, --override_directory OVERRIDE_DIRECTORY
                               Optional directory consisting of benchmarking
                               configs to be present in each app directory. Eg:
                               python3 builder.py -d benchmarking (default:
                               None)

Generate consolidated files for all applicable OEI services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using the Builder tool, OEI auto-generates the configuration files that are required for deploying the OEI services on a single or multiple nodes. The Builder tool auto-generates the consolidated files by getting the relevant files from the OEI service directories that are required for different OEI use-cases. The Builder tool parses the top-level directories under the ``IEdgeInsights`` directory to generate the consolidated files.

The following table shows the list of consolidated files and their details:

Table: Consolidated files

.. list-table::
   :header-rows: 1

   * - file name
     - Description
   * - docker-compose.yml
     - Consolidated ``docker-compose.yml`` file used to launch OEI docker containers in each single node using the ``docker-compose`` tool
   * - docker-compose.override.yml
     - Consolidated ``docker-compose-dev.override.yml`` of every app that is generated only in DEV mode for OEI deployment on a given single node using ``docker-compose`` tool
   * - docker-compose-build.yml
     - Consolidated ``docker-compose-build.yml`` file having OEI base images and ``depends_on`` and ``build`` keys required for building OEI services
   * - docker-compose-push.yml
     - Consolidated ``docker-compose-push.yml`` file (same as ``docker-compose.yml`` file with just dummy ``build`` key added), used for pushing OEI services to docker registry
   * - eii_config.json
     - Consolidated ``config.json`` of every app which will be put into etcd during provisioning
   * - values.yaml
     - Consolidated ``values.yaml`` of every app inside helm-eii/eii-deploy directory, which is required to deploy OEI services via helm
   * - Template yaml files
     - Files copied from helm/templates directory of every app to helm-eii/eii-deploy/templates directory, which are required to deploy OEI services via helm


.. note:: 


   * If you modify an individual OEI app/service directory files, make sure to re-run the ``builder.py`` script before running the OEI stack to regenerate the updated consolidated files as above.
   * Manual editing of consolidated files is not recommended. Instead make changes to the respective files in the OEI app/service directories and use the ``builder.py`` script to generate the consolidated files.
   * Enter the secret credentials in the ``# Service credentials`` section of the `.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ file if you are trying to run that OEI app/service. In case the required credentials are not present, the ``builder.py`` script would be prompting till all the required credentails are entered. Please protect this `.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ file from being read by other users by applying a suitable file access mask.
   * The `builder_config.json <https://github.com/open-edge-insights/eii-core/blob/master/build/builder_config.json>`_ is the config file for the ``builder.py`` script and it contains the following keys:

     * ``subscriber_list``\ : This key contains a list of services that act as a subscriber to the stream being published.
     * ``publisher_list``\ : This key contains a list of services that publishes a stream of data.
     * ``include_services``\ : This key contains the mandatory list of services that are required to be included when the Builder is run without the ``-f`` flag.
     * ``exclude_services``\ : This key contains the mandatory list of services that are required to be excluded when the Builder is run without the ``-f`` flag.
     * ``increment_rtsp_port``\ : This is a boolean key used for incrementing the port number for the RTSP stream pipelines.


To generate the consolidated files, run the following command:

.. code-block:: sh

     python3 builder.py

Generate consolidated files for a subset of OEI services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Builder uses a yml file for configuration. The config yml file consists of a list of services to include. You can mention the service name as the path relative to ``IEdgeInsights`` or full path to the service in the config yml file.
To include only a certain number of services in the OEI stack, you can add the -f or yml_file flag of builder.py. You can find the examples of yml files for different use cases as follows:


* 
  `Azure <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/video-streaming-azure.yml>`_

  The following example shows running Builder with the -f flag:

  .. code-block:: sh

          python3 builder.py -f usecases/video-streaming.yml

* 
  **Main use cases**

.. list-table::
   :header-rows: 1

   * - Usecase
     - yaml file
   * - Video + Timeseries
     - `build/usecases/video-timeseries.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/video-timeseries.yml>`_
   * - Video
     - `build/usecases/video.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/video.yml>`_
   * - Timeseries
     - `build/usecases/time-series.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/time-series.yml>`_



* **Video pipeline sub use cases**

.. list-table::
   :header-rows: 1

   * - Usecase
     - yaml file
   * - Video streaming
     - `build/usecases/video-streaming.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/video-streaming.yml>`_
   * - Video streaming and historical
     - `build/usecases/video-streaming-storage.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/video-streaming-storage.yml>`_
   * - Video streaming with AzureBridge
     - `build/usecases/video-streaming-azure.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/video-streaming-azure.yml>`_
   * - Video streaming and custom udfs
     - `build/usecases/video-streaming-all-udfs.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/video-streaming-all-udfs.yml>`_


When you run the multi-instance config, a ``build/multi_instance`` directory is created in the build directory. Based on the number of ``video_pipeline_instances`` specified, that many directories of VideoIngestion and VideoAnalytics is created in the ``build/multi_instance`` directory.

The following section provides an example for running the Builder to generate the multi-instance boiler plate config for 3 streams of **video-streaming** use case.

Generate multi-instance configs using the Builder
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If required, you can generate the multi-instance ``docker-compose.yml`` and ``config.json`` files using the Builder. You can use the ``-v`` or ``video_pipeline_instances`` flag of the Builder to generate boiler plate config for the multiple -stream use cases. The ``-v`` or ``video_pipeline_instances`` flag creates the multi-stream boiler plate config for the ``docker-compose.yml`` and ``eii_config.json`` files.

The following example shows running builder to generate the multi-instance boiler plate config for 3 streams of video-streaming use case:

.. code-block:: sh

       python3 builder.py -v 3 -f usecases/video-streaming.yml

Using the previous command for 3 instances, the ``build/multi_instance`` directory consists of VideoIngestion1, VideoIngestion2, VideoIngestion3 and VideoAnalytics1, VideoAnalytics2, VideoAnalytics3 directories. Initially each directory will have the default ``config.json`` and the ``docker-compose.yml`` files that are present within the ``VideoIngestion`` and the ``VideoAnalytics`` directories.

.. code-block:: example

       ./build/multi_instance/
       |-- VideoAnalytics1
       |   |-- config.json
       |   `-- docker-compose.yml
       |-- VideoAnalytics2
       |   |-- config.json
       |   `-- docker-compose.yml
       |-- VideoAnalytics3
       |   |-- config.json
       |   `-- docker-compose.yml
       |-- VideoIngestion1
       |   |-- config.json
       |   `-- docker-compose.yml
       |-- VideoIngestion2
       |   |-- config.json
       |   `-- docker-compose.yml
       `-- VideoIngestion3
           |-- config.json
           `-- docker-compose.yml

 You can edit the configs of each of these streams within the ``build/multi_instance`` directory. To generate the consolidated ``docker compose`` and ``eii_config.json`` file, rerun the ``builder.py`` command.

.. note:: 


   * The multi-instance feature support of Builder works only for the video pipeline i.e., **usecases/video-streaming.yml** use case alone and not with any other use case yml files like **usecases/video-streaming-storage.yml** and so on. Also, it doesn't work for cases without the ``-f`` switch. The previous example will work with any positive number for ``-v``. To learn more about using the multi-instance feature with the DiscoverHistory tool, see `Multi-instance feature support for the builder script with the DiscoverHistory tool <https://github.com/open-edge-insights/eii-tools/blob/master/DiscoverHistory/README.md#multi-instance-feature-support-for-the-builder-script-with-the-discoverhistory-tool>`_.
   * If you are running the multi-instance config for the first time, it is recommended to not change the default ``config.json`` file and the ``docker-compose.yml`` file in the ``VideoIngestion`` and ``VideoAnalytics`` directories.
   * If you are not running the multi-instance config for the first time, the existing ``config.json`` and ``docker-compose.yml`` files in the ``build/multi_instance`` directory will be used to generate the consolidated ``eii-config.json`` and ``docker-compose`` files.
   * The docker-compose.yml files present within the ``build/multi_instance`` directory will have the updated service_name, container_name, hostname, AppName, ports and secrets for that respective instance.
   * The config.json file in the ``build/multi_instance`` directory will have the updated Name, Type, Topics, Endpoint, PublisherAppname, ServerAppName and AllowedClients for the interfaces section and incremented rtsp port number for the config section of that respective instance.
   * Ensure that all the containers are down before running the multi-instance configuration. Run the ``docker-compose down`` command before running ``builder.py`` for the multi-instance configuration.


Generate benchmarking configs using Builder
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use the ``-d`` or the ``override_directory`` flag to provide a different set of ``docker-compose.yml`` and ``config.json`` files other than the existing files in every service directory. The ``-d`` or the ``override_directory`` flag indicates to search for the required set of files within a directory provided by the flag.
For example, to pick files from a directory named benchmarking, you can run the following command:

.. code-block:: sh

        python3 builder.py -d benchmarking

.. note:: 


   * If you use the override directory feature of the builder then include all the 3 files mentioned in the previous example. If you do not include a file in the override directory then the Builder will omit that service in the final config that is generated.
   * Adding the ``AppName`` of the subscriber or client container in the ``subscriber_list of builder_config.json`` allows you to spawn a single subscriber or client container that is subscribing or receiving on multiple publishers or server containers.
   * Multiple containers specified by the ``-v`` flag is spawned for services that are not mentioned in the ``subscriber_list``. For example, if you run Builder with ``–v 3`` option and ``Visualizer`` is not added in the ``subscriber_list`` of ``builder_config.json`` then 3 instances of Visualizer are spawned. Each instance subscribes to 3 VideoAnalytics services. If Visualizer is added in the ``subscriber_list`` of ``builder_config.json``\ , a single Visualizer instance subscribing to 3 multiple VideoAnalytics is spawned.


Add OEI services
~~~~~~~~~~~~~~~~

This section provides information about adding a new service, subscribing to the `VideoAnalytics <https://github.com/open-edge-insights/video-analytics>`_\ , and publishing it on a new port.
Add a service to the OEI stack as a new directory in the `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory. The Builder registers and runs any service present in its own directory in the `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory. The directory should contain the following:


* A ``docker-compose.yml`` file to deploy the service as a docker container. The ``AppName`` is present in the ``environment`` section in the ``docker-compose.yml`` file. Before adding the ``AppName`` to the main ``build/eii_config.json``\ , it is appended to the ``config`` and ``interfaces`` as ``/AppName/config`` and ``/AppName/interfaces``.
* A ``config.json`` file that contains the required config for the service to run after it is deployed. The ``config.json`` consists of the following:

  * A ``config`` section, which includes the configuration-related parameters that are required to run the application.
  * An ``interfaces`` section, which includes the configuration of how the service interacts with other services of the OEI stack.

.. note:: 
   For more information on adding new OEI services, refer to the OEI sample apps at `Samples <https://github.com/open-edge-insights/eii-samples/blob/master/README.md>`_ written in C++, python, and Golang using the OEI core libraries.
   The following example shows how to write the **config.json** for any new service, subscribe to **VideoAnalytics**\ , and publish on a new port:


.. code-block:: javascript

       {
           "config": {
               "paramOne": "Value",
               "paramTwo": [1, 2, 3],
               "paramThree": 4000,
               "paramFour": true
           },
           "interfaces": {
               "Subscribers": [
                   {
                       "Name": "default",
                       "Type": "zmq_tcp",
                       "EndPoint": "127.0.0.1:65013",
                       "PublisherAppName": "VideoAnalytics",
                       "Topics": [
                           "camera1_stream_results"
                       ]
                   }
               ],
               "Publishers": [
                   {
                       "Name": "default",
                       "Type": "zmq_tcp",
                       "EndPoint": "127.0.0.1:65113",
                       "Topics": [
                           "publish_stream"
                       ],
                       "AllowedClients": [
                           "ClientOne",
                           "ClientTwo",
                           "ClientThree"
                       ]
                   }
               ]
           }
       }

The ``config.json`` file consists of the following key and values:


* value of the ``config`` key is the config required by the service to run.
* value of the ``interfaces`` key is the config required by the service to interact with other services of OEI stack over the Message Bus.
* the ``Subscribers`` value in the ``interfaces`` section denotes that this service should act as a subscriber to the stream being published by the value specified by ``PublisherAppName`` on the endpoint mentioned in value specified by ``EndPoint`` on topics specified in value of ``Topic`` key.
* the ``Publishers`` value in the ``interfaces`` section denotes that this service publishes a stream of data after obtaining and processing it from ``VideoAnalytics``. The stream is published on the endpoint mentioned in value of ``EndPoint`` key on topics mentioned in the value of ``Topics`` key.
* the services mentioned in the value of ``AllowedClients`` are the only clients that can subscribe to the published stream, if it is published securely over the Message Bus.

.. note:: 


   * Like the interface keys, OEI services can also have "Servers" and "Clients" interface keys. For more information, refer `config.json <https://github.com/open-edge-insights/video-ingestion/blob/master/config.json>`_ of the ``VideoIngestion`` service and `config.json <https://github.com/open-edge-insights/eii-tools/blob/master/SWTriggerUtility/config.json>`_ of SWTriggerUtility tool.
   * For more information on the ``interfaces`` key responsible for the Message Bus endpoint configuration, refer `common/libs/ConfigMgr/README.md#interfaces <https://github.com/open-edge-insights/eii-configmgr/blob/master/README.md#interfaces>`_.
   * For the etcd secrets configuration, in the new OEI service or app ``docker-compose.yml`` file, add the following volume mounts with the right ``AppName`` env value:

   .. code-block:: yaml

      ...
       volumes:
         - ./Certificates/[AppName]:/run/secrets/[AppName]:ro
         - ./Certificates/rootca/cacert.pem:/run/secrets/rootca/cacert.pem:ro


Distribute the OEI container images
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The OEI services are available as pre-built container images in the Docker Hub at https://hub.docker.com/u/openedgeinsights. To access the services that are not available in the Docker Hub, build from source before running the ``docker-compose up -d`` command.
For example:

.. code-block:: sh

   # Update the DOCKER_REGISTRY value in [WORKDIR]/IEdgeInsights/build/.env as DOCKER_RESISTRY=<docker_registry> (Make sure `docker login <docker_registry>` to the docker reigstry works)
   cd [WORKDIR]/IEdgeInsights/build
   # Base images that needs to be built
   docker-compose -f docker-compose-build.yml build ia_eiibase
   docker-compose -f docker-compose-build.yml build ia_common
   # Assuming here that the `python3 builder.py` step has been executed and ia_kapacitor
   # Service exists in the generated compose files
   docker-compose -f docker-compose-build.yml build ia_kapacitor
   docker-compose up -d
   # Push all the applicable OEI images to <docker_registry>. Ensure to use the same DOCKER_REGISTRY value on the deployment machine while deployment
   docker-compose -f docker-compose-push.yml push

The list of pre-built container images that are accessible at https://hub.docker.com/u/openedgeinsights is as follows:


* **Provisioning images**

  * openedgeinsights/ia_configmgr_agent

* **Common OEI images applicable for video and time series use cases**

  * openedgeinsights/ia_etcd_ui
  * openedgeinsights/ia_influxdbconnector
  * openedgeinsights/ia_rest_export
  * openedgeinsights/ia_opcua_export

* **Video pipeline images**

  * openedgeinsights/ia_video_ingestion
  * openedgeinsights/ia_video_analytics
  * openedgeinsights/ia_web_visualizer
  * openedgeinsights/ia_visualizer
  * openedgeinsights/ia_imagestore
  * openedgeinsights/ia_azure_bridge
  * openedgeinsights/ia_azure_simple_subscriber

* **Time series pipeline images**

  * openedgeinsights/ia_grafana
  * openedgeinsights/ia_telegraf
  * openedgeinsights/ia_kapacitor

.. note:: 
   Additionally, we have ``openedgeinsights/ia_edgeinsights_src`` image available at https://hub.docker.com/u/openedgeinsights which consists of source code of the GPL/LGPL/AGPL components of the OEI stack.


List of OEI services
~~~~~~~~~~~~~~~~~~~~

Based on requirement, you can include or exclude the following OEI services in the ``[WORKDIR]/IEdgeInsights/build/docker-compose.yml`` file:


* Provisioning Service - This service is a prerequisite and cannot be excluded from the ``docker-compose.yml`` file.

  * `ConfigMgrAgent <https://github.com/open-edge-insights/eii-configmgr-agent/blob/master/README.md>`_

* Common OEI services

  * `EtcdUI <https://github.com/open-edge-insights/eii-etcd-ui/blob/master/README.md>`_
  * `InfluxDBConnector <https://github.com/open-edge-insights/eii-influxdb-connector/blob/master/README.md>`_
  * `OpcuaExport <https://github.com/open-edge-insights/eii-opcua-export/blob/master/README.md>`_ - Optional service to read from the VideoAnalytics container to publish data to opcua clients.
  * `RestDataExport <https://github.com/open-edge-insights/eii-rest-data-export/blob/master/README.md>`_ - Optional service to read the metadata and image blob from the InfluxDBConnector and ImageStore services respectively.

* Video related services

  * `VideoIngestion <https://github.com/open-edge-insights/video-ingestion/blob/master/README.md>`_
  * `VideoAnalytics <https://github.com/open-edge-insights/video-analytics/blob/master/README.md>`_
  * `Visualizer <https://github.com/open-edge-insights/video-native-visualizer/blob/master/README.md>`_
  * `WebVisualizer <https://github.com/open-edge-insights/video-web-visualizer/blob/master/README.md>`_
  * `ImageStore <https://github.com/open-edge-insights/video-imagestore/blob/master/README.md>`_
  * `AzureBridge <https://github.com/open-edge-insights/eii-azure-bridge/blob/master/README.md>`_
  * `FactoryControlApp <https://github.com/open-edge-insights/eii-factoryctrl/blob/master/README.md>`_ - Optional service to read from the VideoAnalytics container if you want to control the light based on the defective or non-defective data

* Time series-related services

  * `Telegraf <https://github.com/open-edge-insights/ts-telegraf/blob/master/README.md>`_
  * `Kapacitor <https://github.com/open-edge-insights/ts-kapacitor/blob/master/README.md>`_
  * `Grafana <https://github.com/open-edge-insights/ts-grafana/blob/master/README.md>`_
  * `ZMQ Broker <https://github.com/open-edge-insights/eii-zmq-broker/blob/master/README.md>`_

Task 4: Build and run the OEI video and time series use cases
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: 


   * For running the OEI services in the IPC mode, ensure that the same user is mentioned in the publisher and subscriber services.
   * If the publisher service is running as root such as ``VI``\ , ``VA``\ , then the subscriber service should also run as root. For example, in the ``docker-compose.yml``\ file, if you have specified ``user: ${EII_UID}`` in the publisher service, then specify the same ``user: ${EII_UID}`` in the subscriber service. If you have not specified a user in the publisher service then don't specify the user in the subscriber service.
   * If services needs to be running in multiple nodes in the TCP mode of communication, msgbus subscribers, and clients of ``AppName`` are required to configure the "EndPoint" in ``config.json`` with the ``HOST_IP`` and the ``PORT`` under ``Subscribers/Publishers`` or ``Clients/Servers`` interfaces section.
   * Ensure that the port is being exposed in the ``docker-compose.yml`` of the respective ``AppName``.
     For example, if the ``"EndPoint": <HOST_IP>:65012`` is configured in the ``config.json`` file, then expose the port ``65012`` in the ``docker-compose.yml`` file of the ``ia_video_ingestion`` service.


.. code-block:: yaml

       ia_video_ingestion:
         ...
         ports:
           - 65012:65012

Run all the following OEI build and commands from the ``[WORKDIR]/IEdgeInsights/build/`` directory.
OEI supports the following use cases to run the services mentioned in the ``docker_compose.yml`` file. Refer to Task 2 to generate the docker_compose.yml file based on a specific use case. For more information and configuration, refer to the ``[WORK_DIR]/IEdgeInsights/README.md`` file.

Build the OEI stack
~~~~~~~~~~~~~~~~~~~

.. note:: 


   * This is an optional step if you want to use the OEI pre-built container images and don't want to build from source. For more details, refer: `Distribution of OEI container images <#distribution-of-oii-container-images>`__
   * Base OEI services like ``ia_eiibase``\ , ``ia_video_common``\ , and so on, are needed only at the build time and not at the runtime.
     Run the following command to build all OEI services in the ``build/docker-compose-build.yml`` along with the base OEI services.


.. code-block:: sh

   docker-compose -f docker-compose-build.yml build

If any of the services fails during the build, then run the following command to build the service again:

.. code-block:: sh

   docker-compose -f docker-compose-build.yml build --no-cache <service name>

Run OEI services
~~~~~~~~~~~~~~~~

.. note:: 
   Ensure to run ``docker-compose down`` from  `build <https://github.com/open-edge-insights/eii-core/blob/master/build>`_ directory prior to bringing up OEI stack
   in order to remove running containers and avoid sync issues where other services have come up before ``ia_configmgr_agent``
   container has completed the provisioning step
   If the images tagged with the ``EII_VERSION`` label, as in the `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ do not exist locally in the system but are available in the Docker Hub, then the images will be pulled during the ``docker-compose up``\ command.


OEI Provisioning
""""""""""""""""

The OEI provisioning is taken care by the ``ia_configmgr_agent`` service which gets lauched as part of the OEI stack. For more details on the ConfigMgr Agent component, refer to the `Readme <https://github.com/open-edge-insights/eii-configmgr-agent/blob/master/README.md>`_.

Start OEI in Dev mode
#####################

.. note:: 


   * By default, OEI is provisioned in the secure mode.
   * It is recommended to not use OEI in the Dev mode in a production environment. In the Dev mode, all security features, communication to and from the etcd server over the gRPC protocol, and the communication between the OEI services/apps over the ZMQ protocol are disabled.
   * By default, the OEI empty certificates folder `Certificates <https://github.com/open-edge-insights/eii-core/blob/master/Certificates]>`_ will be created in the DEV mode. This happens because of docker bind mounts but it is not an issue.
   * The ``EII_INSTALL_PATH`` in the `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ remains protected both in the DEV and the PROD mode with the Linux group permissions.


Starting OEI in the Dev mode eases the development phase for System Integrators (SI). In the Dev mode, all components communicate over non-encrypted channels. To enable the Dev mode, set the environment variable ``DEV_MODE`` to ``true`` in the ``[WORK_DIR]/IEdgeInsights/build/.env`` file. The default value of this variable is ``false``.

To provision OEI in the developer mode, complete the following steps:


#. Update ``DEV_MODE=true`` in ``[WORK_DIR]/IEdgeInsights/build/.env``.
#. Rerun the ``build/builder.py`` to regenerate the consolidated files.

Start OEI in Profiling mode
###########################

The Profiling mode is used for collecting the performance statistics in OEI. In this mode, each OEI component makes a record of the time needed for processing any single frame. These statistics are collected in the visualizer where System Integrtors (SI) can see the end-to-end processing time and the end-to-end average time for individual frames.

To enable the Profiling mode, in the ``[WORK_DIR]/IEdgeInsights/build/.env`` file, set the environment variable ``PROFILING`` to ``true``.

Run OEI provisioning service and rest of the OEI stack services
###############################################################

.. note:: 


   * Use the `Etcd UI <https://github.com/open-edge-insights/eii-etcd-ui/blob/master/README.md>`_ to make the changes to the service configs post starting the OEI services.
   * As seen in the `build/eii_start.sh <https://github.com/open-edge-insights/eii-core/blob/master/build/eii_start.sh>`_\ , OEI provisioning and deployment happens in a 2 step process where you need to wait for the initialization of the provisioning container (\ ``ia_configmgr_agent``\ ) before bringing up the rest of the stack. Don't use commands like ``docker-compose restart`` as it will randomly restart all the services leading to issues. To restart any service, use command like ``docker-compose restart [container_name]`` or ``docker restart [container_name]``.


.. code-block:: sh

   # The optional TIMEOUT argument passed below is in seconds and if not provided it will wait
   # till the "Provisioning is Done" message show up in `ia_configmgr_agent` logs before
   # bringing up rest of the OEI stack
   cd [WORK_DIR]/IEdgeInsights/build
   ./eii_start.sh [TIMEOUT]

.. code-block:: sh

   # To start the native visualizer service, run this command once in the terminal
   xhost +

On successful run, the Visualizer UI is displays the results of video analytics for all video use cases.

Push the required OEI images to docker registry
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note:: 
   By default, if ``DOCKER_REGISTRY`` is empty in `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ then the images are published to hub.docker.com. Ensure to remove ``openedgeinsights/`` org from the image names while pushing to Docker Hub, as the repository or image names with multiple slashes are not supported. This limitation doesn't exist in other docker registries like the Azure Container Registry (ACR), Harbor registry, and so on.


Run the following command to push all the OEI service docker images in the ``build/docker-compose-push.yml``. Ensure to update the ``DOCKER_REGISTRY`` value in the `.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ file.

.. code-block:: sh

   docker-compose -f docker-compose-push.yml push

Video pipeline analytics
------------------------

This section provides more information about working with the video pipeline.

Enable camera-based video ingestion
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For detailed description on configuring different types of cameras and filter algorithms, refer to the `VideoIngestion/README.md <https://github.com/open-edge-insights/video-ingestion/blob/master/README.md>`_.

Use video accelerators in ingestion and analytics containers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

OEI supports running inference on ``CPU``\ , ``GPU``\ , ``MYRIAD (NCS2)``\ , and ``HDDL`` devices by accepting the ``device`` value ("CPU"|"GPU"|"MYRIAD"|"HDDL"),
part of the ``udf`` object configuration in the ``udfs`` key. The ``device`` field in the UDF config of ``udfs`` key in the ``VideoIngestion`` and ``VideoAnalytics``
configs can be updated at runtime via `EtcdUI <https://github.com/open-edge-insights/eii-etcd-ui>`_ interface, the ``VideoIngestion`` and ``VideoAnalytics``
services will auto-restart.
For more details on the UDF config, refer `common/udfs/README.md <https://github.com/open-edge-insights/video-common/blob/master/udfs/README.md>`_.

.. note:: 
   There is an initial delay of upto ~30 seconds while running inference on ``GPU`` (only for the first frame) as dynamically certain packages get created during runtime.


To run on USB devices
~~~~~~~~~~~~~~~~~~~~~

For actual deployment in case USB camera is required then mount the device node of the USB camera for ``ia_video_ingestion`` service. When multiple USB cameras are connected to host m/c the required camera should be identified with the device node and mounted.
For example, mount the two USB cameras connected to the host machine with device node as ``video0`` and ``video1``.

.. code-block:: yaml

     ia_video_ingestion:
       ...
       devices:
         - "/dev/dri"
         - "/dev/video0:/dev/video0"
         - "/dev/video1:/dev/video1"

.. note:: 
   ``/dev/dri`` is required for graphic drivers.


To run on MYRIAD devices
~~~~~~~~~~~~~~~~~~~~~~~~

.. note:: 
   In the IPC mode when publisher (example, ia_video_ingestion, ia_video_analytics, or custom_udfs) is running with the ``root`` user permissions then the subscribers (example, ia_visualizer,  ia_imagestore, or ia_influxdbconnector) should also run as root.


At runtime, use the ``root`` user permissions to run inference on a ``MYRIAD`` device. To enable the root user at runtime in ``ia_video_ingestion``\ , ``ia_video_analytics``\ , or custom UDF services, add ``user: root`` in the respective ``docker-compose.yml`` file. Refer the following example:

.. code-block:: yaml

     ia_video_ingestion:
       ...
       user: root

.. note:: 

   In the IPC mode when publisher (example, ia_video_ingestion, ia_video_analytics, or custom_udfs) is running with the ``root`` user permissions then the subscribers (For example ia_visualizer, ia_imagestore, ia_influxdbconnectorm, ia_video_profiler etc.) should also run as root by adding ``user: root`` in the respective docker-compose.yml file.
   To enable root user at runtime in ``ia_video_analytics`` or custom UDF services based on ``ia_video_analytics``\ , set ``user: root`` in the respective ``docker-compose.yml`` file. Refer the following example:


.. code-block:: yaml

       ia_video_analytics:
         ...
         user: root

Troubleshooting issues for MYRIAD (NCS2) devices
""""""""""""""""""""""""""""""""""""""""""""""""

If the ``NC_ERROR`` occurs during device initialization of NCS2 stick then use the following workaround. Replug the device for the init, if the NCS2 devices fails to initialize during running OEI. To check if initialization is successful, run **\ *dmesg*\ ** and **\ *lsusb*\ ** as follows:

.. code-block:: sh

   lsusb | grep "03e7" (03e7 is the VendorID and 2485 is one of the  productID for MyriadX)

.. code-block:: sh

         dmesg > dmesg.txt
         [ 3818.214919] usb 3-4: new high-speed USB device number 10 using xhci_hcd
         [ 3818.363542] usb 3-4: New USB device found, idVendor=03e7, idProduct=2485
         [ 3818.363546] usb 3-4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
         [ 3818.363548] usb 3-4: Product: Movidius MyriadX
         [ 3818.363550] usb 3-4: Manufacturer: Movidius Ltd.
         [ 3818.363552] usb 3-4: SerialNumber: 03e72485
         [ 3829.153556] usb 3-4: USB disconnect, device number 10
         [ 3831.134804] usb 3-4: new high-speed USB device number 11 using xhci_hcd
         [ 3831.283430] usb 3-4: New USB device found, idVendor=03e7, idProduct=2485
         [ 3831.283433] usb 3-4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
         [ 3831.283436] usb 3-4: Product: Movidius MyriadX
         [ 3831.283438] usb 3-4: Manufacturer: Movidius Ltd.
         [ 3831.283439] usb 3-4: SerialNumber: 03e72485
         [ 3906.460590] usb 3-4: USB disconnect, device number 11


* If you notice ``global mutex initialization failed`` during device initialization of NCS2 stick, then refer to the following link: https://www.intel.com/content/www/us/en/support/articles/000033390/boards-and-kits.html
* For VPU troubleshooting, refer the following link: https://docs.openvinotoolkit.org/2021.4/openvino_docs_install_guides_installing_openvino_linux_ivad_vpu.html#troubleshooting

To run on HDDL devices
~~~~~~~~~~~~~~~~~~~~~~

Complete the following steps to run inference on HDDL devices:


#. Download the full package for OpenVINO toolkit for Linux version "2021 4.2 LTS"
    (\ ``OPENVINO_IMAGE_VERSION`` used in `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_\ ) from the official website
    (https://software.intel.com/en-us/openvino-toolkit/choose-download/free-download-linux).
#. 
   Refer to the following link to install OpenVINO on the hostsystem


   * 
     OpenVINO install: https://docs.openvinotoolkit.org/2021.4/_docs_install_guides_installing_openvino_linux.html#install-openvino

     ..

        **Note:**
        OpenVINO 2021.4 installation creates a symbolic link to the latest installation with filename as ``openvino_2021`` instead of ``openvino``. You can create a symbolic link with filename as ``openvino`` to the latest installation as follows:

        .. code-block:: sh

            cd /opt/intel
            sudo ln -s <OpenVINO latest installation> openvino

        Example: sudo ln -s openvino_2021.4.752 openvino

        Uninstall the older versions of OpenVINO if it is installed on the host system.


#. 
   Refer the below link and configure HDDL with ``root`` user rights


   * HDDL daemon setup: https://docs.openvinotoolkit.org/2021.4/_docs_install_guides_installing_openvino_linux_ivad_vpu.html

#. 
   Once HDDL setup is complete run the below command with ``root`` user rights.

   .. code-block:: sh

           source /opt/intel/openvino/bin/setupvars.sh
           $HDDL_INSTALL_DIR/bin/hddldaemon

.. note:: 


   * HDDL Daemon should run in a different terminal or in the background on the host system where inference performed.
   * HDDL usecases was tested on hostsystem with Ubuntu 20.04 kernel 5.13.0-27-generic by configuring and running HDDL daemon with ``root`` user rights.
   * HDDL plugin can have the ION driver compatibility issues with some versions of the Ubuntu kernel. If there are compatibility issues then ION driver may not be installed and hddldaemon will make use of shared memory. In order to work with shared memory in docker environment we need to configure and run HDDL with ``root`` user rights.
   * To check the supported Ubuntu kernel versions, refer the `OpenVINO-Release-Notes <https://www.intel.com/content/www/us/en/developer/articles/release-notes/openvino-relnotes.html>`_.
   * For actual deployment, mount only the required devices for services using OpenVINO with HDDL (\ ``ia_video_analytics`` or ``ia_video_ingestion``\ ) in ``docker-compose.yml`` file.
   * For example, mount only the Graphics and HDDL ion device for the ``ia_video_anaytics`` service. Refer to the following code snippet:


.. code-block:: sh

       ia_video_analytics:
         ...
         devices:
                 - "/dev/dri"
                 - "/dev/ion:/dev/ion"

Troubleshooting issues for HDDL devices
"""""""""""""""""""""""""""""""""""""""


* Check if the HDDL Daemon is running on the host machine. This is to check if the HDDL Daemon is using the correct version of OpenVINO libraries in `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_. Enable the ``device_snapshot_mode`` to ``full`` in ``$HDDL_INSTALL_DIR/config/hddl_service.config`` on the host machine to get the complete snapshot of the HDDL device.
* For troubleshooting the VPU-related issues, refer to the following link:
  https://docs.openvinotoolkit.org/2021.4/openvino_docs_install_guides_installing_openvino_linux_ivad_vpu.html#troubleshooting
* For new features and changes from the previous versions, refer to the OpenVINO 2021.4 release notes from the following link:
  https://software.intel.com/content/www/us/en/develop/articles/openvino-relnotes.html
* For more details on the Known issues, limitations, and troubleshooting, refer to the OpenVINO website from the following link:
  https://docs.openvinotoolkit.org/2021.4/index.html

To run on Intel(R) Processor Graphics (GPU/iGPU)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At runtime, use the ``root`` user permissions to run inference on a ``GPU`` device. To enable root user at runtime in ``ia_video_ingestion``\ , ``ia_video_analytics``\ , or custom UDF services, add ``user: root`` in the respective ``docker-compose.yml`` file. Refer the following example:

.. code-block:: yaml

     ia_video_ingestion:
       ...
       user: root

To enable root user at runtime in ``ia_video_analytics`` or any of the custom UDF services based on ``ia_video_analytics``\ , set ``user: root`` in the respective ``docker-compose.yml`` file.
For example, refer to the following:

.. code-block:: yaml

       ia_video_analytics:
         ...
         user: root

.. note:: 


   * In the IPC mode, when the publisher (example, ia_video_ingestion or ia_video_analytics) is running as root then the subscriber (For example ia_visualizer, ia_imagestore, ia_influxdbconnectorm, ia_video_profiler etc.) should also run as root by adding ``user: root`` in the respective docker-compose.yml file.
   * If you get a ``Failed to create plugin for device GPU/ clGetPlatformIDs error`` message then check if the hostsystem supports GPU device. Try installing the required drivers from `OpenVINO-steps-for-GPU <https://docs.openvinotoolkit.org/latest/openvino_docs_install_guides_installing_openvino_linux.html#additional-GPU-steps>`_. Certain platforms like TGL can have compatibility issus with the Ubuntu kernel version. Esure the compatible kernel version is installed.


Custom User Defined Functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

OEI supports the following custom User Defined Functions (UDFs):


* Build or run custom UDFs as standalone applications:

  * For running a custom UDF as a standalone application, download the ``video-custom-udfs`` repo and refer to the `CustomUdfs/README.md <https://github.com/open-edge-insights/video-custom-udfs/blob/master/README.md>`_.

* Build or run custom UDFs in VI or VA:

  * For running custom UDFs either in VI or VA, refer to the `VideoIngestion/docs/custom_udfs_doc.md <https://github.com/open-edge-insights/video-ingestion/blob/master/docs/custom_udfs_doc.md>`_.

Time series analytics
---------------------

For time series data, a sample analytics flow uses Telegraf for ingestion, Influx DB for storage and Kapacitor for classification. This is demonstrated with an MQTT based ingestion of sample temperature sensor data and analytics with a Kapacitor UDF which does threshold detection on the input values.
The services mentioned in `build/usecases/time-series.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/time-series.yml>`_ will be available in the consolidated ``docker-compose.yml`` and consolidated ``build/eii_config.json`` of the OEI stack for time series use case when built via ``builder.py`` as called out in previous steps.
This will enable building of Telegraf and the Kapacitor based analytics containers.
More details on enabling this mode can be referred from `Kapacitor/README.md <https://github.com/open-edge-insights/ts-kapacitor/blob/master/README.md>`_
The sample temperature sensor can be simulated using the `tools/mqtt/README.md <https://github.com/open-edge-insights/eii-tools/blob/master/mqtt/README.md>`_ application.

OEI multi-node cluster deployment
---------------------------------

With k8s orchestrator
^^^^^^^^^^^^^^^^^^^^^

You can use any of the following options to deploy OEI on a multi-node cluster:


* [\ ``Recommended``\ ] For deploying through ansible playbook on multiple nodes automatically, refer `build/ansible/README.md <https://github.com/open-edge-insights/eii-core/blob/master/build/ansible/README.md#deploying-eii-using-helm-in-kubernetes-k8s-environment>`_
* For information about using helm charts to provision the node and deploy the OEI services, refer `build/helm-eii/README.md <https://github.com/open-edge-insights/eii-core/blob/master/build/helm-eii/README.md>`_

OEI tools
---------

The OEI stack consists of the following set of tools that also run as containers:


* Benchmarking

  * `Video Benchmarking <https://github.com/open-edge-insights/eii-tools/blob/master/Benchmarking/video-benchmarking-tool/README.md>`_
  * `Time series Benchmarking <https://github.com/open-edge-insights/eii-tools/blob/master/Benchmarking/time-series-benchmarking-tool/README.md>`_

* `DiscoverHistory <https://github.com/open-edge-insights/eii-tools/blob/master/DiscoverHistory/README.md>`_
* `EmbPublisher <https://github.com/open-edge-insights/eii-tools/blob/master/EmbPublisher/README.md>`_
* `EmbSubscriber <https://github.com/open-edge-insights/eii-tools/blob/master/EmbSubscriber/README.md>`_
* `GigEConfig <https://github.com/open-edge-insights/eii-tools/blob/master/GigEConfig/README.md>`_
* `HttpTestServer <https://github.com/open-edge-insights/eii-tools/blob/master/HttpTestServer/README.md>`_
* `JupyterNotebook <https://github.com/open-edge-insights/eii-tools/blob/master/JupyterNotebook/README.md>`_
* `mqtt <https://github.com/open-edge-insights/eii-tools/blob/master/mqtt/README.md>`_
* `SWTriggerUtility <https://github.com/open-edge-insights/eii-tools/blob/master/SWTriggerUtility/README.md>`_
* `TimeSeriesProfiler <https://github.com/open-edge-insights/eii-tools/blob/master/TimeSeriesProfiler/README.md>`_
* `VideoProfiler <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/README.md>`_

OEI Uninstaller
---------------

The OEI uninstaller script automates the removal of all the OEI Docker configuration that are installed on a system. The uninstaller performs the following tasks:


* Stops and removes all the OEI running and stopped containers.
* Removes all the OEI docker volumes.
* Removes all the OEI docker images [Optional]
* Removes all OEI install directory

To run the uninstaller script, run the following commmand from the ``[WORKDIR]/IEdgeInsights/build/`` directory

.. code-block:: sh

   ./eii_uninstaller.sh -h

Usage: ./eii_uninstaller.sh [-h] [-d]
This script uninstalls the previous OEI version.
Where:
    -h show the help
    -d triggers the deletion of docker images (by default it will not trigger)
Example:


* 
  Run the following command to delete the OEI containers and volumes:

  .. code-block:: sh

         ./eii_uninstaller.sh

* 
  Run the following command to delete the OEI containers, volumes, and images:

  .. code-block:: sh

       export EII_VERSION=2.4
       ./eii_uninstaller.sh -d

The commands in the example will delete the version 2.4 OEI containers, volumes, and all the docker images.

Debugging options
-----------------

Perform the following steps for debugging:


#. 
   Run the following command to check if all the OEI images are built successfully:

   .. code-block:: sh

       docker images|grep ia

#. 
   You can view all the dependency containers and the OEI containers that are up and running. Run the following command to check if all containers are running:

   .. code-block:: sh

       docker ps

#. 
   Ensure that the proxy settings are correctly configured and restart the docker service if the build fails due to no internet connectivity.

#. Run the ``docker ps`` command to list all the enabled containers that are included in the ``docker-compose.yml`` file.
#. From video ingestion>video analytics>visualizer, check if the default video pipeline with OEI is working fine.
#. The ``/opt/intel/eii`` root directory gets created - This is the installation path for OEI:

   * ``data/`` - stores the backup data for persistent imagestore and influxdb
   * ``sockets/`` - stores the IPC ZMQ socket files

The following table displays useful docker-compose and docker commands:

.. list-table::
   :header-rows: 1

   * - Command
     - Description
   * - ``docker-compose -f docker-compose-build.yml build``
     - Builds all the service containers
   * - ``docker-compose -f docker-compose-build.yml build [serv_cont_name]``
     - Builds a single service container
   * - ``docker-compose down``
     - Stops and removes the service containers
   * - ``docker-compose up -d``
     - Brings up the service containers by picking the changes done in the ``docker-compose.yml`` file
   * - ``docker ps``
     - Checks the running containers
   * - ``docker ps -a``
     - Checks the running and stopped containers
   * - ``docker stop $(docker ps -a -q)``
     - Stops all the containers
   * - ``docker rm $(docker ps -a -q)``
     - Removes all the containers. This is useful when you run into issue of already container is in use
   * - ``[docker compose cli]``
     - For more information refer to the `docker documentation <https://docs.docker.com/compose/reference/overview/>`_
   * - ``[docker compose reference]``
     - For more information refer to the `docker documentation <https://docs.docker.com/compose/compose-file/>`_
   * - ``[docker cli]``
     - For more information refer to the `docker documentation <https://docs.docker.com/engine/reference/commandline/cli/#configuration-files>`_
   * - ``docker-compose run --no-deps [service_cont_name]``
     - To run the docker images separately or one by one. For example: ``docker-compose run --name ia_video_ingestion --no-deps   ia_video_ingestion`` to run the VI container and the switch ``--no-deps`` will not bring up its dependencies mentioned in the ``docker-compose`` file. If the container does not launch, there could be some issue with the entrypoint program. You can override by providing the extra switch ``--entrypoint /bin/bash`` before the service container name in the ``docker-compose run`` command. This will let you access the container and run the actual entrypoint program from the container's terminal to root cause the issue. If the container is running and you want to access it then, run the command: ``docker-compose exec [service_cont_name] /bin/bash`` or ``docker exec -it [cont_name] /bin/bash``
   * - ``docker logs -f [cont_name]``
     - Use this command to check logs of containers
   * - ``docker-compose logs -f``
     - To see all the docker-compose service container logs at once


Web Deployment Tool
-------------------

You can use the Web Deployment Tool's GUI to provision video use cases. To learn about launching and using the Web Deployment Tool, refer to the following:


* `Web Deployment Tool back end ReadMe <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-deployment-tool-backend/-/blob/master/README.md>`_
* `Web Deployment Tool front end ReadMe <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-deployment-tool-frontend/-/blob/master/README.md>`_

Troubleshooting guide
---------------------


* For any troubleshooting tips related to the OEI configuration and installation, refer to the `TROUBLESHOOT.md <https://github.com/open-edge-insights/eii-core/blob/master/TROUBLESHOOT.md>`_ guide.
* 
  If you observe any issues with the Python package installation then manually install the Python packages as follows:

  ..

     **Note:**
     To avoid any changes to the Python installation on the system, it is recommended that you use a Python virtual environment to install the Python packages. Th details for setting up and using the Python virtual environment is available here: https://www.geeksforgeeks.org/python-virtual-environment/.


  .. code-block:: sh

     cd [WORKDIR]/IEdgeInsights/build
     # Install requirements for builder.py
     pip3 install -r requirements.txt
