# EmbSubscriber

EmbSubscriber subscribes message coming from a publisher.It subscribes to messagebus topic to get the data.
    
## EII pre-requisites
1.  EmbSubscriber expects a set of config, interfaces & public private keys to be present in ETCD as a pre-requisite.
    To achieve this, please ensure an entry for EmbSubscriber with its relative path from [IEdgeInsights](../../) directory is set in the time-series.yml file present in [build/usecases](../../build/usecases) directory. An example has been provided below:
    ```sh
        AppName:
        - Grafana
        - InfluxDBConnector
        - Kapacitor
        - Telegraf
        - tools/EmbSubscriber
    ```

2. With the above pre-requisite done, please run the below command:
    ```sh
        python3 builder.py -f usecases/time-series.yml
    ```

## Running EmbSubscriber

1. Refer [provision/README.md](../../README.md) to provision, build and run the tool along with the EII time-series recipe/stack.

## Running EmbSubscriber in IPC mode
User needs to modify interface section of **[config.json](./config.json)** to run in IPC mode as following
```
{
  "config": {},
  "interfaces": {
    "Subscribers": [
      {
        "Name": "TestSub",
        "PublisherAppName": "Telegraf",
        "Type": "zmq_ipc",
        "EndPoint": {
                  "SocketDir": "/EII/sockets",
                  "SocketFile": "telegraf-out"
         },
        "Topics": [
          "*"
        ]
      }
    ]
  }
}
```
