
OpcuaBusAbstraction
===================

OpcuaBusAbstraction abstracts underlying messagebus to provide a common set of APIs needed for publish and subscribe.

The python example program demonstrates publish and subscription over OPCUA bus only (\ ``OPCUA is the only messagebus supported for now``\ )

Dependencies
------------


* python3 (for python) installed
* 
  Install databus_requirements.txt (for python) by running cmd: ``sudo -H pip3.6 install -r  databus_requirements.txt``

  **Note**\ : It is highly recommended that you use a python virtual environment to
  install the python packages, so that the system python installation doesn't
  get altered. Details on setting up and using python virtual environment can
  be found here: https://www.geeksforgeeks.org/python-virtual-environment/

* 
  Install open62541 library dependencies (mbedTLS, python dev):

  .. code-block:: sh

     sudo apt-get install -y python3-dev
     wget -q --show-progress https://tls.mbed.org/code/releases/mbedtls-2.16.6-gpl.tgz && \
     tar xf mbedtls-2.16.6-gpl.tgz && \
     cd mbedtls-2.16.6 && \
     make install

  ..

     **NOTE**\ : If ``OpcuaBusAbstraction`` module is referred from dist_libs path, make sure that the ``sub`` client is also run on Ubuntu > 18.04 as the dist libs package is been created by a container with ubuntu 18.04 as the base image. If working from IEI repo,
     one shouldn't be bothered about this


* 
  Set ``PYTHONPATH`` env variable to OpcuaBusAbstraction/py folder path


  * 
    If executing from IEI repo path, set it as below:

    .. code-block:: sh

       export PYTHONPATH=..:../../../../common/

  * 
    If executing from IEI dist libs path (/opt/intel/iei/dist_libs). set it as below:

    .. code-block:: sh

       export PYTHONPATH=..


* Copying certs and keys:

  * Copy opcua client cert and key to /etc/ssl/opcua
  * Copy CA cert to /etc/ssl/ca

How to Test from present working directory
------------------------------------------

1. Pre-requisite
^^^^^^^^^^^^^^^^

.. note:: \ :
   The ``Pre-requisite`` section below is ``only`` needed if executing from
   the IEI repo. It should not be run from IEI dist libs path
   (/opt/intel/iei/dist_libs). If run, it would fail and one may have to re-create
   dist_libs to overcome any issue thereafter.


.. code-block:: sh

     wget -q --show-progress https://tls.mbed.org/code/releases/mbedtls-2.16.6-gpl.tgz && \
     tar xf mbedtls-2.16.6-gpl.tgz && \
     cd mbedtls-2.16.6 && \
     make install
     make clean
     make build_safestring_lib
     make build

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

5. To view the documentation navigate to the below link
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: sh

   /opt/intel/iei/dist_libs/OpcuaBusAbstraction/py/doc/html/index.html
