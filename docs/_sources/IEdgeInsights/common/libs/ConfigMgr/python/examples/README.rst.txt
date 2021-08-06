
**Contents**


* `ConfigMgr python examples <http://localhost:7645/IEdgeInsights/common/libs/ConfigMgr/python/examples/#configmgr-python-examples>`_

  * `Pre-requisites <http://localhost:7645/IEdgeInsights/common/libs/ConfigMgr/python/examples/#pre-requisites>`_
  * `Running examples <http://localhost:7645/IEdgeInsights/common/libs/ConfigMgr/python/examples/#running-examples>`_

ConfigMgr python examples
=========================

Pre-requisites
--------------


#. 
   Please ensure you use the interfaces provided at `examples <https://github.com/open-edge-insights/eii-core/blob/master/examples>`_ for VideoIngestion, VideoAnalytics & Visualizer respectively. If not provisioned using these as the interfaces, please use EtcdUI to update these interfaces at runtime.

#. 
   Ensure you comment/uncomment the required lines for DEV_PROD mode accordingly in `env.sh <https://github.com/open-edge-insights/eii-core/blob/master/examples/env.sh>`_.

#. 
   Run this command from the build/examples directory after installing ConfigMgr WITH_EXAMPLES set to ON:

   .. code-block:: sh

           $ cd ../../examples/ && source ./env.sh && \
             cd -

Running examples
----------------


#. 
   Running the Publisher example:

   .. code-block:: sh

           $ python3 publisher.py

#. 
   Running the Subscriber example:

   .. code-block:: sh

           $ python3 subscriber.py

#. 
   Running the Server example:

   .. code-block:: sh

           $ python3 echo_service.py

#. 
   Running the Client example:

   .. code-block:: sh

           $ python3 echo_client.py

#. 
   Running the GetConfig example:

   .. code-block:: sh

           $ python3 sample_get_value.py
