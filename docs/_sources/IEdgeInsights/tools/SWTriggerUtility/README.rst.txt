
**Contents**


* `Software Trigger Utility for VideoIngestion Module <https://open-edge-insights.github.io/IEdgeInsights/tools/SWTriggerUtility/#software-trigger-utility-for-videoingestion-module>`_

  * `Software Trigger Utilily pre-requisites <https://open-edge-insights.github.io/IEdgeInsights/tools/SWTriggerUtility/#software-trigger-utilily-pre-requisites>`_
  * `Configuration file: <https://open-edge-insights.github.io/IEdgeInsights/tools/SWTriggerUtility/#configuration-file>`_
  * `Running Software Trigger Utility <https://open-edge-insights.github.io/IEdgeInsights/tools/SWTriggerUtility/#running-software%C2%A0trigger%C2%A0utility>`_
  * `Usage of Software Trigger Utility: <https://open-edge-insights.github.io/IEdgeInsights/tools/SWTriggerUtility/#usage%C2%A0of%C2%A0software%C2%A0trigger%C2%A0utility>`_

Software Trigger Utility for VideoIngestion Module
==================================================

This utility is used for invoking various software trigger features of VideoIngestion. The currently supported triggers to VideoIngestion module are:


#. START INGESTION - to start the ingestor
#. STOP_INGESTION -  to stop the ingestor
#. SNAPSHOT - to get frame snapshot which feeds one only frame into the video data pipeline.

Software Trigger Utilily pre-requisites
---------------------------------------

SWTriggerUtility expects a set of config, interfaces & public private keys to be present in ETCD as a pre-requisite.


* To achieve this, please ensure an entry for SWTriggerUtility with its relative path from `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory is set in any of the .yml files present in `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory.

  * An example has been provided below to add the entry in `video-streaming.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/video-streaming.yml>`_
    .. code-block:: yml

         AppContexts:
         ---snip---
         - tools/SWTriggerUtility

Configuration file:
-------------------

**config.json** is the configuration file used for sw_trigger_utility.

.. list-table::
   :header-rows: 1

   * - Field
     - Meaning
     - Type of the value
   * - ``num_of_cycles``
     - ``Number of cyles of start-stop ingestions to repeat``
     - ``integer``
   * - ``dev_mode``
     - ``dev mode ON or OFF``
     - ``boolean (true or false)``
   * - ``log_level``
     - ``Log level to view the logs accordingly``
     - ``integer [DEBUG=3 (default), ERROR=0, WARN=1, INFO=2]``


.. note:: \ :


   * When working with GigE cameras which requires ``network_mode: host``\ , update the ``EndPoint`` key of SWTriggerUtility interface in `config.json <https://github.com/open-edge-insights/eii-tools/blob/master/SWTriggerUtility/config.json>`_ to have the host system IP instead of the service name of the server.


.. code-block::

   Eg. In order to connect to `ia_video_ingestion` service which is configured with GigE camera refer the below `EndPoint` change in the SWTriggerUtility interface:

   ```javascript
   {
       "Clients": [
           {
               "EndPoint": "<HOST_SYSTEM_IP>:64013",
               "Name": "default",
               "ServerAppName": "VideoIngestion",
               "Type": "zmq_tcp"
           }
       ]
   }
   ```


..

   * In case one needs to change the values in `pre-requisites section <https://open-edge-insights.github.io/IEdgeInsights/tools/SWTriggerUtility/#software-trigger-utilily-pre-requisites>`_\ , then ensure to re-run the steps mentioned in `pre-requisites section <https://open-edge-insights.github.io/IEdgeInsights/tools/SWTriggerUtility/#software-trigger-utilily-pre-requisites>`_ to see the updated changes are getting applied ``OR`` one can choose to update the config key of SWTriggerUtility app via ETCD UI and then restart the application.


**This utility works in both dev & prod mode.**  As a pre-requisite make sure to turn ON the flag corresponding to "dev_mode" to true/false in the config.json file.

Running Software Trigger Utility
--------------------------------


#. 
   EII services can be running in prod or dev mode by setting ``DEV_MODE`` value accordingly in `build/.env <https://github.com/open-edge-insights/eii-core/blob/master/build/.env>`_

#. 
   Execute builder.py script:

   .. code-block:: sh

      $ cd [WORKDIR]/IEdgeInsights/build/
      $ python3 builder.py -f usecases/video-streaming.yml

   **NOTE**\ : The same yml file to which the **SWTriggerUtility** entry was added in `pre-requisites <https://open-edge-insights.github.io/IEdgeInsights/tools/SWTriggerUtility/#software-trigger-utilily-pre-requisites>`_ has to be selected while running `pre-requisites <https://open-edge-insights.github.io/IEdgeInsights/tools/SWTriggerUtility/#software-trigger-utilily-pre-requisites>`_

#. 
   Run provisioning step as below:

   .. code-block:: sh

      $ cd [WORKDIR]/IEdgeInsights/build/provision
      $ sudo -E ./provision.sh  ../docker-compose.yml

   ## Usage of Software Trigger Utility:

By default the SW Trigger Utility container will not execute anything and one needs to interact with the running container to generate the trigger commands. Make sure VI service is up and ready to process the commands from the utility.

Software trigger utility can be used in following ways:

1: "START INGESTION" -> "Allows ingestion for default time (120 seconds being default)" -> "STOP INGESTION"

.. code-block::

   ```sh
   $ cd [WORKDIR]/IEdgeInsights/build
   $ docker exec ia_sw_trigger_utility ./sw_trigger_utility
   ```


2: "START INGESTION" -> "Allows ingestion for user defined time (configurable time in seconds)" -> "STOP INGESTION"

.. code-block::

   ```sh
   $ cd [WORKDIR]/IEdgeInsights/build
   $ docker exec ia_sw_trigger_utility ./sw_trigger_utility 300
   ```
   **Note**: In the above example, VideoIngestion starts then does ingestion for 300 seconds then stops ingestion after 300 seconds & cycle repeats for number of cycles configured in the config.json.



3: Selectively send START_INGESTION software trigger:

.. code-block::

   ```sh
   $ cd [WORKDIR]/IEdgeInsights/build
   $ docker exec ia_sw_trigger_utility ./sw_trigger_utility START_INGESTION
   ```


4: Selectively send STOP_INGESTION software trigger:

.. code-block::

   ```sh
   $ cd [WORKDIR]/IEdgeInsights/build
   $ docker exec ia_sw_trigger_utility ./sw_trigger_utility STOP_INGESTION
   ```


5: Selectively send SNAPSHOT software trigger:

.. code-block::

   ```sh
   $ cd [WORKDIR]/IEdgeInsights/build
   $ docker exec ia_sw_trigger_utility ./sw_trigger_utility SNAPSHOT
   ```

.. note:: \ :  



* 
  If duplicate START_INGESTION or STOP_INGESTION sw_triggers are sent by client by mistake then the VI is capable  of catching these duplicates & responds back to client conveying that duplicate triggers were sent & requets to send proper sw_triggers. 

* 
  In order to send SNAPSHOT trigger, ensure that the ingestion is stopped. In case START_INGESTION trigger is sent previously then stop the ingestion using the STOP_INGESTION trigger.
