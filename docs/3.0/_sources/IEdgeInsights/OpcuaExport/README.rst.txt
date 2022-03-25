.. role:: raw-html-m2r(raw)
   :format: html


Contents
========


* `Contents <#contents>`__

  * `OpcuaExport <#opcuaexport>`__

    * `Configuration <#configuration>`__
    * `Service bring up <#service-bring-up>`__
    * `OPCUA client apps <#opcua-client-apps>`__

OpcuaExport
-----------

OpcuaExport service serves as as OPCUA server subscribring to classified results from message bus and starts publishing meta data to OPCUA clients.

..

   IMPORTANT:
   OpcuaExport service can subscribe classified results from both VideoAnalytics(video) or InfluxDBConnector(time-series) use cases. Please ensure the required service to subscribe from is mentioned in the Subscribers configuration in `config.json <https://github.com/open-edge-insights/eii-opcua-export/blob/master/config.json>`_.
   In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights (OEI). This is due to the product name change of EII as OEI.


Configuration
^^^^^^^^^^^^^

For more details on Etcd secrets and messagebus endpoint configuration, visit `Etcd_Secrets_Configuration.md <https://github.com/open-edge-insights/eii-core/blob/master/Etcd_Secrets_Configuration.md>`_ and
`MessageBus Configuration <https://github.com/open-edge-insights/eii-core/blob/master/common/libs/ConfigMgr/README.md#interfaces>`_ respectively.

Service bring up
^^^^^^^^^^^^^^^^


* 
  Please use below steps to generate opcua client certificates before running test client subscriber for production mode.


  #. 
     Please go through the below sections to have OpcuaExport
     service built and launch it:


     * `../README.md#generate-deployment-and-configuration-files <https://github.com/open-edge-insights/eii-core/blob/master/README.md#generate-deployment-and-configuration-files>`_
     * `../README.md#provision <https://github.com/open-edge-insights/eii-core/blob/master/README.md#provision>`_
     * `../README.md#build-and-run-eii-videotimeseries-use-cases <https://github.com/open-edge-insights/eii-core/blob/master/README.md#build-and-run-eii-videotimeseries-use-cases>`_

  #. 
     Update opcua client certificate access so that sample test program
     can access the certificates.

     .. code-block:: sh

             sudo chmod -R 755 ../../build/Certificates

     ..

        **Caution**\ : This step will make the certs insecure. Please do not do it on a production machine.


* 
  To run a test subscriber follow README at `OpcuaExport/OpcuaBusAbstraction/c/test <https://github.com/open-edge-insights/eii-opcua-export/blob/master/OpcuaBusAbstraction/c/test>`_

OPCUA client apps
^^^^^^^^^^^^^^^^^


* OpcuaExport service has been validated with below 3rd party OPCUA client apps:

  * OPCUA CTT tool (https://opcfoundation.org/developer-tools/certification-test-tools/opc-ua-compliance-test-tool-uactt/)
  * UaExpert (https://www.unified-automation.com/downloads/opc-ua-clients.html)
  * Integrated Objects (https://integrationobjects.com/sioth-opc/sioth-opc-unified-architecture/opc-ua-client/)
  * Prosys OPCUA client app(https://www.prosysopc.com/products/opc-ua-browser/)

.. note:: 
   To connect with OPCUA client apps, User needs to take backup `opcua_client_certificate.der <https://github.com/open-edge-insights/eii-core/blob/master/opcua_client_certificate.der>`_ and copy OPCUA client apps certificate to it.


.. code-block:: sh

       sudo chmod -R 755 ../../build/Certificates
       cp <OPCUA client apps certificate> ../build/Certificates/opcua/opcua_client_certificate.der

Make sure not to bring down ConfigMgrAgent(ia_configmgr_agent) service, however restart necessary services like ia_opcua_export to reflect the changes.


* Running in Kubernetes Environment

Install provision and deploy helm chart

.. code-block:: sh

        cd ../build/helm-eii/
        helm install eii-gen-cert eii-gen-cert/

This will generate the Certificates under ``eii-deploy/Certificates`` folder.

.. code-block:: sh

       sudo chmod -R 755 eii-deploy/Certificates

To connect with OPCUA client apps, User needs to copy OPCUA client apps certificate to `opcua_client_certificate.der <https://github.com/open-edge-insights/eii-core/blob/master/opcua_client_certificate.der>`_.

Deploy Helm Chart

.. code-block:: sh

      helm install eii-deploy eii-deploy/

Access Opcua server using "opc.tcp://\ :raw-html-m2r:`<Host IP>`\ :32003" endpoint.
