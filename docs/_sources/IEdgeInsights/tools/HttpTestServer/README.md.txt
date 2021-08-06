# `HttpTestServer`

HttpTestServer runs a simple HTTP Test server with security being optional.

* To install EII libs on bare-metal, follow the [README](../../common/README.md) of eii_libs_installer.

* To Run rest data export test application

    1. Generate the Certificate required to run the Http Test Server using the following command
    ```
    $ ./generate_testserver_cert.sh test-server-ip
    ```

    2. To run the HttpTestServer
    ```
    $ cd IEdgeInsights/tools/HttpTestServer
    $ go run TestServer.go --dev_mode false --host <address of test server> --port <port of test server> --rdehost <address of Rest Data Export server> --rdeport <port of Rest Data Export server>
    ```

    ***NOTE***: server_cert.pem is valid for 365 days from the day of generation

* In PROD mode, you might see intermediate logs like this:

    ```
    $ http: TLS handshake error from 127.0.0.1:51732: EOF
    ```
    These logs are because of RestExport trying to check if the server is present by pinging it without using any certs and can be ignored.
