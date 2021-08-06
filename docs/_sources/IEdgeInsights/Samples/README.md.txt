# Sample Applications for EII Platform #

There are three sample apps for EII Platform:
1. ### cpp_sample_app ###
    Description: [README.md](cpp_sample_app/README.md)
2. ### go_sample_app ###
    Description: [README.md](go_sample_app/README.md)
3. ### python_sample_app ###
    Description: [README.md](python_sample_app/README.MD)

**EII Pre-requisites:**

1. Each sample app expects a set of config, interfaces & public private keys to be present in ETCD as a pre-requisite.
To achieve this, please ensure an entry for a publisher-subscriber pair with its relative path from [IEdgeInsights](../) directory is set in any of the .yml files present in the [build/usecases](../build/usecases) directory. An example has been provided below:

```yaml
    AppContexts:
    - Samples/cpp_sample_app/publisher
    - Samples/cpp_sample_app/subscriber
    - Samples/go_sample_app/publisher
    - Samples/go_sample_app/subscriber
    - Samples/python_sample_app/publisher
    - Samples/python_sample_app/subscriber
```

2.  With the above pre-requisite done, please run the below command:
    ```sh
    $ python3 builder.py -f ./usecases/<yml file used>
    ```


**Refer [README.md](../README.md) to provision, build and run the Sample Apps.**
