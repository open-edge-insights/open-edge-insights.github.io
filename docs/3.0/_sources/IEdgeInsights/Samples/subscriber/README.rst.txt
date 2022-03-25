
Sample subscriber apps
======================

Subscriber apps consists of the following:

C++ subscriber
--------------

The C++ subscriber and client are supported in Ubuntu and Alpine operating systems or docker images. The C++ subscriber runs in the independent containers ``ia_ubuntu_cpp_sample_sub`` and ``ia_alpine_cpp_sample_sub`` respectively.

Go subscriber
-------------

The Go subscriber and client are supported in Ubuntu and Alpine operating systems or docker images.  The Go subscriber runs in the independent containers ``ia_ubuntu_go_sample_sub`` and ``ia_alpine_go_sample_sub`` respectively.

Python subscriber
-----------------

The Python subscriber and client are supported are supported only in Ubuntu operating systems or docker images. The Python subscriber runs in the independent container ``ia_ubuntu_python_sample_sub``.

.. note::  The subscriber containers comprises of both the subscriber and the client functionality.


High-level logical flow for sample subscriber apps
--------------------------------------------------

The high-level logical flow of a sample subscriber app is as below:


#. 
   Sample apps use the `ConfigMgr <https://github.com/open-edge-insights/eii-core/blob/master/common/libs/ConfigMgr/src/cfgmgr.c>`_ APIs to construct the ``msgbus`` config from the ETCD. It fetches its private and public key and the publisher's public key. It then creates the subscriber object (which subscribes to the same topic : publish_test) and start receiving the data and prints the same onto a console.

#. 
   Client contacts the ETCD service and fetches its private key and the server's public key. Then it creates a client object and sends a request to the server and receives the response back.
