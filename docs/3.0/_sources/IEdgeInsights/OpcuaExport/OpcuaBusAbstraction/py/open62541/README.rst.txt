.. role:: raw-html-m2r(raw)
   :format: html


Compilation steps (current directory: :raw-html-m2r:`<repo>`\ /OpcuaBusAbstraction/py/open62541/)
===================================================================================================

For building the open62541 wrapper library ``open62541W.so`` and generating ``open62541W.c``
----------------------------------------------------------------------------------------------------

.. code-block:: sh

   make build

Start server, publish and destroy
=================================

.. code-block:: sh

   make server

Start client, subscribe and destroy
===================================

.. code-block:: sh

   make client

Remove all binaries/object files
================================

.. code-block:: sh

   make clean
