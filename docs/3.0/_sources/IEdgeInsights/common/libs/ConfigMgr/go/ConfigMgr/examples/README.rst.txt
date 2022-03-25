
**Contents**


* `ConfigMgr cpp examples <#configmgr-cpp-examples>`__

  * `Pre-requisites <#pre-requisites>`__
  * `Running examples <#running-examples>`__

ConfigMgr cpp examples
======================

Pre-requisites
--------------


#. 
   Ensure you run builder & provision with Go Sample Apps enabled.

#. 
   Ensure you comment/uncomment the required lines for DEV_PROD mode accordingly in `go_env.sh <https://github.com/open-edge-insights/eii-configmgr/blob/master/go/ConfigMgr/examples/go_env.sh>`_.

#. 
   Run this command from the build/examples directory after installing ConfigMgr WITH_EXAMPLES set to ON:

   .. code-block:: sh

           source ./go_env.sh

Running examples
----------------


#. 
   Running the Publisher example:

   .. code-block:: sh

           go run publisher.go

#. 
   Running the Subscriber example:

   .. code-block:: sh

           go run subscriber.go

#. 
   Running the Server example:

   .. code-block:: sh

           go run echo_service.go

#. 
   Running the Client example:

   .. code-block:: sh

           go run echo_client.go

#. 
   Running the GetConfig example:

   .. code-block:: sh

           go run app_config.go
