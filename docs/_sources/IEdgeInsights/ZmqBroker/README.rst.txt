
**Contents**


* `ZeroMQ Broker <#zeromq-broker>`__

  * `Overview <#overview>`__
  * `Security <#security>`__

    * `IPC <#ipc>`__
    * `TCP <#tcp>`__

  * `Performance Implications <#performance-implications>`__
  * `Configuration <#configuration>`__

    * `Application/Service Configuration <#applicationservice-configuration>`__
    * `Interface Configuration <#interface-configuration>`__

  * `Docker <#docker>`__
  * `Connecting EII Services to the ZeroMQ Broker <#connecting-eii-services-to-the-zeromq-broker>`__
  * `Bare Metal <#bare-metal>`__

    * `Compilation <#compilation>`__
    * `Usage <#usage>`__
    * `Running Unit Tests <#running-unit-tests>`__

ZeroMQ Broker
=============

Overview
--------

The ZeroMQ Broker provides the ability to proxy data coming over the
EII Message Bus ZeroMQ protocol plugin for publishers and subscribers.
Traditionally, each EII Message Bus ZeroMQ publisher has its own IPC socket
or TCP (host, port) combination. This broker allows for publishers to connect
to the proxy's publisher TCP or IPC socket for sending published messages and
subscribers to connect to one central subscriber TCP or IPC socket
to receive incoming messages from the broker.

The ZeroMQ Broker builds off of the functionality provided by the ZeroMQ
``zmq_proxy()`` API, which uses two separate sockets and hands messages off from
one socket to the other.

The following diagram shows a high-level flow of how the ZeroMQ Broker
functions.

.. code-block::

    +-----------+    +-----------+    +-----------+
    | Publisher |    | Publisher |    | Publisher |
    +-----------+    +-----------+    +-----------+
    |  ZMQ_PUB  |    |  ZMQ_PUB  |    |  ZMQ_PUB  |
    +-----------+    +-----------+    +-----------+
       connect          connect          connect
         |                |                 |
         +----------------+-----------------+
                          |
                         bind
                  +----------------+
                  |    ZMQ_XSUB    |
                  +----------------+
                  |     proxy      |
                  +----------------+
                  |    ZMQ_XPUB    |
                  +----------------+
                         bind
                          |
          +---------------+-------------------+
          |               |                   |
       connect          connect            connect
    +------------+    +------------+    +------------+
    |  ZMQ_SUB   |    |  ZMQ_SUB   |    |  ZMQ_SUB   |
    +------------+    +------------+    +------------+
    | Subscriber |    | Subscriber |    | Subscriber |
    +------------+    +------------+    +------------+

As shown in the diagram above, the broker uses two ZeroMQ sockets:


#. A socket for publishers to connect (i.e. the ``ZMQ_XSUB`` socket), which is
    referred to as the **frontend** socket
#. A socket for the subscribers to connect to (i.e. the ``ZMQ_XPUB`` socket), which
    is referred to as the **backend** socket

The broker binds to the TCP (host, port) or IPC socket file (based on its
configuration) for each of its sockets. Messages are sent from publishers into
the **frontend** socket and then relayed onto the **backend** socket which
forwards the messages onto the subscribers.

Security
--------

The security for the broker depends on whether the broker is using TCP or IPC
for its respective sockets.

.. note::  The broker can be used with the **frontend** being a TCP socket and
   the **backend** being an IPC socket (and vice-versa).


The following sections cover the security for the TCP and IPC modes.

IPC
^^^

When the broker is using an IPC socket (for either the **frontend** or **backend**
sockets), the ability to send/receive messages shall be handled by Linux file
permissions. There is no encryption which shall occur for the messages sent
over IPC sockets.

TCP
^^^

For TCP connections, ZeroMQ offers mechanisms for both encryption and authentication.
This is accomplished via elliptical curve keys generated when EII is provisioned.
These keys provide the encryption and authentication via the ZAP protocol and a
list of allowed clients authentication is handled.

The encryption and authentication is all handled under the hood by the EII
Message Bus for publishers and subscribers which are connecting to the broker.
Via the configuration of the broker's messaging interfaces the broker has the
required keys for decrypting messages and for determining whether or not a
given publisher or subscriber is allowed to connect.

See the configuration section for more details on listing out allowed clients
and configuring the security for the ZeroMQ Broker.

**IMPORTANT SECURITY IMPLICATION**

For TCP connections, there is one important security implication to be aware
of. The broker manages a single list of all of the publishers and subscribers
which are allowed to connect. If a given subscriber can connect, then all of
the data coming through that broker instance is available to that subscriber to
receive. Essentially what this means, is that the broker does not do authentication
on a per topic/stream basis, only on a connection basis.

Performance Implications
------------------------

When using the ZeroMQ Broker it is important to understand the implications
on the performance of the networking in the EII platform.

By using the broker, an extra network hop is incurred for all messages sent
over the EII Message Bus ZeroMQ protocol plugin. This will increase the latency
for the messages sent from the publishers connecting to the broker to the
subscribers connecting to the broker.

**IMPORTANT NOTE:**

Having the broker running in an EII deployment does not make every message
from publishers go through the broker. Each publisher must be configured to
connect to the broker. Publishers which are not configured to connect to the
broker will still bind to their respective IPC socket or TCP (host, port)
combination.

Configuration
-------------

.. note::  The JSON schema for the configuration properties of the ZeroMQ
   Broker can be found in the, "schema.json", file.


Configuration of the ZeroMQ Broker is predominantly accomplished from the
EII Configuration Manager APIs. However, it can also be configured via JSON
files, as described in the Usage section below. The configuration via JSON
files is only for development/debugging purposes. This section will focus on
the preduction configuration of the broker.

The configuration of the broker falls into two main areas:


#. **Application/Service Configuration** - Configuration of the runtime behavior
#. **Interface Configuration** - Configuration of the networking interfaces
    (i.e. frontend and backend sockets)

Aside from the interface and service configuration, there are two properties
that are set via environmental variables: ``AppName`` and ``DEV_MODE``. The
``AppName`` variable determines how the service quries its configuration using
the Configuration Manager APIs. For docker-compose, this is set in the
``environment`` section of the, "docker-compose.yml", file under the
``ia_zmq_broker`` section. The default value for this is ``ZmqBroker``.

The ``DEV_MODE`` variable sets whether or not the broker will attempt to use
certificates with the Configuration Manager APIs and over any TCP connections.
This also is obtainerd from the ``environment`` section of the, "docker-compose.yml",
file.

**IMPORTANT NOTE:**
There are some other configuration properties in the ``environment`` section of
the, "docker-compose.yml", file. These are provided to the service from the,
"build/.env", file. See the EII User Guide for more details.

The following two sections cover that various configuration properties the
broker supports for interfaces and application configuration.

Application/Service Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The broker supports two configuration parameters; one for the Linux scheduler
policy and the other for the scheduler priority. These two properties are
applied to the main thread which is running the ZeroMQ proxy.

The configuration values must be specified in the config.json prior to running
the ``builder.py`` script or eii_config.json after using the ``builder.py``
script.

The table below specifies the details for these parameters. Note that neither of
these are required in the configuration and should only be set if the user
knows they need to have a certain scheduling policy and priority for the broker.

.. list-table::
   :header-rows: 1

   * - Key
     - Type
     - Description
   * - ``sched_policy``
     - ``string``
     - Scheduling policy to set for the main thread. Must be: ``SCHED_OTHER``\ , ``SCHED_IDLE``\ , ``SCHED_BATCH``\ , ``SCHED_FIFO``\ , or ``SCHED_RR``.
   * - ``sched_priority``
     - ``integer``
     - Sets the priority for the thread. Must be between ``0`` and ``99``. Only valid for ``SCHED_FIFO`` and ``SCHED_RR`` policies.


An example of applying these two settings to the ZeroMQ Broker's
configuration is shown below. In this example, the configuration is an
excerpt from the, 'IEdgeInsights/build/provision/config/eii_config.json",
generated while using the ``builder.py`` script.

.. code-block:: javascript

   {
       // ... omitted ...

       "/ZmqBroker/config": {
           "sched_policy": "SCHED_FIFO",
           "sched_priority": 42
       }

       // ... omitted ...
   }

.. note::  There will also be an ``/ZmqBroker/interfaces`` section associated
   with the broker, but that is covered in the next section.


The configuration above sets the Linux scheduler policy to ``SCHED_FIFO`` and its
priority to ``42``. As stated before, this can also be left entirely empty and the
broker thread will default to the Linux default thread scheduling policy/priority.

Interface Configuration
^^^^^^^^^^^^^^^^^^^^^^^

The configuration of the networking interfaces for the ZeroMQ Broker is
divided into two sections, "Subscribers", and, "Publishers". The, "Subscribers",
sections sets the configuration for the **frontend** socket (since it subscribes
to the messages coming from publishers), and the, "Publishers", section
determines the configuration for the **backend** socket.

The Configuration Manager API assumes that the, "Subscribers", and, "Publishers",
section is a list. However, since the ZeroMQ Broker does not use multiple
publisher and subscriber sockets, the lists must only contain one element.

The single JSON object which is expected to be in the, "Subscribers", and,
"Publishers", sections is essentially the same. An example is shown below of
the interfaces configuration for a broker instance with TCP sockets for both
the **frontend** and **backend**.

.. code-block:: javascript

   {
       "Subscribers": [
           {
               "Name": "frontend",
               "Type": "zmq_tcp",
               "EndPoint": "0.0.0.0:60514",
               "Topics": ["*"],
               "PublisherAppName": "*",
               "AllowedClients": ["*"]
           }
       ],
       "Publishers": [
           {
               "Name": "backend",
               "Type": "zmq_tcp",
               "EndPoint": "0.0.0.0:60515",
               "Topics": ["*"],
               "AllowedClients": ["*"]
           }
       ]
   }

The primary difference to point out is that the, "Name", and, "Topics",
fields are different. For the, "Subscribers", the, "Name", MUST be the
string ``frontend``\ , and the, "Topics", list must only contain a single value in
it, which is ``"*"``. For the, "Publishers", object the same rules hold
true. Note that, "AllowedClients", also has a ``"*"`` in it. If you wish to only
allowed certain services to connect to the broker, then change this to a list
of the app name's of the services which are allowd to connect.

This allows the broker to query specific configuration objects using the
defined names ``frontend`` and ``backend``. It is important to note that if the
deployment needs to limit the services which can connect to the broker, then
the, "AllowedClients", list should be modified to contain only the app names of
the services allowed to connect.

For IPC sockets, the configuration is very similar, except that the, "EndPoint",
and, "Type", values are different (as shown in the example below).

.. code-block:: javascript

   {
       "Subscribers": [
           {
               "Name": "frontend",
               "Type": "zmq_ipc",
               "EndPoint": {
                   "SocketDir": "/tmp/socks",
                   "SocketFile": "frontend-socket"
               },
               "PublisherAppName": "*",
               "Topics": ["*"],
               "AllowedClients": ["*"]
           }
       ],
       "Publishers": [
           {
               "Name": "backend",
               "Type": "zmq_ipc",
               "EndPoint": {
                   "SocketDir": "/tmp/socks",
                   "SocketFile": "backend-socket"
               },
               "Topics": ["*"],
               "AllowedClients": ["*"]
           }
       ]
   }

In the example above, note that the, "Type", values is now ``zmq_ipc`` and the,
"EndPoint", is a JSON object which defines the socket directory and file to use
for the sockets.

**IMPORTANT NOTES:**

There are two important notes to be aware of with the configurations presented
above:


#. Although the configurations above showcase using IPC and TCP for both the
    frontend and backend sockets they can be used in any combination (e.g. the
    broker can use an IPC socket for the frontend and TCP for the backend).
#. When using IPC sockets, the name of the, "SocketFile", must also be specified
    for all subscribers or publishers which will be connecting to that socket.

Docker
------

The ZeroMQ Broker is integrated into the EII docker-compose ecosystem. To
build and use the Docker container, follow the defined building / provisioning
instructions for EII.

If you are using a YAML file with the ``builder.py`` script, then make sure
to add ``ZmqBroker`` under the ``AppName`` section of the YAML file. This will
include the broker into the resulting docker-compose.yml file and it will
include the broker's configuration into the ``eii_config.json`` which shall
be used when provisioning.

Connecting EII Services to the ZeroMQ Broker
--------------------------------------------

In order to connect EII Services, such as the VideoIngestion or VideoAnalytics
services, user must edit their interfaces configuration to tell them the
broker instance to connect to. Namely, you need to add two keys to the,
"Publishers", configuration to have two additional keys:


#. ``BrokerAppName`` - This specifies the AppName of the targeted broker instance
#. ``brokered`` - This tells the EII Message Bus that the given publisher instance
    is brokered

As an example, below is the changes in the interfaces configuration for enabling
the VideoIngestion and VideoAnalytics to communicate via ZmqBroker.

.. note::  Though the example shows VideoIngestion and VideoAnalytics
   using the ZmqBroker, it is not recommended for video streaming use-cases
   due to the extra hop added to the video frames which will affect performance.


Changes to VideoIngestion/config.json which is a publisher.
The following adds the, "BrokerAppName", and, "brokered", keys to the default
publisher's configuration.

.. code-block:: javascript

       "interfaces": {
           "Publishers": [
               {
                   "Name": "default",
                   "Topics": [
                       "camera1_stream"
                   ],
                   "Type": "zmq_tcp",
                   "EndPoint": "ia_zmq_broker:60514",
                   "brokered": true
                   "BrokerAppName": "ZmqBroker",
                   "AllowedClients": [
                       "*",
                   ],
               }
           ]
        }

For the subscriber which is VideoAnalytics, there wont be much change other than
changing the endpoint to point to ZmqBroker along with PublisherAppName.

.. code-block:: javascript

       "interfaces": {
           "Subscribers": [
               {
                   "Name": "default",
                   "Type": "zmq_tcp",
                   "EndPoint": "ia_zmq_broker:60515",
                   "PublisherAppName": "ZmqBroker",
                   "Topics": [
                       "camera1_stream"
                   ],
                   "zmq_recv_hwm": 50
               }
           ]
       }

Bare Metal
----------

**IMPORTANT NOTE:**

Running the broker using bare-metal is not recommended for production environments.
This way of running the broker is meant purely for development and debugging
purposes.

Compilation
^^^^^^^^^^^

The ZeroMQ Broker is written in C++ and as such utilizes the CMake build
system for building the binary for the broker.

The ZeroMQ Broker has the following dependencies:


* CMake 3.15+
* IntelSafeString
* EIIUtils
* libzmq
* EIIConfigMgr
* EIIMessageBus (only required for unit tests)

Prior to building the ZeroMQ Broker, you must install these libraries on
the target system. This can be done using the, "common/eii_libs_installer.sh"
script.

Once the dependencies for the broker are installed on your system, run the
following commands to build the ZeroMQ Broker.

.. code-block:: sh

   # 1. Create the build directory
   $ mkdir build/

   # 2. Change directories into the newly created build/ directory
   $ cd build/

   # 3. Run the CMake command to setup the build environment (see subsections)

   # 3a. If you want to compile without the unit tests execute the following command
   $ cmake ..

   # 3b. If you wish to compile the unit tests, then execute the following (note:
   # it is recommended to build in Debug mode)
   $ cmake -DCMAKE_BUILD_TYPE=Debug -DWITH_TESTS=ON ..

   # 4. Build the ZeroMQ Broker binary
   $ make

Usage
^^^^^

After building the ZeroMQ Broker you will have a binary named, "zmq-broker".
Use this binary to run the ZeroMQ Broker.

The ZeroMQ Broker can be configured from the EII Configuration Manager as
well as from environmental variables and JSON configuration files.

To use the configuration obtained via the EII Configuration Manager, simply
start the binary with no parameters; however, make sure to set the ``AppName``
and ``DEV_MODE`` environmental variables. As mentioned at the beginning of the,
"Bare Metal", section, this way of running the broker should only be done in
development environments. As such, it is assumed that the ``DEV_MODE`` environmental
variable will be set to ``true`` and no security shall be used with the Configuration
Manager or over EII Message Bus ZeroMQ TCP connections. Additionall, for running
in this way, the, "ia_etcd", container must be running.

.. code-block:: sh

   # Set DEV_MODE environmental variable to "true"
   $ export DEV_MODE=true

   # Set the AppName environmental variable to "ZmqBroker"
   $ export AppName=ZmqBroker

   # Run the broker
   $ ./zmq-broker

The broker also supports receiving its configuration through JSON configuration
files and environmental variables. To set the log level, Linux scheduler policy
and priority set the following environmental variables (defined in Configuration
section of this README).


* ``C_LOG_LEVEL`` - Log level (will default to ``ERROR`` log level)
* ``SCHED_POLCIY`` - Scheduler policy
* ``SCHED_PRIORITY`` - Scheduler priority

Note that none of the above environmental variables are required to run the
broker.

When using JSON files, the broker will expect two positional command line
arguments. The first will be the configuration of the **frontend** socket and
the second will be for the **backend** configuration.

..

   **REMINDER:** The **frontend** refers to the configuration for socket which
   publishers shall connect to, and the **backend** refers to the configuration
   for the socket which subscribers shall connect to.


The contents of the JSON is analygous to the JSON configuration files used in
the EII Message Bus (see the IEdgeInsights/common/libs/EIIMessageBus/README.md
for more information). The contents of the JSON files depends on whether if
the given socket is supposed to use TCP or IPC.

For **IPC** configurations, the frontend and backend configurations need to look
respectively as follows:

**Frontend:**

.. code-block:: javascript

   {
       // Which ZeroMQ protocol to use (TCP vs. IPC)
       "type": "zmq_ipc",

       // Socket directory to create the socket file in
       "socket_dir": "/tmp",

       // Specifies the IPC endpoint config (i.e. socket directory and file)
       "": {
           // IPC socket file to bind to for the frontend socket
           "socket_file": "frontend-sock"
       }
   }

..

   *\ *NOTE:* This example configuration is stored in, "examples/ipc_frontend_example.json",
   and can be used with the, "examples/configs/ipc_publisher_brokered.json",
   EII Message Bus configuration file.


The table below desribes the purpose of each key in the JSON configuration file.

.. list-table::
   :header-rows: 1

   * - Key
     - Description
   * - ``type``
     - Specifies the ZeroMQ protocol to use, must be either ``zmq_ipc`` or ``zmq_tcp``.
   * - ``socket_dir``
     - Specifies the directory in which to create all of the IPC socket files.
   * - ``""``
     - Gives the IPC socket file configuration for the frontend IPC socket.
   * - ``socket_file``
     - This key specifies the name of the socket file to bind to.


For the ``socket_file`` key, when connecting a publisher, the publisher's
configuration must also use this same ``socket_file`` key in the configuration
for it. Otherwise, it will fail to connect because it will attempt to
bind/connect to its topic name (see the EII Message Bus's documentation for
more details).

**Backend:**

.. code-block:: javascript

   {
       // Which ZeroMQ protocol to use (TCP vs. IPC)
       "type": "zmq_ipc",

       // Socket directory to create the socket file in
       "socket_dir": "/tmp",

       // Specifies the IPC endpoint config (i.e. socket directory and file)
       "": {
           // IPC socket file to bind to for the backend socket
           "socket_file": "backend-sock"
       }
   }

..

   *\ *NOTE:* This example configuration is stored in, "examples/ipc_backend_example.json",
   and can be used with the, "examples/configs/ipc_subscriber_brokered.json",
   EII Message Bus configuration file.


The configuration for the backend socket is the exact same as for the frontend.

For **TCP** configurations, the frontend anc backend configurations need to look
respectively as follows:

**Frontend:**

.. code-block:: javascript

   {
       // Which ZeroMQ protocol to use (TCP vs. IPC)
       "type": "zmq_tcp",

       // List of curve public keys for publishers allowed to connect to the
       // frontend socket
       "allowed_clients": ["4J4?(I13cwJgqi+T5nxg:Dyr5)l&reK]cxxTfa9V"],

       // Specifies the configuration for the frontend socket
       "": {
           // TCP host
           "host": "0.0.0.0",

           // TCP port
           "port": 5568,

           // Server secret key to use for encryption / authentication
           "server_secret_key": "H[J1%f:#?0Y1cpUL<nEkPKm.]:n@JI>j]*!u3N#9"
       }
   }

.. note::  This example configuration is stored in, "examples/tcp_frontend_example.json",
   and can be used with the, "examples/configs/tcp_publisher_brokered_with_security.json",
   EII Message Bus configuration file.


The table below desribes the purpose of each key in the JSON configuration file.

.. list-table::
   :header-rows: 1

   * - Key
     - Description
   * - ``type``
     - Specifies the ZeroMQ protocol to use, must be either ``zmq_ipc`` or ``zmq_tcp``.
   * - ``host``
     - TCP host.
   * - ``port``
     - TCP port.
   * - ``allowed_clients``
     - Specifies the list of public keys for the publishers which are allowed to connect.
   * - ``""``
     - Gives security configuration for the frontend TCP socket.
   * - ``server_secret_key``
     - Secret key for the frontend socket for encryption / authentication.


**IMPORTANT NOTE**

When using JSON files for the configuration of the ZeroMQ Broker, the JSON
file is the configuration used by the EII Message Bus, rather than the normal
EII interface configurations. This is why the JSON configuration differs
from the configuration given via the ``eii_config.json``. The JSON shown above
for the frontend, and the following backend configuration, represent how the
configuration manager alters that configuration to be in these forms.

**Backend:**

.. code-block:: javascript

   {
       // Which ZeroMQ protocol to use (TCP vs. IPC)
       "type": "zmq_tcp",

       // List of curve public keys for subscribers allowed to connect to the
       // backend socket
       "allowed_clients": ["y-G]27SC}{!mC4(]}=c=KH1M{x)Kd/{i%o]j7YT3"],

       // Specifies the configuration for the frontend socket
       "zmq_tcp_publish": {
           // TCP host
           "host": "0.0.0.0",

           // TCP port (note that this is a different port from the frontend)
           "port": 5569

           // Server secret key to use for encryption / authentication
           "server_secret_key": "qydUsM#PP4r<E*2]<]kbLRwk1IX:H^y{Gk56e:tc"
       }
   }

..

   *\ *NOTE:* This example configuration is stored in, "examples/tcp_backend_example.json",
   and can be used with the, "examples/configs/tcp_subscriber_with_security.json",
   EII Message Bus configuration file.


All of the keys above have the same purpose and meaning as for the frontend
configuration. Except, there is one key difference; instead of the empty string
key, the backend configuration has the ``zmq_tcp_publish`` key. This is because
the broker adopts the same configuration definitions as the EII Message Bus
ZeroMQ protocol plugin. The ZeroMQ protocol plugin uses the ``zmq_tcp_publish``
key for setting the TCP (host, port) combinations for all publishers under a
single message bus context. The broker uses the same configuration to keep it
consistent between the two entities.

Once the frontend and backend JSON configuration files have been created,
run the broker binary as follows:

.. code-block:: sh

   # IPC Example
   $ ./zmq-broker examples/ipc_frontend_example.json examples/ipc_backend_example.json

   # TCP Example
   $ ./zmq-broker examples/tcp_frontend_example.json examples/tcp_backend_example.json

.. note::  It is assumed the configurations are named, "frontend_config.json",
   and, "backend_config.json", respectively. These could be named anything the
   user wishes. For more examples of configurations, see the JSON files in the,
   "tests/configs/", directory.


Running Unit Tests
^^^^^^^^^^^^^^^^^^

If the ZeroMQ Broker was built with the ``-DWITH_TESTS=ON`` flag set for the
CMake command, then do the following to run the unit tests:

.. note::  The commands below assume you are in the, "build/", directory created
   when compiing the broker.


.. code-block:: sh

   # 1. Change directories into the tests/ directory
   $ cd tests/

   # 2. Execute the unit tests
   $ ./broker-tests

If you wish to run the unit tests with a different log level, then provide
one of the following log levels: ``DEBUG``\ , ``INFO``\ , ``WARN``\ , ``ERROR`` with the
command line argument ``--log-level``. An example of using the ``DEBUG`` log
level is shown below:

.. code-block:: sh

   $ ./broker-tests --log-level DEBUG
