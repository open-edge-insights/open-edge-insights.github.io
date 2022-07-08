
Contents
========


* `Contents <#contents>`__

  * `Grafana Overview <#grafana-overview>`__
  * `Configuration <#configuration>`__
  * `Run Grafana <#run-grafana>`__

    * `Run Grafana in the PROD mode <#run-grafana-in-the-prod-mode>`__
    * `Run Grafana in the DEV mode <#run-grafana-in-the-dev-mode>`__
    * `Execute queries <#execute-queries>`__
    * `Run Grafana for video use cases <#run-grafana-for-video-use-cases>`__

Grafana Overview
----------------

Grafana is an open-source metric analytics and visualization suite. Its uses include:


* Visualizing time series data for infrastructure and application analytics
* Industrial sensors, home automation, weather, and process control

Grafana supports various storage backends for the time-series data (data source). Open Edge Insights for Industrial (Open EII) uses InfluxDB as the data source. Grafana connects to the InfluxDB data source which has been preconfigured as a part of the Grafana setup. The ``ia_influxdbconnector`` service must be running for Grafana to be able to collect the time-series data. After the data source starts working, you can use the preconfigured dashboard to visualize the incoming data. You can also edit the dashboard as required.

Configuration
-------------

The following are the configuration details for Grafana:


* 
  `dashboard.json <https://github.com/open-edge-insights/ts-grafana/blob/master/dashboard.json>`_\ : This is the dashboard json file that is loaded when Grafana starts. It is preconfigured to display the time-series data.

* 
  `dashboard_sample.yml <https://github.com/open-edge-insights/ts-grafana/blob/master/dashboard_sample.yml>`_\ : This is the config file for all the dashboards. It specifies the path to locate all the dashboard json files.

* 
  `datasource_sample.yml <https://github.com/open-edge-insights/ts-grafana/blob/master/datasource_sample.yml>`_\ : This is the config file for setting up the data source. It has various fields for data source configuration.

* 
  `grafana_template.ini <https://github.com/open-edge-insights/ts-grafana/blob/master/grafana_template.ini>`_\ : This is the config file for Grafana. It specifies how Grafana should start after it is configured.

.. note::  You can edit the contents of these files based on your requirement.


Run Grafana
-----------

Based on requirement, you can run Grafana in the ``Prod mode`` or the ``DEV mode``.

Complete the following steps to run Grafana:


#. Open the `docker-compose.yml <https://github.com/open-edge-insights/eii-core/blob/master/docker-compose.yml>`_ file.
#. In the ``docker-compose.yml`` file, uncomment ``ia_grafana``.
#. Check if the ``ia_influxdbconnector``\ , ``ia_kapacitor``\ , and ``ia_telegraph`` services are running for the time-series data.
#. Check if the `publisher <https://github.com/open-edge-insights/eii-tools/blob/master/mqtt-publisher/publisher_temp.sh>`_ service is running.
#. Run the ``docker-compose build`` command to build image.
#. Run the ``docker-compose up`` to start the service.

Complete the previous steps and based on the mode that you want to run Grafana refer to the following sections:

Run Grafana in the PROD mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note::  Skip this section, if you are running Grafana in the DEV mode.


To run Grafana in the PROD mode, import ``cacert.pem`` from the ``build/Certificates/rootca/`` directory to the browser certificates. Complete the following steps to import certificates:


#. In Chrome browser, go to **Settings**.
#. In **Search settings**\ , enter **Manage certificates**.
#. Click **Security**.
#. On the **Advanced** section, click **Manage certificates**.
#. On the **Certificates** window, click the **Trusted Root Certification Authorities** tab.
#. Click **Import**.
#. On the **Certificate Import Wizard**\ , click **Next**.
#. Click **Browse**.
#. Go to the ``IEdgeInsights/build/Certificates/rootca/`` directory.
#. Select the **cacert.pem** file.
#. Select all checkboxes and then, click **Import**.

Run Grafana in the DEV Mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^

To run Grafana in the DEV mode, complete the following steps:


#. After starting the ``ia_grafana`` service, go to ``http://< host ip >:3000``.
#. Enter the default credentials details, username: "admin" and password: "admin".
#. On the **Home Dashboard** page, on the left corner, click the Dashboards icon.
#. Click the **Manage Dashboards** tab.
#. From the list of preconfigured dashboards, click **Point_Data_Dashboard**.
#. Click **Panel Title** and then, select **Edit**.
#. On the **Point_Data_Dashboard** page, if required make modifications to the query.

Execute Queries
^^^^^^^^^^^^^^^

On the ``Point_Data_Dashboard``\ , the green spikes visible in the graph are the results of the default query. To run queries, perform the following steps:


#. 
   In the **FROM** section of query, click **default_classifier_results**. A list is displayed with the name of measurements present in InfluxDB.

   ..

      **Note:** If any other measurement is set the graph will switch to the measurement query results.
      By default, the **FROM** section will have **default point_classifier_results WHERE +**.


#. 
   In the **SELECT** section, click **temperature**. A list will display the fields tags present in the schema of the measurements set in the **FROM** section.

   ..

      **Note:** By default the **SELECT** section will have **field(temperature) mean() +**.
      The graph will change according to the values you select.


Run Grafana for Video Use Cases
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Perform the following steps to run Grafana for a video use case:


#. Ensure that the endpoint of the publisher, that you want to subscribe to, is mentioned in the **Subscribers** section of the `config <https://github.com/open-edge-insights/ts-grafana/blob/master/config.json>`_ file.
#. On the **Home Dashboard** page, on the left corner, click the Dashboards icon.
#. Click the **Manage Dashboards** tab, to view the list of all the preconfigured dashboards.
#. Select **Open EII Video and Time Series Dashboard**\ , to view multiple panels with topic names of the subscriber as the panel names along with a time-series panel named ``Time Series``.
#. Hover over the topic name. The panel title will display multiple options.
#. Click **View** to view the subscribed frames for each topic.

.. note:: 


   #. The only supported browser for Grafana support for video use case is **Google Chrome**.
   #. Changing gridPos for the video frame panels is prohibited since these values are altered internally to support multi instance.
   #. Grafana does not support visualization for GVA, CustomUDF streams
   #. Grafana currently supports running a maximum number of **12 streams** only.
   #. Helm chart for Grafana video use case is not supported.

