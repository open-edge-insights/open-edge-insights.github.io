
Contents
========

Run the Web Deployment Tool back end
------------------------------------

Web Deployment Tool (WDT) provides a graphical interface to set up and complete the deployment process for OEI services. The following sections provide details for the prerequisites and configuration for the Web GUI Deployment tool back end.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights (OEI). This is due to the product name change of EII as OEI.


Prerequisites
^^^^^^^^^^^^^

Ensure that all the prerequisites for OEI are installed. For more details, refer to the `OEI ReadMe <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_.

Configuration
^^^^^^^^^^^^^

The back-end server runs on the port value that you enter for the env variable ``DEPLOYMENT_TOOL_BACKEND_PORT`` in the ``docker-compose.yml`` file. Also, based on the value for the env variable ``dev_mode`` in the ``docker-compose.yml`` file, the back-end server will run in the ``DEV mode`` (http/insecure) or the ``PROD mode`` (https/secure). By default, the PROD mode is enabled. The following snippet shows the default value for the env variable ``env_mode``\ :

.. code-block:: sh

   dev_mode: "false"

Configure Logging
~~~~~~~~~~~~~~~~~

Web Deployment Tool supports the following log levels:


* Debug
* Info
* Error

In the ``docker-compose.yml`` file, you can set the env variable ``LOG_LEVEL`` to configure logging. Refer to the following code snippet:

.. code-block:: sh

     LOG_LEVEL: "INFO"

.. note:: 
   You must enter your user credentials in the ``./creds.json`` file to log in to the Web Deployment Tool's back end. You must use the same user credentials to access the frontend of the Web Deployment tool. Refer to the following code snippet for the JSON format of the user credentials:


.. code-block:: sh

    {
     "username" : "password"
    }

Run the Web Deployment Tool
---------------------------

Perform the steps mentioned in this section to run the Web Deployment Tool.

Run the container without building
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To run the container without building it, run the following commands:

.. code-block:: shell

   cd [WORKDIR]/IEdgeInsights/DeploymentToolBackend
   ./run.sh

Build and run the container
^^^^^^^^^^^^^^^^^^^^^^^^^^^

To build and run the container, run the following commands:

.. code-block:: shell

   ./run.sh --build

or

.. code-block:: shell

   ./run.sh -b

To build and run with --no-cache or to provide any other build argument, you can append the build argument after running the previous commands.

For example:

.. code-block:: shell

   ./run.sh --build --no-cache

.. note::  If you are building the container for the first time, the Web Deployment Tool will prompt you for the host credentials and the credentials for the SSH key generation. The host user is then added to the sudoers with the NOPASSWORD option. This is for the container to seemlessly interact with the host.


Restart the container
^^^^^^^^^^^^^^^^^^^^^

Run any the following commands to restart the container:

.. code-block:: shell

   ./run.sh --restart

or

.. code-block:: shell

   ./run.sh -r

Bring down the container
^^^^^^^^^^^^^^^^^^^^^^^^

Run any of the following commands to bring down the container:

.. code-block:: shell

   ./run.sh --down

or

.. code-block:: shell

   ./run.sh -d

Web Deployment Tool API Documentation
-------------------------------------

The Web Deployment Tool auto-generates the OpenAPI documentation for all the REST APIs it exposes. This documentation can be accessed at its **/docs** endpoint.

For example:

.. code-block:: sh

   https://127.0.0.1:5100/docs

Un-install Web Deployment Tool backend
--------------------------------------

To completely remove the Web Deployment Tool backend, project files and other files
created on the host machine by the tool, run the *uninstall.sh* script as below:

.. code-block:: sh

   ./uninstall.sh

Running unit tests
------------------

Unit tests are available under [WORK_DIR]/IEdgeInsights/DeploymentToolBackend/tests directory.
These can be executed using PyTest. 
To start with, user needs to install PyTest utility, if not already done.

Run the below command to install PyTest:

.. code-block:: sh

   $ pip3 install --user pytest

User also need to configure tests/config.ini with various values needed for the test.

For example:

.. code-block:: sh

   [backend]
   # IP address of the machine where backend is running
   ip = 127.0.0.1

   # Port at which backend is running. Please refer to the corresponding
   # docker-compose.yml to get the port value. Default value is 5100
   port = 5100

   # Backend login credentials.
   # Please refer to the correspnding creds.json to know the credentails.
   username = admin
   password = admin@123

   [deploy_target]
   # IP address of the machine where the remote deployment will be made.
   ip = 10.10.1.1

   # Credentails for the remote machine
   username = testuser
   password = testpassword

   # Directory path in the remote machine, where the deployment files will be copied.
   dir = /home/testuser/deploy

   # Docker image names of the services that will be copied to the remote machine.
   # At least one image name need to be specified
   images = ["openedgeinsights/ia_configmgr_agent:3.0"]

To run the unit tests, execute the following commands:

.. code-block:: sh

   $ cd [WORK_DIR]/IEdgeInsights/DeploymentToolBackend/tests
   # pytest -v .

You may also run the tests individually. For e.g:

.. code-block:: sh

   $ pytest -v test_basic.py
   # pytest -v test_build_deploy.py

More info on PyTest is available at `https://docs.pytest.org/en/6.2.x/contents.html <https://docs.pytest.org/en/6.2.x/contents.html>`_
