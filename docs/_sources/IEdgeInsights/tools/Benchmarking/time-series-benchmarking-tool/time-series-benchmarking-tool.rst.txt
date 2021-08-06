
Time Series Benchmarking Tool
=============================

These scripts are designed to automate the running of benchmarking tests and the collection of the performance data. This performance data includes the Average Stats of each data stream, and also the CPU%, Memory%, and Memory Read/Write bandwidth.

The Processor Counter Monitor (PCM) is required for measuring memory read/write bandwidth, which can be downloaded and built here: https://github.com/opcm/pcm 

If you do not have PCM on your system, those values will be blank in the output.ppc

Steps for running a benchmarking test case:


#. 
   Configure **\ `TimeSeriesProfiler README.md <https://github.com/open-edge-insights/eii-tools/blob/master/TimeSeriesProfiler/README.md>`_\ ** file to recieve rfc_results according to **\ `TimeSeriesProfiler README.md <https://github.com/open-edge-insights/eii-tools/blob/master/TimeSeriesProfiler/README.md>`_\ **.

#. 
   change the "command" option in MQTT publisher **\ `docker-compose.yml <https://github.com/open-edge-insights/eii-core/blob/master/mqtt/publisher/docker-compose.yml>`_ to:

   .. code-block:: sh

        ["--topic", "test/rfc_data", "--json", "./json_files/*.json", "--streams", "<streams>"]

   For example:

   .. code-block:: sh

        ["--topic", "test/rfc_data", "--json", "./json_files/*.json", "--streams", "1"]

#. 
   Run execute test to execute the time series test case.Before running following command, make sure that "export_to_csv" value in **\ `TimeSeriesProfiler config.json <https://github.com/open-edge-insights/eii-tools/blob/master/TimeSeriesProfiler/config.json>`_\ ** is set to "True":

   .. code-block::

       USAGE:
         ./execute_test.sh TEST_DIR STREAMS SLEEP PCM_HOME [EII_HOME]

       WHERE:
         TEST_DIR  - Directory containing services.yml and config files for influx, telegraf, and kapacitor
         STREAMS   - The number of streams (1, 2, 4, 8, 16)
         SLEEP     - The number of seconds to wait after the containers come up
             PCM_HOME  - The absolute path to the PCM repository where pcm.x is built
             [EII_HOME] - [Optional] Absolut path to EII home directory, if running from a non-default location

   For example:

   .. code-block:: sh

       sudo -E ./execute_test.sh $PWD/samples 2 10 /opt/intel/pcm /home/intel/IEdgeInsights

#. 
   The execution log, performance logs, and the output.ppc will be saved in TEST_DIR/output/< timestamp >/ so that the same test case can be ran multiple times without overwriting the output. You can see if the test ocurred any errors in the execution.log, and you can see the results of a successful test in output.ppc

#. 
   The timeseries profiler output file (named "avg_latency_Results.csv" ) will be stored in TEST_DIR/output/< timestamp >/.
