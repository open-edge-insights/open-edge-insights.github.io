
Video Benchmarking Tool
=======================

These scripts are designed to automate the running of benchmarking tests and the collection of the performance data. This performance data includes the FPS of each video stream, and also the CPU%, Memory%, and Memory Read/Write bandwidth.

The Processor Counter Monitor (PCM) is required for measuring memory read/write bandwidth, which can be downloaded and built here: https://github.com/opcm/pcm

If you do not have PCM on your system, those columns will be blank in the output.csv.

.. note:: \ :


   * 
     For running the gstreamer pipeline mentioned in `sample_test/config.json <https://github.com/open-edge-insights/eii-tools/blob/master/Benchmarking/video-benchmarking-tool/sample_test/config.json>`_ one needs to copy the required model files to [WORKDIR]/IEdgeInsights/VideoIngestion/models by referring `sample_test/config.json <https://github.com/open-edge-insights/eii-tools/blob/master/Benchmarking/video-benchmarking-tool/sample_test/config.json>`_.

   * 
     For using ``MYRIAD`` device for inference please add ``user: root`` in docker-compose.yml file for VI. If IPC mode is used then ``user: root`` needs to be added for VideoProfiler docker-compose.yml file as well.

   * 
     The docker-compose.yml files of VI and VideoProfiler gets picked from their respective repos. So any changes needed should be applied in their repsective repos.

   * 
     Refer `README-Using-video-accelerators <https://github.com/open-edge-insights/eii-core#using-video-accelerators-in-ingestionanalytics-containers>`_ for using video accelerators.


Steps for running a benchmarking test case:


#. 
   Please ensure the VideoProfiler requirements are installed by following the README at `README <https://github.com/open-edge-insights/eii-tools/blob/master/VideoProfiler/README.md>`_

#. 
   Start RTSP server on a sepereate system on the network:

   .. code-block:: sh

       $ ./stream_rtsp.sh <number-of-streams> <starting-port-number> <bitrate> <width> <height> <framerate>

   For example:

   .. code-block:: sh

       $ ./stream_rtsp.sh 16 8554 4096 1920 1080 30

#. 
   Run execute_test.sh with the desired benchmarking config:

   .. code-block::

       USAGE:
           ./execute_test.sh TEST_DIR STREAMS SLEEP PCM_HOME [EII_HOME]

       WHERE:
           TEST_DIR  - The absolute path to directory containing services.yml for the services to be tested, and the config.json and docker-compose.yml for VI and VA if applicable.
           STREAMS   - The number of streams (1, 2, 4, 8, 16)
           SLEEP     - The number of seconds to wait after the containers come up
           PCM_HOME  - The absolute path to the PCM repository where pcm.x is built
           EII_HOME - [Optional] The absolute path to EII home directory, if running from a non-default location

   For example:

   .. code-block:: sh

       sudo -E ./execute_test.sh $PWD/sample_test 16 60 /opt/intel/pcm /home/intel/IEdgeInsights

#. 
   The execution log, performance logs, and the output.csv will be saved in TEST_DIR/< timestamp >/ so that the same test case can be ran multiple times without overwriting the output. You can see if the test ocurred any errors in the execution.log, and you can see the results of a successful test in output.csv.
