# ConfigMgr cpp examples

## Pre-requisites

1. Please ensure you use the interfaces provided in this directory for VideoIngestion, VideoAnalytics & Visualizer respectively. If not provisioned using these as the interfaces, please use EtcdUI to update these interfaces at runtime.

2. Ensure you comment/uncomment the required lines for DEV_PROD mode accordingly in [env.sh](env.sh).

3. Run this command from the build/examples directory after installing ConfigMgr WITH_EXAMPLES set to ON:

    ```sh
        $ cd ../../examples/ && source ./env.sh && \
          cd -
    ```

4. For running the examples having IPC endpoints, please ensure the required sockets directory mentioned in the Endpoints are created. Eg: Create a directory called **.socks** within **build/examples** if the Endpoint provided is:

    ```sh
        $ "EndPoint": ".socks"
    ```

## Running C API examples

1. Running the Publisher example:

    ```sh
        $ ./pub_c
    ```

2. Running the Subscriber example:

    ```sh
        $ ./sub_c
    ```

3. Running the Server example:

    ```sh
        $ ./server_c
    ```

4. Running the Client example:

    ```sh
        $ ./client_c
    ```

5. Running the KV store example:

    ```sh
        $ ./kv_store_etcd ./etcd_kv_store_config.json
    ```

## Running Cpp API examples

1. Running the Publisher example:

    ```sh
        $ ./pub
    ```

2. Running the Subscriber example:

    ```sh
        $ ./sub
    ```

3. Running the Server example:

    ```sh
        $ ./server
    ```

4. Running the Client example:

    ```sh
        $ ./client
    ```

5. Running the GetConfig & watchCallback example:

    ```sh
        $ ./sample_app
    ```