
OEI provisioning and deployment via bundle generation
=====================================================

Perform the following steps to provision and deploy through bundle generation:


* `OEI provisioning and deployment via bundle generation <#oei-provisioning-and-deployment-via-bundle-generation>`__

  * `Step 1 Prerequisite <#step-1-prerequisite>`__
  * `Step 2 Set up Docker Registry URL then Build and Push Images <#step-2-set-up-docker-registry-url-then-build-and-push-images>`__
  * `Step 3 Choosing the OEI services to deploy <#step-3-choosing-the-oei-services-to-deploy>`__

Step 1 Prerequisite
-------------------

Before provisioning, complete the OEI prerequisites . For more information, see `OEI Prerequisites <https://github.com/open-edge-insights/eii-core/blob/master/README.md#oii-prerequisites>`_.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as OEI. This is due to the product name change of EII as OEI.


Step 2 Set up Docker Registry URL then Build and Push Images
------------------------------------------------------------

OEI Deployment through bundle generation must be done using a docker registry. Complete the following:


* Update docker registry url in DOCKER_REGISTRY variable in  `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_. Please use full registry URL with a traliling /
* 
  Building OEI images and pushing the same to docker registry.

  .. code-block::

     ```sh
     docker-compose -f docker-compose-build.yml build
     docker-compose -f docker-compose-push.yml push
     ```

Step 3 Choosing the OEI services to deploy
------------------------------------------

.. note::  This deployment bundle generated can be used to provision and also to deploy the OEI stack



#. This software works only on Python3+ version.
#. 
   Update the `config.json <https://github.com/open-edge-insights/eii-core/blob/master/build/deploy/config.json>`_ file.

   .. code-block:: sh

          {
              "docker_compose_file_version": "<docker_compose_file_version which is compose file version supported by deploying docker engine>",
              "exclude_services": [list of services which you want exclude for your deployement]
              "include_services": [list of services needed in final deployment docker-compose.yml]

         }

   **Note:**
    >

   ..

      * Validate the json.
      * Ensure that you have updated the DOCKER_REGISTRY in `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_ file.


Step 4 Creating OEI bundle for deployment
-----------------------------------------

.. note::  Before proceeding this step, ensure that you have completed steps 1 to 3.


.. code-block:: sh

     # commands to be executed for bundle creation:
     cd build/deploy
     sudo python3 generate_eii_bundle.py -gb

  This will generate the .tar.gz which has all the required artifacts

**Note:**


* Default Values of Bundle Name is "eii_bundle.tar.gz" and the tag name is "eii_bundle"
* Using the ``-t`` options, you can give your custom tag name which will be used as same name for bundle generation.

For more help:

.. code-block::

   $sudo python3 generate_eii_bundle.py


   usage: generate_eii_bundle.py [-h] [-f COMPOSE_FILE_PATH] [-t BUNDLE_TAG_NAME]

   OEI Bundle Generator: This utility helps to generate the bundle to deploy OEI.

   optional arguments:
       -h, --help            show this help message and exit
       -t BUNDLE_TAG_NAME    Tag Name used for Bundle Generation (default:eii_bundle)


Now this bundle can be used to deploy an OEI on the instance. This bundle has all the required artifacts to start the OEI services.
