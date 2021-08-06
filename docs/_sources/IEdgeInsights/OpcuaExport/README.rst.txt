.. role:: raw-html-m2r(raw)
   :format: html


**Contents**


* `OpcuaExport <http://localhost:7645/IEdgeInsights/OpcuaExport/#opcuaexport>`_

  * `Configuration <http://localhost:7645/IEdgeInsights/OpcuaExport/#configuration>`_
  * `Service bring up <http://localhost:7645/IEdgeInsights/OpcuaExport/#service-bring-up>`_
  * `Known issues <http://localhost:7645/IEdgeInsights/OpcuaExport/#known-issues>`_

OpcuaExport
===========

OpcuaExport service serves as as OPCUA server subscribring to classified results from EII message bus(VideoAnalytics) and starts publishing meta data to OPCUA clients

Configuration
-------------

For more details on Etcd secrets and messagebus endpoint configuration, visit `Etcd_Secrets_Configuration.md <https://github.com/open-edge-insights/eii-core/blob/master/Etcd_Secrets_Configuration.md>`_ and
`MessageBus Configuration <https://github.com/open-edge-insights/eii-core/blob/master/common/libs/ConfigMgr/README.md#interfaces>`_ respectively.

Service bring up
----------------


* 
  Please use below steps to generate opcua client certificates before running test client subscriber for production mode.


  #. 
     Append following key in `certs' in `build/provision/config/x509_cert_config.json <https://github.com/open-edge-insights/eii-core/blob/master/build/provision/config/x509_cert_config.json>`_ file.

     .. code-block::

                  {
                    "opcua": {
                      "client_alt_name": "",
                      "output_format": "DER"
                    }
                  }


     #. 
        Please go through the below sections to have RestDataExport 
        service built and launch it:


        * `../README.md#generate-deployment-and-configuration-files <https://github.com/open-edge-insights/eii-core/blob/master/README.md#generate-deployment-and-configuration-files>`_
        * `../README.md#provision <https://github.com/open-edge-insights/eii-core/blob/master/README.md#provision>`_
        * `../README.md#build-and-run-eii-videotimeseries-use-cases <https://github.com/open-edge-insights/eii-core/blob/master/README.md#build-and-run-eii-videotimeseries-use-cases>`_

     #. 
        Update opcua client certificate access so that sample test program 
        can access the certificates.

        .. code-block:: sh

              sudo chmod -R 755 ../../build/provision/Certificates/ca
              sudo chmod -R 755 ../../build/provision/Certificates/opcua

        ..

           **Caution**\ : This step will make the certs insecure. Please do not do it on a production machine.


* 
  To run a test subscriber follow README at `OpcuaExport/OpcuaBusAbstraction/c/test <https://github.com/open-edge-insights/eii-opcua-export/blob/master/OpcuaBusAbstraction/c/test>`_

Known issues
------------


* 
  OpcuaExport service has been validated with below 3rd party OPCUA client apps:


  * OPCUA CTT tool (https://opcfoundation.org/developer-tools/certification-test-tools/opc-ua-compliance-test-tool-uactt/) - Works well in both DEV and PROD modes
  * UaExpert (https://www.unified-automation.com/downloads/opc-ua-clients.html) - Works well in DEV mode only and with PROD mode, facing issues while trusting the self-signed certs
  * Integrated Objects (https://integrationobjects.com/sioth-opc/sioth-opc-unified-architecture/opc-ua-client/) - Works well in DEV mode only and with PROD mode, facing issues while trusting the self-signed certs

  However, the Prosys OPCUA client app(https://www.prosysopc.com/products/opc-ua-browser/) doesn't work both in DEV and PROD modes as it expects LDS(Local discovery server) to be implemented in the OPCUA Server to be able to discover\ :raw-html-m2r:`<br>`
  server and LDS is not part of out OpcuaExport service now.
