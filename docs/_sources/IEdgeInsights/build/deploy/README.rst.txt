
Multi-node Provisioning & Deployment
====================================

Perform the below steps  to achieve provisioning & deployment on multiple nodes

`Step 1 Provision the Leader node <http://localhost:7645/IEdgeInsights/build/deploy/#step-1-provision-the-leader-node>`_

`Step 2 Set up Docker Registry URL then Build and Push Images <http://localhost:7645/IEdgeInsights/build/deploy/#step-2-set-up-docker-registry-url-then-build-and-push-images>`_

`Step 3 Choosing the EII services to run on worker node <http://localhost:7645/IEdgeInsights/build/deploy/#step-3-choosing-the-eii-services-to-run-on-worker-node>`_

`Step 4 Provisioning the worker node <http://localhost:7645/IEdgeInsights/build/deploy/#step-4-provisioning-the-worker-node>`_

`Step 5 Creating eii bundle for worker node <http://localhost:7645/IEdgeInsights/build/deploy/#step-5-creating-eii-bundle-for-worker-node>`_

`Step 6 EII-Multinode deployment <http://localhost:7645/IEdgeInsights/build/deploy/#step-6-eii-multinode-deployment>`_

Step 1 Provision the Leader node
================================

..

   **Pre-requisite**\ :
   Please follow the EII Pre-requisites before Provisioning.
   `EII Pre-requisites <https://github.com/open-edge-insights/eii-core/blob/master/README.md#eii-pre-requisites>`_

   **NOTE**\ :


   * EII services can run on leader as well as worker nodes
   * Leader node should have the entire repo/source code present
   * Leader node is the primary administative node and has following attributes:

     #. Generating required certificates and secrets.
     #. Loading Initial ETCD values.
     #. Generating bundles to provision new nodes.
     #. Generating bundles to deploy EII services on new/worker nodes.


For running EII in multi node, we have to identify one node to run ETCD server (this node is called as ``leader`` node). For a leader node, ETCD_NAME in `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/env>`_ must be set to ``leader``. Rest other nodes are ``Worker`` nodes which doesn't run ETCD server, instead all the worker nodes remotely connect to the ETCD server running on the ``Leader`` node only. Make sure the ``date`` and ``time`` are in sync in all nodes.

Provision the Leader node using the below command:

.. code-block:: sh

   $ cd [WORK_DIR]/IEdgeInsights/build/provision
   $ sudo -E ./provision.sh <path_to_eii_docker_compose_file>

   $ # eq. $ sudo -E ./provision.sh ../docker-compose.yml

This creates the ETCD server (Container ia_etcd) on the leader edge node.

Step 2 Set up Docker Registry URL then Build and Push Images
============================================================

EII Deployment on multiple node must be done using a docker registry.

Follow below steps:


* 
  Please update docker registry url in DOCKER_REGISTRY variable in  `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/env>`_ on any node(leader/worker). Please use full registry URL with a traliling /

* 
  Building EII images and pushing the same to docker registry.

  .. code-block::

     ```sh
     docker-compose -f docker-compose-build.yml build
     docker-compose -f docker-compose-push.yml push
     ```

Step 3 Choosing the EII services to run on worker node.
=======================================================

..

   Note: This deployment bundle generated can be used to provision and also to deploy EII stack on new node



#. This software works only on python3+ version.
#. 
   Please update the `config.json <https://github.com/open-edge-insights/eii-core/blob/master/config.json>`_ file

   .. code-block::

           {
               "docker_compose_file_version": "<docker_compose_file_version which is compose file version supported by deploying docker engine>",

               "exclude_services": [list of services which you want exclude for your deployement]
               "include_services": [list of services needed in final deployment docker-compose.yml]

           }

    **\ *Note*\ **\ :

   ..

      #. 
         Please validate the json.

      #. 
         Please ensure that you have updated the DOCKER_REGISTRY in `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/env>`_ file


Step 4 Provisioning the worker node
===================================

.. code-block::

       # Please ensure the following fields are correctly set in the leader node before generating the provisioning bundle:
       ETCD_HOST=<IP address of leader node>
       DOCKER_REGISTRY=<Docker registry details>

       # commands to be executed on leader node:
       $ cd build/deploy
       $ sudo python3 generate_eii_bundle.py -p

       This will generate the 'worker_provisioning.tar.gz'.
       Do a manual copy of this bundle on worker node. And then follow below commands
       on worker node.

.. code-block::

       # commands to be executed on worker node.
       $ tar -xvzf worker_provisioning.tar.gz
       $ cd worker_provisioning/provision/
       $ sudo -E ./provision.sh

Step 5 Creating eii bundle for worker node
==========================================

.. note:: \ : Before proceeding this step, please make sure, you have followed steps 1-4.


.. code-block::

       # Please ensure the following fields are correctly set in the leader node before generating the eii bundle:
       ETCD_HOST=<IP address of leader node>
       DOCKER_REGISTRY=<Docker registry details>

       # commands to be executed on leader node:
       $ cd build/deploy
       $ sudo python3 generate_eii_bundle.py

       This will generate the .tar.gz which has all the required artifacts by which eii services
       can be started on worker node.

**\ *Note*\ **\ :

.. code-block::

   1. Default Values of Bundle Name is "eii_bundle.tar.gz" and the tag name is "eii_bundle"

   2. Using *-t* options you can give your custom tag name which will be used as same name for bundle generation.

   3. Default values for docker-compose yml path is "../docker-compose.yml"


For more help:

.. code-block::

   $sudo python3 generate_eii_bundle.py


   usage: generate_eii_bundle.py [-h] [-f COMPOSE_FILE_PATH] [-t BUNDLE_TAG_NAME]

   EII Bundle Generator: This Utility helps to Generate the bundle to deploy EII.

   optional arguments:
       -h, --help            show this help message and exit
       -t BUNDLE_TAG_NAME    Tag Name used for Bundle Generation (default:eii_bundle)



Now this bundle can be used to deploy an eii on worker node. This bundle has all the required artifacts to start the eii
services on worker node.

Step 6 EII-Multinode deployment
===============================

.. note:: \ : Before proceeding this step, please make sure, you have followed steps 1-5. Please make sure to copy and untar the above bundle on a secure location having root only access as it contains secrets. Please ensure you mandatorily run the below commands with sudo option as shown below.


.. code-block::

       $ sudo tar -xvzf <eii_bundle gz generated in Step No. 5>
       $ cd eii_bundle
       $ docker-compose up -d

       eq.

       $ sudo tar -xvzf eii_bundle.tar.gz
       $ cd eii_bundle
       $ docker-compose up -d
