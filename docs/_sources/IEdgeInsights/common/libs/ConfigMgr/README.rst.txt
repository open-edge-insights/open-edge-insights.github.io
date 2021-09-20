
ConfigMgr
=========

ConfigMgr (Config manager) provides CPP/Python/Golang APIs to:


* fetch an applications configs values from the KV store.
* fetch an applications interface values from KV store for pub, sub, server and client.
* watch on application's config changes.
* generate EII MessageBus config.
* read and set /GlobalEnv variables.
* fetch env variables: appname, dev_mode

All these data are stored into the kv store of EII during the provisioning phase and can be changed   dynamically by the admin.

Installation
------------

The EIIUtils depends on CMake version 3.11+. For Ubuntu 18.04 this is not
the default version installed via ``apt-get``. To install the correct version
of CMake and other ConfigMgr dependencies, please follow `eii_libs_installer README <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_

CMAKE_INSTALL_PREFIX needs to be set for the installation:

.. code-block:: sh

       $ export CMAKE_INSTALL_PREFIX="/opt/intel/eii"

ConfigMgr installs to ``/opt/intel/eii/lib/``. On some platforms this is not included in the ``LD_LIBRARY_PATH`` by default. As a result, you must add this directory to your ``LD_LIBRARY_PATH``\ ,
otherwise you will encounter issues using the ConfigMgr. This can
be accomplished with the following ``export``\ :

.. code-block:: sh

   $ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/eii/lib/

.. note::  You can also specify a different library prefix to CMake through
   the ``CMAKE_INSTALL_PREFIX`` flag. If different installation path is given via ``CMAKE_INSTALL_PREFIX``\ , then ``$LD_LIBRARY_PATH`` should be appended by $CMAKE_INSTALL_PREFIX/lib.


Install ConfigMgr with Python bindings, Go bindings, Examples, Test suits and Debug Build.
------------------------------------------------------------------------------------------

.. code-block:: sh

   $ rm -rf build
   $ mkdir build
   $ cd build
   $ cmake -DCMAKE_INSTALL_INCLUDEDIR=$CMAKE_INSTALL_PREFIX/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX -DWITH_PYTHON=ON -DWITH_GO=ON -DWITH_EXAMPLES=ON -DWITH_TESTS=ON -DCMAKE_BUILD_TYPE=Debug ..
   $ make
   $ sudo make install

``WITH_PYTHON=ON``\ , ``WITH_GO=ON``\ , ``WITH_EXAMPLES=ON``\ , ``WITH_TESTS=ON`` and ``CMAKE_BUILD_TYPE=Debug`` to compile ConfigMgr with Python bindings, Go bindings, Examples, Unit Tests and Debug mode respectively.

Interfaces
==========

ConfigMgr parses the data from the kv store (eg: etcd) and the application environment variables for its functionality.
It supports Publisher, Subscriber, Server and Client interfaces. Below are the examples for providing 
different interfaces and different usecases.

Please refer `different ways of giving endpoints <https://open-edge-insights.github.io/IEdgeInsights/common/libs/ConfigMgr/###**Note**-"endpoint"-can-be-given-in-different-ways:>`_

**Publisher Interface**
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   {
       "Publishers": [
           {
               "Name": "default",
               "Type": "zmq_ipc",
               "EndPoint": "/EII/sockets",
               "Topics": [
                   "camera1_stream"
               ],
               "AllowedClients": [
                   "*"
               ]
           },
           {
               "Name": "example",
               "Type": "zmq_tcp",
               "EndPoint": "127.0.0.1:65015",
               "Topics": [
                   "*"
               ],
               "AllowedClients": [
                   "Visualizer", "VideoAnalytics"
               ]
           }
       ]
   }

.. list-table::
   :header-rows: 1

   * - Key
     - Type
     - Required (Mandatory)
     - Description
   * - ``Publishers``
     - ``array``
     - Yes
     - Entire publisher interface will be added with in the array. Multiple publish endpoints can be added by adding elements in the array.
   * - ``Name``
     - ``string``
     - Yes
     - Name of different publisher interfaces
   * - ``Type``
     - ``string``
     - Yes
     - Specifies ZeroMQ protocol ("zmq_tcp" or "zmq_ipc") on which data will be published
   * - ``EndPoint``
     - ``string`` or ``object``
     - Yes
     - In case of TCP or IPC (socket directory only), endpoint should be string as shown in the above examples. In case IPC explicitly specifying socket file, either object or string can be used for EndPoint. Please refer `Different ways of specifying endpoint <https://open-edge-insights.github.io/IEdgeInsights/common/libs/ConfigMgr/###**Note**-"endpoint"-can-be-given-in-different-ways:>`_
   * - ``Topics``
     - ``array``
     - Yes
     - Specifying the topics on which data will be published on. Multiple elements in this array can denote multiple topics published on the same endpoint
   * - ``AllowedClients``
     - ``array``
     - Yes
     - Specifying who can subscribe to the the topic on which data is published. If AllowedClients is "*", then all the provisioned services can receive the data published.


**Subscriber Interface**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   {
       "Subscribers": [

           // tcp usecase
           {
               "Name": "example",
               "Type": "zmq_tcp",
               "EndPoint": "127.0.0.1:65013",
               "PublisherAppName": "VideoIngestion",
               "Topics": [
                   "*"
               ]
           },

           // ipc usecase
           {
               "Name": "default",
               "Type": "zmq_ipc",
               "EndPoint": ".socks",
               "PublisherAppName": "VideoIngestion",
               // Topics cannot be "*", if the only IPC directory is given
               // if it Topics "*" to be used in ipc, then socket file should be given explicitly.
               "Topics": [
                   "camera1_stream"
               ]
           }
       ]
   }

.. list-table::
   :header-rows: 1

   * - Key
     - Type
     - Required (Mandatory)
     - Description
   * - ``Subscribers``
     - ``array``
     - Yes
     - Entire subscriber interface will be added with in the array. Multiple subscribe endpoints can be added by adding elements in the array
   * - ``Name``
     - ``string``
     - Yes
     - Name of different subscriber interfaces
   * - ``Type``
     - ``string``
     - Yes
     - Specifies ZeroMQ protocol ("zmq_tcp" or "zmq_ipc") through which subscription happens
   * - ``EndPoint``
     - ``string`` or ``object``
     - Yes
     - In case of TCP or IPC (socket directory only), endpoint should be string as shown in the above examples. In Case IPC explicitly specifying socket file, either object or string can be used for EndPoint. Please refer `Different ways of specifying endpoint <https://open-edge-insights.github.io/IEdgeInsights/common/libs/ConfigMgr/###**Note**-"endpoint"-can-be-given-in-different-ways:>`_
   * - ``PublisherAppName``
     - ``string``
     - Yes
     - Specifies the publisher's AppName through which data will be received
   * - ``Topics``
     - ``array``
     - Yes
     - Specifying the topics on which data will be published on. If Topics is "*", the subscriber receives all the data published on the endpoint, irrespective of the topic names data is published on. Multiple elements in this array can denote multiple topics subscribed on the same endpoint


**Server Interface**
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   {
       "Servers": [
           // tcp usecase
           {
               "Name": "default",
               "Type": "zmq_tcp",
               "EndPoint": "127.0.0.1:9006",
               "AllowedClients": [
                   "VideoAnalytics"
               ]
           },
           //ipc usecase
           {
               "Name": "example",
               "Type": "zmq_ipc",
               "EndPoint": "/EII/sockets",
               "AllowedClients": [
                   "VideoAnalytics"
               ]
           }
       ]
   }

.. list-table::
   :header-rows: 1

   * - Key
     - Type
     - Required (Mandatory)
     - Description
   * - ``Servers``
     - ``array``
     - Yes
     - Entire server interface will be added with in the array. Multiple server endpoints can be added by adding elements in the array
   * - ``Name``
     - ``string``
     - Yes
     - Name of different server interfaces
   * - ``Type``
     - ``string``
     - Yes
     - Specifies ZeroMQ protocol ("zmq_tcp" or "zmq_ipc") through which Server pushes data
   * - ``EndPoint``
     - ``string`` or ``object``
     - Yes
     - In case of TCP or IPC (socket directory only), endpoint should be string as shown in the above examples. In case IPC explicitly specifying socket file, either object or string can be used for EndPoint. Please refer `Different ways of specifying endpoint <https://open-edge-insights.github.io/IEdgeInsights/common/libs/ConfigMgr/###**Note**-"endpoint"-can-be-given-in-different-ways:>`_
   * - ``AllowedClients``
     - ``array``
     - Yes
     - Specifying who can get data is from the Server. If AllowedClients is "*", then all the provisioned services can connect to Server.


**Client Interface**
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   {
       "Clients": [
           // tcp usecase
           {
               "Name": "default",
               "ServerAppName": "VideoIngestion",
               "Type": "zmq_tcp",
               "EndPoint": "127.0.0.1:9006"
           },

           // ipc usecase
           {
               "Name": "example",
               "ServerAppName": "VideoIngestion",
               "Type": "zmq_ipc",
               "EndPoint": "/EII/sockets"
           }
       ]
   }

.. list-table::
   :header-rows: 1

   * - Key
     - Type
     - Required (Mandatory)
     - Description
   * - ``Clients``
     - ``array``
     - Yes
     - Entire client interface will be added with in the array. Multiple client endpoints can be added by adding elements in the array
   * - ``Name``
     - ``string``
     - Yes
     - Name of different client interfaces
   * - ``ServerAppName``
     - ``string``
     - Yes
     - Server's AppName to which client connection is established
   * - ``Type``
     - ``string``
     - Yes
     - Specifies ZeroMQ protocol ("zmq_tcp" or "zmq_ipc") through which client connection is established
   * - ``EndPoint``
     - ``string`` or ``object``
     - Yes
     - In case of TCP or IPC (socket directory only), endpoint should be string as shown in the above examples. In case IPC explicitly specifying socket file, either object or string can be used for EndPoint. Please refer `Different ways of specifying endpoint <https://open-edge-insights.github.io/IEdgeInsights/common/libs/ConfigMgr/###**Note**-"endpoint"-can-be-given-in-different-ways:>`_


Overriding of Type and EndPoint:
--------------------------------

In a given publisher, subscriber, client or server interfaces, the "Type" and "Endpoint" values mentioned in the respective interfaces can be overriden by setting them as env variables.

Let's take publisher as an example having multiple endpoints and below is its interface:

.. code-block:: javascript

   {
       "Publishers": [
           {
               "Name": "default",
               "Type": "zmq_ipc",
               "EndPoint": "/EII/sockets",
               "Topics": [
                   "camera1_stream"
               ],
               "AllowedClients": [
                   "*"
               ]
           },
           {
               "Name": "example",
               "Type": "zmq_tcp",
               "EndPoint": "127.0.0.1:65015",
               "Topics": [
                   "*"
               ],
               "AllowedClients": [
                   "Visualizer", "VideoAnalytics"
               ]
           }
       ]
   }

Let's say we want to override the Type ("zmq_ipc") and Endpoint ("/EII/sockets") of the first publisher's interface (interface having "Name":"default"), then env variable should be set with below syntax.

.. code-block:: sh

   export PUBLISHER_default_TYPE="zmq_tcp"
   export PUBLISHER_default_ENDPOINT="127.0.0.1:65013"

In the above command ``default`` is the ``Name`` of the interface.

Similarly, we can override subscriber's, client's and server's Type and Endpoint.

.. code-block:: sh

   export SUBSCRIBER_default_TYPE="zmq_tcp"
   export SUBSCRIBER_default_ENDPOINT="127.0.0.1:65013"

   export SERVER_default_TYPE="zmq_tcp"
   export SERVER_default_ENDPOINT="127.0.0.1:65013"

   export CLIENT_default_TYPE="zmq_tcp"
   export CLIENT_default_ENDPOINT="127.0.0.1:65013"

Let's take publisher as an example having single endpoint and below is its interface:

.. code-block:: javascript

   {
       "Publishers": [
           {
               "Name": "default",
               "Type": "zmq_ipc",
               "EndPoint": "/EII/sockets",
               "Topics": [
                   "camera1_stream"
               ],
               "AllowedClients": [
                   "*"
               ]
           }
   }

If we want to override the Type ("zmq_ipc") and Endpoint ("/EII/sockets") of publisher's interface, then env variable should be set with below syntax.

.. code-block:: sh

   export PUBLISHER_TYPE="zmq_tcp"
   export PUBLISHER_ENDPOINT="127.0.0.1:65013"

Similarly, we can override subscriber's, client's and server's Type and Endpoint.

.. code-block:: sh

   export SUBSCRIBER_TYPE="zmq_tcp"
   export SUBSCRIBER_ENDPOINT="127.0.0.1:65013"

   export SERVER_TYPE="zmq_tcp"
   export SERVER_ENDPOINT="127.0.0.1:65013"

   export CLIENT_TYPE="zmq_tcp"
   export CLIENT_ENDPOINT="127.0.0.1:65013"

Overriding feature of ConfigMgr will be used in orchestrated scenarios including Kubernetes.

Broker Usecase
--------------

If publisher and subscriber wants to communicate via broker(ZmqBroker), i.e., if publisher publish data to ZmqBroker and subscriber subscribes from ZmqBroker, then the interfaces of ZmqBroker, subscriber and publisher with respect to ``zmq_tcp`` and ``zmq_ipc`` protocol as follows.

zmq_tcp Protocol usecase
------------------------

ZmqBroker:
^^^^^^^^^^

.. code-block:: javascript

   {
       //X-PUB
       "Publishers": [
           {
               "AllowedClients": [
                   "*"
               ],
               "EndPoint": "127.0.0.1:5568",
               "Name": "backend",
               "Topics": [
                   "*"
               ],
               "Type": "zmq_tcp"
           }
       ],
       // X-SUB
       "Subscribers": [
           {
               "AllowedClients": [
                   "*"
               ],
               "EndPoint": "127.0.0.1:5569",
               "Name": "frontend",
               "PublisherAppName": "*",
               "Topics": [
                   "*"
               ],
               "Type": "zmq_tcp"
           }
       ]
   }

Subscriber
^^^^^^^^^^

.. code-block:: javascript

   {
       "Subscribers": [
           {
               "Name": "default",
               "Type": "zmq_tcp",
               "EndPoint": "127.0.0.1:5568",

               // PublisherAppName will be "ZmqBroker" in case of brokered usecase 
               "PublisherAppName": "ZmqBroker",
               "Topics": [
                   "*"
               ]
           }
       ]
   }

Publisher
^^^^^^^^^

.. code-block:: javascript

   {
       "Publishers": [
           {
               "Name": "default",
               "Type": "zmq_tcp",
               "EndPoint": "127.0.0.1:5569",
               "Topics": [
                   "camera1_stream"
               ],
               // With broker usecase, AllowedClients in Publisher's interface is not required, as it acts as a subscriber to X-SUB

               // "brokered" and "BrokerAppName" should be added for ZmqBroker usecase
               "brokered": true,
               "BrokerAppName" : "ZmqBroker"
           }
       ]
   }

zmq_ipc Protocol usecase
------------------------

ZmqBroker:
^^^^^^^^^^

.. code-block:: javascript

   {
       "Publishers": [
           {
               "Name": "backend",
               "Type": "zmq_ipc",
               "EndPoint": {
                   "SocketDir": "/EII/sockets",
                   "SocketFile": "backend-socket"
               },
               "Topics": ["*"],
               "AllowedClients": ["*"]
           }
       ],
       "Subscribers": [
           {
               "Name": "frontend",
               "Type": "zmq_ipc",
               "EndPoint": {
                   "SocketDir": "/EII/sockets",
                   "SocketFile": "frontend-socket"
               },
               "PublisherAppName": "*",
               "Topics": ["*"],
               "AllowedClients": ["*"]
           }
       ]
   }

Subscriber
^^^^^^^^^^

.. code-block:: javascript

   {
       "Subscribers": [
           {
               "Name": "default",
               "Type": "zmq_ipc",
               "EndPoint": {
                   "SocketDir" : "/EII/sockets",
                   "SocketFile": "backend-socket"
               },

               // PublisherAppName will be "ZmqBroker" in case of brokered usecase 
               "PublisherAppName": "ZmqBroker",

               // Topics cannot be "*", if the only IPC directory is given
               // if it Topics "*" to be used in ipc, then socket file should be given explicitly.
               "Topics": [
                   "*"
               ]
           }
       ]
   }

Publisher
^^^^^^^^^

.. code-block:: javascript

   {
       "Publishers": [
           {
               "Name": "default",
               "Type": "zmq_ipc",
               "EndPoint": {
                   "SocketDir" : "/EII/sockets",
                   "SocketFile": "frontend-socket"
               },
               "Topics": [
                   "camera1_stream"
               ],

               // With broker usecase, AllowedClients in Publisher's interface is not required, as it acts as a subscriber to X-SUB

               // "brokered" and "BrokerAppName" should be added for ZmqBroker usecase
               "brokered": true,
               "BrokerAppName" : "ZmqBroker"
           }
       ]
   }

.. list-table::
   :header-rows: 1

   * - Key
     - Type
     - Required (Mandatory)
     - Description
   * - ``brokered``
     - ``boolean``
     - Yes
     - (Required if publishing via ZmqBroker) Specifies if publisher is using broker or not. use "brokered": true for use with broker.
   * - ``BrokerAppName``
     - ``string``
     - Yes
     - (Required if publishing via ZmqBroker) Specifies ZeroMQ Broker app name


**Note** "Endpoint" can be given in different ways:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* 
  zmq_tcp Endpoint should look like below:

  .. code-block:: javascript

       "Endpoint":"127.0.0.1:65013"

* 
  zmq_ipc Endpoint have different ways of giving.


  * 
    Specifying just socket directory

    .. code-block:: javascript

         "Endpoint":"/EII/sockets"

  * 
    Specifying socket directory and socket file

    .. code-block:: javascript

         "Endpoint":{
             "SocketDir" : "/EII/sockets",
             "SocketFile": "socketfile"
         }

         or

         "Endpoint":"/EII/sockets, socketfile"

Running Examples
----------------

The ConfigMgr library also supports Cpp APIs and Python & Go bindings. These APIs/bindings can be used in Cpp and Python/Go services in the EII stack to fetch required config/interfaces/msgbus config. 

Examples will only be compiled if the ``WITH_EXAMPLES=ON`` option is specified while running CMake.
Please refer `Examples installation <https://open-edge-insights.github.io/IEdgeInsights/common/libs/ConfigMgr/###-install-configMgr-with-examples,-test-suits-and-debug-build-enabled.>`_ 

Refer the interfaces of publisher and server in `./examples/configs/VideoAnalytics_interfaces.json <https://github.com/open-edge-insights/eii-core/blob/master/examples/configs/VideoAnalytics_interfaces.json>`_ and for subscriber and client, refer `./examples/configs/VideoAnalytics_interfaces.json <https://github.com/open-edge-insights/eii-core/blob/master/examples/configs/VideoAnalytics_interfaces.json>`_

Examples on demonstrating the usage of these APIs in the bindings have been given in respective sections below.


* `CPP Examples <https://open-edge-insights.github.io/IEdgeInsights/common/libs/ConfigMgr/###-cpp-examples>`_
* `Python Examples <https://open-edge-insights.github.io/IEdgeInsights/common/libs/ConfigMgr/###-python-examples>`_
* `Go Examples <https://open-edge-insights.github.io/IEdgeInsights/common/libs/ConfigMgr/###-go-examples>`_

CPP Examples
^^^^^^^^^^^^

.. code-block:: sh

   $ Navigate to [WORKDIR]/IEdgeInsights/common/libs/ConfigMgr
   $ sudo rm -rf build
   $ mkdir build
   $ cd build
   $ cmake -DCMAKE_INSTALL_INCLUDEDIR=$CMAKE_INSTALL_PREFIX/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX -DWITH_EXAMPLES=ON ..
   $ make
   $ sudo make install

There are currently 5 CPP examples:


#. ``examples/sample_pub.cpp``
#. ``examples/sample_sub.cpp``
#. ``examples/sample_server.cpp``
#. ``examples/sample_client.cpp``
#. ``examples/sample_getvalue.cpp``

All of the Cpp example executables are in the ``build/examples/`` directory. To run
them, execute the following command:

Before executing any of the examples, please run below command from ``build/examples/``

.. code-block:: sh

    cd ../../examples/ && source ./env.sh && cd -

Publisher example.

.. code-block:: sh

   $ ./pub

Subscriber example.

.. code-block:: sh

   $ ./sub

Server example.

.. code-block:: sh

   $ ./server

Client example.

.. code-block:: sh

   $ ./client

Sample_getvalue used to get the values from application's config

.. code-block:: sh

   $ ./sample_app

Python Examples
^^^^^^^^^^^^^^^

.. code-block:: sh

   $ Navigate to [WORKDIR]/IEdgeInsights/common/libs/ConfigMgr
   $ sudo rm -rf build
   $ mkdir build
   $ cd build
   $ cmake -DCMAKE_INSTALL_INCLUDEDIR=$CMAKE_INSTALL_PREFIX/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX -DWITH_PYTHON=ON ..
   $ make
   $ sudo make install

There are currently 5 Python examples:


#. ``examples/publisher.py``
#. ``examples/subscriber.py``
#. ``examples/echo_service.py``
#. ``examples/echo_client.py``
#. ``examples/sample_get_value.py``

All of the py examples are in ``python/examples/`` directory. To run
them, execute the following command:

Before executing any of the examples, please run below command from ``python/examples/``

.. code-block:: sh

    cd ../../examples/ && source ./env.sh && cd -

Publisher example.

.. code-block:: sh

   $ python3 publisher.py

Subscriber example.

.. code-block:: sh

   $ python3 subscriber.py

Server example.

.. code-block:: sh

   $ python3 echo_service.py

Client example.

.. code-block:: sh

   $ python3 echo_client.py

sample_get_value used to get the values from application's config

.. code-block:: sh

   $ python3 sample_get_value.py

Go Examples
^^^^^^^^^^^

.. code-block:: sh

   $ Navigate to [WORKDIR]/IEdgeInsights/common/libs/ConfigMgr/go
   $ cp -r ConfigMgr/ $GOPATH/src
   $ cd ConfigMgr/examples
   $ source go_env.sh

There are currently 5 Go examples:


#. ``publisher.go``
#. ``subscriber.go``
#. ``echo_service.go``
#. ``echo_client.go``
#. ``app_config.go``

All of the go examples are in ``go/examples/`` directory. To run
them, execute the following command:

Before executing any of the examples, please run below command from ``go/ConfigMgr/examples/``

.. code-block:: sh

    cd ../../../examples/ && source ./env.sh && cd -
    source ./go_env.sh

Publisher example.

.. code-block:: sh

   $ go build publisher.go
   $ ./publisher

Subscriber example.

.. code-block:: sh

   $ go build subscriber.go
   $ ./subscriber

Server example.

.. code-block:: sh

   $ go build echo_service.go
   $ ./echo_service

Client example.

.. code-block:: sh

   $ go build echo_client.go
   $ ./echo_client

app_config used to get the values from application's config

.. code-block:: sh

   $ go build app_config.go
   $ ./app_config

Running Unit Tests
------------------

.. note:: 



* 
  The unit tests will only be compiled if the ``WITH_TESTS=ON`` option is specified when running CMake.
  Please refer `Unit Test installation <https://open-edge-insights.github.io/IEdgeInsights/common/libs/ConfigMgr/###-install-configMgr-with-examples,-test-suits-and-debug-build-enabled.>`_ installation.

* 
  Provisioning should be done to start etcd server in dev/prod mode and to generate application specific certificates(only in prod mode).

Before executing any of the test files, please run below command from ``build/tests/``

.. code-block:: sh

    cd ../../examples/ && source ./env.sh && cd -


* To run ConfigMgr unit tests
  .. code-block::

     $ ./config_manager_unit_tests
     $ ./kvstore_client-tests
  ## Creation of grpc .deb file (Optional)

**Note**\ : This is an optional as we have already created .deb file in the repo.
If user wants to create .deb file freshly, then one has to follow this step.

Navigate to ``[WORKDIR]/IEdgeInsights/common/libs/ConfigMgr`` and run the ``grpc_deb.sh``

.. code-block:: sh

   $ sudo ./grpc_deb.sh

By executing the above script, ``grpc-1.29.0-Linux.deb`` will be created in
``[WORKDIR]/IEdgeInsights/common/libs/ConfigMgr``.
