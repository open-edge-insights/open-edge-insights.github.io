
Grafana
=======


#. Grafana is an open source metric analytics & visualization suite. It is most commonly used for visualizing time series data for infrastructure and        application analytics but many use it in other domains including industrial sensors, home automation, weather, and process control.
#. Grafana supports many different storage backends for our time series data (Data Source). Here in EII we are using InfluxDB as datasource.
#. Grafana connects to the InfluxDB datasource which has been preconfigured as a part of grafana setup. The data source i.e. ia_influxdbconnector service must be running in order for grafana to collect time series data.
#. Once the data source is working, we can visualize the incoming data using the preconfigured dashboard. We can aslo edit the dashboard as per our convenience.

Configuration
-------------


#. 
   `dashboard.json <https://github.com/open-edge-insights/ts-grafana/blob/master/dashboard.json>`_
    This is the dashboard json file that is loaded when grafana starts. It has been preconfigured to display time-series data. 

#. 
   `dashboard_sample.yml <https://github.com/open-edge-insights/ts-grafana/blob/master/dashboard_sample.yml>`_
    This is the config file for all the dashboards. It specifies the path where all the dashboard json files will be looked for.

#. 
   `datasource_sample.yml <https://github.com/open-edge-insights/ts-grafana/blob/master/datasource_sample.yml>`_
    This is the config file for setting up the datasource. It has got various fields for datasource configuration.

#. 
   `grafana_template.ini <https://github.com/open-edge-insights/ts-grafana/blob/master/grafana_template.ini>`_
    This is the config for Grafana itself. It specifies how grafana should start up, once it has been configured.

NOTE: The contents of these files can be edited according to the requirement.

Procedure to run Grafana
------------------------


#. Open `docker-compose.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/docker-compose.yml>`_ and uncomment ia_grafana.
#. Check ia_influxdbconnector, ia_kapacitor, ia_telegraph are running for time-series data.
#. Check `publisher <https://github.com/open-edge-insights/eii-tools/blob/master/mqtt-publisher/publisher_temp.sh>`_ is running.
#. Use "docker-compose build" to build image.
#. 
   Use "docker-compose up" to run the service.


   * 
     **Prod mode [To be followed when running in PROD mode, skip if DEV mode]**

     Import 'ca_certificate.pem' from 'build/provision/Certificates/ca' Directory to your Browser Certifcates.

     Steps to Import Certificates


     * Go to *Settings* in Chrome
     * Search *Manage Certificates* Under Privacy & Security
     * Select Manage Certificates Option
     * Under *Authorities* Tab Click Import Button
     * With Import Wizard navigate to
       *IEdgeInsights/build/provision/Certificates/ca* Dir
     * Select *ca_certificate.pem* file
     * Select All CheckBoxes and Click Import Button.

   * **Dev mode [To be followed when running in DEV mode]**

#. 
   Once ia_grafana service is up, go to http://< host ip >:3000 .

#. Provide default username: "admin" and password: "admin".
#. On "Home Dashboard" page, click on Dashboards icon from the left corner.
#. Click on Manage Dashboards tab, it will list out all the preconfigured dashboards.
#. Click on "Point_Data_Dashboard" which is a preconfigured dashboard.
#. Click on "Panel Title" and select the "edit" option.
#. Now you will be on Point_Data_Dashboard page and here you can make modifications to the query.

Steps to execute query
----------------------


#. 
   Once landed on the ``Point_Data_Dashboard``\ , green spikes visible in the
   graph are the results of the default query

#. 
   In the "FROM" section of query, by default it will have 'default
   point_classifier_results WHERE +',
   click on 'default_classifier_results', drop down will open with the name of Measurements present in InfluxDB.
   If any other measurement is set the graph will switch to the measurement query results.

#. 
   In the "SELECT" section, by default it will have 'field(temperature) mean() +'. Click on 'temperature', a drop down will
   open with the fields tags present in the schema of the measurements set in the FROM section.
   Select any options the graph will react accordingly.
