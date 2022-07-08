
Contents
========


* `Contents <#contents>`__

  * `Azure Bridge Module <#azure-bridge-module>`__
  * `Running Unit Tests <#running-unit-tests>`__

Azure Bridge Module
-------------------

This directory contains the source code for the Azure Bridge Open EII service which bridges communication from the Message bus and the Azure IoT Edge Runtime. For more information on this service, see the top level README. The purpose of this README is to cover some specifics related to the code itself, and not the usage of the module in Open EII. Refer to the Open EII and Azure Bridge READMEs for more information.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights for Industrial (Open EII). This is due to the product name change of EII as Open EII.


Running Unit Tests
------------------

The Azure Bridge contains unit tests for various utility functions in the service. It does not contain unit tests for every single method, because most of it is not unit test-able, meaning, you must have a fully up and running Azure IoT Edge Runtime in order to run the code succesfully. Testing the bridge in this way can be accomplished via using the Azure IoT Edge Runtime simulator documented in the root directory of the Azure Bridge service.

To run the unit tests for the Azure Bridge, first install the Azure Bridge python dependencies:

.. note::  It is highly recommended that you use a python virtual environment to install the python packages, so that the system python installation doesn't get altered. Details on setting up and using python virtual environment can be found `here <https://www.geeksforgeeks.org/python-virtual-environment/>`_.


.. code-block:: sh

    sudo -H -E pip3 install -r requirements.txt

Next, set up your ``PYTHONPATH`` to contian the necessary Open EII Python libraries for the test:

.. note::  This can be skipped if you have installed the Open EII libraries on your system already. This step assumes none of the Open EII libraries for Python, Go, or C have been installed on your system.


.. code-block:: sh

   export PYTHONPATH=${PYTHONPATH}:../../../common:../../../common/libs/ConfigManager/python

Next, run the unit tests with the following Python command:

.. code-block:: sh

   python3 -m unittest discover

If everything runs successfully, you should see the following output:

.. code-block:: sh

   ..
   ----------------------------------------------------------------------
   Ran 2 tests in 0.001s

   OK
