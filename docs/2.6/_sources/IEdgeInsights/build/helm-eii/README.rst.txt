.. role:: raw-html-m2r(raw)
   :format: html


**Contents**


* `EII provision and deployment <#eii-provision-and-deployment>`__

  * `Pre requisites <#pre-requisites>`__
  * `Update the helm charts directory <#update-the-helm-charts-directory>`__
  * `Provision and deploy in the kubernetes node. <#provision-and-deploy-in-the-kubernetes-node>`__
  * `Provision and deploy mode in times switching between dev and prod mode OR changing the usecase <#provision-and-deploy-mode-in-times-switching-between-dev-and-prod-mode-or-changing-the-usecase>`__
  * `Steps to enable Accelarators <#steps-to-enable-accelarators>`__
  * `Steps for Enabling GiGE Camera with helm <#steps-for-enabling-gige-camera-with-helm>`__
  * `Accessing Web Visualizer and EtcdUI <#accessing-web-visualizer-and-etcdui>`__

EII provision and deployment
============================

For deployment of EII, helm charts are provided for both provision and deployment.

.. note:: \ :


   * Same procedure has to be followed for single or multi node.
   * Please login/configure docker registry before running helm. This would be required when not using public docker hub for accessing images.


Pre requisites
--------------

----

**Note**\ :


* K8s installation on single or multi node should be done as pre-requisite to continue the following deployment. Please note we
  have tried the kubernetes cluster setup with ``kubeadm``\ , ``kubectl`` and ``kubelet`` packages on single and multi nodes with ``v1.22.2``.
  One can refer tutorials like https://adamtheautomator.com/install-kubernetes-ubuntu/#Installing_Kubernetes_on_the_Master_and_Worker_Nodes and many other
  online tutorials to setup kubernetes cluster on the web with host OS as ubuntu 18.04/20.04.
* For helm installation, please refer `helm website <https://helm.sh/docs/intro/install/>`_

----

For preparing the necessary files required for the provision and deployment, user needs to execute the build and provision steps on an Ubuntu 18.04 / 20.04 machine.
Follow the Docker pre-requisites, EII Pre-requisites, Provision EII and Build and Run EII mentioned in `README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_ on the Ubuntu dev machine.


* 
  To run EII services with helm in fresh system where EII services are going to run for the first time(no ``eiiuser`` is present on that system), user needs to run below steps:


  #. Create EII user if not exists:
     .. code-block:: sh

        $ set -a
        $ source ../.env
        $ set +a
        $ sudo groupadd $EII_USER_NAME -g $EII_UID
        $ sudo useradd -r -u $EII_UID -g $EII_USER_NAME $EII_USER_NAME

  #. Create required directory and change ownership to EII user
     .. code-block:: sh

        $ sudo mkdir -p $EII_INSTALL_PATH/data/influxdata
        $ sudo mkdir -p $EII_INSTALL_PATH/sockets/
        $ sudo chown -R $EII_USER_NAME:$EII_USER_NAME $EII_INSTALL_PATH

* Execute `builder.py <https://github.com/open-edge-insights/eii-core/blob/master/README.md#using-builder-script>`_ with the preferred usecase for generating the consolidated helm charts for the provisioning and deployment.
  As EII don't distribute all the docker images on docker hub, one would run into issues of those pods status showing ``ImagePullBackOff`` and few pods status like visualizer, factory ctrl etc., 
  showing ``CrashLoopBackOff`` due to additional configuration required. For ``ImagePullBackOff`` issues, please follow the steps mentioned at [../README.md#distribution-of-eii-container-images]> (../README.\ :raw-html-m2r:`<br>`
  md#distribution-of-eii-container-images) to push the images that are locally built to the docker registry of choice. Please ensure to update the ``DOCKER_REGISTRY`` value in ``[WORKDIR]/IEdgeInsights/build/.env``
  file and re-run the `../builder.py <https://github.com/open-edge-insights/eii-core/blob/master/build/builder.py>`_ script to regenerate the helm charts for provision and deployment.

----

Update the helm charts directory
--------------------------------


#. Copy the docker-compose.yml, eii_config.json into the eii-provision helm chart.
   .. code-block:: sh

      $ cd [WORKDIR]/IEdgeInsights/build
      $ cp docker-compose.yml provision/config/eii_config.json helm-eii/eii-provision/

#. To generate only Certificates by provisioning. 
   .. code-block:: sh

      $ cd [WORKDIR]/IEdgeInsights/build/provision
      $ sudo -E ./provision.sh ../docker-compose.yml --generate_certs

#. Copy the Certificates generated by provisioning process to the eii-provision helm chart.
   .. code-block:: sh

      $ cd [WORKDIR]/IEdgeInsights/build
      $ sudo chmod -R 755 provision/Certificates/
      $ cp -a provision/Certificates/ helm-eii/eii-provision/

   ..

      **Note**\ :
      The Certificates/ directory contains sensitive information. So post the installation of eii-provision helm chart, it is recommended to delete the Certificates from it.


Provision and deploy in the kubernetes node.
--------------------------------------------

Copy the helm charts in helm-eii/ directory to the node.


#. 
   Install provision helm chart

   .. code-block:: sh

      $ cd [WORKDIR]/IEdgeInsights/build/helm-eii/
      $ helm install eii-provision eii-provision/

   Verify the pod is in running state:

   .. code-block:: sh

      $ kubectl get pods

      NAME                       READY   STATUS    RESTARTS   AGE
      ia-etcd-58866469b9-dl66k   2/2     Running   0          8s

#. 
   Install deploy helm chart

   .. code-block:: sh

      $ cd [WORKDIR]/IEdgeInsights/build/helm-eii/
      $ helm install eii-deploy eii-deploy/

   Verify all the pod are running:

   .. code-block:: sh

      $ kubectl get pods

      NAME                                          READY   STATUS    RESTARTS   AGE
      deployment-etcd-ui-6c7c6cd769-rwqm6           1/1     Running   0          11s
      deployment-video-analytics-546574f474-mt7wp   1/1     Running   0          11s
      deployment-video-ingestion-979dd8998-9mzkh    1/1     Running   0          11s
      deployment-webvisualizer-6c9d56694b-4qhnw     1/1     Running   0          11s
      ia-etcd-58866469b9-dl66k                      2/2     Running   0          2m26s

The EII is now successfully deployed.

For running helm charts and deploying kube pods with specific namespace
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: \ : By default all our helm charts are deployed with ``default`` namespace, below commands will help us to deploy helm chart and kube pods with specific namespace

   .. code-block:: sh

          helm install --set namespace=<namespace> <helm_app_name> <helm_charts_directory>/ --namespace <namespace> --create-namespace

     For Eg.:


   * For Deploying ``eii-provision`` helm chart with ``eii`` namespace.
     .. code-block:: sh

          helm install --set namespace=eii eii-provision eii-provision/ --namespace eii --create-namespace

   * For Deploying ``eii-deploy`` helm chart with ``eii`` namespace.
     .. code-block:: sh

          helm install --set namespace=eii eii-deploy eii-deploy/ --namespace eii --create-namespace

   * Now all the ``pods`` and ``helm`` charts are deployed under ``eii`` namespace
   * For listing ``helm charts`` deployed with specific namespace
     .. code-block:: sh

          helm ls -n <namespace>

   * For listing ``kube pods`` deployed with specific namespace
     .. code-block:: sh

          kubectl get pods -n <namespace>
     ## Provision and deploy mode in times switching between dev and prod mode OR changing the usecase



#. 
   Set the DEV_MODE as "true/false" in  `.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ depending on dev or prod mode.

#. 
   Run builder to copy templates file to eii-deploy/templates directory and generate consolidated values.yaml file for eii-services:

   .. code-block:: sh

      $ cd [WORKDIR]/IEdgeInsights/build
      $ python3 builder.py -f usecases/<usecase>.yml

#. 
   Remove the etcd storage directory

   .. code-block:: sh

      $sudo rm -rf /opt/intel/eii/data/*

Do helm install of provision and deploy charts as per previous section.

.. note:: \ :
    During re-deploy(\ ``helm uninstall`` and ``helm install``\ ) of helm chart for eii-provision and eii-deploy wait for all the pervious pods to terminated successfully.


Steps to enable Accelarators
----------------------------

.. note:: \ :
   ``nodeSelector`` is the simplest recommended form of node selection constraint.
   ``nodeSelector`` is a field of PodSpec. It specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels (it can have additional labels as well). The most common usage is one key-value pair.



#. 
   Setting the ``label`` for a particular node

   .. code-block:: sh

      $ kubectl label nodes <node-name> <label-key>=<label-value>

#. 
   For HDDL/NCS2 dependenecies follow the steps for setting ``labels``.


   * For HDDL

   .. code-block:: sh

      $ kubectl label nodes <node-name> hddl=true


   * For NCS2

   .. code-block:: sh

      kubectl label nodes <node-name> ncs2=true

.. note:: \ : Here the node-name is your worker node machine hostname



#. 
   Open the ``[WORKDIR]/IEdgeInsights/VideoIngestion/helm/values.yaml`` or ``[WORKDIR]/IEdgeInsights/VideoAnalytics/helm/values.yaml`` file.

#. 
   Based on your workload preference. Add hddl or ncs2 to accelerator in values.yaml of video-ingestion or video-analytics. 


   * For HDDL

   .. code-block:: yml

      config:
       video_ingestion:
          .
          .
          .
         accelerator: "hddl"
         .
         .
         .


   * For NCS2

   .. code-block:: yml

      config:
       video_ingestion:
          .
          .
          .
         accelerator: "ncs2"
         .
         .
         .

#. 
   set device as "MYRIAD" in case of ncs2 and as HDDL in case of hddl in the `VA config <https://github.com/open-edge-insights/video-analytics/blob/master/config.json>`_


   * In case of ncs2.

   .. code-block:: yml

      "udfs": [{
       .
       .
       .
       "device": "MYRIAD"
       }]


   * In case of hddl.

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

      $ sudo rm -f /run/cni/dhcp.sock
      $ cd /opt/cni/bin
      $ sudo ./dhcp daemon

   ### Setting up Multus CNI and Enabling it.


   * Multus CNI is a container network interface (CNI) plugin for Kubernetes that enables attaching multiple network interfaces to pods. Typically, in Kubernetes each pod only has one network interface (apart from a loopback) -- with Multus you can create a multi-homed pod that has multiple interfaces. This is accomplished by Multus acting as a "meta-plugin", a CNI plugin that can call multiple other CNI plugins.


   #. Get the name of the ``ethernet`` interface in which gige camera & host system connected
      **Note**\ : Identify the network interface name by following command

   .. code-block:: sh

      $ ifconfig


   #. Execute the Following Script with Identified ``ethernet`` interface name as Argument for ``Multus Network Setup``
      **Note:** Pass the ``interface`` name without ``quotes``

   .. code-block:: sh

      $ cd [WORKDIR]/IEdgeInsights/build/helm-eii/gige_setup
      $ sudo -E sh ./multus_setup.sh <interface_name>

   ..

      **Note**\ : Verify ``multus`` pod is in ``Running`` state

      .. code-block:: sh

           $ kubectl get pods --all-namespaces | grep -i multus



   #. Set gige_camera to true in values.yaml

   .. code-block:: sh

      $ vi [WORKDIR]/IEdgeInsights/VideoIngestion/helm/values.yaml
      .
      .
      .
      gige_camera: true
      .
      .
      .


   #. 
      Follow the `Deployment Steps <#provision-and-deploy-in-the-kubernetes-node>`__

   #. 
      Verify ``pod``\ ip & ``host`` ip are same as per Configured ``Ethernet`` interface by using below command.

   .. code-block:: sh

      $ kubectl -n eii exec -it <pod_name> -- ip -d address

   **Note:**

   User needs to deploy as root user for MYRIAD(NCS2) device and GenICam USB3.0 interface cameras.

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
