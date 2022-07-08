.. role:: raw-html-m2r(raw)
   :format: html


Contents
========


* `Contents <#contents>`__

  * `NodeRedHttpClientApp <#noderedhttpclientapp>`__

    * `Configure NodeRed <#configure-nodered>`__
    * `Getting Open EII UDF Classifier results data to Node-RED Environment Using Node-RED HTTPClient <#getting-open-eii-udf-classifier-results-data-to-node-red-environment-using-node-red-httpclient>`__
    * `Sample Workflow <#sample-workflow>`__

NodeRedHttpClientApp
--------------------

This Node-RED in-built http node based client App acts as client for the Open EII RestDataExport and brings the Open EII Classifier data to Node-RED ecosystem.

Configure NodeRed
^^^^^^^^^^^^^^^^^

  Node-RED provides various options to install and set up Node-RED in your environment. For more information on installation and setup, refer to the `Node-RED documenation <https://nodered.org/docs/getting-started/local>`_.

.. note::  For quick setup, install using ``docker``

   .. code-block:: sh

         docker run -it -p 1880:1880 --name myNodeRed nodered/node-red


Getting Open EII UDF Classifier results data to Node-RED Environment Using Node-RED HTTPClient
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: \ : RestDataExport should be running already as a prerequisite.\ :raw-html-m2r:`<br>`
   Refer to the RestDataExport `Readme <https://github.com/open-edge-insights/eii-rest-data-export>`_



#. 
   Drag the ``http request`` node of Node-RED's default nodes to your existing workflow.


   .. image:: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/NodeRedHttpClientApp/images/imagehttprequestnode.png
      :target: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/NodeRedHttpClientApp/images/imagehttprequestnode.png
      :alt: images/imagehttprequestnode.png


#. 
   Update the ``properties`` of node as follows:

   For ``DEV mode``\ :


   * 
     Refer to the dialog properties for setting up the ``DEV`` mode in the Node-Red dashboard


     .. image:: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/NodeRedHttpClientApp/images/imagedevmode.png
        :target: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/NodeRedHttpClientApp/images/imagedevmode.png
        :alt: images/imagedevmode.png


   For ``PROD Mode``\ :


   * 
     Refer to the dialog properties for setting up the ``PROD`` mode in the Node-RED dashboard

      
     .. image:: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/NodeRedHttpClientApp/images/imageprodmode.png
        :target: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/NodeRedHttpClientApp/images/imageprodmode.png
        :alt: imageprodmode.png


   For Prod Mode TLS ``ca_cert.pem`` import.
   **Note:** This ``ca_cert.pem`` will be part of the Open EII certificate bundle. Refer the ``[WORKDIR]/IEdgeInsights/build/Certificates/`` directory.


   .. image:: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/NodeRedHttpClientApp/images/imageprodmodetlscert.png
      :target: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/NodeRedHttpClientApp/images/imageprodmodetlscert.png
      :alt: imageprodmodetlscert.png


   ..

      **Note:**


      #. For the ``DEV`` mode, do not enable or attach the certificates.
      #. Update the ``IP`` address as per the ``RestDataExport`` module running machine IP.
      #. For more details on Node-RED's ``http request`` module, refer to `Http requset <https://stevesnoderedguide.com/node-red-http-request-node-beginners>`_.


Sample Workflow
^^^^^^^^^^^^^^^

The attached workflow document is sample workflow by updating the ``RestDataExport`` IP Address in the ``http request`` module,


#. 
   Import the Sample Workflow `flows.json <https://github.com/open-edge-insights/eii-tools/blob/master/NodeRedHttpClientApp/flows.json>`_ file to NodeRed dashboard using ``menu`` icon in top right corner as follows


   .. image:: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/NodeRedHttpClientApp/images/imageimportnodes.png
      :target: https://raw.githubusercontent.com/open-edge-insights/eii-tools/master/NodeRedHttpClientApp/images/imageimportnodes.png
      :alt: images/imageimportnodes.png


#. 
   Click ``Import``

#. 
   Update the ``URL`` of ``http request`` node with ``RestDataExport`` module running the machine IP Address

   ..

      **Note:**


      * For detail, refer to [import export] (https://nodered.org/docs/user-guide/editor/workspace/import-export)
      * The classifier results will be logged in the debug window.

