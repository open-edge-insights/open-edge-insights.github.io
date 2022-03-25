
Contents
========


* `Contents <#contents>`__

  * `OEI provision and deployment <#oei-provision-and-deployment>`__
  * `Prerequisites <#prerequisites>`__
  * `Update the helm charts directory <#update-the-helm-charts-directory>`__
  * `Provision OEI in the kubernetes node <#provision-oei-in-the-kubernetes-node>`__
  * `Deploy OEI in the kubernetes node <#deploy-oei-in-the-kubernetes-node>`__
  * `Provision and deploy mode in times switching between dev and prod mode OR changing the usecase <#provision-and-deploy-mode-in-times-switching-between-dev-and-prod-mode-or-changing-the-usecase>`__
  * `Steps to enable Accelarators <#steps-to-enable-accelarators>`__
  * `Steps for Enabling GiGE Camera with helm <#steps-for-enabling-gige-camera-with-helm>`__

    * `Setting up Multus CNI and Enabling it <#setting-up-multus-cni-and-enabling-it>`__

  * `Accessing Web Visualizer and EtcdUI <#accessing-web-visualizer-and-etcdui>`__
  * `Pre requisites on K8s MultiNode Cluster Environment <#pre-requisites-on-k8s-multinode-cluster-environment>`__

OEI provision and deployment
----------------------------

For deployment of Open Edge Insights (OEI), helm charts are provided for both provision and deployment.

.. note:: \ :


   * In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as OEI. This is due to the product name change of EII as OEI.
   * Same procedure has to be followed for single or multi node.
   * Please login/configure docker registry before running helm. This would be required when not using public docker hub for accessing images.


Prerequisites
-------------

**Note**\ :


* K8s installation on single or multi node should be done as pre-requisite to continue the following deployment. Please note we
   have tried the kubernetes cluster setup with ``kubeadm``\ , ``kubectl`` and ``kubelet`` packages on single and multi nodes with ``v1.23.4``.
   One can refer tutorials like https://adamtheautomator.com/install-kubernetes-ubuntu/#Installing_Kubernetes_on_the_Master_and_Worker_Nodes and many other
   online tutorials to setup kubernetes cluster on the web with host OS as ubuntu 18.04/20.04.
* For helm installation, please refer `helm website <https://helm.sh/docs/intro/install/>`_

..

   For time series usecase make sure ia_mqtt_broker & ia_mqtt_publisher are running.
   Make sure 'MQTT_BROKER_HOST' Environment Variable is updated with HOST IP address of the system where MQTT Broker is running.


For preparing the necessary files required for the provision and deployment, user needs to execute the build and provision steps on an Ubuntu 18.04 / 20.04 machine.
Follow the Docker prerequisites, OEI Prerequisites, Provision OEI and Build and Run OEI mentioned in `README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_ on the Ubuntu dev machine.

  As OEI don't distribute all the docker images on docker hub, one would run into issues of those pods status showing ``ImagePullBackOff`` and few pods status like visualizer, factory ctrl etc.,
  showing ``CrashLoopBackOff`` due to additional configuration required. For ``ImagePullBackOff`` issues, please follow the steps mentioned at [../README.md#distribution-of-eii-container-images]> (../README.
  md#distribution-of-eii-container-images) to push the images that are locally built to the docker registry of choice. Ensure to update the ``DOCKER_REGISTRY`` value in ``[WORKDIR]/IEdgeInsights/build/.env``
  file and re-run the `../builder.py <https://github.com/open-edge-insights/eii-core/blob/master/build/builder.py>`_ script to regenerate the helm charts for provision and deployment.

Update the helm charts directory
--------------------------------


#. 
   Edit the "EII_HOME_DIR" in `.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ with /home/username/\<dir&gt;/IEdgeInsights/.

#. 
   Make sure you have updated the OEI Service Secrets Username and password in `.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ file.

#. 
   Run builder to copy templates file to eii-deploy/templates directory and generate consolidated values.yaml file for eii-services:

   ..

      **Note:** Execute `builder.py <https://github.com/open-edge-insights/eii-core/blob/master/README.md#using-builder-script>`_ with the preferred usecase for generating the consolidated helm charts for the provisioning and deployment.


   .. code-block:: sh

      cd [WORKDIR]/IEdgeInsights/build
      python3 builder.py -f usecases/<usecase>.yml

#. 
   Following steps are required both in DEV and PROD mode:

Provision OEI in the kubernetes node
------------------------------------

.. note::  Make sure you have ``deleted`` older certificates.


   a. Execute ``eii-provision`` chart to provision OEI

.. code-block:: sh

     cd [WORKDIR]/IEdgeInsights/build/helm-eii
     helm install eii-provision eii-provision/

   b. Update permission of certificates dir incase of ``PROD`` mode

.. code-block:: sh

     cd [WORKDIR]/IEdgeInsights/build/helm-eii/
     sudo chmod -R 777 eii-deploy/Certificates

.. note:: 
   The Certificates/ directory contains sensitive information. So post the installation of eii-provision helm chart, it is recommended to delete the Certificates from it.


Deploy OEI in the kubernetes node
---------------------------------

.. note:: 


   * OEI helm/k8s deployment does not support native visualizer. Remove ``visualizer.yml`` template, if exists as a part of your usecase from ``EII_HOME_DIR/build/helm-eii/eii-deploy/templates/visualizer.yml``
   * OEI helm/k8s deployment ``Grafana`` application is not enabled for ``Video`` usecases. Only ``timeseries`` usecases are supported


Copy the helm charts in helm-eii/ directory to the node.


#. 
   Install deploy helm chart

   .. code-block:: sh

      cd [WORKDIR]/IEdgeInsights/build/helm-eii/
      helm install eii-deploy eii-deploy/

   ..

      **Note:** If one wants to set the ingestion pipeline for the video ingestion pod, please install the deploy helm chart as below:

      .. code-block:: sh

         helm install --set env.PIPELINE="<INGESTION_PIPELINE>" eii-deploy eii-deploy/

      **Note:** ConfigMgrAgent service needs to be initialized before other services during runtime. In case other services are initialized before ConfigMgrAgent one might notice "cfgmgr initialization failed" exception. After generating this exception the services should restart and continue to run.


   Verify all the pod are running:

   .. code-block:: sh

      kubectl get pods

OEI is now successfully deployed.

Provision and deploy mode in times switching between dev and prod mode OR changing the usecase
----------------------------------------------------------------------------------------------


#. 
   Set the DEV_MODE as "true/false" in  `.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ depending on dev or prod mode.

#. 
   Run builder to copy templates file to eii-deploy/templates directory and generate consolidated values.yaml file for eii-services:

   .. code-block:: sh

      cd [WORKDIR]/IEdgeInsights/build
      python3 builder.py -f usecases/<usecase>.yml

#. 
   Remove the etcd storage directory

   .. code-block:: sh

      sudo rm -rf /opt/intel/eii/data/*

Do helm install of provision and deploy charts as per previous section.

.. note:: \ :
   During re-deployment (\ ``helm uninstall`` and ``helm install``\ ) of helm charts, wait for all the pervious pods to terminate.


Steps to enable Accelarators
----------------------------

.. note:: \ :
   ``nodeSelector`` is the simplest recommended form of node selection constraint.
   ``nodeSelector`` is a field of PodSpec. It specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels (it can have additional labels as well). The most common usage is one key-value pair.



#. 
   Setting the ``label`` for a particular node

   .. code-block:: sh

      kubectl label nodes <node-name> <label-key>=<label-value>

#. 
   For HDDL/NCS2 dependenecies follow the steps for setting ``labels``.


* 
  For HDDL

  .. code-block:: sh

     kubectl label nodes <node-name> hddl=true

* 
  For NCS2

  .. code-block:: sh

     kubectl label nodes <node-name> ncs2=true

.. note:: \ : Here the node-name is your worker node machine hostname



#. 
   Open the ``[WORKDIR]/IEdgeInsights/VideoIngestion/helm/values.yaml`` or ``[WORKDIR]/IEdgeInsights/VideoAnalytics/helm/values.yaml`` file.

#. 
   Based on your workload preference. Add hddl or ncs2 to accelerator in values.yaml of video-ingestion or video-analytics.


* 
  For HDDL

  .. code-block:: yml

     config:
       video_ingestion:
         .
         .
         accelerator: "hddl"
         .
         .

* 
  For NCS2

  .. code-block:: yml

     config:
       video_ingestion:
         .
         .
         accelerator: "ncs2"
         .
         .


#. set device as "MYRIAD" in case of ncs2 and as HDDL in case of hddl in the `VA config <https://github.com/open-edge-insights/video-analytics/blob/master/config.json>`_


* 
  In case of ncs2.

  .. code-block:: yml

     "udfs": [{
       .
       .
       .
       "device": "MYRIAD"
       }]

* 
  In case of hddl.

  .. code-block:: yml

     "udfs": [{
       .
       .
       .
       "device": "HDDL"
       }]


#. 
   Run the ``[WORKDIR]/IEdgeInsights/build/builder.py`` for generating latest consolidated ``deploy`` yml file based on your ``nodeSelector`` changes set in the respective Modules.

   .. code-block:: sh

      cd [WORKDIR]/IEdgeInsights/build/
      python3 builder.py

#. 
   Follow the `Deployment Steps <#provision-and-deploy-in-the-kubernetes-node>`__

#. 
   Verify the respecitve workloads are running based on the ``nodeSelector`` constraints.

Steps for Enabling GiGE Camera with helm
----------------------------------------

**Note:** For more information on ``Multus`` please refer this git https://github.com/intel/multus-cni
 Skip installing multus if it is already installed.


#. 
   Prequisites
   For enabling gige camera with helm. Helm pod networks should be enabled ``Multus`` Network Interface
   to attach host system network interface access by the pods for connected camera access.

   **Note**\ : Please follow the below steps & make sure ``dhcp daemon`` is running fine.If there is an error on ``macvlan`` container creation on accessing the socket or if socket was not running. Please execute the below steps again

   .. code-block:: sh

      sudo rm -f /run/cni/dhcp.sock
      cd /opt/cni/bin
      sudo ./dhcp daemon

Setting up Multus CNI and Enabling it
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* 
  Multus CNI is a container network interface (CNI) plugin for Kubernetes that enables attaching multiple network interfaces to pods. Typically, in Kubernetes each pod only has one network interface (apart from a loopback) -- with Multus you can create a multi-homed pod that has multiple interfaces. This is accomplished by Multus acting as a "meta-plugin", a CNI plugin that can call multiple other CNI plugins.


  #. Get the name of the ``ethernet`` interface in which gige camera & host system connected
     **Note**\ : Identify the network interface name by following command

  .. code-block:: sh

     ifconfig


  #. Execute the Following Script with Identified ``ethernet`` interface name as Argument for ``Multus Network Setup``
     **Note:** Pass the ``interface`` name without ``quotes``

  .. code-block:: sh

     cd [WORKDIR]/IEdgeInsights/build/helm-eii/gige_setup
     sudo -E sh ./multus_setup.sh <interface_name>

  ..

     **Note**\ : Verify ``multus`` pod is in ``Running`` state

     .. code-block:: sh

          kubectl get pods --all-namespaces | grep -i multus



  #. Set gige_camera to true in values.yaml

  .. code-block:: sh

     $ vi [WORKDIR]/IEdgeInsights/VideoIngestion/helm/values.yaml
     .
     .
     gige_camera: true
     .
     .


  #. 
     Follow the `Deployment Steps <#provision-and-deploy-in-the-kubernetes-node>`__

  #. 
     Verify ``pod``\ ip & ``host`` ip are same as per Configured ``Ethernet`` interface by using below command.

  .. code-block:: sh

     kubectl exec -it <pod_name> -- ip -d address

.. note:: 


   * User needs to deploy with ``root`` user rights for GPU device, MYRIAD(NCS2) device and GenICam USB3.0 interface cameras.
   * Refer the below configuration file snip to deploy with ``root`` user rights using ``runAsUser`` field.
     .. code-block::

        apiVersion: apps/v1
        kind: Deployment
        ...
        spec:
          ...
          spec:
            ...
            containers:
              ....
              securityContext:
                runAsUser: 0


Accessing Web Visualizer and EtcdUI
-----------------------------------

  Environment EtcdUI & WebVisualizer will be running in Following ports.


* EtcdUI

  * ``https://master-nodeip:30010/``

* WebVisualizer

  * PROD Mode --  ``https://master-nodeip:30007/``
  * DEV Mode -- ``http://master-nodeip:30009/``

Pre requisites on K8s MultiNode Cluster Environment
---------------------------------------------------

.. note:: 


   * For running OEI in Prod Mode, it is required to have self-signed certificates that are getting generated need to go in as kubernetes secrets. To make this happen, it is mandatory that ConfigManager Agent Pod & generating Certs Pod gets scheduled in k8s Cluster's Master/Control Plane node
   * By default, OEI deployment charts are deployed in PROD mod and only ConfigManager Agent & generating Certs pods gets scheduled on Master/Control Plane node of K8s Cluster
   * Ensure that the Master Node is *tainted* to schedule the pod


.. code-block:: sh

     kubectl taint nodes --all node-role.kubernetes.io/master-
