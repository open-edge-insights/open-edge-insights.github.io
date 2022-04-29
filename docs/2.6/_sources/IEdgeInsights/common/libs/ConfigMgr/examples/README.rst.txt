
**Contents**


* `ConfigMgr cpp examples <#configmgr-cpp-examples>`__

  * `Pre-requisites <#pre-requisites>`__
  * `Running C API examples <#running-c-api-examples>`__
  * `Running Cpp API examples <#running-cpp-api-examples>`__

ConfigMgr cpp examples
======================

Pre-requisites
--------------


#. 
   Please ensure you use the interfaces provided in this directory for VideoIngestion, VideoAnalytics & Visualizer respectively. If not provisioned using these as the interfaces, please use EtcdUI to update these interfaces at runtime.

#. 
   Ensure you comment/uncomment the required lines for DEV_PROD mode accordingly in `env.sh <https://github.com/open-edge-insights/eii-configmgr/blob/master/examples/env.sh>`_.

#. 
   Run this command from the build/examples directory after installing ConfigMgr WITH_EXAMPLES set to ON:

   .. code-block:: sh

           $ cd ../../examples/ && source ./env.sh && \
             cd -

#. 
   For running the examples having IPC endpoints, please ensure the required sockets directory mentioned in the Endpoints are created. Eg: Create a directory called **.socks** within **build/examples** if the Endpoint provided is:

   .. code-block:: sh

           $ "EndPoint": ".socks"

Running C API examples
----------------------


#. 
   Running the Publisher example:

   .. code-block:: sh

           $ ./pub_c

#. 
   Running the Subscriber example:

   .. code-block:: sh

           $ ./sub_c

#. 
   Running the Server example:

   .. code-block:: sh

           $ ./server_c

#. 
   Running the Client example:

   .. code-block:: sh

           $ ./client_c

#. 
   Running the KV store example:

   .. code-block:: sh

           $ ./kv_store_etcd ./etcd_kv_store_config.json

Running Cpp API examples
------------------------


#. 
   Running the Publisher example:

   .. code-block:: sh

           $ ./pub

#. 
   Running the Subscriber example:

   .. code-block:: sh

           $ ./sub

#. 
   Running the Server example:

   .. code-block:: sh

           $ ./server

#. 
   Running the Client example:

   .. code-block:: sh

           $ ./client

#. 
   Running the GetConfig & watchCallback example:

   .. code-block:: sh

           $ ./sample_app
