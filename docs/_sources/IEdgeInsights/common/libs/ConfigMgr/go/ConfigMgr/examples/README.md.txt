# ConfigMgr cpp examples

## Pre-requisites

1. Ensure you run builder & provision with Go Sample Apps enabled.

2. Ensure you comment/uncomment the required lines for DEV_PROD mode accordingly in [go_env.sh](go_env.sh).

3. Run this command from the build/examples directory after installing ConfigMgr WITH_EXAMPLES set to ON:

    ```sh
        $ source ./go_env.sh
    ```

## Running examples

1. Running the Publisher example:

    ```sh
        $ go run publisher.go
    ```

2. Running the Subscriber example:

    ```sh
        $ go run subscriber.go
    ```

3. Running the Server example:

    ```sh
        $ go run echo_service.go
    ```

4. Running the Client example:

    ```sh
        $ go run echo_client.go
    ```

5. Running the GetConfig example:

    ```sh
        $ go run app_config.go
    ```