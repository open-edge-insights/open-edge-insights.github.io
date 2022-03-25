
ConfigMgr
=========

ConfigMgr or Config manager provides CPP, Python, and Golang APIs to:


* fetch an applications configs values from the KV store.
* fetch an applications interface values from the KV store for pub, sub, server, and client.
* monitor application's config changes.
* generate MessageBus config.
* read and set the ``/GlobalEnv`` variables.
* fetch the env variables: appname, dev_mode

All the ConfigMgr operations related data is stored in the KV store of OEI during the provisioning phase. An admin can dynamically change these data.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights (OEI). This is due to the product name change of EII as OEI.


ConfigMgr Installation
----------------------

You can install the OEI Config Manager library in any of the following ways:


* Through published Debian, Fedora, or Alpine APK packages
* Installing from source

If you are installing from one of the packages then from the releases assets, select the package to install. Based on the OS that on which you are installing, run the following:

.. code-block:: sh

   # Debian
   sudo apt install libcjson1 libzmq5 zlib1g
   sudo dpkg -i <debian package>

   # Fedora
   sudo dnf install cjson zeromq zlib
   sudo rpm -i <rpm package>

   # Alpine (NOTE: the depencies get automatically installed by the apk command)
   sudo apk add --allow-untrusted <apk package>

In the above commands, installing the cJSON and ZeroMQ dependencies is required, however, in general, installation of the dev module is not required (i.e. the OS packages which include all of the headers for the libraries). If you are compiling an application that is linking to this library, then it is recommended that you install the ``dev`` versions of the libraries.


* For Ubuntu, install ``libcjson-dev libzmq3-dev zlib1g-dev``
* For Fedora, install ``cjson-devel zeromq-devel zlib-devel``
* For Alpine, install ``cjson-dev zeromq-dev zlib-dev``

If you want to compile the Config Manager from source, follow the
intructions below.

The ConfigMgr depends on CMake version 3.11+. For Ubuntu 18.04 this is not the default version installed via ``apt-get``. To install the correct version
of CMake and other ConfigMgr dependencies, refer to the `eii_libs_installer README <https://github.com/open-edge-insights/eii-core/blob/master/common/README.md>`_

To set ``CMAKE_INSTALL_PREFIX`` for the installation, run the following command:

.. code-block:: sh

       export CMAKE_INSTALL_PREFIX="/opt/intel/eii"

ConfigMgr installs to ``/opt/intel/eii/lib/``. On some platforms, this is not included in the ``LD_LIBRARY_PATH`` by default. As a result, you must add this directory to your ``LD_LIBRARY_PATH``\ , otherwise you will encounter issues using the ConfigMgr. This can be accomplished with the following ``export``\ :

.. code-block:: sh

   export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/eii/lib/

.. note::  You can also specify a different library prefix to CMake through the ``CMAKE_INSTALL_PREFIX`` flag. If different installation path is given via ``CMAKE_INSTALL_PREFIX``\ , then ``$LD_LIBRARY_PATH`` should be appended by $CMAKE_INSTALL_PREFIX/lib.


To install the remaining dependencies for the message bus execute the following command:

.. note:: \ : It is highly recommended that you use a python virtual environment to install the python packages, so that the system python installation doesn't get altered. For details on setting up and using Python virtual environment, refer to `Python virtual environment <https://www.geeksforgeeks.org/python-virtual-environment/>>`_.


.. code-block:: sh

   sudo apt install libcjson-dev libzmq5 zlib1g-dev

.. note::  For Fedora, the packages is ``cjson-devel zeromq-devel zlib-devel``. For Alpine, the package is ``cjson-dev zeromq-dev zlib-dev``.


If you wish to compile the Python binding as well, then you must also install the Python requirements. To do this, execute the following ``pip`` command:

.. code-block:: sh

   pip3 install --user -r ./python/requirements.txt

Using System gRPC
^^^^^^^^^^^^^^^^^

The ConfigMgr library can be built using the gRPC version already installed on
the system. To do this, use the ``SYSTEM_GRPC`` flag when running the ``cmake``
command. Below is an example of what this would look like:

.. code-block:: sh

    cmake -DSYSTEM_GRPC=ON ..

.. note::  The ConfigMgr library depends on a specific gRPC version, 1.29.0. A debian package is provided for this in the grpc-package directory. To install
   this in Ubuntu run the following command:


.. code-block:: sh

   sudo dpkg -i grpc-package/grpc-1.29.0-Linux.deb

Compilation
-----------

The Config Manager utilizes CMake as the build tool for compiling the library. The simplest sequence of commands for building the library are shown below.

.. code-block:: sh

   mkdir build
   cd build
   cmake ..
   make

This will compile only the C library for the ConfigMgr. To build with the Python binding, specify the ``WITH_PYTHON`` flag, when
executing the ``cmake`` command. Refer to the following:

.. code-block:: sh

   cmake -DWITH_PYTHON=ON ..

If you wish to include installation of the Go binding with the installation of the OEI library, then specify the ``WITH_GO`` flag when executing the ``cmake``
command.

.. code-block:: sh

   cmake -DWITH_GO=ON ..

.. note::  This only copies the Go binding library to your system's ``$GOPATH``. If you do not have your ``$GOPATH`` specified in your system's environmental
   variables then an error will occur while executing the ``cmake`` command.


In addition to the ``WITH_PYTHON`` and ``WITH_GO`` flags, the ConfigMgr CMake files add flags for building the C examples and the unit tests associated
with the library. The following table specifies all of the available flags that can be given to CMake for building the ConfigMgr.

.. list-table::
   :header-rows: 1

   * - Flag
     - Default
     - Description
   * - ``WITH_TESTS``
     - ``OFF``
     - If set to ``ON``\ , builds the C unit tests with the ConfigMgr compilation
   * - ``WITH_EXAMPLES``
     - ``OFF``
     - If set to ``ON``\ , then CMake will compile the C examples in addition to the library
   * - ``WITH_DOCS``
     - ``OFF``
     - If set to ``ON``\ , then CMake will add a ``docs`` build target to generate documentation


.. note:: 


   * These flags are in addition to any and all flags that are available for the ``cmake`` command. See the CMake documentation for additional flags.
   * See the `Generating Documentation <#generating-documentation>`__ section.


If you wish to compile the ConfigMgr in debug mode, then you can set the
the ``CMAKE_BUILD_TYPE`` to ``Debug`` when executing the ``cmake`` command. Refer to the following:

.. code-block:: sh

   cmake -DCMAKE_BUILD_TYPE=Debug ..

Generating Documentation
^^^^^^^^^^^^^^^^^^^^^^^^

Generating the documentation has several dependencies which are not installed by the ``install.sh`` script. You must install the following packages to generate the documentation:

.. code-block:: sh

   sudo apt install doxygen texlive-full

**Warning:** This install way take a very long time. It will install more than ``4GB`` of packages.

If you are building the Python binding by using the ``WITH_PYTHON`` flag, then you must also install Sphinx and an extension for Sphinx. This can be accomplished with the following commands:

.. code-block:: sh

   sudo apt install python3-sphinx
   sudo -H -E pip3 install m2r

.. note::  The commands above assume you already have Python 3.6 and pip
   installed on your system.


**Go documentation generation is WIP.**

Once you have completed these steps, the documentation can be generated by running the following make command:

.. code-block:: sh

   make docs && make docs

.. note::  Currently you need to run ``make docs`` twice so that the table of contents is generated correctly for each of the documents. This will be fixed in the future.


The PDF documents will be in the ``docs/pdfs/`` directory within your ``build`` directory. There will be other log files and output files associated with the building of the PDFs. Any file that does not end in ``.pdf`` can be ignored.

Packaging
---------

.. note::  If the build is done using the system installed gRPC, then packaging cannot be done . This is due to the various linking issues that can occur in that scenario.


This library supports being packaged as a Debian, RPM, or Alpine APK packages. This is all accomplished via CMake. By default, packaging is disabled. To enable packaging, add the ``-DPACKAGING=ON`` flag to your CMake command (see Compilation section). This command will look something like:

.. code-block:: sh

   cmake -DPACKAGING=ON ..

By default, the packaging utilities will scan the system for the required
toolchains it needs to build each package type (Deb, RPM, and APK). If it does
not find the required toolsets, then it will disable that form of packaging.
The packaging utilities provide CMake flags to force packaging as any of the
supported package types. If a given package type, ex. APK, is set to be enabled
manually by its CMake flag and its required packaging toolchain does not exist,
then CMake will raise a fatal error.

The table below provides the required toolchains for each package type as well
as the CMake flag to set to ``ON`` to manually enable a packaging type:

.. list-table::
   :header-rows: 1

   * - Package Type
     - Required Tools
     - Manual Package Flag
   * - ``deb``
     - ``dpkg-deb``
     - ``PACKAGE_DEB``
   * - ``rpm``
     - ``rpmbuild``
     - ``PACKAGE_RPM``
   * - ``apk``
     - ``docker``
     - ``PACKAGE_APK``


.. note::  Manually setting a given package type to be built (e.g. setting
   ``-DPACKAGE_DEB=ON``\ ) still requires that the ``-DPACKAGING=ON`` to be set.


After the required toolchains have been installed and CMake has been run with
some combination of the packaging flags, the library can be packaged with the
following commands:

.. code-block:: sh

   make package

The command above will build the Debian and RPM packages (depending on the
specified CMake flags).

To build the Alpine APK package, execute the following command:

.. code-block:: sh

   make package-apk

**IMPORTANT:**

The Config Manager depends on the OEI Utils and Message Bus libraries.
In order to compile the Alpine APK package for the Config Manager it must
have the APK packages for the OEI Utils and Message Bus modules.

To provide this, you must first build or download the Alpine APK package for the
OEI Utils library (see it's repo `here <https://github.com/open-edge-insights/eii-c-utils>`_
to obtain the library) and also for the Message Bus (see it's repo
`here <https://github.com/open-edge-insights/eii-messagebus>`_ to obtain the library).

Once you have the APKs, create an, "apks" directory at the top level of this
repository.

.. code-block:: sh

   mkdir apks/

Next, place the OEI Utils and Message Bus APK packages into the, "apks",
directory. Then execute the ``make package-apk`` command. If this is not done,
then the build will fail.

A Note on Alpine APK Packaging
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to package the library as an Alpine APK package, the packaging utility
must use a Docker container to have access to the proper Alpine APK toolchains.
This container will automatically be built when the CMake command is ran to
configure your build environment.

By default, Alpine 3.14 is used to build the package. However, this version
can be changed by setting the ``APKBUILD_ALPINE_VERSION`` CMake flag to the
version of Alpine you wish to use (ex. ``-DAPKBUILD_ALPINE_VERSION=3.12``\ ).

Install ConfigMgr with Python bindings, Go bindings, Examples, Test suits, and Debug Build
------------------------------------------------------------------------------------------

.. code-block:: sh

   rm -rf build
   mkdir build
   cd build
   cmake -DCMAKE_INSTALL_INCLUDEDIR=$CMAKE_INSTALL_PREFIX/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX -DWITH_PYTHON=ON -DWITH_GO=ON -DWITH_EXAMPLES=ON -DWITH_TESTS=ON -DCMAKE_BUILD_TYPE=Debug ..
   make
   sudo make install

``WITH_PYTHON=ON``\ , ``WITH_GO=ON``\ , ``WITH_EXAMPLES=ON``\ , ``WITH_TESTS=ON`` and ``CMAKE_BUILD_TYPE=Debug`` to compile ConfigMgr with Python bindings, Go bindings, Examples, Unit Tests and Debug mode respectively.

Interfaces
----------

ConfigMgr parses the data from the kv store (eg: etcd) and the application environment variables for its functionality.
It supports Publisher, Subscriber, Server and Client interfaces. Below are the examples for providing
different interfaces and different usecases.

Please refer `different ways of giving endpoints <#note-endpoint-can-be-given-in-different-ways>`__

Publisher Interface
-------------------

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
     - In case of TCP or IPC (socket directory only), endpoint should be string as shown in the above examples. In case IPC explicitly specifying socket file, either object or string can be used for EndPoint. Please refer `Different ways of specifying endpoint <#note-endpoint-can-be-given-in-different-ways>`__
   * - ``Topics``
     - ``array``
     - Yes
     - Specifying the topics on which data will be published on. Multiple elements in this array can denote multiple topics published on the same endpoint
   * - ``AllowedClients``
     - ``array``
     - Yes
     - Specifying who can subscribe to the the topic on which data is published. If AllowedClients is "*", then all the provisioned services can receive the data published.


Subscriber Interface
^^^^^^^^^^^^^^^^^^^^

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
     - In case of TCP or IPC (socket directory only), endpoint should be string as shown in the above examples. In Case IPC explicitly specifying socket file, either object or string can be used for EndPoint. Please refer `Different ways of specifying endpoint <#note-endpoint-can-be-given-in-different-ways>`__
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
     - In case of TCP or IPC (socket directory only), endpoint should be string as shown in the above examples. In case IPC explicitly specifying socket file, either object or string can be used for EndPoint. Please refer `Different ways of specifying endpoint <#note-endpoint-can-be-given-in-different-ways>`__
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
     - In case of TCP or IPC (socket directory only), endpoint should be string as shown in the above examples. In case IPC explicitly specifying socket file, either object or string can be used for EndPoint. Please refer `Different ways of specifying endpoint <#note-endpoint-can-be-given-in-different-ways>`__


Overriding of Type and EndPoint
-------------------------------

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

ZmqBroker (use case)
^^^^^^^^^^^^^^^^^^^^

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

ZmqBroker
^^^^^^^^^

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

Subscriber Details
^^^^^^^^^^^^^^^^^^

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

Publisher Details
^^^^^^^^^^^^^^^^^

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


**Note** "Endpoint" can be given in different ways
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


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

The ConfigMgr library also supports Cpp APIs and Python & Go bindings. These APIs/bindings can be used in Cpp and Python/Go services in the OEI stack to fetch required config/interfaces/msgbus config.

Examples will only be compiled if the ``WITH_EXAMPLES=ON`` option is specified while running CMake.
Please refer `Examples installation <#-install-configmgr-with-examples,-test-suits-and-debug-build-enabled.>`__

Refer the interfaces of publisher and server in `./examples/configs/VideoAnalytics_interfaces.json <https://github.com/open-edge-insights/eii-configmgr/blob/master/examples/configs/VideoAnalytics_interfaces.json>`_ and for subscriber and client, refer `./examples/configs/VideoAnalytics_interfaces.json <https://github.com/open-edge-insights/eii-configmgr/blob/master/examples/configs/VideoAnalytics_interfaces.json>`_

**Note** : In the above json files, if the message communication type is ``zmq_ipc``\ , user needs to make sure the socket directory is present in the host machine.

Examples on demonstrating the usage of these APIs in the bindings have been given in respective sections below.


* 
  `CPP Examples <#-cpp-examples>`__

* 
  `Python Examples <#-python-examples>`__

* `Go Examples <#-go-examples>`__

CPP Examples
^^^^^^^^^^^^

.. code-block:: sh

   Navigate to [WORKDIR]/IEdgeInsights/common/libs/ConfigMgr
   sudo rm -rf build
   mkdir build
   cd build
   cmake -DCMAKE_INSTALL_INCLUDEDIR=$CMAKE_INSTALL_PREFIX/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX -DWITH_EXAMPLES=ON ..
   make
   sudo make install

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

   ./pub

Subscriber example.

.. code-block:: sh

   ./sub

Server example.

.. code-block:: sh

   ./server

Client example.

.. code-block:: sh

   ./client

Sample_getvalue used to get the values from application's config

.. code-block:: sh

   ./sample_app

Python Examples
^^^^^^^^^^^^^^^

.. code-block:: sh

   Navigate to [WORKDIR]/IEdgeInsights/common/libs/ConfigMgr
   sudo rm -rf build
   mkdir build
   cd build
   cmake -DCMAKE_INSTALL_INCLUDEDIR=$CMAKE_INSTALL_PREFIX/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX -DWITH_PYTHON=ON ..
   make
   sudo make install

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

   python3 publisher.py

Subscriber example.

.. code-block:: sh

   python3 subscriber.py

Server example.

.. code-block:: sh

   python3 echo_service.py

Client example.

.. code-block:: sh

   python3 echo_client.py

sample_get_value used to get the values from application's config

.. code-block:: sh

   python3 sample_get_value.py

Go Examples
^^^^^^^^^^^

.. code-block:: sh

   Navigate to [WORKDIR]/IEdgeInsights/common/libs/ConfigMgr/go
   cp -r ConfigMgr/ $GOPATH/src
   cd ConfigMgr/examples
   source go_env.sh

There are currently 5 Go examples:


#. ``publisher.go``
#. ``subscriber.go``
#. ``echo_service.go``
#. ``echo_client.go``
#. ``app_config.go``

All of the go examples are in the ``go/examples/`` directory. To run them, execute the following command:

.. note::  Before executing any of the examples, run the following command from ``go/ConfigMgr/examples/``


.. code-block:: sh

    cd ../../../examples/ && source ./env.sh && cd -
    source ./go_env.sh

The Publisher example is as follows:

.. code-block:: sh

   go build publisher.go
   ./publisher

The Subscriber example is as follows:

.. code-block:: sh

   go build subscriber.go
   ./subscriber

The Server example is as follows:

.. code-block:: sh

   go build echo_service.go
   ./echo_service

The Client example is as follows:

.. code-block:: sh

   go build echo_client.go
   ./echo_client

The app_config used to get the values from application's config is as follows:

.. code-block:: sh

   go build app_config.go
   ./app_config

Running Unit Tests
------------------

.. note:: 


   * The unit tests will only be compiled if the ``WITH_TESTS=ON`` option is specified when running CMake. Refer `Unit Test installation <#-install-configmgr-with-examples,-test-suits-and-debug-build-enabled.>`__ installation.
   * Provisioning should be done to start etcd server in the Dev or the Prod mode and to generate application specific certificates (only in prod mode).


Before executing any of the test files, run the followins command from ``build/tests/``\ :

.. code-block:: sh

    cd ../../examples/ && source ./env.sh && cd -


* To run ConfigMgr unit tests

.. code-block:: sh

   ./config_manager_unit_tests
   ./kvstore_client-tests

Creation of grpc .zip file (Optional)
-------------------------------------

.. note::  This is an optional as we have already created .zip file in the repo.
   If user wants to create .zip file freshly, then one has to follow this step.


Navigate to ``[WORKDIR]/IEdgeInsights/common/libs/ConfigMgr/grpc-package`` and run the ``grpc_zip.sh``

.. code-block:: sh

   sudo ./grpc_zip.sh

By executing the above script, grpc zip package will be created in ``[WORKDIR]/IEdgeInsights/common/libs/ConfigMgr/grpc-package``.

Generation of python .whl file (Optional)
-----------------------------------------

**Note:** This is an optional as we have already hosted .whl file. If user wants to create .whl file freshly, then complete the following steps.


#. 
   Installation of ``wheel``

   .. code-block:: sh

       pip3 install –upgrade setuptools wheel

#. 
   Navigate to ``[WORKDIR]/IEdgeInsights/common/libs/ConfigMgr/python`` and run the following command:

   .. code-block:: sh

       python3 setup_packaging.py sdist bdist_wheel --plat-name=manylinux2014_x86_64

#. 
   ConfigMgr .whl package will be created in the ``dist`` folder.
