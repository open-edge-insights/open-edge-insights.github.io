
Contents
========


* `Contents <#contents>`__

  * `HttpTestServer <#httptestserver>`__

    * `Prerequisites for running the HttpTestServer <#prerequisites-for-running-the-httptestserver>`__
    * `Starting HttpTestServer <#starting-httptestserver>`__

HttpTestServer
--------------

HttpTestServer runs a simple HTTP Test server with security being optional.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights (OEI). This is due to the product name change of EII as OEI.


Prerequisites for running the HttpTestServer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* 
  To install OEI libs on bare-metal, follow the `README <https://github.com/open-edge-insights/eii-core/blob/master/common/README.md>`_ of eii_libs_installer.

* 
  Generate the certificates required to run the Http Test Server using the following command:

  .. code-block:: sh

       ./generate_testserver_cert.sh test-server-ip

* 
  Update no_proxy to connect to RestDataExport server

  .. code-block:: sh

       export no_proxy=$no_proxy,<HOST_IP>

Starting HttpTestServer
^^^^^^^^^^^^^^^^^^^^^^^


* 
  Run the following command to start the HttpTestServer:

  .. code-block:: sh

       cd IEdgeInsights/tools/HttpTestServer
       go run TestServer.go --dev_mode false --host <address of test server> --port <port of test server> --rdehost <address of Rest Data Export server> --rdeport <port of Rest Data Export server>

  .. code-block:: sh

       Eg: go run TestServer.go --dev_mode false --host=0.0.0.0 --port=8082 --rdehost=localhost --rdeport=8087

    For Helm Usecases

  .. code-block:: sh

       Eg: go run TestServer.go --dev_mode false --host=0.0.0.0 --port=8082 --rdehost=<maser_node_ip>--rdeport=31509 --client_ca_path ../../build/helm-eii/eii-deploy/Certificates/rootca/cacert.pem

  ..

     **Note:** server_cert.pem is valid for 365 days from the day of generation


* 
  In PROD mode, you might see intermediate logs like this:

  .. code-block:: sh

       http: TLS handshake error from 127.0.0.1:51732: EOF

  These logs are because of RestExport trying to check if the server is present by pinging it without using any certs and can be ignored.
