
OpcuaBusAbstraction
===================

OpcuaBusAbstraction abstracts underlying messagebus to provide a common set of APIs needed for publish and subscribe.
The C example program demonstrates publish and subscription over OPCUA bus only (\ ``OPCUA is the only messagebus supported for now``\ )

How to Test from present working directory
------------------------------------------

1. Prerequisite
^^^^^^^^^^^^^^^

.. note:: 


   * In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights (OEI). This is due to the product name change of EII as OEI.
   * The ``Prerequisite`` section below is ``only`` needed if executing from the OEI repo.


.. code-block:: sh

     wget -q --show-progress https://tls.mbed.org/code/releases/mbedtls-2.16.6-gpl.tgz
     tar xf mbedtls-2.16.6-gpl.tgz
     cd mbedtls-2.16.6
     make install
     make clean
     make build_safestring_lib

2. Testing with security enabled
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* Start publisher, publish and destroy

.. code-block:: sh

   make pub


* Start subscriber, subscribe and destroy

.. code-block:: sh

   make sub

3. Testing with security disabled
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


* Start publisher, publish and destroy

.. code-block:: sh

   make pub_insecure


* Start subscriber, subscribe and destroy

.. code-block:: sh

   make sub_insecure

4. Remove all binaries/object files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: sh

   make clean
