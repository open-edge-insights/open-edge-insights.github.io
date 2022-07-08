
ETCD UI Service
===============

.. note::  In this document, you will find labels of 'Edge Insights for Industrial (EII)' for file names, paths, code snippets, and so on. Consider the references of EII as Open Edge Insights for Industrial (Open EII). This is due to the product name change of EII as Open EII.


After the Open EII Configuration Management (a_configmgr_agent) service is successfully up, you can access the ETCD web UI through the steps below. You can make configuration changes for respective Open EII container services.


* Open your browser and enter the address: https://$(HOST_IP):7071/etcdkeeper/ (when Open EII is running in secure mode). In this case, CA cert has to be imported in the browser. For insecure mode i.e. DEV mode, it can be accessed at https://$(HOST_IP):7071/etcdkeeper/.
* Click on the version of the title to select the version of ETCD. The default is V3. Reopening will remember your choice.
* Right-click on the tree node to add or delete.
* For secure mode, authentication is required. User name and password needs to be entered in the dialogue box.
* Username is 'root' and default password is located at ETCDROOT_PASSWORD key under environment section in `docker-compose.yml <https://github.com/open-edge-insights/eii-configmgr-agent/blob/master/docker-compose.yml>`_.
* This service can accessed from a remote system at address: https://$(HOST_IP):7071 (when Open EII is running in secure mode). In this case, CA cert has to be imported in the browser. For insecure mode i.e. DEV mode, it can be accessed at http://$(HOST_IP):7071


.. image:: https://raw.githubusercontent.com/open-edge-insights/eii-etcd-ui/master/img/fig_6_3.png
   :target: https://raw.githubusercontent.com/open-edge-insights/eii-etcd-ui/master/img/fig_6_3.png
   :alt: ETCD UI Interface


.. note:: 



#. 
   If ETCDROOT_PASSWORD is changed, there must be consolidated docker-compose.yml generated using builder script and Open EII must to be provisioned again. Run the following commands:

   .. code-block:: sh

       cd [WORKDIR]/IEdgeInsights/build
       python3 builder.py -f usecases/<usecase.ml>
       docker-compose up -d ia_configmgr_agent

#. 
   Only VideoIngestion and VideoAnalytics based services will have watch for any changes. Any changes done to those keys will be reflected at runtime in Open EII.

#. 
   For changes done to any other keys, Open EII stack needs to be restarted for it to take effect. Run the following commands in working directory build/ to restart Open EII.

   .. code-block:: sh

        cd [WORKDIR]/IEdgeInsights/build
        docker-compose down
        eii_start.sh

#. 
   Refer to the `prerequisites for video accelerators <https://github.com/open-edge-insights/eii-core#using-video-accelerators-in-ingestionanalytics-containers>`_ and `prerequisities for cameras <https://github.com/open-edge-insights/video-ingestion#camera-configuration>`_ before trying to change the config dynamically through ETCD UI.
