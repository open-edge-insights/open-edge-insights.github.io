
**Contents**


* `EII Message Bus <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#eii-message-bus>`_

  * `Dependency Installation <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#dependency-installation>`_
  * `Compilation <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#compilation>`_

    * `Generating Documentation <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#generating-documentation>`_
    * `Potential Compilation Issues <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#potential-compilation-issues>`_

  * `Installation <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#installation>`_

    * `Install Python Binding <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#install-python-binding>`_
    * `Install Golang Binding <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#install-golang-binding>`_

  * `Running Unit Tests <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#running-unit-tests>`_
  * `Configuration <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#configuration>`_

    * `ZeroMQ IPC Configuration <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#zeromq-ipc-configuration>`_
    * `ZeroMQ TCP Configuration <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#zeromq-tcp-configuration>`_

      * `Publishers <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#publishers>`_
      * `Subscribers <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#subscribers>`_
      * `Services <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#services>`_
      * `Requesters <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#requesters>`_
      * `Using ZAP Authentication <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#using-zap-authentication>`_

    * `Additional ZeroMQ Configuration Properties <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#additional-zeromq-configuration-properties>`_

  * `Example Usage <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#example-usage>`_

    * `C Examples <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#c-examples>`_

      * `Publisher Many Example <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#publisher-many-example>`_

    * `Python Examples <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#python-examples>`_
    * `Go Examples <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#go-examples>`_
    * `Running Go Examples without Installing <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#running-go-examples-without-installing>`_
    * `Brokered Publish/Subscribe <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#brokered-publishsubscribe>`_

  * `Security <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#security>`_

    * `Using Only CurveZMQ Encryption <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#using-only-curvezmq-encryption>`_

      * `Publish/Subscribe <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#publishsubscribe>`_
      * `Request/Response <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#requestresponse>`_

    * `Using ZAP Authentication <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#using-zap-authentication-1>`_
    * `Disabling Security <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#disabling-security>`_

  * `Known issues <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#known-issues>`_

EII Message Bus
===============

Message bus used between containers inside of EII.

Dependency Installation
-----------------------

The EII Message Bus depends on CMake version 3.11+. For Ubuntu 18.04 this is not
the default version installed via ``apt-get``. To install the correct version
of CMake, execute the following commands:

.. code-block:: sh

   # Remove old CMake version
   $ sudo apt -y purge cmake
   $ sudo apt -y autoremove

   # Download CMake
   $ wget https://cmake.org/files/v3.15/cmake-3.15.0-Linux-x86_64.sh

   # Installation CMake
   $ sudo mkdir /opt/cmake
   $ sudo cmake-3.15.0-Linux-x86_64.sh --prefix=/opt/cmake --skip-license

   # Make the command available to all users
   $ sudo update-alternatives --install /usr/bin/cmake cmake /opt/cmake/bin/cmake 1 --force

To install the remaining dependencies for the message bus execute the following
command:

**Note**\ : It is highly recommended that you use a python virtual environment to
install the python packages, so that the system python installation doesn't
get altered. Details on setting up and using python virtual environment can
be found here: https://www.geeksforgeeks.org/python-virtual-environment/

.. code-block:: sh

   $ sudo -E ./install.sh

Additionally, EIIMessageBus depends on the below libraries. Follow their documentation to install them.


* `IntelSafeString <https://github.com/open-edge-insights/eii-c-utils/blob/master/IntelSafeString/README.md>`_
* `EIIUtils <https://github.com/open-edge-insights/eii-c-utils/blob/master/README.md>`_

If you wish to compile the Python binding as well, then run the ``install.sh``
script with the ``--cython`` flag (as shown below).

.. code-block:: sh

   $ sudo -E ./install.sh --cython

Compilation
-----------

The EII Message Bus utilizes CMake as the build tool for compiling the library.
The simplest sequence of commands for building the library are shown below.

.. code-block:: sh

   $ mkdir build
   $ cd build
   $ cmake ..
   $ make

This will compile only the C library for the EII Message Bus. If you wish to
build with the Python binding, then specify the ``WITH_PYTHON`` flag when
executing the ``cmake`` command (as shown below).

.. code-block:: sh

   $ cmake -DWITH_PYTHON=ON ..

If you wish to include installation of the Go binding with the installation of
the EII library, then specify the ``WITH_GO`` flag when executing the ``cmake``
command (as shown below).

.. code-block:: sh

   $ cmake -DWITH_GO=ON ..

Note that this only copies the Go binding library to your system's ``$GOPATH``.
If you do not have your ``$GOPATH`` specified in your system's environmental
variables then an error will occur while executing the ``cmake`` command.

In addition to the ``WITH_PYTHON`` and ``WITH_GO`` flags, the EII Message Bus
CMake files add flags for building the C examples and the unit tests associated
with the library. The table below specifies all of the available flags that can
be given to CMake for building the EII Message Bus.

.. list-table::
   :header-rows: 1

   * - Flag
     - Default
     - Description
   * - ``WITH_TESTS``
     - ``OFF``
     - If set to ``ON``\ , builds the C unit tests with the EII Message Bus compilation
   * - ``WITH_EXAMPLES``
     - ``OFF``
     - If set to ``ON``\ , then CMake will compile the C examples in addition to the library
   * - ``WITH_DOCS``
     - ``OFF``
     - If set to ``ON``\ , then CMake will add a ``docs`` build target to generate documentation


.. note::  These flags are in addition to any and all flags that are available
   for the ``cmake`` command. See the CMake documentation for additional flags.

   **NOTE:** See the `Generating Documentation <http://localhost:7645/IEdgeInsights/common/libs/EIIMessageBus/#generating-documentation>`_
   section.


If you wish to compile the EII Message Bus in debug mode, then you can set the
the ``CMAKE_BUILD_TYPE`` to ``Debug`` when executing the ``cmake`` command (as shown
below).

.. code-block:: sh

   $ cmake -DCMAKE_BUILD_TYPE=Debug ..

Generating Documentation
^^^^^^^^^^^^^^^^^^^^^^^^

Generating the documentation has several dependencies which are not installed
by the ``install.sh`` script. You must install the following packages in order
to generate the documentation:

.. code-block:: sh

   $ sudo apt install doxygen texlive-full

**WARNING:** This install way take a very long time. It will install > ``4GB`` of
packages.

If you are building the Python binding (by using the ``WITH_PYTHON`` flag), then
you must also install Sphinx and an extension for Sphinx. This can be
accomplished with the following commands:

.. code-block:: sh

   $ sudo apt install python3-sphinx
   $ sudo -H -E pip3 install m2r

.. note::  The commands above assume you already have Python 3.6 and pip
   installed on your system.


**Go documentation generation is WIP.**

Once you have completed these steps, the documentation can be generated by
running the following make command:

.. code-block:: sh

   $ make docs && make docs

Note that currently you need to run ``make docs`` twice so that the table of
contents is generated correctly for each of the documents. This will be fixed
in the future.

The PDF documents will be in the ``docs/pdfs/`` directory within your ``build``
directory. There will be other log files and output files associated with the
building of the PDFs. Any file that does not end in ``.pdf`` can be ignored.

Potential Compilation Issues
^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* **CMake Python Version Issue:** If CMake raises an error for finding the
    incorrect Python version, then add the following flag to the CMake
    command: ``-DPYTHON_EXECUTABLE=/usr/bin/python3``
* **Python Binding Changes Not Compiling:** If a change is made to the Python
    binding and make has already been previously ran, then you must run
    ``make clean`` before running make again to compile the changes in the
    Python binding. This will need to be fixed later.

Installation
------------

If you wish to install the EII Message Bus on your system, execute the
following command after building the library:

.. code-block:: sh

   $ sudo make install

By default, this command will install the EII Message Bus C library into
``/opt/intel/eii/lib``. On some platforms this is not included in the ``LD_LIBRARY_PATH``
by default. As a result, you must add this directory to you ``LD_LIBRARY_PATH``\ ,
otherwise you will encounter issues using the EII Message Bus. This can
be accomplished with the following ``export``\ :

.. code-block:: sh

   $ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/eii/lib

.. note::  You can also specify a different library prefix to CMake through
   the ``CMAKE_INSTALL_PREFIX`` flag. If different installation path is given via
   ``CMAKE_INSTALL_PREFIX``\ , then ``$LD_LIBRARY_PATH`` should be appended by
   $CMAKE_INSTALL_PREFIX/lib.


Install Python Binding
^^^^^^^^^^^^^^^^^^^^^^

To install the Python binding for the EII Message Bus execute the following
commands:

.. code-block:: sh

   # Change directories into the python/ directory
   $ cd python/

   # Install the Python package
   $ sudo python3 setup.py install

.. note::  In order for the installation to be successful, you must have run
   the ``install.sh`` script with the ``--cython`` flag when installing the
   message bus dependencies.


Install Golang Binding
^^^^^^^^^^^^^^^^^^^^^^

To install the Golang binding for the EII Message Bus execute the following
command:

.. code-block:: sh

   # Copy the Golang source to your $GOPATH/src directory
   $ cp -a go/EIIMessageBus/ $GOPATH/src/

.. note::  The above command assumes Golang is installed and configured on
   the target system.


Running Unit Tests
------------------

.. note::  The unit tests will only be compiled if the ``WITH_TESTS=ON`` option
   is specified when running CMake.


Execute one of the following commands from the ``build/tests`` folder to execute
the message bus unit tests.

.. code-block:: sh

   $ ./msgbus-tests
   $ ./msg-envelope-tests
   $ ./crc32-tests

It is important to note that the ``msgbus-tests`` executable has an extra CLI
option which the other unit test binaries do not have. This option allows for
running the message bus tests to run over TCP rather than the IPC.

To run the message bus tests over TCP, execute the following command:

.. code-block:: sh

   $ ./msgbus-tests --tcp

Configuration
-------------

The EII Message Bus is configured through a ``key, value`` pair interface. The
values can be objects, arrays, integers, floating point, boolean, or strings.
The keys that are required to be available in the configuration are largly
determined by the underlying protocol which the message bus will use. The
protocol is specified via the ``type`` key and currently must be one of the
following:


* ``zmq_ipc`` - ZeroMQ over IPC protocol
* ``zmq_tcp`` - ZeroMQ over TCP protocol

The following sections specify the configuration attributes expected for the
TCP and IPC ZeroMQ protocols.

ZeroMQ IPC Configuration
^^^^^^^^^^^^^^^^^^^^^^^^

The ZeroMQ IPC protocol implementation only requires one configuration
attribute: ``socket_dir``. The value of this attribute specifies the directory
where the message bus should create the Unix socket files to establish the IPC
based communication.

The ZeroMQ IPC protocol also supports a few optional configuration properties.
For all communication patterns (request/response and publisher/subscribe), the
``socket_file`` property can be specified for any service name or publish/subscribe
topic. This makes it so that all communication happens over the specified socket
file, instead of the default, which is for the message bus to use the topic/service
name for the socket file name. This enables two specific use cases:


#. Having a single application publish multiple topics over a single IPC socket
    file
#. Connecting ZeroMQ IPC publishers to the ZeroMQ Broker

For the second use case, there is an additional property which must be specified;
the ``brokered`` configuration value. This is must be a boolean value, with ``true``
telling the publisher to connect to the broker and ``false`` for not connecting.
Note that this value is not required. If it is ``false`` or is not specified, then
the publisher will not connect to a broker and will bind to the given ``socket_file``
value.

**IMPORTANT NOTE:**

The ``socket_file`` value should only be the name of a file. The socket itself
will still be created in the specified ``socket_dir`` directory.

Also, both the ``socket_file`` and ``brokered`` values must be specified in the
configuration under an object with its key being the topic or service name.
An example of this is shown below.

.. code-block:: javascript

   {
       // Specifies the ZeroMQ IPC protocol
       "type": "zmq_ipc",

       // Specifies the socket directory
       "socket_dir": "/tmp/socks",

       // The key, "example-topic", is the topic which the publisher shall publish
       // on. This could also be a service name.
       "example-topic": {
           // For the example-topic, the below property specifies the socket file
           // to use
           "socket_file": "socket-filename",

           // The below property is only supported by publishers, this specifies
           // that the given publisher will be brokered through the ZeroMQ Broker
           "brokered": true
       }
   }

The example above uses JSON to represent the EII Message Bus configuration.
See the, "Examples", below section for more details and more examples.

ZeroMQ TCP Configuration
^^^^^^^^^^^^^^^^^^^^^^^^

The ZeroMQ TCP protocol has several configuration attributes which must be
specified based on the communication pattern the application is using and
based on the security the application wishes to enable for its communication.

Publishers
~~~~~~~~~~

For an application which wishes to publish messages over specific topics, the
configuration must contain the key ``zmq_tcp_publish``. This attribute must be
an object which has the following keys:

.. list-table::
   :header-rows: 1

   * - Key
     - Type
     - Required
     - Description
   * - ``host``
     - ``string``
     - Yes
     - Specifies the host to publish as
   * - ``port``
     - ``int``
     - Yes
     - Specifies the port to publish messages on
   * - ``server_secret_key``
     - ``string``
     - No
     - Specifies the secret key for the port for authentication
   * - ``brokered``
     - ``boolean``
     - No
     - Specifies whether or not to connect to the ZeroMQ Broker


The ``server_secret_key`` must be a Curve Z85 encoded string value that is
specified if the application wishes to use CurveZMQ authentication with to
secure incoming connections from subscribers.

The ``brokered`` key must be a boolean value. This will determine whether or not
the publishers attempt to bind or connect to the given  TCP (host, port)
combination.

Subscribers
~~~~~~~~~~~

To subscribe to messages coming from a publisher over TCP, the configuration
must contain a key for the topic you wish to subscribe to. For example, if
*Application 1* were publishing on topic ``sensor-1``\ , then the subscribing
application *Application 2* would need to contain a configuration key ``sensor-1``
which contains the keys required to configure the TCP connection to
*Application 1*.

The key that can be specified for a subscribers configuration are outline in the
table below.

.. list-table::
   :header-rows: 1

   * - Key
     - Type
     - Required
     - Description
   * - ``host``
     - ``string``
     - Yes
     - Specifies the host of the publisher
   * - ``port``
     - ``int``
     - Yes
     - Specifies the port of the publisher
   * - ``server_public_key``
     - ``string``
     - No
     - Specifies the publisher's public key for authentication
   * - ``client_secret_key``
     - ``string``
     - No
     - Specifies the subscribers's secret key for authentication
   * - ``client_public_key``
     - ``string``
     - No
     - Specifies the subcribers's public key for authentication


.. note::  If one of the ``*_key`` values is specifed, then all of them must be
   specified.


Services
~~~~~~~~

The configuration to host a service to receive and respond to requests is
similar to the configuration for doing publications on a message bus context.
The only difference, is the configuration for a service is placed under a key
which is the name of the service.

For example, if *Application 1* wishes to host a service named ``example-service``\ ,
then the configuration must contain an a key called ``example-service``. The value
for that key must be an object containing the keys listed in the table of
the Publishers section.

Requesters
~~~~~~~~~~

The configuration to issue requests to a service is the exact same as a
subscriber. In the case of a requester, instead of the configuration being
under the name of the topic, the configuration is placed under the name of
the service it wishes to connect to. For the details of the allowed values,
see the table in the Subscribers section above.

Using ZAP Authentication
~~~~~~~~~~~~~~~~~~~~~~~~

For services and publishers additional security can be enabled for all incoming
connections (i.e. requesters and subscribers). This method utilizes the ZMQ
ZAP protocol to verify the incoming client public keys against a list of
whitelisted clients.

The list of allowed clients is given to the message bus via the ``allowed_clients``
key. This key must be a list of Z85 encoded CurveZMQ keys.

Additional ZeroMQ Configuration Properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The configuration interface for the ZeroMQ protocol exposes additional socket
properties. The table below specifies each of the supported properties.

.. list-table::
   :header-rows: 1

   * - Configuration
     - Type
     - Default
     - Description
   * - ``zmq_recv_hwm``
     - ``int``
     - ``1000``
     - Sets ``ZMQ_RCVHWM`` socket property (queue size for pending received messages)
   * - ``zmq_connect_retries``
     - ``int``
     - ``1000``
     - Sets number of connect failures before recreating ZMQ socket object


Example Usage
-------------

..

   **IMPORTANT NOTE:** Some of the example configurations contain public/private
   keys for the purpose of show how to use the message bus with security enabled.
   THESE KEYS SHOULD **NEVER** BE USED IN PRODUCTION.

   **NOTE:** The examples will only be compiled if the ``WITH_EXAMPLES=ON`` option
   is set when CMake is executed during compilation.


All of the examples provided for the EII Message Bus use a JSON configuration
file to configure the EII Message Bus. There are several example configurations
provided with the message bus for running in IPC and TCP mode accross the
various different messaging patterns (i.e. Publish/Subscribe and Request/Response).
All of these example configurations are in the ``examples/configs/`` directory.
However, all of them are copied into the ``examples/build/configs/`` directory
as well when build the examples.

The table below specifies all of the provided example configurations.

.. list-table::
   :header-rows: 1

   * - Configuration
     - Description
   * - ipc_example_config.json
     - Configuration for IPC based communication. Works with all examples.
   * - ipc_example_config_multi_topics.json
     - Configuration for IPC based communication to be used with multi-topic publishing/subscribing. Works with publisher-many & subscriber examples.
   * - ipc_publisher_brokered.json
     - Publisher configuration for IPC based communication using the ZeroMQ Broker.
   * - ipc_subscriber_brokered.json
     - Subscriber configuration for IPC based communication using the ZeroMQ Broker.
   * - tcp_publisher_brokered_no_security.json
     - TCP configuration for publishing with no security through the ZeroMQ Broker.
   * - tcp_publisher_no_security.json
     - TCP configuration for publishing with no security.
   * - tcp_publisher_with_security_no_auth.json
     - TCP configuration for publishing with key based auth without ZAP auth.
   * - tcp_publisher_with_security_with_auth.json
     - TCP configuration for publishing with key based auth and ZAP auth.
   * - tcp_subscriber_no_security.json
     - TCP configuration for subscribing to a topic with no security.
   * - tcp_subscriber_no_security_prefix_match.json
     - TCP configuration for subscribing to multiple topics which share a common prefix, with no security.
   * - tcp_subscriber_with_security.json
     - TCP configuration for subscribing to a topic with security enabled.
   * - tcp_subscriber_with_security_prefix_match.json
     - TCP configuration for subscribing to multiple topics which share a common prefix, with security.
   * - tcp_service_server_no_security.json
     - TCP configuration for a service server side (i.e. ``echo-service``\ ) without security.
   * - tcp_service_server_with_security_no_auth.json
     - TCP configuration for a service server side with key based auth without ZAP auth.
   * - tcp_service_server_with_security_with_auth.json
     - TCP configuration for a service server side with key based auth and ZAP auth.
   * - tcp_service_client_no_security.json
     - TCP configuration for a service client side (i.e. ``echo-client``\ ) with no security.
   * - tcp_service_client_with_security.json
     - TCP configuration for a service client side with security enabled.


.. note::  When using the brokered examples, you must also launch the broker
   first. For more information on the broker and how to use it, see the
   ZmqBroker/README.md.


You will notice that for the publisher configurations and service server side
configurations there are 3 configurations each, where as subscribers and service
client side configurations only have 2. This is because for publishers and service
server side applications there are two forms of security to enable: with ZAP authentication,
and no ZAP authentication. In the configurations with ZAP authentication, an
additional configuration value is provided which specifies the list of clients
(i.e. subscribers or service client side connections) which are allowed to connect
to the specified port. This list oporates as a whitelist of allowed client public
keys. If a connection is attempted with a key not in that list, then the connection
is denied.

C Examples
^^^^^^^^^^

There are currently 5 C examples:


#. ``examples/publisher.c``
#. ``examples/subscriber.c``
#. ``examples/echo_server.c``
#. ``examples/echo_client.c``
#. ``examples/publisher_many.c``

All of the C example executables are in the ``build/examples/`` directory. To run
them, execute the following command:

.. code-block:: sh

   $ ./publisher ./configs/ipc_example_config.json

.. note::  The ``tcp_example_config.json`` can also be used in lieu of the IPC
   configuration file.


All of the examples follow the command structure above, i.e.
``<command> <json-config-file>.json``\ , except for the ``publisher_many.c``
example. This example is explained more in-depth in the next section.

Publisher Many Example
~~~~~~~~~~~~~~~~~~~~~~

The ``examples/publisher_many.c`` example serves as a reference for implementing
an application which contains many publishers. This also serves as a way of
testing this functionality in the EII Message Bus.

The example can be run with the following command (from the ``build/examples/``
directory):

.. code-block:: sh

   $ ./publisher-many ./configs/ipc_example_config.json 5

In the case above, the example will create 5 publishers where the topic
strings follow the pattern ``pub-{0,N-1}`` where ``N`` is the number of publishers
specified through the CLI. Can replace this JSON config file with any other JSON
config as mention in the table above.

The behavior of how these topics are published depends on if the configuration
is IPC or TCP (i.e. if ``type`` is set to ``zmq_ipc`` vs. ``zmq_tcp`` in the JSON
configuration file).

If IPC communication is being used, then each topic will be a different Unix
socket file in the ``socket_dir`` directory specified in the configuration, if default IPC config file ``ipc_example_config.json`` is used. If the IPC config file ``ipc_example_config_multi_topics.json`` is used then each topic is published over
the socket file that is mentioned in the configuration file. For example, in the
default file ``ipc_example_config_multi_topics.json``\ , all topics publish & subscribe over the same socket file "multi-topics". However, we can have each topic or a set of topics publish/subscribe over a different socket file.

If TCP communication is being used, then each message will be published over
the ``host`` and ``port`` specified under the ``zmq_tcp_publish`` JSON object in the
configuration.

In order to subscribe to the topics published by this example, use the
``subscriber.c`` example. If you are using TCP or even IPC with multiple topics subscription, then you will need to specify the topic in your configuration. For example, your JSON configuration will
need to contain the following to subscribe to the ``pub-0`` topic:

.. code-block:: json

   {
       "pub-0": {
           "host": "127.0.0.1",
           "port": 5569
       }

.. note::  The ``host`` and ``port`` are assumed above, they may be different.


In order to simplify the creation of the configuration for subscribing to
topics over TCP, the ``gen_tcp_sub_conf.py`` helper script is provided. This
Python script will generate a JSON file for you based on your TCP JSON
configuration for the ``publisher-many`` example which contains all of the
topics specified so you can subscribe to any of them.

This helper script can be ran as follows:

.. code-block:: sh

   $ python3 ./gen_tcp_sub_conf.py <CONFIG-FILE-PATH>/tcp_publisher_no_security.json output.json 5

The command above uses the ``tcp_publisher_no_security.json`` for the ``publisher-many``
configuration. Then it generates all 5 topics and outputs them into the
``output.json`` file.

After generating this configuration, you can use the ``subscriber.c`` example as
shown below to subscribe to the ``pub-1`` topic:

.. code-block:: sh

   $ ./subscriber output.json pub-1

Similiarly for IPC mode of communicatin with multi topics, the sample JSON configuration would look like below:

.. code-block:: json

       {
       "type": "zmq_ipc",
       "socket_dir": "${CMAKE_CURRENT_BINARY_DIR}/.socks",
       "pub-0": {
           "socket_file": "multi-topics"
       },
       "pub-1": {
          "socket_file": "multi-topics"
       },
       "pub-": {
          "socket_file": "multi-topics"
       }
   }

Here, ``pub-0`` & ``pub-1`` are the PUB topics & ``pub-`` is the SUB topics, where we have given just the prefix name. If we don't intend to give the SUB topic prefix, we can as well give the entire SUB topic name. In this example all these topics communicate over a common socket file ``multi-topics``.

Python Examples
^^^^^^^^^^^^^^^

.. note::  The Python examples will only be present if the ``WITH_EXAMPLES=ON``
   and ``WITH_PYTHON=ON`` flags are set when CMake is executed during compilation.


There are currently 4 Python examples:


#. ``python/examples/publisher.py``
#. ``python/examples/subscriber.py``
#. ``python/examples/echo_server.py``
#. ``python/examples/echo_client.py``

To run the Python examples, go to the ``build/examples/`` directory. Then source
the ``source.sh`` script that is in the examples directory.

.. code-block:: sh

   $ source ./source.sh

Then, execute one of the following commands:

.. code-block:: sh

   $ python3 ./publisher.py <CONFIG-FILE-PATH>/ipc_example_config.json

.. note::  The ``tcp_example_config.json`` can also be used in lieu of the IPC
   configuration file.


All of the examples follow the same command structure as the ``publisher.py``
script, i.e. ``python3 <python-script>.py <json-config-file>.json``.

Go Examples
^^^^^^^^^^^

..

   **IMPORANT NOTE:** It is assumed that when compiling the C library prior to
   running the examples that the ``WITH_GO=ON`` flag was specified when executing
   the ``cmake`` command. It is also assume that, ``sudo make install`` has been
   ran. If it has not and you do not wish to install the library, see the
   "Running Go Examples without Installing" section below.


When the ``sudo make install`` command is executed on your system, the Go binding
will be copied to your system's ``$GOPATH``. To execute the examples provided
with the EII Message Bus Go binding go to the ``$GOPATH/src/EIIMessageBus/examples``
directory on your system in a terminal window.

Once you are in this directory choose an example (i.e. publisher, subscriber,
etc.) and ``cd`` into that directory. Then, to run the example execute the
following command:

.. code-block:: sh

   $ go run main.go -configFile <CONFIG-FILE>.json -topic publish_test

The example command above will run either the subscriber or publisher examples.
For the echo-client and echo-server examples the ``-topic`` flag should be
``-serviceName``.

Additionally, there are example configurations provided in the
``build/examples/configs/`` directory after building the EII Message Bus library.

Running Go Examples without Installing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you wish to run the Go binding examples with out installing the EII Message
Bus library, then this can be accomplished by either copying or creating a
soft-link to the ``go/EIIMessageBus`` directory in your ``$GOPATH``. This can be
accomplished with one of the commands shown below.

.. code-block:: sh

   $ cp -r go/EIIMessageBus/ $GOPATH/src

   # OR

   $ ln -s go/EIIMessageBus/ $GOPATH/src

.. note::  The command above assumes that you are currently in the
   EIIMessageBus source root directory.


Since it is assumed you have not ran the ``sudo make install`` command to install
the EII Message Bus library, you must set the environmental variables specified
below prior to running the examples.

.. code-block:: sh

   $ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$MSGBUS_DIR/build
   $ export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:$MSGBUS_DIR/build

Note that in the ``export`` commands above the ``$MSGBUS_DIR`` variable represents
the absolute path to the ``libs/EIIMessageBus`` directory. It is very important
that this is the absolute path.

Once you have exported these variables, once you have done these steps, you can
run any of the Go examples as specified in the previous section.

Brokered Publish/Subscribe
^^^^^^^^^^^^^^^^^^^^^^^^^^

EII provides a ZeroMQ Broker. Any of the publisher and subscriber examples can
be used with this broker. There are three example JSON configuration provided
to showcase the required configuration of the publishers/subscribers to connect
to the broker. These examples are listed below:


#. ``ipc_publisher_brokered.json`` - IPC brokered publisher configuration
#. ``ipc_subscriber_brokered.json`` - IPC brokered subscriber configuration
#. ``tcp_publisher_brokered_no_security.json`` - TCP brokered publisher configuration

Note that there is not TCP brokered subscriber configuration. This is because
no configuration change is needed to connect a subscriber to the broker.

For the publisher side of the configuration, it is important to note that in the
IPC configuration, under the ``publish-test`` JSON object the configuration key
``brokered`` is set to ``true``. Similarly, in the TCP configuration, under the
``zmq_tcp_publish`` JSON object the key ``brokered`` is also set to true. This
tells the publisher to connect to the broker (vs. binding to the given socket
configuration).

Since there is no code change to use a brokered publisher or subscriber, the
examples can be ran the same as before, just with these brokered configurations.
An example of this is shown below:

**IPC Example:**

.. code-block:: sh

   # Start the publisher
   $ ./publisher ./configs/ipc_publisher_brokered.json

   # Start the subscriber
   $ ./subscriber ./configs/ipc_subscriber_brokered.json

.. note::  These configurations work with the, "examples/ipc_frontend_example.json",
   and, "examples/ipc_backend_example.json", examples provided with the EII
   ZeroMQ Broker.


**TCP Example:**

.. code-block:: sh

   # Start the publisher
   $ ./publisher ./configs/tcp_publisher_brokered_with_security.json

   # Start the subscriber
   $ ./subscriber ./configs/tcp_subscriber_with_security.json

.. note::  These configurations work with the, "examples/tcp_frontend_example.json",
   and, "examples/tcp_backend_example.json", examples provided with the EII
   ZeroMQ Broker.


**IMPORTANT NOTE:**

Before runnint the examples below, you must start the ZeroMQ Broker. See
the ZmqBroker/README.md for more details on configuring/launching the broker.
Keep in mind that your broker configuration will impact the configuration
needed for these examples.

For example, in the ``ipc_publisher_brokered.json`` configuration file, if the
socket for publishers to connect to is not named ``frontend-sock`` then this
configuration file needs to be changed to reflect the different socket file
name.

To make this easy, the ZeroMQ Broker provides example configurations for TCP
and IPC which use the same socket directory / files and (host, port) combinations
to easily try out this feature.

Security
--------

..

   **IMPORTANT NOTE:** Security is only available for TCP communications. If IPC
   is being used, then all access must be controlled using Linux file
   permissions.

   **NOTE:** Example configurations using for enabling security in the examples
   are provided in the ``examples`` directory.


The ZeroMQ protocol for the EII Message Bus enables to usage of
`CurveZMQ <http://curvezmq.org/>`_ for encryption and authentication where the
`ZAP <https://rfc.zeromq.org/spec:27/ZAP/>`_ protocol is used for the
authentication.

The ZeroMQ protocol for the message bus allows for using both CurveZMQ and ZAP
together, only CurveZMQ encryption, or no encryption/authentication for TCP
communication.

Enabling the security features is done through the configuration object which
is given to the ``msgbus_initialize()`` method. The example configurations below
showcase how to use the security features enabled in the message bus. It is
important to note that although the examples below use JSON to convey the
configurations it is not required that you use a JSON configuration for the
message bus. However, utilities are provided in the C library for the EII
message bus for using a JSON file to configure the bus.

Using Only CurveZMQ Encryption
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you wish to use the message bus with only CurveZMQ encryption, then you
specify the following keys for the communication types specified in the
sections below.

**IMPORTANT NOTE:** All keys must be Z85 encoded (see ZeroMQ documentation for
more information).

Publish/Subscribe
~~~~~~~~~~~~~~~~~

For publications over TCP, the configuration must contain a ``server_secret_key``
value which the secret key of the Curve key pair that is Z85 encoded (see
the ZeroMQ documentation for more information).

Additionally, every subscriber configuration object (which is specified under
the key for the topic it is subscribing to) must contain the following three
keys: ``server_public_key``\ , ``client_public_key``\ , and ``client_secret_key``.

**Example:**

Below is an example configuration in JSON (note: the keys are not Z85 encoded,
but are more clear text to help the example).

**Publisher Config:**

.. code-block:: json

   {
       "type": "zmq_tcp",
       "zmq_tcp_publish": {
           "host": "127.0.0.1",
           "port": 3000,
           "server_secret_key": "publishers-secret-key"
       }
   }

**Subscriber Config:**

.. code-block:: json

   {
       "type": "zmq_tcp",
       "pub-sub-topic": {
           "host": "127.0.0.1",
           "port": 3000,
           "server_public_key": "publishers-public-key",
           "client_secret_key": "subscriber-secret-key",
           "client_public_key": "subscriber-public-key"
       }
   }

In the example configurations above, it is assumed that the publisher is
sending messages on the ``pub-sub-topic`` topic.

Request/Response
~~~~~~~~~~~~~~~~

For every service which is going to accept and respond to requests, there must
exist the ``server_secret_key`` in the configuration object for the service. The
key for the configuration of the service is its service name.

For every service which is going to issue requests to another service, there
must exist a configuration object for the destination service name which
contains the following three keys: ``server_public_key``\ , ``client_public_key``\ ,
and ``client_secret_key``.

**Example:**

**Service Config:**

.. code-block:: json

   {
       "type": "zmq_tcp",
       "example-service": {
           "host": "127.0.0.1",
           "port": 3000,
           "server_secret_key": "service-secret-key"
       }
   }

**Service Requester Config:**

.. code-block:: json

   {
       "type": "zmq_tcp",
       "example-service": {
           "host": "127.0.0.1",
           "port": 3000,
           "server_public_key": "service-public-key",
           "client_secret_key": "service-requester-secret_key",
           "client_public_key": "service-requester-public-key"
       }
   }

In the example above, the service requester will connect to the ``example-service``
and issue requests to it on the port: ``127.0.0.1:3000``.

Using ZAP Authentication
^^^^^^^^^^^^^^^^^^^^^^^^

To enable ZAP authentication protocol using CurveZMQ on top of the encryption,
then in the configuration specify the key ``allowed_clients``. This key must have
a value which is a list of Z85 encoded strings which are the public keys of the
clients which are allowed to connect to the application.

For example, using the publish/subscribe example from before, to make it so
that only the subscriber client can connect to the publisher the publisher's
configuration would be modified to be the following:

.. code-block:: json

   {
       "type": "zmq_tcp",
       "allowed_clients": ["subscriber-public-key"],
       "zmq_tcp_publish": {
           "host": "127.0.0.1",
           "port": 3000,
           "server_secret_key": "publishers-secret-key"
       }
   }

Disabling Security
^^^^^^^^^^^^^^^^^^

To disable all encryption and authentication for TCP communication do not
specify any of the configuration keys documented above. This will cause the
message bus to initialize the ZeroMQ protocol without any of the CurveZMQ
security primitives.

Known issues
------------

Due to certain limitations imposed by cJSON, there is no proper distinction
between an integer and a floating point in **EIIMsgEnv**. As a result of this limitation,
the floating point values defined as whole numbers(1.0, 50.00 etc) are always deserialized
as integers(1, 50 etc) on the subscriber's end in C, Python & Go APIs.

The workaround for this limitation in the C APIs is to check for the type of
``msg_envelope_elem_body_t`` struct before accessing the respective type's data. One such example
is provided below:

.. code-block:: c

   msg_envelope_elem_body_t* data;
   msgbus_ret_t ret = msgbus_msg_envelope_get(msg, "key", &data);
   if (ret != MSG_SUCCESS) {
       LOG_ERROR_0("Failed to retreive message");
   }
   if (data->type == MSG_ENV_DT_INT) {
       LOG_INFO("Received integer: %d", data->body.integer);
   } else if (data->type == MSG_ENV_DT_FLOATING) {
       LOG_INFO("Received float: %f", data->body.floating);
   }
