
HttpTestServer
==============

HttpTestServer runs a simple HTTP Test server with security being optional.

Pre-requisites for running the HttpTestServer
---------------------------------------------


* 
  To install EII libs on bare-metal, follow the `README <https://github.com/open-edge-insights/eii-core/blob/master/common/README.md>`_ of eii_libs_installer.

* 
  Generate the certificates required to run the Http Test Server using the following command

  .. code-block::

       $ ./generate_testserver_cert.sh test-server-ip

Starting HttpTestServer
-----------------------


* 
  Please run the below command to start the HttpTestServer

  .. code-block::

       $ cd IEdgeInsights/tools/HttpTestServer
       $ go run TestServer.go --dev_mode false --host <address of test server> --port <port of test server> --rdehost <address of Rest Data Export server> --rdeport <port of Rest Data Export server>

  .. code-block::

       Eg: go run TestServer.go --dev_mode false --host=0.0.0.0 --port=8082 --rdehost=localhost --rdeport=8087

    **\ *NOTE*\ **\ : server_cert.pem is valid for 365 days from the day of generation

* 
  In PROD mode, you might see intermediate logs like this:

  .. code-block::

       $ http: TLS handshake error from 127.0.0.1:51732: EOF

    These logs are because of RestExport trying to check if the server is present by pinging it without using any certs and can be ignored.
