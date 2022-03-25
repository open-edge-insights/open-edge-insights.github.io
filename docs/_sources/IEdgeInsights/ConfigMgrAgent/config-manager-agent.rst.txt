
Config Manager Agent
====================

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights (OEI). This is due to the product name change of EII as OEI.


Config Manager Agent is an OEI service responsible for the following:


* Puts the OEI services configs to the OEI config manager data store
* Additionally in PROD mode, generates the following:

  * Required config manager data store keys/certificates to interact with OEI config manager data store like etcd and puts in the volume mounts to be shared with
    other OEI services
  * Required messagebus keys for OEI services communication

* Creates required provisioning folders with the right permissions needed for other OEI services via volume mounts

The diagram below shows a high level flow of ``ConfigMgrAgent`` service .

.. code-block:: mermaid

   %% name: EII Provisioning

   sequenceDiagram
       participant User
       participant EdgeNode
       participant EIIService
       Participant ConfigMgrAgent
       participant ETCD

       User->>EdgeNode: Start container (docker-compose up)
       EIIService->>EIIService: Wait for ETCD certificates
       ConfigMgrAgent->>ConfigMgrAgent: Generate x509 certificates for ETCD
       ConfigMgrAgent->>ETCD: Start ETCD
       ConfigMgrAgent->>ETCD: Register services to be able to connect
       ConfigMgrAgent->>ETCD: Load default configuration
       ConfigMgrAgent->>ETCD: Generate and load ZeroMQ Keys
       ConfigMgrAgent->>EIIService: Copy ETCD certificates to the shared volume
       Note right of ConfigMgrAgent: Each service has its own certs volume
       EIIService->>ETCD: Connect to ETCD with certificate to get configuration

.. note:: 
   Any EII service ``waits/restarts`` if the config manager data store client key and certificates are yet to be made available for the container.
   EII Certificates will be re-generated everytime the ConfigMgrAgent service brought up/restarted


 **Optional:** For capturing the data back from Etcd to a JSON file, run the etcd_capture.sh script. This can be achieved using the following command:

.. code-block:: sh

    docker exec -it ia_configmgr_agent ./scripts/etcd_capture.sh
