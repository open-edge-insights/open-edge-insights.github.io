
EII Orchestration using k8s (a.k.a) Kuberenetes Orchestrator
============================================================

EII Orchestration using k8s Orchestrator.


#. 
   `k8s Setup <http://localhost:7645/IEdgeInsights/build/k8s/#k8s-setup>`_

#. 
   `Steps for Multinode Deployment <http://localhost:7645/IEdgeInsights/build/k8s/#steps-for-multinode-deployment>`_

#. 
   `Provisioning with k8s <http://localhost:7645/IEdgeInsights/build/k8s/#provisioning-with-k8s>`_

#. 
   `Deploying EII Application with K8s <http://localhost:7645/IEdgeInsights/build/k8s/#deploying-eii-application-with-k8s>`_

#. 
   `Accessing Web Visualizer and EtcdUI <http://localhost:7645/IEdgeInsights/build/k8s/#Accessing-Web-Visualizer-and-EtcdUI>`_

#. 
   `Steps for Enabling Basler Camera with k8s <http://localhost:7645/IEdgeInsights/build/k8s/#steps-for-enabling-basler-camera-with-k8s>`_

#. 
   `Steps to enable Accelarators <http://localhost:7645/IEdgeInsights/build/k8s/#steps-to-enable-accelarators>`_

#. 
   `Steps to enable IPC Mode <http://localhost:7645/IEdgeInsights/build/k8s/#steps-to-enable-ipc-mode>`_

#. 
   `Accessing Web Visualizer and EtcdUI <http://localhost:7645/IEdgeInsights/build/k8s/#accessing-web-visualizer-and-etcdui>`_

K8s Setup
---------

.. note:: \ :
   Please follow the standard proxy settings if your installation machines are behind proxy environment.
   For orchestrator deployment in a proxy environment, make sure all communicating machines IP addresses
   are in ``no_proxy`` list both in ``docker`` & ``system`` environment files as follows.


   #. /etc/environment
   #. /etc/systemd/system/docker.service.d/http-proxy.conf
   #. ~/.docker/config.json

   ``K8s installation should be done as pre-requisite to continue the following deployment``

   Steps for Multinode Deployment
   ------------------------------


   #. ``K8s Master node & Worker(s) node Setup should be done as pre-requisite to continue the following deployment``
   #. ``This step is not required for **Single Node** deployment``

      * Pre-Requisites


.. code-block::

   **Note** For Running EII Modules in k8s multi node cluster, corresponding module Container Images should be present in the master and worker(s) nodes. The container images can be pushed to a Docker registry and pulled on the required nodes. For more information on Docker registry- Kindly refer:
   [build/deploy/README.html/#step-2-set-up-docker-registry-url-then-build-and-push-images](http://localhost:7645/IEdgeInsights/#step-2-set-up-docker-registry-url-then-build-and-push-images)



* Master Node provisoning: Follow steps mentioned in below links.

  #. `Change for Master Provision for Multinode Deployment <http://localhost:7645/IEdgeInsights/build/k8s/###change-for-master-provision-for-multinode-deployment>`_
  #. `Provisioning with k8s <http://localhost:7645/IEdgeInsights/build/k8s/#provisioning-with-k8s>`_

* Worker Node provisoning: Follow steps mentioned in below links.

  #. `Provision Bundle creation for k8s worker node <http://localhost:7645/IEdgeInsights/build/k8s/#Provision-Bundle-creation-for-k8s-worker-node>`_
  #. `EII k8s Worker Node provisioning for MultiNode deployment <http://localhost:7645/IEdgeInsights/build/k8s/#EII-k8s-Worker-Node-provisioning-for-MultiNode-deployment>`_

Change for Master Provision for Multinode Deployment.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* 
  Get the master node name by running below command.

  .. code-block:: sh

      $ kubectl get nodes

     It will display NAME along with ROLES. Copy the complete ``name`` of Node with ROLES as ``master``.

  .. code-block::

        E.g.
        $ kubectl get nodes
        NAME     STATUS          ROLES    AGE   VERSION
        coolm    Ready           master   67d   v1.18.6

     For example, copied ``coolm`` from above.


  * 
    Add copied master node name(e.g. coolm) with label ``nodeName under spec of Pod ia-etcd as nodeName:`` in below yaml based on DEV mode.


    * If DEV mode =TRUE  `build/provision/dep/k8s/k8s_etcd_devmode.yml <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-core/-/blob/master/k8s_etcd_devmode.yml>`_
    * IF DEV mode=FALSE  `build/provision/dep/k8s/k8s_etcd_prodmode.yml <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-core/-/blob/master/k8s_etcd_prodmode.yml>`_

    This will enable etcd to run always on master node.

    For example, added  ``nodeName:coolm`` in etcd_devmode.yaml or etcd_prodmode.yaml which was copied above.

    .. code-block::

       E.g.
       spec:
         restartPolicy: OnFailure
         nodeName: coolm

    ### Provision Bundle creation for k8s worker node.

    ..

       #. This step is not required for **Single Node** deployment
       #. Provisioning bundle will be generated on k8s ``Master node``.
       #. Make Sure ETCD_NAME=[any name other than ``master``\ ] in `build/.env <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-core/-/blob/master/.env>`_ of Master node.
       #. Below steps to be run on Master node.


  * Set ``PROVISION_MODE=k8s`` in `build/provision/.env <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-core/-/blob/master/.env>`_ file.
  * Go to ``$[WORK_DIR]/IEdgeInsights/build/deploy`` directory
  * 
    Generate Provisioning bundle for k8s worker node provisioning.

    .. code-block:: sh

        $ sudo python3 generate_eii_bundle.py -p

    ### EII ``k8s Worker Node`` provisioning for MultiNode deployment

    ..

       **Note**\ :


       #. This should be executed in k8s worker nodes in **\ *Multi node
          scenario*\ ** only.
       #. This step is not required for **Single Node** deployment
       #. Provisioning bundle generated on k8s Master node will be copied to worker node.


  * 
    Copy the ``eii_provisioning.tar.gz`` file from k8s master node to worker node(s).

    .. code-block:: sh

            $ sudo scp <eii_provisioning.tar.gz> <any-directory_on-worker-Filesystem>
            $ sudo tar -xvf <eii_provisioning.tar.gz>
            $ cd <eii_provisioning>

  * 
    Provision the EII in k8s Worker(s) Node.

    .. code-block:: sh

            $ cd provision
            $ sudo ./provision.sh

    ## Provisioning with k8s

    ..

       **Note:** If Provisioning of EII with k8s is already done under Multinode deployment, please skip it.


       #. EII Deployment with k8s can be done in **PROD** & **DEV** mode.
       #. For running EII in multi node, we have to identify one master node.For a master node ``ETCD_NAME`` in `build/.env <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-core/-/blob/master/build/.env>`_ must be set to ``master``.
       #. Please follow the `EII Pre-requisites <http://localhost:7645/IEdgeInsights/README.html#eii-pre-requisites>`_ before k8s Provisioning.


  * 
    Please Update following Environment Variables in `build/.env <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-core/-/blob/master/build/.env>`_


    * HOST_IP           =     [masternodeip]
    * ETCD_HOST         =     [masternodeip]
    * ETCD_CLIENT_PORT  =     8379
    * ETCD_PEER_PORT    =     8380

  * 
    Please Update following Environment Variables in `build/provision/.env <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-core/-/blob/master/.env>`_


    * EII_LOCAL_PATH    =     [EII Source path]
    * PROVISION_MODE    =     k8s

Provisioning ``EII`` with ``k8s`` in ``DEV`` mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

..

   **Note:** ``k8s_eii_deploy.yml`` file be generated in ``build/k8s`` directory by ``eiibuilder``.
   Once 'DEV_MODE\ ``is updated. We should re-run``\ eiibuilder\ ``to generate
   updated``\ build/k8s/k8s_eii_deploy.yml` file.
     Please follow the `EII Pre-requisites <http://localhost:7645/IEdgeInsights/README.html#eii-pre-requisites>`_



* 
  Please Update ``DEV_MODE=true`` in `build/.env <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-core/-/blob/master/build/.env>`_ file.

* 
  Provision with K8s.

  .. code-block:: sh

     $ sudo ./provision.sh <path_to_eii_docker_compose_file>

     eq. $ sudo ./provision.sh ../docker-compose.yml

  ### Provisioning ``EII`` with ``k8s`` in ``PROD`` mode

  ..

     **Note:** ``k8s_eii_deploy.yml`` file be generated in ``build/k8s`` directory by ``eiibuilder``.
     Once 'DEV_MODE\ ``is updated. We should re-run``\ eiibuilder\ ``to generate
     updated``\ build/k8s/k8s_eii_deploy.yml` file.
     Please follow the `EII Pre-requisites <http://localhost:7645/IEdgeInsights/README.html#eii-pre-requisites>`_


* 
  Please Update ``DEV_MODE=false`` in `build/.env <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-core/-/blob/master/build/.env>`_ file.

* 
  Provision with K8s.

  .. code-block:: sh

     $ sudo ./provision.sh <path_to_eii_docker_compose_file>

     eq. $ sudo ./provision.sh ../docker-compose.yml

  ## Deploying EII Application with K8s

  ..

     **Note:** ``eii-k8s-deploy.yml`` file be generated in ``build/k8s`` directory by ``eiibuilder``.


* 
  Goto ``build/k8s`` directory.

  .. code-block:: sh

     $ cd build/k8s

* Deploy ``k8s`` using ``kubectl`` utility by following command
  .. code-block:: sh

     $  kubectl apply -f ./eii-k8s-deploy.yml

* Make sure the pods are ``Running`` fine as per deploy yml file.
  .. code-block:: sh

     $  kubectl -n eii get pods
  ## Accessing Web Visualizer and EtcdUI
  In K8s Environment EtcdUI & WebVisualizer will be running in Following ports. 
* EtcdUI

  * PROD Mode --  ``https://master-nodeip:30010/``
  * DEV Mode -- ``http://master-nodeip:30010/``

* WebVisualizer

  * PROD Mode --  ``https://master-nodeip:30007/``
  * DEV Mode -- ``http://master-nodeip:30009/``

Steps for Enabling Basler Camera with k8s
-----------------------------------------

..

   **Note:** For more information on ``Multus`` please refer this git https://github.com/intel/multus-cni
   Skip installing multus if it is already installed and execute others.



* 
  Prequisites
  For enabling basler camera with k8s. K8s pod networks should be enabled ``Multus`` Network Interface
  to attach host system network interface access by the pods for connected camera access.

  ..

     **Note**\ : Please follow the below steps & make sure ``dhcp daemon`` is running fine.If there is an error on ``macvlan`` container creation on accessing the socket or if socket was not running. Please execute the below steps again

     .. code-block:: sh

          $ sudo rm -f /run/cni/dhcp.sock
          $ cd /opt/cni/bin
          $ sudo ./dhcp daemon


  ### Setting up Multus CNI and Enabling it.


  * 
    Multus CNI is a container network interface (CNI) plugin for Kubernetes that enables attaching multiple network interfaces to pods. Typically, in Kubernetes each pod only has one network interface (apart from a loopback) -- with Multus you can create a multi-homed pod that has multiple interfaces. This is accomplished by Multus acting as a "meta-plugin", a CNI plugin that can call multiple other CNI plugins.

  * 
    Get the name of the ``ethernet`` interface in which basler camera & host system connected

    ..

       **Note**\ : Identify the network interface name by following command

       .. code-block:: sh

            $ ifconfig


  * 
    Execute the Following Script with Identified ``ethernet`` interface name as Argument for ``Multus Network Setup``

    ..

       **Note:** Pass the ``interface`` name without ``quotes``

       .. code-block:: sh

          $ sudo -E sh ./k8s_multus_setup.sh <interface_name>


  * 
    Update ``macvlan`` network config ``VideoIngestion`` deployment ``yml`` file in ``VideoIngestion/k8s-service.yml``

  .. code-block:: yml

           annotations:
               k8s.v1.cni.cncf.io/networks: macvlan-conf

  Eg.

  .. code-block:: yml

       template:
         metadata:
           labels:
             app: video
           annotations:
             k8s.v1.cni.cncf.io/networks: macvlan-conf

         spec:
           containers:
           - name: ia-video-ingestion


  * 
    Follow the `Deployment Steps <http://localhost:7645/IEdgeInsights/build/k8s/#deploying-eii-application-with-k8s>`_

  * 
    Verify ``pod``\ ip & ``host`` ip are same as per Configured ``Ethernet`` interface by using below command.

    .. code-block:: sh

           $ kubectl -n eii exec -it <pod_name> -- ip -d address

    ## Steps to enable Accelarators

    ..

       **Note**\ :
       ``nodeSelector`` is the simplest recommended form of node selection constraint.
       ``nodeSelector`` is a field of PodSpec. It specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels (it can have additional labels as well). The most common usage is one key-value pair.


  * 
    Setting the ``label`` for a particular node

    .. code-block:: sh

       $ kubectl label nodes <node-name> <label-key>=<label-value>

  * 
    For HDDL/NCS2 dependenecies follow the steps for setting ``labels``.

  * For HDDL
    .. code-block:: sh

       kubectl -n eii label nodes <node-name> hddl=true

  * 
    For NCS2

    .. code-block:: sh

       kubectl -n eii label nodes <node-name> ncs2=true

    **Note** Here the node-name is your worker node machine hostname

  * 
    Updating the ``yml`` File.

  * 
    Open the ``build/k8s/eii-k8s-deploy.yml`` file.

  * 
    Based on your
    workload preference. Add a ``nodeSelector`` field to your ``pod`` configuration for ``VideoIngestion`` or ``VideoAnalytics`` Module(s) ``k8s-service.yml`` yml file. 

    .. code-block:: sh

       $   spec:
             nodeSelector:
               <key>: <value>

  * For HDDL
    .. code-block:: sh

         .
         .
         .
          spec:
             nodeSelector:
             hddl: "true"
             containers:
               - name: ia-video-ingestion
               image: ia_video_ingestion:2.4
         .
         .
         .

  * For NCS2
    .. code-block:: sh

         .
         .
         .
          spec:
             nodeSelector:
             ncs2: "true"
             containers:
               - name: ia-video-ingestion
               image: ia_video_ingestion:2.4
         .
         .
         .

  * 
    Run the ``build/builder.py`` for generating latest consolidated ``deploy`` yml file based on your ``nodeSelector`` changes set in the respective Modules ``k8s-service.yml`` files.

    .. code-block:: sh

       cd build/
       python3 builder.py

  * 
    Deploy the ``latest`` generated ``yml`` file

    .. code-block:: sh

       cd build/k8s/
       kubectl apply -f ./eii-k8s-deploy.yml

  * 
    Verify the respecitve workloads are running based on the ``nodeSelector`` constraints.

Steps to enable IPC mode
------------------------

.. note::  To achieve VideoIngestion & VideoAnalytics Module run in IPC mode. Both the Services should be running in same node. By following the below steps it makes the modules to the select the same node in ipc mode based on the label & config updated in ``k8s/eii-k8s-deploy.yml`` file.



* 
  Open the ``VideoIngestion/k8s-service.yml`` file

* 
  Update the ``PUBLISHER_TYPE`` and ``PUBLISHER_ENDPOINT`` env value for ``IPC`` connection in ``videoingestion`` deployment yaml section like below:

  .. code-block:: yml

             - name: PUBLISHER_TYPE
                 value: "zmq_ipc"
             - name: PUBLISHER_ENDPOINT
                 value: "${SOCKET_DIR}/"

* 
  Open the ``VideoAnalytics/k8s-service.yml`` file

* 
  Update the ``SUBSCRIBER_TYPE`` and  ``SUBSCRIBER_ENDPOINT`` env value for ``IPC`` connection in ``videoanalytics`` deployment yaml section like below:

  .. code-block:: yml

             - name: SUBSCRIBER_TYPE
                 value: "zmq_ipc"
             - name: SUBSCRIBER_ENDPOINT
                 value: "${SOCKET_DIR}/"

* 
  Add a ``nodeSelector`` field to your ``deployment`` configuration of both ``videoingestion`` & ``videoanalytics`` yaml above the ``containers`` in ``spec`` section.

  .. code-block:: yml

           spec:
             nodeSelector:
               ipc: enabled

    For Eg. Following snippet shows the ``nodeSelector`` section in ``videoanalytics`` yaml file.

  .. code-block:: yml

             // ... omitted ...
             .
             .
             .
           spec:
             replicas: 1
             selector:
               matchLabels:
                 app: videoanalytics
             template:
               metadata:
                 labels:
                   app: videoanalytics
               spec:
                 nodeSelector:
                   ipc: enabled
                 containers:
                 - name: ia-video-analytics
                   image: ...
               .
               .
               .
               .
               .
               // ... omitted ...


* 
  Attach label to the ``node``


  #. 
     Run ``kubectl get nodes`` to get the names of your cluster's nodes. 
      Pick out the one that you want to add a label to, and then run 

     .. code-block:: sh

         $ kubectl -n eii label nodes <node-name> ipc=enabled

      to add a ``label`` to the node you've chosen.

  #. 
     Verify the node labelled properly with following command.

     .. code-block:: sh

         $ kubectl -n eii get nodes --show-labels <node-name> | grep ipc=enabled

* Run ``builder`` from ``build`` directory to generate the updated yaml file.
  .. code-block::

     ```sh
        python3 ./builder.py
     ```

* Deploy the ``latest`` generated ``yaml`` file from ``k8s`` directory
  .. code-block:: sh

           $ cd build/k8s/
           $ kubectl apply -f ./eii-k8s-deploy.yml

* Verify the respecitve workloads are running based on the ``nodeSelector`` constraints with ``ipc`` mode.

Accessing Web Visualizer and EtcdUI
-----------------------------------

  In K8s Environment EtcdUI & WebVisualizer will be running in Following ports. 


* EtcdUI

  * PROD Mode --  ``https://master-nodeip:30010/``
  * DEV Mode -- ``http://master-nodeip:30010/``

* WebVisualizer

  * PROD Mode --  ``https://master-nodeip:30007/``
  * DEV Mode -- ``http://master-nodeip:30009/``
