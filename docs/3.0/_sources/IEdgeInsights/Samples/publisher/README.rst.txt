
Sample publisher apps
=====================

Publisher apps consists of the following:

C++ publisher
-------------

The C++ publisher and server are supported in Ubuntu and Alpine operating systems or docker images. The C++ publisher runs in the independent containers ``ia_ubuntu_cpp_sample_pub`` and ``ia_alpine_cpp_sample_pub`` respectively.

Go publisher
------------

The Go publisher and server are supported in Ubuntu and Alpine operating systems or docker images. The Go publisher runs in the independent containers ``ia_ubuntu_go_sample_pub`` and ``ia_alpine_go_sample_pub`` respectively.

Python publisher
----------------

The Python publisher and server are supported only in Ubuntu operating systems or docker images. The Python publisher will run in the independent containers ``ia_ubuntu_python_sample_pub``.

.. note::  The sample publisher containers comprises of both the publisher and the server functionality.


High-level logical flow for sample publisher apps
-------------------------------------------------

The high-level logical flow of a sample publisher app is as follows:


#. 
   Sample apps use the `ConfigMgr <https://github.com/open-edge-insights/eii-core/blob/master/common/libs/ConfigMgr/src/cfgmgr.c>`_ APIs to construct the ``msgbus`` config from the ETCD. It fetches its private key and the allowed client's public keys. Then it creates the publisher object and starts publishing the sample data at a given topic (topic name: publish_test).

#. 
   The server contacts the ETCD service and fetches its private key and the allowed client's public keys. Then it creates a server object according to the given service name and waits for the client's request.
