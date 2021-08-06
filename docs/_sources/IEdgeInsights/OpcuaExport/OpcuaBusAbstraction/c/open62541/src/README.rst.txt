
How to Test from present working directory
==========================================

1. Pre-requisite
----------------

.. code-block:: sh

     make clean
     make build_safestring_lib

2. Testing with security enabled
--------------------------------


* Start publisher, publish and destroy

.. code-block:: sh

   make pub


* Start subscriber, subscribe and destroy

.. code-block:: sh

   make sub

3. Testing with security disabled
---------------------------------


* Start publisher, publish and destroy

.. code-block:: sh

   make pub_insecure


* Start subscriber, subscribe and destroy

.. code-block:: sh

   make sub_insecure

4. Remove all binaries/object files
-----------------------------------

.. code-block:: sh

   make clean
