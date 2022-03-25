
**Contents**


* `EII Utils <#eii-utils>`__

  * `Dependency Installation <#dependency-installation>`__
  * `Compilation <#compilation>`__
  * `Installation <#installation>`__
  * `Running Unit Tests <#running-unit-tests>`__

EII Utils
=========

EII Utils is a C library providing commonly used APIs across the C/C++ EII modules, EII Message Bus and Config Manager libraries.

Dependency Installation
-----------------------

The EIIUtils depends on CMake version 3.11+. For Ubuntu 18.04 this is not
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

To install the remaining dependencies for the EIIUtils execute the following
command:

.. code-block:: sh

   $ sudo -E ./install.sh

Additionally, EIIUtils depends on the below libraries. Follow their documentation to install them.


* `IntelSafeString <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_
* `EIIMsgEnv <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_

Compilation
-----------

The EII Utils utilizes CMake as the build tool for compiling the C
library. 

CMAKE_INSTALL_PREFIX needs to be set for the build and installation:

.. code-block:: sh

       $ export CMAKE_INSTALL_PREFIX="/opt/intel/eii"

The simplest sequence of commands for building the library are
shown below.

.. code-block:: sh

   $ mkdir build
   $ cd build
   $ cmake -DCMAKE_INSTALL_INCLUDEDIR=$CMAKE_INSTALL_PREFIX/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX ..
   $ make

If you wish to compile the EII Utils in debug mode, then you can set
the ``CMAKE_BUILD_TYPE`` to ``Debug`` when executing the ``cmake`` command (as shown
below).

.. code-block:: sh

   $ cmake -DCMAKE_INSTALL_INCLUDEDIR=$CMAKE_INSTALL_PREFIX/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX -DCMAKE_BUILD_TYPE=Debug ..

Installation
------------

.. note::  This is a mandatory step to use this library in
   C/C++ EII modules and EII Message Bus library.


If you wish to install the EII Utils on your system, execute the
following command after building the library:

.. code-block:: sh

   $ sudo make install

By default, this command will install the EII Utils C library into
``/opt/intel/eii/lib/``. On some platforms this is not included in the ``LD_LIBRARY_PATH``
by default. As a result, you must add this directory to you ``LD_LIBRARY_PATH``.
This can be accomplished with the following ``export``\ :

.. code-block:: sh

   $ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/eii/lib/

.. note::  You can also specify a different library prefix to CMake through
   the ``CMAKE_INSTALL_PREFIX`` flag. If different installation path is given via ``CMAKE_INSTALL_PREFIX``\ , then ``$LD_LIBRARY_PATH`` should be appended by $CMAKE_INSTALL_PREFIX/lib.


Running Unit Tests
------------------

.. note::  The unit tests will only be compiled if the ``WITH_TESTS=ON`` option
   is specified when running CMake.


Run the following commands from the ``build/tests`` folder to cover the unit
tests.

.. code-block:: sh

   $ ./config-tests
   $ ./frame-tests
   $ ./log-tests
   $ ./thp-tests
   $ ./tsp-tests
