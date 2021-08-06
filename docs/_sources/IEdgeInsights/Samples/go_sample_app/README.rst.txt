
The sample Golang application demonstrates the usage of EII core libraries like EIIMessageBus and ConfigManager
===============================================================================================================

Short Description of App containers
===================================

This is a Golang Application which uses the EII message bus and EII client side
libraries. In this app, there is a publisher, subscriber, client and server. 
Subscriber and client are running inside ia_go_subscriber container.
Publisher and server are running inside ia_go_publisher container.

The high level logical flow of Sample App is as below:
======================================================


#. 
   Publisher contacts to ETCD service, using the 'msgbusutil'
   (IEdgeInsights/common/util/msgbusutil) library. It fetches it's private key
   and allowed client's public keys. Then it creates the publisher object and
   starts publishing the sample data at given topic(topic name: publish_test).

#. 
   Server contacts to ETCD service, and fetches it's private key and
   allowed client's public keys. Then it creates a server object according
   to given service name and waits for the client's request.

#. 
   Subscriber contacts to ETCD service using the 'msgbusutil'
   (IEdgeInsights/common/util/msgbusutil) library. And fetches it's private
   and public key and public key of publisher. It then creates the subscriber
   object(which subscribes to the same topic: publish_test) and starts
   receiving the data and prints the same onto a console.

#. 
   Client contacts to ETCD service, and fetches its private key and public
   key of server. Then it creates a client object and sends a request to the
   server and gets the response back.
