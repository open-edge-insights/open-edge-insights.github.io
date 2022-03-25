
Contents
========


* `Contents <#contents>`__

  * `OEI Utils <#oei-utils>`__
  * `Dependency Installation <#dependency-installation>`__
  * `Compilation <#compilation>`__
  * `Packaging <#packaging>`__

    * `A Note on Alpine APK Packaging <#a-note-on-alpine-apk-packaging>`__

  * `Installation <#installation>`__
  * `Linking <#linking>`__
  * `Running Unit Tests <#running-unit-tests>`__

OEI Utils
---------

Open Edge Insights (OEI) Utils is a C library providing commonly used APIs across the C/C++ OEI modules, Message Bus, and Config Manager libraries.

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as OEI. This is due to the product name change of EII as OEI.


Dependency Installation
-----------------------

The OEI Utils depends on CMake version 3.11+. For Ubuntu 18.04 this is not the default version installed via ``apt-get``. To install the correct version of CMake, execute the following commands:

.. code-block:: sh

   # Remove old CMake version
   sudo apt -y purge cmake
   sudo apt -y autoremove

   # Download CMake
   wget https://cmake.org/files/v3.15/cmake-3.15.0-Linux-x86_64.sh

   # Installation CMake
   sudo mkdir /opt/cmake
   sudo cmake-3.15.0-Linux-x86_64.sh --prefix=/opt/cmake --skip-license

   # Make the command available to all users
   sudo update-alternatives --install /usr/bin/cmake cmake /opt/cmake/bin/cmake 1 --force

To install the remaining dependencies for the OEI Utils execute the following
command:

.. code-block:: sh

   sudo apt install libcjson-dev

.. note::  For Fedora, the package installed should be ``cjson-devel`` and for
   Alpine it is ``cjson-dev``.


Compilation
-----------

The OEI Utils utilizes CMake as the build tool for compiling the C library.

CMAKE_INSTALL_PREFIX needs to be set for the build and installation:

.. code-block:: sh

   export CMAKE_INSTALL_PREFIX="/opt/intel/eii"

The simplest sequence of commands for building the library are shown below.

.. code-block:: sh

   mkdir build
   cd build
   cmake -DCMAKE_INSTALL_INCLUDEDIR=$CMAKE_INSTALL_PREFIX/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX ..
   make

If you wish to compile the OEI Utils in debug mode, then you can set
the ``CMAKE_BUILD_TYPE`` to ``Debug`` when executing the ``cmake`` command (as shown
below).

.. code-block:: sh

   cmake -DCMAKE_INSTALL_INCLUDEDIR=$CMAKE_INSTALL_PREFIX/include -DCMAKE_INSTALL_PREFIX=$CMAKE_INSTALL_PREFIX -DCMAKE_BUILD_TYPE=Debug ..

Packaging
---------

This library supports being packaged as a Debian, RPM, or Alpine APK packages.
This is all accomplished via CMake. By default, packaging is disabled. To
enable packaging, add the ``-DPACKAGING=ON`` flag to your CMake command (see
Compilation section above). This command will look something like:

.. code-block:: sh

   cmake -DPACKAGING=ON ..

By default, the packaging utilities will scan the system for the required
toolchains it needs to build each package type (Deb, RPM, and APK). If it does
not find the required toolsets, then it will disable that form of packaging.
The packaging utilities provide CMake flags to force packaging as any of the
supported package types. If a given package type, ex. APK, is set to be enabled
manually by its CMake flag and its required packaging toolchain does not exist,
then CMake will raise a fatal error.

The table below provides the required toolchains for each package type as well
as the CMake flag to set to ``ON`` to manually enable a packaging type:

.. list-table::
   :header-rows: 1

   * - Package Type
     - Required Tools
     - Manual Package Flag
   * - ``deb``
     - ``dpkg-deb``
     - ``PACKAGE_DEB``
   * - ``rpm``
     - ``rpmbuild``
     - ``PACKAGE_RPM``
   * - ``apk``
     - ``docker``
     - ``PACKAGE_APK``


.. note::  Manually setting a given package type to be built (e.g. setting
   ``-DPACKAGE_DEB=ON``\ ) still requires that the ``-DPACKAGING=ON`` to be set.


After the required toolchains have been installed and CMake has been run with
some combination of the packaging flags, the library can be packaged with the
following commands:

.. code-block:: sh

   make package

The command above will build the Debian and RPM packages (depending on the
specified CMake flags).

To build the Alpine APK package, execute the following command:

.. code-block:: sh

   make package-apk

A Note on Alpine APK Packaging
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to package the library as an Alpine APK package, the packaging utility
must use a Docker container to have access to the proper Alpine APK toolchains.
This container will automatically be built when the CMake command is ran to
configure your build environment.

By default, Alpine 3.14 is used to build the package. However, this version
can be changed by setting the ``APKBUILD_ALPINE_VERSION`` CMake flag to the
version of Alpine you wish to use (ex. ``-DAPKBUILD_ALPINE_VERSION=3.12``\ ).

Installation
------------

.. note::  This is a mandatory step to use this library in
   C/C++ OEI modules and Message Bus library.


The OEI Utils library can be installed in two different ways.


#. Through published Debian, Fedora, or Alpine APK packages
#. Installing form source

If you are installing from one of the packages, select the package you wish to
install from the releases assets, and then run one of the following depending
on the OS you are installing on:

.. code-block:: sh

   # Debian
   sudo apt install libcjson1
   sudo dpkg -i <debian package>

   # Fedora
   sudo dnf install cjson
   sudo rpm -i <rpm package>

   # Alpine (NOTE: the depencies get automatically installed by the apk command)
   sudo apk add --allow-untrusted <apk package>

.. note::  In the above commands, installing the cJSON dependency is required,
   however, in general installation of the dev module is not required. However,
   when building against this library the installing the dev version of the
   library is required. For Ubuntu this is ``libcjson-dev``\ , Fedora is ``cjson-devel``
   and Alpine is ``cjson-dev``.


If you wish to install the OEI Utils on your system from source, execute the
following command after building the library:

.. code-block:: sh

   sudo make install

By default, this command will install the OEI Utils C library into
``/opt/intel/eii/lib/``. On some platforms this is not included in the ``LD_LIBRARY_PATH``
by default. As a result, you must add this directory to you ``LD_LIBRARY_PATH``.
This can be accomplished with the following ``export``\ :

.. code-block:: sh

   export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/eii/lib/

.. note::  You can also specify a different library prefix to CMake through
   the ``CMAKE_INSTALL_PREFIX`` flag. If different installation path is given via ``CMAKE_INSTALL_PREFIX``\ , then ``$LD_LIBRARY_PATH`` should be appended by $CMAKE_INSTALL_PREFIX/lib.


Linking
-------

It is recommended to link to the library using the CMake build system. To do
this you must use the ``find_package()`` method in CMake to find and include the
package.

The example below showcases how this can be accomplished.

.. code-block:: cmake

   # Find the package
   find_package(EIIUtils REQUIRED)

   # Add the include directories
   include_directories(${EIIUtils_INCLUDE})

   # Link to your executable (NOTE: 'src/example.cpp' is just an example path)
   add_executable(example "src/example.cpp")
   target_link_libraries(example ${EIIUtils_LIBRARIES})

Running Unit Tests
------------------

.. note::  The unit tests will only be compiled if the ``WITH_TESTS=ON`` option
   is specified when running CMake.


Run the following commands from the ``build/tests`` folder to cover the unit
tests.

.. code-block:: sh

   ./config-tests
   ./frame-tests
   ./log-tests
   ./thp-tests
   ./tsp-tests
