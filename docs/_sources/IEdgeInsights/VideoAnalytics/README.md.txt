# `VideoAnalytics Module`

The VideoAnalytics module is mainly responsibly for running the classifier UDFs
and doing the required inferencing on the chosen Intel(R) Hardware
(CPU, GPU, VPU, HDDL) using openVINO.

The high level logical flow of VideoAnalytics pipeline is as below:

1. App reads the application configuration via EII Configuration Manager which
   has details of `encoding` and `udfs`.
2. App gets the msgbus endpoint configuration from system environment.
3. Based on above two configurations, app subscribes to the published topic/stream
   coming from VideoIngestion module.
4. The frames received in the subscriber are passed onto one or more chained
   native/python UDFs for running inferencing and doing any post-processing as
   required. One can refer [UDFs README](../common/video/udfs/README.md) for more details
5. The frames coming out of chained udfs are published on the different topic/stream
   on EII MessageBus.

## `Configuration`

---
**NOTE**:

* The `max_workers` and `udfs` are configuration keys related to udfs.
  For more details on udf configuration, please visit
  [../common/video/udfs/README.md](../common/video/udfs/README.md)
* For details on Etcd and MessageBus endpoint configuration, visit
  [Etcd_and_MsgBus_Endpoint_Configuration](../Etcd_and_MsgBus_Endpoint_Configuration.md).
* In case the VideoAnalytics container found to be consuming a lot of memory, then one of the suspects could be that Algo processing is slower than the frame ingestion rate. Hence a lot of frames are occupying RAM waiting to be processed. In that case user can reduce the high watermark value to acceptable lower number so that RAM consumption will be under control and stay stabilzed. The exact config parameter is called **ZMQ_RECV_HWM** present in [docker-compose.yml](../build/docker-compose.yml). This config is also present in other types of container, hence user can tune them to control the memory bloat if applicable. The config snippet is pasted below:
```bash
      ZMQ_RECV_HWM: "1000"
```
---

All the app module configuration are added into distributed
key-value data store under `AppName` env as mentioned in the
environment section of this app's service definition in docker-compose.

Developer mode related overrides go into docker-compose-dev.override.yml

If `AppName` is `VideoAnalytics`, then the app's config would be fetched from
`/VideoAnalytics/config` key via EII Configuration Manager.
Below is the JSON schema for app's config:

```javascript
{
  "type": "object",
  "additionalProperties": false,
  "required": [
    "encoding",
    "udfs"
  ],
  "properties": {
    "encoding": {
      "description": "Encoding object",
      "type": "object",
      "required": [
        "type",
        "level"
      ],
      "properties": {
        "type": {
          "description": "Encoding type",
          "type": "string",
          "enum": [
              "jpeg",
              "png"
            ]
        },
        "level": {
          "description": "Encoding value",
          "type": "integer",
          "default": 0
        }
      }
    },
    "max_workers": {
      "description": "Number of threads acting on queued jobs",
      "type": "integer",
      "default": 4
    },
    "udfs": {
      "description": "Array of UDF config objects",
      "type": "array",
      "items": [
        {
          "description": "UDF config object",
          "type": "object",
          "properties": {
            "type": {
              "description": "UDF type",
              "type": "string",
              "enum": [
                "native",
                "python",
                "raw_native"
              ]
            },
            "name": {
              "description": "Unique UDF name",
              "type": "string"
            },
            "device": {
              "description": "Device on which inference occurs",
              "type": "string",
              "enum": [
                "CPU",
                "GPU",
                "HDDL",
                "MYRIAD"
              ]
            }
          },
          "additionalProperties": true,
          "required": [
            "type",
            "name",
            "device"
          ]
        }
      ]
    }
  }
}
```
**Note**:

* For `jpeg` encoding type, `level` is the quality from `0 to 100` (the higher is the better)
* For `png` encoding type, `level` is the compression level from `0 to 9`. A higher value means a smaller size and longer compression time.

One can use [JSON validator tool](https://www.jsonschemavalidator.net/) for
validating the app configuration against the above schema.
