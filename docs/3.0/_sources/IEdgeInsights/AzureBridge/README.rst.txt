
Contents
========


* `Contents <#contents>`__

  * `Introduction <#introduction>`__
  * `Prerequisites and Setup <#prerequisites-and-setup>`__

    * `Azure Cloud Setup <#azure-cloud-setup>`__

      * `Setting up AzureML <#setting-up-azureml>`__
      * `Important <#important>`__

        * `Pushing a Model to AzureML <#pushing-a-model-to-azureml>`__

    * `Development System Setup <#development-system-setup>`__

  * `Build and Push OEI Containers <#build-and-push-oei-containers>`__
  * `Single-Node Azure IoT Edge Deployment <#single-node-azure-iot-edge-deployment>`__

    * `Step 1 - Provisioning <#step-1-provisioning>`__
    * `Step 2 - Configuring OEI <#step-2-configuring-oei>`__
    * `Step 3 - Configuring Azure IoT Deployment Manifest <#step-3-configuring-azure-iot-deployment-manifest>`__
    * `Step 4 - Deployment <#step-4-deployment>`__
    * `Helpful Debugging Commands <#helpful-debugging-commands>`__
    * `Final Notes <#final-notes>`__

  * `Configuration <#configuration>`__

    * `Azure Bridge <#azure-bridge>`__
    * `ZeroMQ TCP Subscription Implications <#zeromq-tcp-subscription-implications>`__
    * `ZeroMQ IPC Subscription Implications <#zeromq-ipc-subscription-implications>`__
    * `Sample OEI ONNX UDF <#sample-oei-onnx-udf>`__
    * `Simple Subscriber <#simple-subscriber>`__
    * `OEI ETCD Pre-Load <#oei-etcd-pre-load>`__
    * `Azure Blob Storage <#azure-blob-storage>`__
    * `Azure Deployment Manifest <#azure-deployment-manifest>`__

  * `Azure IoT Edge Simulator <#azure-iot-edge-simulator>`__
  * `Supported OEI Services <#supported-oei-services>`__
  * `Additional Resources <#additional-resources>`__

Introduction
------------

.. note:: 


   * In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights (OEI). This is due to the product name change of EII as OEI.
   * For the various scripts and commands mentioned in this document to work, place the source code for this project in the ``IEdgeInsights`` directory in the source code for OEI.


The Azure Bridge serves as a connector between OEI and the Microsoft Azure IoT Edge Runtime ecosystem. It does this by allowing the following forms of bridging:


* Publishing of incoming data from OEI onto the Azure IoT Edge Runtime bus
* Storage of incoming images from the OEI video analytics pipeline into a local instance of the Azure Blob Storage service
* Translation of configuration for OEI from the Azure IoT Hub digital twin for the bridge into ETCD via the OEI Configuration Manager APIs

This code base is structured as an Azure IoT Edge Runtime module. It includes the following:


* Deployment templates for deploying the OEI video analytics pipeline with the bridge on top of the Azure IoT Edge Runtime
* The Azure Bridge module
* A simple subscriber on top of the Azure IoT Edge Runtime for showcasing the end-to-end transmission of data
* Various utilities and helper scripts for deploying and developing on the Azure Bridge

The following sections will cover the configuration/usage of the Azure Bridge, the deployment of OEI on the Azure IoT Edge Runtime, as well as the usage of the tools and scripts included in this code base for development.

.. note::  The following sections assume an understanding of the configuration
   for OEI. It is recommended that you read the main README and User Guide for
   OEI prior to using this service.


Prerequisites and Setup
-----------------------

To use and develop with the Azure Bridge there are a few steps which must be taken to configure your environment. The setup must be done to configure your Azure Cloud account, your development system, and also the node which you are going to deploy the Azure Bridge on.

The following sections cover the setup for the first two environments listed.
Setting up your system for a single-node deployment will be covered in the
`Single-Node Azure IoT Edge Deployment <#single-node-azure-iot-edge-deployment>`__ section below.

.. note::  When you deploy with Azure IoT Hub you will also need to configure
   the Azure IoT Edge Runtime and OEI on your target device.


Azure Cloud Setup
^^^^^^^^^^^^^^^^^

Prior to using the Azure Bridge there are a few cloud services in Azure
which must be initialized.

Primarily, you need an Azure Containter Registry instance, an Azure IoT Hub,
as well as an Azure IoT Device. Additionally, if you wish to use the sample ONNX
UDF in OEI to download a ML/DL model from AzureML, then you must follow a few
steps to get this configured as well. For these steps, see the `Setting up AzureML <#setting-up-azureml>`__
section below.

To create these instances, follow the guides provided by Microsoft below:

.. note::  In the quickstart guides below it is recommended that you create an
   Azure Resource Group. This is a good practice as it makes for easy clean up
   of your Azure cloud environment.



* `Create Azure Container Registry <https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal>`_
* `Create Azure IoT Hub <https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-create-through-portal>`_
* `Register an Azure IoT Device <https://docs.microsoft.com/en-us/azure/iot-edge/how-to-register-device>`_

..

   **IMPORTANT:**
   In the tutorials above you will receive credentials/connection strings for your
   Azure Container Registry, Azure IoT Hub, and Azure IoT Device. Save these for
   later, as they will be important for setting up your development and single node
   deployment showcased in this README.


All of the tutorials provided above provide options for creating these instances
via Visual Studio Code, the Azure Portal, or the Azure CLI. If you wish to use
the Azure CLI, it is recommended that you follow the Development System Setup
instructions below.

Setting up AzureML
~~~~~~~~~~~~~~~~~~

To use the sample OEI ONNX UDF, you must do the following:


#. Create an AzureML Workspace (see `these <https://docs.microsoft.com/en-us/azure/machine-learning/how-to-manage-workspace>`_
    instructions provided by Microsoft)
#. Configure Service Principle Authentication on your AzureML workspace by following
    instructions provided `here <https://docs.microsoft.com/en-us/azure/machine-learning/how-to-setup-authentication#set-up-service-principal-authentication>`_

Important
~~~~~~~~~

During the setup process provided for step 2 above, you will run a command similar
to the following:

.. code-block:: sh

   az ad sp create-for-rbac --sdk-auth --name ml-auth

After executing this command you will see a JSON blob printed to your console
window. Save the ``clientId``\ , ``clientSecret``\ , ``subscriptionId``\ , and ``tenantId``
for configuring the sample ONNX OEI UDF later.

Pushing a Model to AzureML
""""""""""""""""""""""""""

If you already have an ONNX model you wish to push to your AzureML Workspace, then
follow `these instructions <https://docs.microsoft.com/en-us/azure/machine-learning/how-to-deploy-existing-model>`_
to push your model.

If you do not have a model, and want an easy model to use, follow
`this <https://notebooks.azure.com/azureml/projects/azureml-getting-started/html/how-to-use-azureml/deployment/onnx/onnx-modelzoo-aml-deploy-resnet50.ipynb>`_
notebook provided my Microsoft to train a simple model to push to your AzureML Workspace.

Also, you can find pre-trained models in the `ONNX Model Zoo <https://github.com/onnx/models>`_.

Development System Setup
^^^^^^^^^^^^^^^^^^^^^^^^

The development system will be used for the following actions:


* Building and pushing the OEI containers (including the bridge) to your Azure Container Registry
* Creating your Azure IoT Hub deployment manifest
* Deploying your manifest to a single node

For testing purposes, your development system can serve to do the actions detailed
above, as well as being the device you use for your single-node deployment. This
should not be done in a production environment, but it can be helpful when
familiarizing yourself with the Azure Bridge.

First, setup your system for building OEI. To do this, follow the instructions
detailed in the main OEI README and the OEI User Guide. At the end, you should
have installed Docker, Docker Compose, and other OEI Python dependencies for
the Builder script in the ``../build/`` directory.

Once this is completed, install the required components to user the Azure CLI
and development tools. The script ``./tools/install-dev-tools.sh`` automates this
process. To run this script, execute the following command:

.. note:: 


   * It is highly recommended that you use a python virtual environment to install the python packages, so that the system python installation doesn't get altered. Details on setting up and using python virtual environment can be found here: https://www.geeksforgeeks.org/python-virtual-environment/
   * If one encounter issues with conflicting dependencies between python packages, please upgrade the pip version: ``pip3 install -U pip`` and try.


.. code-block:: sh

   sudo -H -E -u ${USER} ./tools/install-dev-tools.sh

Please set the ``PATH`` environmental variable as below in the terminal
where you are using ``iotedgedev`` and ``iotedgehubdev`` commands:

.. code-block:: sh

   export PATH=~/.local/bin:$PATH

.. note:: 


   * 
     The ``-u ${USER}`` flag above allows the Azure CLI to launch your browser (if it can) so you can login to your Azure account.

   * 
     Occasionally, pip's local cache can get corrupted. If this happens,
     pip may ``SEGFAULT``. In the case that this happens, delete the ``~/.local`` directory
     on your system and re-run the script mentioned above. You may consider creating
     a backup of this directory just in case.


While running this script you will be prompted to sign-in to your Azure
account so you can run commands from the Azure CLI that interact with your
Azure instance.

This script will install the following tools:


* Azure CLI
* Azure CLI IoT Edge/Hub Extensions
* Azure ``iotedgehubdev`` development tool
* Azure ``iotedgedev`` development tool

Next, login to your Azure Container Registry with the following command:

.. code-block:: sh

   az acr login --name <ACR Name>

.. note::  Fill in ``<ACR Name>`` with the name of your Azure Container Registry


**IMPORTANT NOTE:**

Please see the list of supported services at the end of this README for the
services which can be pushed to an ACR instance. Not all OEI services are
supported by and validated to work with the Azure Bridge.

Build and Push OEI Containers
-----------------------------

.. note::  By following the steps below, the Azure Bridge and Simple
   Subscriber Azure IoT Modules will be pushed to your ACR instance as well.


After setting up your development system, build and push the OEI containers
to your Azure Contianer Registry instance. Note that the Azure Bridge only
supports a few of the OEI services currently. Before building and pushing your
OEI containers, be sure to look at the `Supported OEI Services <#supported-oei-services>`__
section below, so as to not build/push uneeded containers to your registry.

To do this go to the ``../build/`` directory in the OEI source code, modify the
``DOCKER_REGISTRY`` variable in the ``../build/.env`` file to point to your Azure
Container Registry.

Next, execute the following commands:

.. code-block:: sh

   python3 builder.py -f usecases/video-streaming-azure.yml
   docker-compose -f docker-compose-build.yml build
   docker-compose -f docker-compose-push.yml push

For more detailed instructions on this process, see the OEI README and User Guide.

Single-Node Azure IoT Edge Deployment
-------------------------------------

.. note::  Outside of the Azure ecosystem, OEI can be deployed and communicate
   across nodes. In the Azure IoT Edge ecosystem this is not possible with OEI.
   All OEI services must be running on the same edge node. However, you can
   deploy OEI on multiple nodes, but intercommunication between the nodes will
   not work.
   **Important Note:**
   If you are using TCP communication between VI or VA and the Azure Bridge,
   then you must modify the ``AllowedClients`` list under the ``Publishers`` section
   of the interfaces configuration of VI or VA to include ``AzureBridge``.
   This must be done prior to provisioning to that the proper certificates will
   be generated to encrypt/authenticate connections.


In the Azure IoT ecosystem you can deploy to single-nodes and you can do bulk
deployments. This section will cover how to deploy the Azure Bridge and
associated OEI services to a single Linux edge node. For more details on deploying
modules at scale with the Azure IoT Edge Runtime, see
`this guide <https://docs.microsoft.com/en-us/azure/iot-edge/how-to-deploy-monitor>`_

Note that this section will give a high-level overview of how to deploy the
modules with the Azure CLI. For more information on developing and deploying
Azure modules, see `this guide <https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-develop-for-linux>`_.

The deloyment of the Azure IoT Edge and the OEI modules can be broken down into
the following steps:


#. Provisioning
#. Configuring OEI
#. Configuring Azure IoT Deployment Manifest
#. Deployment

Prior to deploying a single Azure IoT Edge node you must have already
configured your Azure cloud instance (see instructions in the `Azure Cloud Setup <#azure-cloud-setup>`__
section). Additionally, you need to have already built and pushed the OEI services to your
Azure Container Registry (follow the instructions in the `Build and Push OEI Containers <#build-and-push-eii-containers>`__
section).

Provided you have met these two prerequisites, follow the steps below to do a
single node deployment with the Azure Bridge on the Azure IoT Edge Runtime.

Step 1 - Provisioning
^^^^^^^^^^^^^^^^^^^^^

The provisioning must take place on the node you wish to deploy your Azure IoT
Edge modules onto.

.. note::  This may be your development system, which was setup earlier. Keep in
   mind, however, that having your system setup as a development system and a
   targeted node for a single-node deployment should never be done in production.


First, you must then install the Azure IoT Edge Runtime on your target deployment
system. To do that, follow the instructions provided by Microsoft in
`this guide <https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-linux>`_.

Next, you must provision on your target deployment system. This provisioning is supported
by the configmanager agent azure module itself.

While provisioning on your system, note that you only need to setup the
Video Ingesiton and/or the Video Analytics containers. All other services are
not supported by the Azure Bridge currently.

Be sure to note down which directory you generate your certificates into, this
will be important later. Unless, you are running OEI in dev mode, in that case
you will have no certificates generated.

**IMPORTANT NOTE:**

If you previously installed OEI outside of Azure on your system, then make sure
that all of the OEI containers have been stopped. You can do this by going to
the ``build/`` directory in the OEI source code and running the following command:

.. code-block:: sh

   docker-compose down

This will stop and remove all of the previously running OEI containers, allowing
the Azure Bridge to run successfully.

Step 2 - Configuring OEI
^^^^^^^^^^^^^^^^^^^^^^^^

This step should be done from your development system, and not the edge node you
are deploying OEI onto. The configuration you will do during this setup will
allow your system to deploy OEI to your edge node. As noted earlier, for development
and testing purposes this could be the same system as your targeted edge device,
but this is not recommended in a production environment.

To configure OEI, modify the ``build/eii_config.json`` file. This
should have been generated when the ``build/builder.py`` script was executed
when building/pushing the OEI containers to your ACR instance. If it does not
exist, run this script based on the instructions provided in the OEI README.

Next, configure the ``build/.env`` file. You must make sure to modify the following
values in the ``.env`` file:


* ``DOCKER_REGISTRY`` - This should have been set when building/pushing the OEI
    containers to your ACR instance. Make sure it is set to the URL for your
    ACR instance.
* ``HOST_IP`` - This must be the IP address of the edge node you are deploying
    your containers to
* ``ETCD_HOST`` - This should be set to the same value as your ``HOST_IP`` address
* ``DEV_MODE`` - Set this to the same value you used when provisioning your edge node
    in the previous step

Next, in the ``AzureBridge/`` source directory, modify the ``.env`` file. Make
sure to set the following values:


* ``EII_CERTIFICATES``              - The directory with the OEI certificates on your edge system
* ``AZ_CONTAINER_REGISTY_USERNAME`` - User name for the container registry login (obtained during creation)
* ``AZ_CONTAINER_REGISTY_PASSWORD`` - Password for the container registry login (obtained during creation)

  * **IMPORTANT NOTE:** Make sure to surround the password in single quotes, i.e. ``'``\ , because bash
    .. code-block::

       may escape certain characters when the file is read, leading to incorrect configuration

* ``AZ_BLOB_STORAGE_ACCOUNT_NAME``  - **(OPTIONAL)** User name for the local Azure Blob Storage instance

**IMPORTANT NOTE #1:**

It is important to note that for the ``AZ_CONTAINER_REGISTY_PASSWORD`` variable you
must wrap the password in single quotes, i.e. ``'``. Otherwise, there may be
characters that get escaped in a weird way when the values are populated into
your deployment manifest leading to configuration errors.

**IMPORTANT NOTE #2:**

If you wish to use the sample OEI ONNX UDF, now is the time to configure the UDF
to run. See the `Sample OEI ONNX UDF <#sample-oei-onnx-udf>`__ configuration section
below for how to configure the UDF.

Once the following step has been completed, then you should have correctly configured
``.env`` files to deploying OEI via Azure. If some of the values were incorrect, then
you will encounter issues in the proceeding steps.

Step 3 - Configuring Azure IoT Deployment Manifest
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once you have your target edge system provisioned and OEI configured, you need to
create your Azure IoT Hub deployment manifest. The Azure Bridge provides some
convenience scripts to ease this process.

.. note::  These steps should be done from your development system setup in
   the `Development System Setup <#development-system-setup>`__ section. Note, that for testing
   and development purposes, these could be the same system.


To generate your deployment manifest template, execute the following command:

.. code-block:: sh

   ./tools/generate-deployment-manifest.sh example ia_configmgr_agent AzureBridge SimpleSubscriber ia_video_ingestion ia_video_analytics

.. note:: 


   * If you are using Azure Blob Storage, include ``AzureBlobStorageonIoTEdge`` in the argument list above.
   * When you run the command above, it will pull some values from your OEI ``build/.env`` file. If the ``build/.env`` file is configured incorrectly, you may run into issues.


The above command will generate two files: ``./example.template.json`` and ``config/example.amd64.json``. The first is a deployment template, and the second is the fully populated/generated configuration for Azure IoT Hub. In executing the script above, you should have a manifest which includes the Azure Bridge, Simple Subscriber, as well as the OEI video ingestion service.

The list of services given to the bash script can be changed if you wish to run different services.

You may want/need to modify your ``./example.template.json`` file after running
this command. This could be because you wish to change the topics that VI/VA use
or because you want to configure the Azure Bridge in some different way. If you
modify this file, you must regenerate the ``./config/example.amd64.json`` file.
To do this, execute the following command:

.. code-block:: sh

   iotedgedev genconfig -f example.template.json

If you wish to modify your ``eii_config.json`` file after generating your template,
you can re-add this to the Azure Bridge digital twin by running the following
command:

.. code-block:: sh

   python3 tools/serialize_eii_config.py example.template.json ../build/eii_config.json

If all of the commands above ran correctly, then you will have a valid ``*.template.json`` file and a valid ``config/*.amd64.json`` file.

If, for some reason, these commands fail, revisit Step 2 and make sure all of your
environmental variables are set correctly. And if that does not resolve your issue,
verify that your development system is setup correctly by revisiting the
`Development System Setup <#development-system-setup>`__ section.

Step 4 - Deployment
^^^^^^^^^^^^^^^^^^^

Now that you have generated your deployment manifest, deploy the modules to your
Azure IoT Edge Device using the Azure CLI command shown below:

.. code-block:: sh

   az iot edge set-modules -n <azure-iot-hub-name> -d <azure-iot-edge-device-name> -k config/<deployment-manifest>

If this command run successfully, then you will see a large JSON string print out
on the console with information on the deployment which it just initiated. If it
failed, then the Azure CLI will output information on the potential reason for the
failure.

Provided all of the setups above ran correctly, your edge node should now be running
your Azure IoT Edge modules, the Azure Bridge, and the OEI services you
selected.

It is possible that for the Azure Bridge (and any Python Azure IoT Edge modules)
you will see that the service crashes the first couple of times it attempts to come
up on your edge system with an exception similar to the following:

.. code-block:: shell

   Traceback (most recent call last):
       File "/usr/local/lib/python3.7/site-packages/azure/iot/device/common/mqtt_transport.py", line 340, in connect
           host=self._hostname, port=8883, keepalive=DEFAULT_KEEPALIVE
       File "/usr/local/lib/python3.7/site-packages/paho/mqtt/client.py", line 937, in connect
           return self.reconnect()
       File "/usr/local/lib/python3.7/site-packages/paho/mqtt/client.py", line 1071, in reconnect
           sock = self._create_socket_connection()
       File "/usr/local/lib/python3.7/site-packages/paho/mqtt/client.py", line 3522, in _create_socket_connection
           return socket.create_connection(addr, source_address=source, timeout=self._keepalive)
       File "/usr/local/lib/python3.7/socket.py", line 728, in create_connection
           raise err
       File "/usr/local/lib/python3.7/socket.py", line 716, in create_connection
           sock.connect(sa)
       ConnectionRefusedError: [Errno 111] Connection refused

This occurs because the container is starting before the ``edgeHub`` container for
the Azure IoT Edge Runtime has come up, and so it it unable to connect. Once the
``edgeHub`` container is fully launched, then this should go away and the containers
should launch correctly.

If everything is running smoothly, you should see messages being printed in the
Simple Subscriber service using the following command:

.. code-block:: sh

   docker logs -f SimpleSubscriber

For more debugging info, see the following section.

Helpful Debugging Commands
^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are encountering issues, the following commands can help with debugging:


* **Azure IoT Edge Runtime Daemon Logs:** ``sudo iotedge system logs -- -f``
* **Container Logs:** ``docker logs -f <CONTAINER-NAME>``

Final Notes
^^^^^^^^^^^

When deploying with Azure IoT Edge Runtime there are many security considerations
to be taken into account. Please consult the following Microsoft resources regarding
the security in your deployments.


* `Securing Azure IoT Edge <https://docs.microsoft.com/en-us/azure/iot-edge/security>`_
* `IoT Edge Security Manager <https://docs.microsoft.com/en-us/azure/iot-edge/iot-edge-security-manager>`_
* `IoT Edge Certificates <https://docs.microsoft.com/en-us/azure/iot-edge/iot-edge-certs>`_
* `Securing the Intelligent Edge blob post <https://azure.microsoft.com/en-us/blog/securing-the-intelligent-edge/>`_

Configuration
-------------

The configuration of the Azure Bridge is a mix of the configuration for the
OEI services, the Azure Bridge module, and configuration for the other
Azure IoT Edge Modules (i.e. the Simple Subscriber, and the Azure Blob Storage
modules). All of this configuration is wrapped up into your deployment manifest
for Azure IoT Hub.

The following sections cover the configuration of the aforementioned servies
and then the generation of your Azure Deployment manifest.

Azure Bridge
^^^^^^^^^^^^

The Azure Bridge spans OEI and Azure IoT Edge Runtime environments, as such
its configuration is a mix of OEI configuration and Azure IoT Edge module configuration
properties. The configuration of the bridge is split between environmental
variables specified in your Azure IoT Hub deployment manifest and the module's
digital twin. Additionally, the digital twin for the Azure Bridge module
contains the entire configuration for the OEI services running in your edge
environment.

The configuration of the OEI Message Bus is done in a method similar to that of
the other OEI services, such as the Video Analytics service. To provided the
configuration for the topics which the bridge should subscribe to,
you must set the ``Subscribers`` list in the `modules/AzureBridge/config.json <https://github.com/open-edge-insights/eii-azure-bridge/blob/master/modules/AzureBridge/config.json>`_
file. The list is comprised of JSON objects for every subscription you wish the
Azure Bridge to establish. Below is an example of the configuration for
subscribing to the the publications coming from the Video Analytics container.

.. code-block:: javascript

   {
       "Subscribers": [
           {
               // Specifies that this is the default subscriber
               "Name": "default",

               // Gives the type of connection, i.e. zmq_tcp, this could
               // also be zmq_ipc
               "Type": "zmq_tcp",

               // The EndPoint specifies the details of the connect, for an
               // IPC connection, this would be a JSON object with the
               // SocketDir key pointing to the directory of the IPC sockets
               "EndPoint": "127.0.0.1:65013",

               // Specification of the AppName of the service publishing the
               // messages. This allows the Azure Bridge to get the needed
               // authentication keys to subscribe
               "PublisherAppName": "VideoAnalytics",

               // Specifies the list of all of the topics which the
               // AzureBridge shall subscribe to
               "Topics": [
                   "camera1_stream_results"
               ]
           }
       ]
   }

There are a few important implications to be aware of for both ZeroMQ TCP and IPC
subscribers over the OEI Message Bus. These implications are specified below.

ZeroMQ TCP Subscription Implications
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For ZeroMQ TCP subscribers, like the example shown above, the ``EndPoint`` in
the subscriber's configuration object has to be overridden through an
environmental variable. The reason for this, is that the Azure Bridge
service runs attached to a bridged Docker network created by the Azure IoT
Edge Runtime, whereas the other OEI services run on the different bridged network.
In order to subscribe, the Azure Bridge must use the host's IP address to
connect.

If the Azure Bridge is only subscribing to a single service, then the
``EndPoint`` can be overridden by setting the ``SUBSCRIBER_ENDPOINT`` environmental
variable. The environmental variable changes if there are multiple subscribers.
For instance, if the configuration example had another object in the Subscribers
list which had the ``Name`` key set to ``example_name``\ , then the environmental
variable name would need to be ``SUBSCRIBER_example_name_ENDPOINT``. Essentially,
for multiple subscribers the ``Name`` property must be in the environmental
variable name between the ``SUBSCRIBER_`` and ``_ENDPOINT``. The same holds true for
``CLIENT_ENDPOINT`` and ``CLIENT_<Name>_ENDPOINT`` usage of environmental variables too.

In either case, the value of the environmental variable must be set to
``$HOST_IP:<PORT>`` where you must fill in what the desired port is. Note that the
IP address is the variable ``$HOST_IP``. This will be pulled from the ``.env`` file
when generating your deployment manifest.

The final implication is on the configuration of the services which the
Azure Bridge is subscribing to. Most OEI services publishing over TCP set
their host to ``127.0.0.1``. This keeps the communication only available to
subscribers which are on the local host network on the system. In order for
the Azure Bridge to subscribe to these publications this must be changed
to ``0.0.0.0``.

This can be accomplished by overridding the service's publisher ``EndPoint``
configuration via environmental variables, just like with the Azure Bridge
service. For each service which the Azure Bridge needs to subscribe to over
TCP, add the environmental variable ``PUBLISHER_ENDPOINT=0.0.0.0:<PORT>`` to the
environmental variable configuration of the serivce's module configuration in
your deployment manifest (note: be sure to replace the port).  Or if there are
multiple topics being published, use the variable ``PUBLISHER_<Name>_ENDPOINT``.
The same holds true for ``SERVER_ENDPOINT`` and ``SERVER_<Name>_ENDPOINT`` usage
of environmental variables too.

These variables have already been set for to have the Azure Bridge subscribe
to a single instance of the Video Analytics service. This configuration can be
seen in your deployment manifest under the, "AzureBridge", and, "ia_video_analytics",
modules. Or, you can see this configuration being set in the,
"config/templates/ia_video_analytics.template.json", and,
"config/templates/AzureBridge.template.json", files.

ZeroMQ IPC Subscription Implications
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If AzureBridge is subscribing to publisher over a ZeroMQ IPC socket, ensure the following:


* 
  AzureBridge app's subscriber interfaces configuration matches to that of the publisher app's publisher interfaces configuration in ``build/eii_config.json`` file.  The following is an example of the AzureBridge interface configuration subscribing to the publications coming from the VideoIngestion container.

  .. code-block:: javascript

     {
       "Subscribers": [
           {
               // Specifies that this is the default subscriber
               "Name": "default",

               // Gives the type of connection, i.e. zmq_tcp/zmq_ipc
               "Type": "zmq_ipc",

               // The EndPoint specifies the details of the connect, for an
               // IPC connection, this would be a JSON object with the
               // SocketDir key pointing to the directory of the IPC sockets
               "EndPoint": "/EII/sockets",

               // Specification of the AppName of the service publishing the
               // messages. This allows the Azure Bridge to get the needed
               // authentication keys to subscriber
               "PublisherAppName": "VideoIngestion",

               // Specifies the list of all of the topics which the
               // AzureBridge shall subscribe to
               "Topics": [
                   "camera1_stream"
               ]
           }
        ]
       }

* 
  Follow ``Step 3 - Configuring Azure IoT Deployment Manifest`` to generate the manifest template file and deployment manifest files. Ensure to remove all the ``PUBLISHER_ENDPOINT`` ``PUBLISHER_<Name>_ENDPOINT``\ , ``SUBSCRIBER_ENDPOINT``\ , and ``SUBSCRIBER_<Name>_ENDPOINT`` environmental variables from the generated deployment manifest template file i.e., `example.template.json <https://github.com/open-edge-insights/eii-azure-bridge/blob/master/example.template.json>`_ as these ENVs are not applicable for IPC configuration. Additionally, update the `example.template.json <https://github.com/open-edge-insights/eii-azure-bridge/blob/master/example.template.json>`_ as per the recommendations mentioned in the ``Important Note`` section for IPC configuration. Run the following command to regenerate the deployment manifest file (\ ``config/example.amd64.json``\ ):

  .. code-block:: sh

     iotedgedev genconfig -f example.template.json

* 
  Follow  `Step 4 - Deployment <#step-4-deployment>`__ for deployment. The following is an example of digital twin for the Azure Bridge:

.. code-block:: json

   {
       "log_level": "DEBUG",
       "topics": {
           "camera1_stream_results": {
               "az_output_topic": "camera1_stream_results"
           }
       },
       "eii_config": "{\"/AzureBridge/config\": {}, \"/AzureBridge/interfaces\": {\"Subscribers\": [{\"EndPoint\": \"127.0.0.1:65013\", \"Name\": \"default\", \"PublisherAppName\": \"VideoAnalytics\", \"Topics\": [\"camera1_stream_results\"], \"Type\": \"zmq_tcp\"}]}, \"/EtcdUI/config\": {}, \"/EtcdUI/interfaces\": {}, \"/GlobalEnv/\": {\"C_LOG_LEVEL\": \"DEBUG\", \"ETCD_KEEPER_PORT\": \"7070\", \"GO_LOG_LEVEL\": \"INFO\", \"GO_VERBOSE\": \"0\", \"PY_LOG_LEVEL\": \"DEBUG\"}, \"/VideoAnalytics/config\": {\"encoding\": {\"level\": 95, \"type\": \"jpeg\"}, \"max_jobs\": 20, \"max_workers\": 4, \"queue_size\": 10, \"udfs\": [{\"device\": \"CPU\", \"model_bin\": \"common/udfs/python/pcb/ref/model_2.bin\", \"model_xml\": \"common/udfs/python/pcb/ref/model_2.xml\", \"name\": \"pcb.pcb_classifier\", \"ref_config_roi\": \"common/udfs/python/pcb/ref/roi_2.json\", \"ref_img\": \"common/udfs/python/pcb/ref/ref.png\", \"type\": \"python\"}]}, \"/VideoAnalytics/interfaces\": {\"Publishers\": [{\"AllowedClients\": [\"*\"], \"EndPoint\": \"0.0.0.0:65013\", \"Name\": \"default\", \"Topics\": [\"camera1_stream_results\"], \"Type\": \"zmq_tcp\"}], \"Subscribers\": [{\"EndPoint\": \"/EII/sockets\", \"Name\": \"default\", \"PublisherAppName\": \"VideoIngestion\", \"Topics\": [\"camera1_stream\"], \"Type\": \"zmq_ipc\", \"zmq_recv_hwm\": 50}]}, \"/VideoIngestion/config\": {\"encoding\": {\"level\": 95, \"type\": \"jpeg\"}, \"ingestor\": {\"loop_video\": true, \"pipeline\": \"./test_videos/pcb_d2000.avi\", \"poll_interval\": 0.2, \"queue_size\": 10, \"type\": \"opencv\"}, \"max_jobs\": 20, \"max_workers\": 4, \"sw_trigger\": {\"init_state\": \"running\"}, \"udfs\": [{\"n_left_px\": 1000, \"n_right_px\": 1000, \"n_total_px\": 300000, \"name\": \"pcb.pcb_filter\", \"scale_ratio\": 4, \"training_mode\": \"false\", \"type\": \"python\"}]}, \"/VideoIngestion/interfaces\": {\"Publishers\": [{\"AllowedClients\": [\"VideoAnalytics\", \"Visualizer\", \"WebVisualizer\", \"TLSRemoteAgent\", \"RestDataExport\"], \"EndPoint\": \"/EII/sockets\", \"Name\": \"default\", \"Topics\": [\"camera1_stream\"], \"Type\": \"zmq_ipc\"}], \"Servers\": [{\"AllowedClients\": [\"*\"], \"EndPoint\": \"127.0.0.1:66013\", \"Name\": \"default\", \"Type\": \"zmq_tcp\"}]}}"
   }

..

   For the full JSON schema, see ``modules/AzureBridge/config_schema.json``
   For the digital twin of the Azure Bridge module.


Each key in the configuration above is described in the table below.

.. list-table::
   :header-rows: 1

   * - Key
     - Description
   * - ``log_level``
     - This is the logging level for the Azure Bridge module, must be INFO, DEBUG, WARN, or ERROR
   * - ``topics``
     - Configuration for the topics to map from the OEI Message Bus into the Azure IoT Edge Runtime
   * - ``eii_config``
     - Entire serialized configuration for OEI; this configuration will be placed in ETCD


You will notice that the ``eii_config`` is a serialized JSON string. This is due to a limitation with the Azure IoT Edge Runtime. Currently, module digital twins do not support arrays; however, the OEI configuration requires array support. To workaround this limitation, the OEI configuration must be a serialized JSON string in the digital twin for the Azure Bridge module.

The ``topics`` value is a JSON object, where each key is a topic from the OEI Message Bus which will be re-published onto the Azure IoT Edge Runtime. The value for the topic key will be an additional JSON object, where there is one required key, ``az_output_topic``\ , which is the topic on Azure IoT Edge Runtime to use and then an optional key, ``az_blob_container_name``.

Sample OEI ONNX UDF
^^^^^^^^^^^^^^^^^^^

OEI provides a sample UDF which utilizes the ONNX RT to execute your machine learning or deep learning model. It also supports connecting to an AzureML Workspace to download the model and then run it. The source code for this UDF is in ``[WORKDIR]/IEdgeInsights/common/video/udfs/python/sample_onnx/``\ , also refer ``Sample ONNX UDF`` section in ``[WORKDIR]/IEdgeInsights/common/video/udfs/README.md`` for doing the required configuration for running this UDF.

To use this UDF with OEI, you need to modify your ``build/eii_config.json``\ configuration file to run the UDF in either your Video Ingesiton or Video Analytics instance. Ensure to remove the existing PCB filter or classifier UDFs or any other UDFs in Video Ingestion and Video Analytics config keys in ``build/eii_config.json`` aswe are doing some basic pre-processing, inferencing and post-processing in the ONNX UDF itself. Then, you need to modify the environmental variables in the ``AzureBridge/.env`` file to provide the connection
information to enable the UDF to download your model from AzureML. Make sure to follow the instructions provided in the `Setting up AzureML <#setting-up-azureml>`__ section above to configure your workspace correctly so that the UDF can download your model.

The sample ONNX UDF requires that the following configuration values be set for the UDF in your ``eii_config.json`` file:

.. list-table::
   :header-rows: 1

   * - Key
     - Value
   * - ``aml_ws``
     - AzureML workspace name
   * - ``aml_subscription_id``
     - ``subscriptionId`` saved from creating the Service Principal Authentication
   * - ``model_name``
     - Name of the model in your AzureML workspace
   * - ``download_mode``
     - Whether or not to attempt to download the model


.. note::  If ``download_mode`` is ``false``\ , then it expects the ``model_name`` to where the ``*.onnx`` model file is in the container.


This should be added into the ``udfs`` list for your Video Ingestion or Video Analytics instance you wish to have run the UDF. The configuration should look similar to the following:

.. code-block:: javascript

   {
       // ... omited rest of OEI configuration ...

       "udfs": [
           {
               "name": "sample_onnx.onnx_udf",
               "type": "python",
               "aml_ws": "example-azureml-workspace",
               "aml_subscription_id": "subscription-id",
               "model_name": "example-model-name",
               "download_model": true
           }
       ]

       // ... omited rest of OEI configuration ...
   }

The following environmental variables must be set in the ``AzureBridge/.env`` file in order to have the sample ONNX UDF download your model from an AzureML Workspace:

.. list-table::
   :header-rows: 1

   * - Setting
     - Description
   * - ``AML_TENANT_ID``
     - The ``tenantId`` saved in the Azure Cloud setup
   * - ``AML_PRINCIPAL_ID``
     - The ``clientId`` saved in the Azure Cloud setup
   * - ``AML_PRINCIPAL_PASS``
     - The ``clientSecret`` saved in the Azure Cloud setup


It is important to note that for the ``AML_PRINCIPAL_PASS`` variable you must wrap the password in single quotes, i.e. ``'``. Otherwise, there may be characters that get escaped in a weird way when the values are populated into your deployment manifest leading to configuration errors.

The ``tenantId``\ , ``clientId``\ , ``clientSecret``\ , and ``subscriptionId`` should all have been obtained when following the instructions in the `Setting up AzureML <#setting-up-azureml>`__
section.

Run the below steps when the ``sample_onnx`` UDF is failing with error like ``The provided client secret keys are expired. Visit the Azure Portal to create new keys for your app``\ :

.. code-block:: sh

   az login
   az ad sp create-for-rbac --sdk-auth --name ml-auth

The output of above command will be in json format. Please update the ``AML_`` env variables in ``AzureBridge/.env`` as per above table and follow the steps `Step 4 <#step-4-deployment>`__ and `Step 4 <#step-4-deployment>`__ to see the ``sample_onnx`` UDF working fine.

**IMPORTANT NOTE:**

If your system is behind a proxy, you may run into an issue where the download of your ONNX model from AzureML times out. This may happen even if the proxy is set globally for Docker on your system. To fix this, update your deployment manifest template so that the Video Ingestion and/or Video Analytics containers have the ``http_proxy`` and ``https_proxy`` values set. The manifest should look something like the following:

.. code-block:: javascript

   {
       // ... omitted ...

       "modules": {
           "ia_video_ingestion": {
               // ... omitted ...

               "settings": {
                   "createOptions": {
                       "Env": [
                           // ... omitted ...

                           "http_proxy=<YOUR PROXY>",
                           "https_proxy=<YOUR PROXY>",

                           // ... omitted ...
                       ]
                   }
               }

               // ... omitted ...
           }
       }

       // ... omitted ...
   }

Simple Subscriber
^^^^^^^^^^^^^^^^^

The Simple Subscriber module provided with the Azure Bridge is a very simple service which only receives messages over the Azure IoT Edge Runtime and prints them to stdout. As such, there is no digital twin required for this module. The only configuration required is that a route be established in the Azure IoT Edge Runtime from the Azure Bridge module to the Simple Subscriber module. This routewill look something like the following in your deployment manifest:

.. code-block:: javascript

   {
       "$schema-template": "2.0.0",
       "modulesContent": {
           // ... omitted for brevity ...

           "$edgeHub": {
               "properties.desired": {
                   "schemaVersion": "1.0",
                   "routes": {
                       "BridgeToSimpleSubscriber": "FROM /messages/modules/AzureBridge/outputs/camera1_stream INTO BrokeredEndpoint(\"/modules/SimpleSubscriber/inputs/input1\")"
                   },
                   "storeAndForwardConfiguration": {
                       "timeToLiveSecs": 7200
                   }
               }
           }

           // ... omitted for brevity ...
       }
   }

For more information on establishing routes in the Azure IoT Edge Runtime, see `this documentation <https://docs.microsoft.com/en-us/azure/iot-edge/module-composition#declare-routes>`_.

OEI ETCD Pre-Load
^^^^^^^^^^^^^^^^^

The configuration for OEI is given to the Azure Bridge via the ``eii_config`` key in the module's digital twin. As specified in the Azure Bridge configuration
section, this must be a serialized string. For the scripts included with the
Azure Bridge for generating your deployment manifest the ETCD pre-load
configuration is stored at ``config/eii_config.json``. See the OEI documentation
for more information on populating this file with your desired OEI configuration.
The helper scripts will automatically serialize this JSON file and add it to your
deployment manifest.

Azure Blob Storage
^^^^^^^^^^^^^^^^^^

The Azure Bridge enables to use of the Azure Blob Storage edge IoT service
from Microsoft. This service can be used to save images from OEI into the blob
storage.

If you wish to have the Azure Blob Storage service save the images to your
host filesystem, then you must do the following:


#. 
   Create the directory to save the data on your host filesystem, it is recommended to use the following commands:

   .. code-block:: sh

       source [WORK_DIR]/IEdgeInsights/build/.env
       sudo mkdir -p /opt/intel/eii/data/azure-blob-storage
       sudo chown ${EII_UID}:${EII_UID} /opt/intel/eii/data/azure-blob-storage

#. 
   Next, modify your deployment manifest to alter the bind location which the Azure Blob Storage service uses. To do this, open your ``*.template.json`` file. Provided you specified the Azure Blob Storage service, you should see something like the following in your deployment manifest template:

   .. code-block:: json

      {
        "AzureBlobStorageonIoTEdge": {
               "type": "docker",
               "status": "running",
               "restartPolicy": "always",
               "version": "1.0",
               "settings": {
                   "image": "mcr.microsoft.com/azure-blob-storage",
                   "createOptions": {
                       "User": "${EII_UID}",
                       "Env": [
                           "LOCAL_STORAGE_ACCOUNT_NAME=$AZ_BLOB_STORAGE_ACCOUNT_NAME",
                           "LOCAL_STORAGE_ACCOUNT_KEY=$AZ_BLOB_STORAGE_ACCOUNT_KEY"
                       ],
                       "HostConfig": {
                           "Binds": [
                               "az-blob-storage-volume:/blobroot"
                           ]
                       }
                   }
               }
           }
      }

   Change the ``Binds`` location to the following:

   .. code-block:: javascript

      {
       "AzureBlobStorageonIoTEdge": {
               // ... omitted ...
               "settings": {
                   "createOptions": {
                       // ... omitted ...
                       "HostConfig": {
                           "Binds": [
                               "/opt/intel/eii/data/azure-blob-storage/:/blobroot"
                           ]
                       }
                   }
               }
           }
      }

#. 
   Add the ``az_blob_container_name`` key as below in ``example.template.json`` file, this specifies the Azure Blob Storage container to store the images from the OEI video analytics pipeline in. If the ``az_blob_container_name`` key is not specified, then the images will not be saved.

   .. code-block:: javascript

      {
           "AzureBridge": {
               "properties.desired": {
                   // ...
                   "topics": {
                       "camera1_stream_results": {
                           "az_output_topic": "camera1_stream_results",
                           "az_blob_container_name": "camera1streamresults"
                       }
                   },
                   // ...
               }
           }
       }

   ..

      **Important Notes:**


      * "Container" in the Azure Blob Storage context is not referencing a Docker container, but rather a storage structure within the Azure Blob Storage instance running on your edge device. For more information on the data structure of Azure Blob Storage, see `this link <https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction#blob-storage-resources>`_.
      * The Azure Blob Storage service places strict requirements on the name of the, "container", under which it stores blobs. This impacts the value given for the ``az_blob_container_name`` configuration key. According to the Azure documentation, the name must a valid DNS name adhering to the following rules:

        * Container names must start or end with a letter or number, and can contain only letters, numbers, and the dash (-) character.
        * Every dash (-) character must be immediately preceded and followed by a letter or number; consecutive dashes are not permitted in container names.
        * All letters in a container name must be lowercase.
        * Container names must be from 3 through 63 characters long.

      * For more information on the name conventions/restrictions for Azure Blob Storage container names, see `this <https://docs.microsoft.com/en-us/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata>`_ page of the Azure documentation.


#. 
   Ensure to run the ``iotedgedev genconfig -f example.template.json`` command for the changes to be applied to the actual deployment manifest: ``./config/example.amd64.json``\ /. Follow `Step  4 - Deployment <#step-4-deployment>`__ to deploy the azure modules. Run the following command to see the images:

   .. code-block:: sh

      sudo ls -l /opt/intel/eii/data/azure-blob-storage/BlockBlob/

   In that directory, you will see a folder for each container. Inside that directory will be the individually saved images.

#. 
   **(OPTIONAL)** For pushing the saved images to Azure portal, update the ``properties.desired`` key of `AzureBlobStorageonIOTEdge.template.json <https://github.com/open-edge-insights/eii-azure-bridge/blob/master/config/templates/AzureBlobStorageonIOTEdge.template.json>`_ as below with the right applicable values:

   .. code-block:: javascript

       "properties.desired": {
           "deviceToCloudUploadProperties": {
               "uploadOn": true,
               "cloudStorageConnectionString" : "<KEY_IN_THE_REQUIRED_CONNECTION_STRING",
               "storageContainersForUpload":{
                   "camera1streamresults": {
                           "target": "camera1streamresults"
                   }
               }

           },
           "deviceAutoDeleteProperties": {
               "deleteOn": false,
               "deleteAfterMinutes": 5
           }
       }

   Rerun step 4 to redeploy the Azure Blob storage with the above config.

   ..

      **Note:**


      * For more information on configuring your Azure Blob Storage instance at the edge, see the documentation for the service `here <https://docs.microsoft.com/en-us/azure/iot-edge/how-to-store-data-blob>`_.
      * Also see `this guide <https://docs.microsoft.com/en-us/azure/iot-edge/how-to-deploy-blob>`_ as well.


Azure Deployment Manifest
^^^^^^^^^^^^^^^^^^^^^^^^^

For more information on creating / modifying Azure IoT Hub deployment manifests, see `this guide <https://docs.microsoft.com/en-us/azure/iot-edge/module-composition>`_.

Azure IoT Edge Simulator
------------------------

.. note:: 
   We are facing issue with ``iotedgehubdev`` Azure IoT edge simulator tool where we aren't seeing the modules being deployed. An issue on the same is been raised at `Issue 370 <https://github.com/Azure/iotedgehubdev/issues/370>`_ and is been followed up.


Microsoft provides a simluator for the Azure IoT Edge Runtime. During the setup of your development system (covered in the `Development System Setup <#development-system-setup>`__
section), the simulator is automatically installed on your system.

Additionally, the Azure Bridge provides the ``./tools/run-simulator.sh`` script to easily use the simulator with the bridge.

To do this, follow steps 1 - 3 in the `Single-Node Azure IoT Edge Deployment <#single-node-azure-iot-edge-deployment>`__ section above. Then, instead of step 4, run the following command to setup the simulator:

.. code-block:: sh

   sudo -E env "PATH=$PATH" iotedgehubdev setup -c "<edge-device-connection-string>"

.. note::  The ``env "PATH=$PATH"`` above ensures that the ``PATH`` env variable set in the regular user context gets exported to ``sudo`` environment.


Next, start the simulator with your deployment manifest template using the following command:

.. code-block:: sh

   ./tools/run-simulator.sh ./example.template.json

If everything is running smoothly, you should see messages being printed in the Simple Subscriber service using the following command:

.. code-block:: sh

   docker logs -f SimpleSubscriber

**Important Note:**

You *cannot* run both the Azure IoT Edge Runtime and the simulator simultaneously on the same system. If you are using the same system, first stop the Azure IoT Edge Runtime daemon with the following command:

.. code-block:: sh

   sudo iotedge system stop

Then, run the simulator as specified above.

Supported OEI Services
----------------------

Azure Bridge supports the following services:


* Config Manager Agent
* Video Ingestion
* Video Analytics

.. note:: 


   * As ``Config Manager Agent`` responsible for OEI provisioning is deployed as azure module, it becomes essential to have other OEI services to be deployed as azure modules to talk to other OEI azure modules.
   * Ensure to add the app names to the ``SERVICES`` environment key of ``Config Manager Agent`` module template file at `ia_configmgr_agent.template.json <https://github.com/open-edge-insights/eii-core/blob/master/ia_configmgr_agent.template.json>`_.


Additional Resources
--------------------

For more resources on Azure IoT Hub and Azure IoT Edge, see the following references:


* `Azure IoT Hub <https://docs.microsoft.com/en-us/azure/iot-hub/>`_
* `Azure IoT Edge <https://docs.microsoft.com/en-us/azure/iot-edge/>`_
* `How to Deploy AzureML Models <https://docs.microsoft.com/en-us/azure/machine-learning/how-to-deploy-and-where>`_
* `AzureML Tutorial: Train your first ML Model <https://docs.microsoft.com/en-us/azure/machine-learning/tutorial-1st-experiment-sdk-train>`_
