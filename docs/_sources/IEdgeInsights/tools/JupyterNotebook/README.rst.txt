
**Contents**


* `Jupyter Notebook usage to develop python UDFs <http://localhost:7645/IEdgeInsights/tools/JupyterNotebook/#jupyter-notebook-usage-to-develop-python-udfs>`_

  * `Jupyter Notebook pre-requisites <http://localhost:7645/IEdgeInsights/tools/JupyterNotebook/#jupyter-notebook-pre-requisites>`_
  * `Running Jupyter Notebook <http://localhost:7645/IEdgeInsights/tools/JupyterNotebook/#running-jupyter-notebook>`_

Jupyter Notebook usage to develop python UDFs
=============================================

UDF development in python can be done using the web based IDE of Jupyter Notebook.
Jupyter Notebook is an open-source web application that allows you to create and share documents that contain live code, equations, visualizations and narrative text. Uses include: data cleaning and transformation, numerical simulation, statistical modeling, data visualization, machine learning, and much more.

This tool acts as an interface between user and Jupyter Notebook service allowing the user to interact with Jupyter Notebook to write, edit, experiment and create python UDFs.

It works along with the `jupyter_connector <https://github.com/open-edge-insights/video-common/tree/master/udfs/python/jupyter_connector.py>`_ UDF for enabling the IDE for udf development.

Jupyter Notebook pre-requisites
-------------------------------


#. 
   Jupyter Notebook expects a set of config, interfaces & public private keys to be present in ETCD as a pre-requisite.


   * To achieve this, please ensure an entry for Jupyter Notebook with its relative path from `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory is set in any of the .yml files present in `IEdgeInsights <https://github.com/open-edge-insights/>`_ directory.
   * An example has been provided below to add the entry in `video-streaming.yml <https://github.com/open-edge-insights/eii-core/tree/master/build/usecases/video-streaming.yml>`_
     .. code-block:: yml

          AppContexts:
          ---snip---
          - tools/JupyterNotebook

#. 
   Ensure the `jupyter_connector <https://github.com/open-edge-insights/video-common/tree/master/udfs/python/jupyter_connector.py>`_ UDF is enabled in the config of either **VideoIngestion** or **VideoAnalytics** to be connected to JupyterNotebook. An example has been provided here for connecting **VideoIngestion** to **JupyterNotebook**\ , the config to be changed being present at `config.json <https://github.com/open-edge-insights/video-ingestion/blob/master/config.json>`_\ :

   .. code-block:: javascript

           {
               "config": {
                   "encoding": {
                       "type": "jpeg",
                       "level": 95
                   },
                   "ingestor": {
                       "type": "opencv",
                       "pipeline": "./test_videos/pcb_d2000.avi",
                       "loop_video": true,
                       "queue_size": 10,
                       "poll_interval": 0.2
                   },
                   "sw_trigger": {
                       "init_state": "running"
                   },
                   "max_workers":4,
                   "udfs": [{
                       "name": "jupyter_connector",
                       "type": "python",
                       "param1": 1,
                       "param2": 2.0,
                       "param3": "str"
                   }]
               }
           }

Running Jupyter Notebook
------------------------


#. 
   With the above pre-requisite done, please run the below command:

   .. code-block:: sh

           python3 builder.py -f usecases/video-streaming.yml

#. 
   Refer `IEdgeInsights/README.md <https://github.com/open-edge-insights/eii-core/blob/master/README.md>`_ to provision, build and run the tool along with the EII recipe/stack.

#. 
   Run the following command to see the logs:

   .. code-block:: sh

           docker logs -f ia_jupyter_notebook

#. 
   Copy paste the URL (along with the token) from the above logs in a browser. Below is a sample URL 

   .. code-block:: sh

           http://127.0.0.1:8888/?token=5839f4d1425ecf4f4d0dd5971d1d61b7019ff2700804b973

   Replace the '127.0.0.1' IP address with host IP, if you are accessing the server remotely.

#. 
   Once the Jupyter Notebook service is launched in the browser, run the `main.ipynb <https://github.com/open-edge-insights/eii-tools/blob/master/JupyterNotebook/main.ipynb>`_ file visible in the list of files available.

#. 
   The process method of `udf_template.ipynb <https://github.com/open-edge-insights/eii-tools/blob/master/JupyterNotebook/udf_template.ipynb>`_ file available in the list of files can be altered and re-run to experiment and test the UDF.

#. 
   If any parameters are to be sent to the custom udf by the user, they can be added in the **jupyter_connector** UDF config provided to either **VideoIngestion** or **VideoAnalytics** and can be accessed in the `udf_template.ipynb <https://github.com/open-edge-insights/eii-tools/blob/master/JupyterNotebook/udf_template.ipynb>`_ constructor in the **udf_config** parameter which is a dict containing all these parameters. A sample UDF for reference has been provided at `udf_template.ipynb <https://github.com/open-edge-insights/eii-tools/blob/master/JupyterNotebook/udf_template.ipynb>`_.

**Note**\ : After altering/creating a new udf, run main.ipynb  and restart **VideoIngestion** or **VideoAnalytics** with which you have enabled jupyter notebook service.


#. 
   Once the user is satisfied with the functionality of the UDF, the udf can be saved/exported by clicking on the **Download as** option and selecting **(.py)** option. The downloaded udf can then be directly used by placing it in the `../../common/video/udfs/python <https://github.com/open-edge-insights/video-common/blob/master/udfs/python>`_ directory or can be integrated and used with **CustomUDFs**.

   ..

      **Note:**


      * JupyterNotebook is not to be used with **CustomUDFs** like **GVASafetyGearIngestion** since they are specific to certain usecases only. Instead, the VideoIngestion pipeline can be modified to use GVA ingestor pipeline and config can be modifed to use **jupyter_connector** UDF.

