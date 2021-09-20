
**Contents**


* `Intel SafeString library <https://open-edge-insights.github.io/IEdgeInsights/common/util/c/IntelSafeString/#intel-safestring-library>`_

  * `Compilation <https://open-edge-insights.github.io/IEdgeInsights/common/util/c/IntelSafeString/#compilation>`_
  * `Installation <https://open-edge-insights.github.io/IEdgeInsights/common/util/c/IntelSafeString/#installation>`_

Intel SafeString library
========================

This library includes routines for safe string operations (like strcpy) and memory routines (like memcpy) that are recommended for Linux/Android operating systems, and will also work for Windows. This library is especially useful for cross-platform situations where one library for these routines is preferred.

Compilation
-----------

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

Installation
------------

.. code-block:: sh

   $ sudo make install

By default, this command will install the Intel safestring library into
``/opt/intel/eii/lib/``. On some platforms this is not included in the ``LD_LIBRARY_PATH``
by default. As a result, you must add this directory to you ``LD_LIBRARY_PATH``.
This can be accomplished with the following ``export``\ :

.. code-block:: sh

   $ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/eii/lib/

.. note::  You can also specify a different library prefix to CMake through
   the ``CMAKE_INSTALL_PREFIX`` flag. If different installation path is given via ``CMAKE_INSTALL_PREFIX``\ , then ``$LD_LIBRARY_PATH`` should be appended by $CMAKE_INSTALL_PREFIX/lib.

