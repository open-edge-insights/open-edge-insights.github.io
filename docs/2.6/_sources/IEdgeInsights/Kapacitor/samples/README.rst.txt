
**Contents**


* `Introduction to samples of multiple UDFs in a single task and multiple tasks with single UDF <#introduction-to-samples-of-multiple-udfs-in-a-single-task-and-multiple-tasks-with-single-udf>`__

  * `Steps to run the samples <#steps-to-run-the-samples>`__
  * `Steps to verify the results in single task with multiple UDFs <#steps-to-verify-the-results-in-single-task-with-multiple-udfs>`__
  * `Steps to verify the results in multiple tasks with single UDF <#steps-to-verify-the-results-in-multiple-tasks-with-single-udf>`__

Introduction to samples of multiple UDFs in a single task and multiple tasks with single UDF
============================================================================================

Implementation of 2 simple examples to showcase the use of a single task with multiple UDFs and multiple tasks with single UDF in Kapacitor.


* 
  Sample implemented in single task with multiple UDFs.
  MQTT Publisher is publishing 2 parameters (temperature and humidity). TICK script is calling 2 UDFs (one written in GO, another one in python), Go-based UDF is filtering the data based on the condition (i.e. temperature > 25), similarly for the python based UDF the condition is set to “humidity > 25”, filtered data is written back to “point_classifier_results” measurement in InfluxDB.

* 
  Sample implemented in multiple tasks with single UDF.
  MQTT Publisher is publishing 2 parameters (temperature and humidity). One TICK script is calling Go UDF, it is filtering the data based on the condition “temperature > 25” and the data is written back to “point_classifier_results” measurement.

  Second TICK script is calling python UDF, it is filtering the data based on the condition “humidity > 25” and the data is written back to “humidity_classifier_results” measurement.

* 
  Samples directory contain 3 directories, kapacitor_config which is common to both the samples and the tick scripts and eii config for each sample is kept in respective directory.

Steps to run the samples
------------------------


#. It is recommended to take a backup of the following original files before running the below mentioned steps as it will replace the original file.


* `config <https://github.com/open-edge-insights/ts-kapacitor/blob/master/config>`_ directory.
* `config.json <https://github.com/open-edge-insights/ts-kapacitor/blob/master/config.json>`_ file.


#. 
   Copy the sample `config <https://github.com/open-edge-insights/ts-kapacitor/blob/master/config>`_ or `config <https://github.com/open-edge-insights/ts-kapacitor/blob/master/config>`_ files based on prod or dev mode configuration of EII in the `config <https://github.com/open-edge-insights/ts-kapacitor/blob/master/config>`_ directory.

   .. code-block::

      $ cp kapacitor_config/kapacitor.conf kapacitor_config/kapacitor_devmode.conf ../config/

#. To run the multiple UDFs in a single task sample


* 
  Copy the `config.json <https://github.com/open-edge-insights/ts-kapacitor/blob/master/config.json>`_ file and replace the `config.json <https://github.com/open-edge-insights/ts-kapacitor/blob/master/config.json>`_ file.

  .. code-block::

     $ cp multi_udfs_single_task/eii_config/config.json ../config.json

* 
  Copy the `tick_scripts <https://github.com/open-edge-insights/ts-kapacitor/blob/master/tick_scripts>`_ and paste it in the `tick_scripts <https://github.com/open-edge-insights/ts-kapacitor/blob/master/tick_scripts>`_ directory.

  .. code-block::

     $ cp multi_udfs_single_task/tick_scripts/point_classifier.tick ../tick_scripts/


#. To run the multiple tasks with single UDF


* 
  Copy the `config.json <https://github.com/open-edge-insights/ts-kapacitor/blob/master/config.json>`_ file and replace the `config.json <https://github.com/open-edge-insights/ts-kapacitor/blob/master/config.json>`_ file.

  .. code-block::

     $ cp single_udf_multi_tasks/eii_config/config.json ../config.json

* 
  Copy the `tick_scripts <https://github.com/open-edge-insights/ts-kapacitor/blob/master/tick_scripts>`_ and `tick_scripts <https://github.com/open-edge-insights/ts-kapacitor/blob/master/tick_scripts>`_ and paste it in the `tick_scripts <https://github.com/open-edge-insights/ts-kapacitor/blob/master/tick_scripts>`_ directory.

  .. code-block::

     $ cp single_udf_multi_tasks/tick_scripts/humidity_classifier.tick ../tick_scripts/

     $ cp single_udf_multi_tasks/tick_scripts/temperature_classifier.tick ../tick_scripts/


#. 
   Please go through the below sections to bring up EII stack:


   * `../README.md#generate-deployment-and-configuration-files <https://github.com/open-edge-insights/eii-core/blob/master/README.md#generate-deployment-and-configuration-files>`_
   * `../README.md#provision <https://github.com/open-edge-insights/eii-core/blob/master/README.md#provision>`_
   * `../README.md#build-and-run-eii-videotimeseries-use-cases <https://github.com/open-edge-insights/eii-core/blob/master/README.md#build-and-run-eii-videotimeseries-use-cases>`_

#. 
   To start the mqtt-publisher with temperature and humidity data, please refer `tools/mqtt-publisher/README.md <https://github.com/open-edge-insights/eii-tools/blob/master/mqtt-publisher/README.md>`_

Steps to verify the results in single task with multiple UDFs
-------------------------------------------------------------


#. 
   InfluxDBConnector logs output

   Command

   .. code-block::

      $ docker logs -f ia_influxdbconnector

   Output

   .. code-block::

      I0717 00:53:46.636838      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 humidity=28.57107003902059,ts=1594927426.634632,temperature=26.655493463976832 1594927426635557092 idbconn_pub:true]
      I0717 00:53:46.639011      10 pubManager.go:121] Published message: map[data:point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 humidity=28.57107003902059,temperature=26.655493463976832,ts=1594927426.634632 1594927426635557092 idbconn_pub:true]
      I0717 00:53:47.638366      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 temperature=22.619720484770554,humidity=16.731963502272897,ts=1594927427.6358457 1594927427636952571 idbconn_pub:true]
      I0717 00:53:48.642098      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 ts=1594927428.6372812,temperature=17.571037421047784,humidity=23.928420142806388 1594927428640200053 idbconn_pub:true]
      I0717 00:53:49.640132      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 ts=1594927429.6386497,temperature=13.926727845681501,humidity=14.61523554472623 1594927429639197778 idbconn_pub:true]
      I0717 00:53:50.641966      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 ts=1594927430.6400163,temperature=29.535498572099197,humidity=17.40839676245851 1594927430640922707 idbconn_pub:true]
      I0717 00:53:51.644246      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 ts=1594927431.6406047,temperature=22.014647866484523,humidity=23.24245661794304 1594927431641579125 idbconn_pub:true]
      I0717 00:53:52.643982      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 ts=1594927432.641993,temperature=16.77923134191675,humidity=18.01282784469746 1594927432642890643 idbconn_pub:true]
      I0717 00:53:53.644850      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 ts=1594927433.642745,temperature=11.96364985814431,humidity=24.91437844150262 1594927433643691025 idbconn_pub:true]
      I0717 00:53:54.646171      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 temperature=27.031131147255834,humidity=29.017961635659024,ts=1594927434.6439903 1594927434644892174 idbconn_pub:true]
      I0717 00:53:54.648654      10 pubManager.go:121] Published message: map[data:point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 humidity=29.017961635659024,temperature=27.031131147255834,ts=1594927434.6439903 1594927434644892174 idbconn_pub:true]

#. 
   Check data in the InfluxDB


* 
  Enter in the container using the following command

  .. code-block::

     $ docker exec -it ia_influxdbconnector bash

* 
  Use the following to enter into the database

  prod mode

  .. code-block::

     $ influx -ssl -unsafeSsl -username "username" -password "password" -database "database_name"

  dev mode

  .. code-block::

     $ influx -username "username" -password "password" -database "database_name"

  replace the "username", "database", and "password" keys with your DB username, password and DB name.`

* 
  Use Influx CLI command to interact with DB.

  .. code-block::

     $ show measurements

  Output of the command

  .. code-block::

     > show measurements
     name: measurements
     name
     ----
     point_classifier_results
     point_data

* 
  To see the data in point_classifier_results measurement use the following command

  .. code-block::

     $ select * from point_classifier_results

  Sample output of the command

  .. code-block::

     name: point_classifier_results
     time                host        humidity           temperature        topic                   ts
     ----                ----        --------           -----------        -----                   --
     1594927630832507623 ia_telegraf 29.855404019751816 25.53367159780972  temperature/simulated/0 1594927630.8313158
     1594927641839961485 ia_telegraf 29.913849074313813 25.18000949751805  temperature/simulated/0 1594927641.8386183
     1594927644843838821 ia_telegraf 26.631081501636743 29.99623149385496  temperature/simulated/0 1594927644.842906
     1594927678873799703 ia_telegraf 25.362676867888464 25.420215215181166 temperature/simulated/0 1594927678.872921
     1594927703898638431 ia_telegraf 27.55019173839383  26.603218492677055 temperature/simulated/0 1594927703.8980274
     1594927705900115913 ia_telegraf 26.309594644310167 25.68362497454113  temperature/simulated/0 1594927705.8995953
     1594927709904380759 ia_telegraf 27.52768187637576  29.3945157174943   temperature/simulated/0 1594927709.9034615
     1594927744936530293 ia_telegraf 26.844547285321887 29.498420224006598 temperature/simulated/0 1594927744.9355264
     1594927753945020633 ia_telegraf 25.71321771687117  25.07137538609002  temperature/simulated/0 1594927753.9441714

  Here, both the temperature and himidity data is above 25

Steps to verify the results in multiple tasks with single UDF
-------------------------------------------------------------


#. 
   InfluxDBConnector logs output

   Command

   .. code-block::

      $ docker logs -f ia_influxdbconnector

   Output

   .. code-block::

      I0716 23:58:01.428896      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 humidity=23.50196006313319,ts=1594924081.4278295,temperature=25.758773258956786 1594924081428303867 idbconn_pub:true]
      I0716 23:58:01.429801      10 pubManager.go:121] Published message: map[data:point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 humidity=23.50196006313319,temperature=25.758773258956786,ts=1594924081.4278295 1594924081428303867 idbconn_pub:true]
      I0716 23:58:02.429889      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 humidity=12.785272290966788,ts=1594924082.4282343,temperature=25.88015317051446 1594924082428980179 idbconn_pub:true]
      I0716 23:58:02.431062      10 pubManager.go:121] Published message: map[data:point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 humidity=12.785272290966788,temperature=25.88015317051446,ts=1594924082.4282343 1594924082428980179 idbconn_pub:true]
      I0716 23:58:03.430484      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 humidity=27.393076181834594,ts=1594924083.4287212,temperature=18.16039229846337 1594924083429552256 idbconn_pub:true]
      I0716 23:58:03.433508      10 pubManager.go:121] Published message: map[data:humidity_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 humidity=27.393076181834594,temperature=18.16039229846337,ts=1594924083.4287212 1594924083429552256 idbconn_pub:true]
      I0716 23:58:04.431786      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 temperature=27.32044250166733,humidity=12.040910740537647,ts=1594924084.430019 1594924084430766594 idbconn_pub:true]
      I0716 23:58:04.433122      10 pubManager.go:121] Published message: map[data:point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 humidity=12.040910740537647,temperature=27.32044250166733,ts=1594924084.430019 1594924084430766594 idbconn_pub:true]
      I0716 23:58:05.431732      10 pubManager.go:121] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 ts=1594924085.4304821,temperature=24.03411329966985,humidity=17.1642901921084 1594924085431039073 idbconn_pub:true]

#. 
   Check data in the InfluxDB


* 
  Enter in the container using the following command

  .. code-block::

     $ docker exec -it ia_influxdbconnector bash

* 
  Use the following to enter into the database

  prod mode

  .. code-block::

     $ influx -ssl -unsafeSsl -username "username" -password "password" -database "database_name"

  dev mode

  .. code-block::

     $ influx -username "username" -password "password" -database "database_name"

  replace the "username", "database", and "password" keys with your DB username, password and DB name.`

* 
  Use Influx CLI command to interact with DB.

  .. code-block::

     $ show measurements

  Output of the command

  .. code-block::

     > show measurements
     name: measurements
     name
     ----
     humidity_classifier_results
     point_classifier_results
     point_data

* 
  To see the data in point_classifier_results measurement use the following command

  .. code-block::

     $ select * from point_classifier_results

  Sample output of the command

  .. code-block::

     name: point_classifier_results
     time                host        humidity           temperature        topic                   ts
     ----                ----        --------           -----------        -----                   --
     1594923545769758418 ia_telegraf 19.81604257073865  26.879897943430848 temperature/simulated/0 1594923545.7691243
     1594923614838551423 ia_telegraf 28.95738078521109  29.250270883294515 temperature/simulated/0 1594923614.8377328
     1594923616840732537 ia_telegraf 21.3038992065728   27.128756451611928 temperature/simulated/0 1594923616.839861
     1594923621850408245 ia_telegraf 24.17081234022867  28.12092358748019  temperature/simulated/0 1594923621.8499172
     1594923622851693103 ia_telegraf 16.991377116304164 25.942235861646243 temperature/simulated/0 1594923622.8506734
     1594923625853194299 ia_telegraf 17.790742401040156 26.029039405898416 temperature/simulated/0 1594923625.8526921
     1594923627854797339 ia_telegraf 18.636291434358817 29.326483056022322 temperature/simulated/0 1594923627.8538675
     1594923629861235060 ia_telegraf 13.110781672981076 26.452111132568664 temperature/simulated/0 1594923629.8601944
     1594923636868340122 ia_telegraf 27.663236146599328 26.29884980515991  temperature/simulated/0 1594923636.867832
     1594923647880734785 ia_telegraf 25.3924565876123   28.078243101597526 temperature/simulated/0 1594923647.8796468
     1594923648881486348 ia_telegraf 12.022217148202829 27.96651437168266  temperature/simulated/0 1594923648.8805144
     1594923654886479873 ia_telegraf 17.333022980266087 28.368384241660806 temperature/simulated/0 1594923654.8855999
     1594923655888191773 ia_telegraf 29.70650230680635  26.11839631300789  temperature/simulated/0 1594923655.8870778
     1594923656888741310 ia_telegraf 17.591632563610442 27.549975342107892 temperature/simulated/0 1594923656.8878617

  Here, all the temperature data is above 25

* 
  To see the data in humidity_classifier_results measurement use the following command

  .. code-block::

     $ select * from humidity_classifier_results

  Sample output of the command

  .. code-block::

     name: humidity_classifier_results
     time                host        humidity           temperature        topic                   ts
     ----                ----        --------           -----------        -----                   --
     1594922435670934407 ia_telegraf 29.14477182235822  28.24835641087284  temperature/simulated/0 1594922435.6704926
     1594922438672749397 ia_telegraf 28.333961058034408 23.79412059324136  temperature/simulated/0 1594922438.6722746
     1594922440673573262 ia_telegraf 28.403412014434576 15.849326161213199 temperature/simulated/0 1594922440.6730607
     1594922441674124435 ia_telegraf 27.629699521677637 13.964340936065131 temperature/simulated/0 1594922441.6733868
     1594922451683956862 ia_telegraf 26.31819680956764  11.757978239277323 temperature/simulated/0 1594922451.6834583
     1594922452684404597 ia_telegraf 26.942886935205237 27.954762973394395 temperature/simulated/0 1594922452.6838408
     1594922454685507436 ia_telegraf 28.198384986303513 21.958077952286747 temperature/simulated/0 1594922454.6847117
     1594922459691136679 ia_telegraf 28.928009573873968 27.33375706494249  temperature/simulated/0 1594922459.6906385
     1594922461692967897 ia_telegraf 27.300210084769425 21.84679722208667  temperature/simulated/0 1594922461.6924229
     1594922464695220305 ia_telegraf 26.031621145742253 19.33596257126575  temperature/simulated/0 1594922464.694672
     1594922465696773126 ia_telegraf 25.312548153267265 25.93090843830029  temperature/simulated/0 1594922465.695979
     1594922466697439030 ia_telegraf 29.384297962446905 27.409752918067007 temperature/simulated/0 1594922466.6964452
     1594922472701537004 ia_telegraf 27.621241070234706 26.68898759908584  temperature/simulated/0 1594922472.7007167
     1594922473703266299 ia_telegraf 28.558485895926765 21.455033422854758 temperature/simulated/0 1594922473.7021594
     1594922475704646142 ia_telegraf 27.423792582269076 12.596418464102804 temperature/simulated/0 1594922475.7040672
     1594922479707734013 ia_telegraf 27.770484735726605 19.20584244268835  temperature/simulated/0 1594922479.706727

  Here, all the humidity data is above 25
