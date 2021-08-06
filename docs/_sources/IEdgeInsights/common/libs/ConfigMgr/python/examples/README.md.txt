# ConfigMgr python examples

## Pre-requisites

1. Please ensure you use the interfaces provided at [examples](../../examples) for VideoIngestion, VideoAnalytics & Visualizer respectively. If not provisioned using these as the interfaces, please use EtcdUI to update these interfaces at runtime.

2. Ensure you comment/uncomment the required lines for DEV_PROD mode accordingly in [env.sh](../../examples/env.sh).

3. Run this command from the build/examples directory after installing ConfigMgr WITH_EXAMPLES set to ON:

    ```sh
        $ cd ../../examples/ && source ./env.sh && \
          cd -
    ```

## Running examples

1. Running the Publisher example:

    ```sh
        $ python3 publisher.py
    ```

2. Running the Subscriber example:

    ```sh
        $ python3 subscriber.py
    ```

3. Running the Server example:

    ```sh
        $ python3 echo_service.py
    ```

4. Running the Client example:

    ```sh
        $ python3 echo_client.py
    ```

5. Running the GetConfig example:

    ```sh
        $ python3 sample_get_value.py
    ```