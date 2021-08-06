# EII TimeSeriesProfiler

1. This module calculates the SPS(samples per second) of any EII time-series modules based on the stream published by that respective module.
2. This module calculates the average e2e time for every sample data to process and it's breakup. The e2e time end to end time required
   for a metric from mqtt-publisher to TimeSeriesProfiler (mqtt-publisher->telegraf->influx->kapacitor->influx->influxdbconnector->
   TimeSeriesProfiler)


## EII pre-requisites

1. TimeSeriesProfiler expects a set of config, interfaces & public private keys to be present in ETCD as a pre-requisite.
    To achieve this, please ensure an entry for TimeSeriesProfiler with its relative path from [IEdgeInsights](../../) directory is set in the time-series.yml file present in [build/usecases](../../build/usecases) directory. An example has been provided below:
    ```sh
        AppContexts:
        - Grafana
        - InfluxDBConnector
        - Kapacitor
        - Telegraf
        - tools/TimeSeriesProfiler
    ```

2. With the above pre-requisite done, please run the below command:
    ```sh
        python3 builder.py -f ./usecases/time-series.yml
    ```



## EII TimeSeriesProfiler modes

    By default the EII TimeSeriesProfiler supports two modes, which are 'sps' & 'monitor' mode.

1. SPS mode

    Enabled by setting the 'mode' key in [config](./config.json) to 'sps', this mode calculates the samples
    per second of any EII module by subscribing to that module's respective stream.
    ```sh
        "mode": "sps"
    ```

2. Monitor mode

    Enabled by setting the 'mode' key in [config](./config.json) to 'monitor', this mode calculates average & per sample stats

    Refer the below exmaple config where TimeSeriesProfiler is used in monitor mode.

    ```javascript
        "config": {
        "mode": "monitor",
        "monitor_mode_settings": {
                                    "display_metadata": false,
                                    "per_sample_stats":false,
                                    "avg_stats": true
                                },
        "total_number_of_samples" : 5,
        "export_to_csv" : false
    }
    ```

    ```sh
        "mode": "monitor"
    ```

    The stats to be displayed by the tool in monitor_mode can be set in the monitor_mode_settings key of [config.json](config.json).
    1. 'display_metadata': Displays the raw meta-data with timestamps associated with every sample.
    2. 'per_sample_stats': Continously displays the per sample metrics of every sample.
    3. 'avg_stats': Continously displays the average metrics of every sample.

  > **Note:**
  > * Pre-requisite for running in profiling or monitor mode: Time series containers should be running with PROFILING_MODE set to **true** in [.env](../../build/.env)
  > * For running TimeSeriesProfiler in SPS mode, it is recommended to keep PROFILING_MODE set to false in [.env](../../build/.env) for better performance.

## EII TimeSeriesProfiler configurations

1. total_number_of_samples

    If mode is set to 'sps', the average SPS is calculated for the number of samples set by this variable.
    If mode is set to 'monitor', the average stats is calculated for the number of samples set by this variable.
    Setting it to (-1) will run the profiler forever unless terminated by stopping container TimeSeriesProfiler manually.
    total_number_of_samples should never be set as (-1) for 'sps' mode.

2. export_to_csv

    Setting this switch to **true** exports csv files for the results obtained in TimeSeriesProfiler. For monitor_mode, runtime stats printed in the csv
    are based on the the following precdence: avg_stats, per_sample_stats, display_metadata.


## Running TimeSeriesProfiler

1. Pre-requisite :

   Profiling UDF returns "ts_kapacitor_udf_entry" and "ts_kapacitor_udf_exit" timestamp.  
   
    These 2 as examples to refer:
    1. [profiling_udf.go](../../Kapacitor/udfs/profiling_udf.go)
    2. [rfc_classifier.py](../../Kapacitor/udfs/rfc_classifier.py)
   
* **Additional:** Adding timestamps in ingestion and UDFs:

  In case user wants to enable his/her own ingestion and UDFs, timestamps need to be added to ingestion and UDFs modules respectively.
  The TS Profiler needs three timestamps.

  1. "ts" timestamp which is to be filled by the ingestor (done by the mqtt-publisher app).

  2. The udf to give "ts_kapacitor_udf_entry" and "ts_kapacitor_udf_exit" timestamps to profile the udf execution time.

     ts_kapacitor_udf_entry : timestamp in UDF before execution of the of the algorithm

     ts_kapacitor_udf_exit : timestamp in UDF after execution of the algorithm.

    The sample profiling UDFs can be referred at [profiling_udf.go](../../Kapacitor/udfs/profiling_udf.go) and [rfc_classifier.py](../../Kapacitor/udfs/rfc_classifier.py).

* configuration required to run profiling_udf.go as profiling udf

   In **[Kapacitor config.json](../../Kapacitor/config.json)** , update "task" key as below:
   ```
   "task": [{
       "tick_script": "profiling_udf.tick",
       "task_name": "profiling_udf",
       "udfs": [{
          "type": "go",
          "name": "profiling_udf"
       }]
   }]
   ```
   In **[kapacitor.conf](../../Kapacitor/config/kapacitor.conf)** under udf section:

   ```
      [udf.functions]
         [udf.functions.profiling_udf]
           socket = "/tmp/profiling_udf"
           timeout = "20s"

   ```
 * configuration required to run rfc_classifier.py as profiler udf
   In **[Kapacitor config.json](../../Kapacitor/config.json)** , update "task" key as below:
   ```
   "task": [{
       {
        "tick_script": "rfc_task.tick",
        "task_name": "random_forest_sample"
        }
   }]
   ```
   In **[kapacitor.conf](../../Kapacitor/config/kapacitor.conf)** under udf section:

   ```
    [udf.functions.rfc]
      prog = "python3.7"
      args = ["-u", "/EII/udfs/rfc_classifier.py"]
      timeout = "60s"
      [udf.functions.rfc.env]
         PYTHONPATH = "/EII/go/src/github.com/influxdata/kapacitor/udf/agent/py/"
   ```
  keep **[config.json](./config.json)** file as following:
```
  {
    "config": {
        "total_number_of_samples": 10,
        "export_to_csv": "False"
    },
    "interfaces": {
        "Subscribers": [
            {
                "Name": "default",
                "Type": "zmq_tcp",
                "EndPoint": "ia_influxdbconnector:65032",
                "PublisherAppName": "InfluxDBConnector",
                "Topics": [
                    "rfc_results"
                ]
            }
        ]
    }
}
```



   In [.env](../../build/.env):

   Set the profiling mode as true.

2. Set environment variables accordingly in [config.json](config.json)

3. Set the required output stream/streams and appropriate stream config in [config.json](config.json) file.

4. To run this tool in IPC mode, User needs to modify subscribers interface section of **[config.json](./config.json)** as following

```
    {
        "type": "zmq_ipc",
        "EndPoint": "/EII/sockets"
    }
```

5. Refer [README.md](../../README.md) to provision, build and run the tool along with the EII time-series recipe/stack.

6. Run the following command to see the logs:

    ```sh
        docker logs -f ia_timeseries_profiler
    ```

