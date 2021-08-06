.. role:: raw-html-m2r(raw)
   :format: html


**Contents**


* `FactoryControlApp: <http://localhost:7645/IEdgeInsights/FactoryControlApp/#factorycontrolapp>`_

  * `\ ``Pre-requisites`` <http://localhost:7645/IEdgeInsights/FactoryControlApp/#pre-requisites>`_
  * `\ ``Service bring up`` <http://localhost:7645/IEdgeInsights/FactoryControlApp/#service-bring-up>`_

FactoryControlApp:
==================

This module controls the Alarm Light and Reset button.
The Output of VA (Video Analytics) i.e. the classified results data (MetaData, Frame) is published on a topic (which is mentioned in the docker compose.yml file in VA section) & the Factory control app subscribes to this topic (again sub topic mentioned in compose file of factory control app section). This meta data is parsed by the FCA & is used to determine if the frame is Defective or a proper one & glow the Red or green light in the IO module (Adam controller) respectively.

FactoryControlApp uses Message Bus Library to subscribe the output of VA (Video analytics) on the topic where VA publishes the classified data on the same topic.
The config about Adamcontroller (IO module) like port, IP address, register bits is stored in ETCD (A distributed key-value store).

``Pre-requisites``
----------------------


#. 
   Configure io_module using a tool **AdamApax.Net Utility V2.05.11 B19.msi**. This can be downloaded from [here]
   (https://support.advantech.com/support/DownloadSRDetail_New.aspx?SR_ID=1-2AKUDB&Doc_Source=Download) by selecting **Primary** Download Site with the **AdamApax.Net Utility V2.05.11 B19.msi**.

    **Note**\ : System from which io_module is configures and io_module both should be in same subnet.
    During the next step, if password prompt is asked then the password is "00000000"

#. 
   Install and open the downloaded app and follow the below instructions:\ :raw-html-m2r:`<br>`
    a. On the left-hand side pannel, right click on ``Ethernet`` and select ``Search Device``\ :raw-html-m2r:`<br>`
    b. io_module will be detected (ADAM-6050) under Ethernet, click on it and go to ``Network`` tab and set the ``<ip_address>`` to the io_module.\ :raw-html-m2r:`<br>`

.. note:: \ : For the circuit connections of the lab > setup, refer 4.5 (4.5.2 IO module) in the
   document `HW_Configuration.pdf <https://github.com/open-edge-insights/eii-factoryctrl/blob/master/HW_Configuration.pdf>`_


``Service bring up``
------------------------


* 
  Update "io_module_ip" key with host ip address in the `config <https://github.com/open-edge-insights/eii-factoryctrl/blob/master/config.json>`_ file.

* 
  Please go through the below sections to have visualizer service built
  and launch it:


  * `../README.md#generate-deployment-and-configuration-files <https://github.com/open-edge-insights/eii-core/blob/master/README.md#generate-deployment-and-configuration-files>`_
  * `../README.md#provision <https://github.com/open-edge-insights/eii-core/blob/master/README.md#provision>`_
  * `../README.md#build-and-run-eii-videotimeseries-use-cases <https://github.com/open-edge-insights/eii-core/blob/master/README.md#build-and-run-eii-videotimeseries-use-cases>`_
