
Contents
========

Web Deployment Tool front end
-----------------------------

The Web Deployment Tool provides a graphical user interface to configure and deploy single and multiple video streams. The following sections provide details for the prerequisites and configuration for the Web GUI Deployment tool front end.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights (OEI). This is due to the product name change of EII as OEI.
   To learn more about the back end of the Web Deployment Tool, refer to the `ReadMe <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-deployment-tool-backend/-/blob/master/README.md>`_.
   The current version of the Web Deployment Tool doesn't fully support running back end and front end containers on different machines.


Prerequisites
-------------

Complete the following prerequisites before using the Web Deployment tool:


* Install the prerequisites for running OEI. For more details, refer to the `OEI ReadMe <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_.
* `Prerequisites for video accelerators <https://github.com/open-edge-insights/eii-core#using-video-accelerators-in-ingestionanalytics-containers>`_
* `Prerequisites for cameras <https://github.com/open-edge-insights/video-ingestion#camera-configuration>`_

Configuration
-------------

The front-end server runs on the port value that you enter for the env variable ``PORT`` in the ``docker-compose.yml`` file. Based on the value for the env variable ``dev_mode`` in the ``docker-compose.yml`` file, the front-end server will run in the ``DEV mode`` (http/insecure) or the ``PROD mode`` (https/secure). By default the PROD mode is enabled. The following code snippet shows the default value for the env variable ``env_mode``\ :

.. code-block:: sh

   dev_mode: "false"

Run the Web Deployment Tool front end
-------------------------------------

This section provides details for running the Web Deployment Tool for various scenarios.

Run the container without building
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To run the container without building it, run the following commands:

.. code-block:: shell

    cd [WORKDIR]/IEdgeInsights/DeploymentToolFrontend
    ./run.sh

Build and run container
^^^^^^^^^^^^^^^^^^^^^^^

To build and run the container, run any of the following commands:

.. code-block:: shell

   ./run.sh --build

or

.. code-block:: shell

   ./run.sh -b

To build and run with ``--no-cache`` or to provide any other build argument, append the build arguement after the previous commands. For example:

.. code-block:: shell

   ./run.sh --build --no-cache

Restart the container
^^^^^^^^^^^^^^^^^^^^^

To restart the container, run any of the following commands:

.. code-block:: shell

    ./run.sh --restart

or

.. code-block:: shell

    ./run.sh -r

Bring down the container
^^^^^^^^^^^^^^^^^^^^^^^^

To bring down the container, run any of the following commands:

.. code-block:: shell

   ./run.sh --down

or

.. code-block:: shell

   ./run.sh -d

Web Deployment Tool workflow
----------------------------

This section provides details for how to use the Web Deployment Tool (WDT) to configure and deploy video streams using the front end of the WDT.

Start the front-end server
^^^^^^^^^^^^^^^^^^^^^^^^^^

After the ``./run.sh`` file runs, it may take 30 seconds for the front-end server to come up. You may need to wait longer if you are running the tool for the first time. You can run the following command to view the logs and check if the front-end server is up:

.. code-block:: shell

   docker logs -f deployment_tool_frontend

Log in to the Web Deployment Tool
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After you start the front-end server and when the log shows the ``"Starting the development server..."`` message, complete the following steps to log in to the WDT front end:


#. 
   From a browser, go to http(s)://\<host-ip&gt;:\<host-port&gt;

    For example, for the ``PROD mode`` go to:

   .. code-block:: sh

       https://127.0.0.1:3100

    For the ``DEV mode``\ , go to:

   .. code-block:: sh

       http://127.0.0.1:3100

   ..

      **Note:** Currently the Web Deployment Tool is supported only on the Google Chrome browser.


#. 
   On the **Sign in** page, enter your user credentials and click **Sign in**.

   ..

      **Note:** To log in, use the credentials mentioned in the ``creds.json`` file in the ``DeploymentToolBackend`` repo. For more details, refer to the `ReadMe <https://gitlab.devtools.intel.com/Indu/edge-insights-industrial/eii-deployment-tool-backend/-/blob/master/README.md>`_
      Ensure that the front end and the back end are running in the same mode (DEV mode or PROD mode). By default, the front end and the back end are configured to run in the ``PROD`` mode.


Configure and deploy video streams
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After you sign in to the WDT, complete the following steps to configure and deploy the video streams:


#. In the **Create or select a project** section, select **Create a new project**.
#. In the **Project name** field, enter the name of the project.
#. From the **Number of datastreams** list, select the required number of datastreams.
   ..

      **Note:** Currently WDT supports maximum of 6 streams or instances.


#. Click **Next**.
#. On the **Configure & Build** page, in the **Data Streams**\ , view the layout of the use case.
#. If required, on the left pane, from the **Components List**\ , drag and drop the required components to the data stream. This will add the component and video stream to the use case layout.
   ..

      **Note:** The new stream is added as a Video Ingestion and Video Analytics pair (VI-VA pair).
      In the current version of WDT, the VI only use case is not supported.


#. If required, to remove a component, click and select the component and then press the Delete key on the keyboard.
   ..

      **Note:** If you delete a stream, then only the last stream that was added gets deleted, irrespective of the stream that is selected. This is due to a limitation in the platform.


#. If required, from the **Settings** section, change the settings for the component.
#. Click **Save**.
#. If required, you can configure the VI to use camera as an ``ingestor`` (to receive camera input). If required, to optimize the output preview you can adjust the camera settings on the fly from the *Test* screen.
   ..

      **Note:** In the current version of WDT, the camera adjustment is supported only for the USB cameras. Also, for other camera types, if you need to do additional configuration for that device to work then you must do it manually on the platform.


#. If required, to add an algorithm or User Defined Function (UDF) to the config, from the **Components List**\ , click **Import UDF**.
#. On the **Import UDF** screen, browse and select the required UDF.
   ..

      **Note:** To import a new UDF that doesn't exist in the common/video/udfs folder manually copy it to the udfs folder. As of now, only the Python UDFs are supported to import via the Import UDF functionality.


#. On the **Add to** section, select the required component.
#. Click **Import**.
#. On the **Configure & Build** screen, after completing the configuration, in the **Build** section, click **Start**.
   ..

      **Note:** The progress bar indicates the progress of the build.
      Currently there's no option to stop or cancel a build. If the build fails, click **Start** to retry the build. You can check the build failure reason by checking the build logs.
      To view the build logs, click  **View Logs**. User can select the 'Enable Auto Refresh' option to refresh the logs. The logs are refreshed every 5 seconds. 


#. After completing the build 100 percent, click the **Next**.
#. On the **Test**\ screen, to preview the output from VA for a data stream, click and select the required Video Analytics component.
   ..

      **Note:** When you click a VI or VA component, the settings associated to the selected component is displayed in the **Settings* section. You can view settings such as **\ Camera Settings**, Pipeline Settings, and UDF settings.


#. If required, update the settings, and click **Save & Restart**. This will save the changes to setting and restart the containers and you can view the updated output.
#. If you have configured a USB camera in VI, then you can view and adjust the camera controls from the **Camera Settings** section. You can make the setting changes the fly and view the preview.
   ..

      **Note:** On low end machines the camera control adjustments may not be fast enough.


#. If required, you can click **Back** to go to the **Configure & Build** screen. You can also go to the **Project** screen and click **Cancel** to cancel the current project and choose another project.
#. After checking the preview, on the **Test** screen, click **Next**.
#. On the **Deploy** screen, to deploy on a local machine, in the **Target Machine** section, select **Local Machine**.
#. In the **Deployment Mode** section, select the required deployment mode (Dev or Prod mode) and then, click **Deploy**.
#. To deploy on a remote machine, from the **Target Machine** section, select **Remote Machine**.
#. From the **Deployment Mode** section, select the required deployment mode (DEV or PROD mode).
   ..

      **Note:** Before you start the deployment, ensure that the specified directory in the remote machine is empty, otherwise the deployment might fail.


#. In the **Remote Machine Detail** section, enter the details of the remote machine and then, click **Deploy**.
   ..

      **Note:** After the deployment is triggered or done, you cannot go back to the **Test** screen or the **Configure** screen. You can either click **Sign out** to sign out or click **Cancel** to go to the *Project selection* screen.


#. On the **Deployment Successful** dialog, click **Close**.

Bring up the containers
^^^^^^^^^^^^^^^^^^^^^^^

To bring up the front-end containers in the remote machine, complete the deployment, and then, log in to the remote machine. Go to the build directory (under the directory that was specified while deploying). Update the ``HOST_IP`` and the ``ETCD_HOST`` values to the current machine IP, manually in the ``build/.env`` and run the following commands:

.. code-block:: shell

   ./source.sh
   ./eii_start.sh

.. note::  It may take several minutes for the front end to come up if you are building and running WDT on a fresh machine.

