.. role:: raw-html-m2r(raw)
   :format: html


Contents
========


* `Contents <#contents>`__

  * `Ansible based OEI Prequisites setup, provisioning, build & deployment <#ansible-based-oei-prequisites-setup-provisioning-build-deployment>`__
  * `Installing Ansible on Ubuntu {Control node} <#installing-ansible-on-ubuntu-control-node>`__
  * `Prerequisite step needed for all the control/worker nodes <#prerequisite-step-needed-for-all-the-controlworker-nodes>`__

    * `Generate SSH KEY for all nodes <#generate-ssh-key-for-all-nodes>`__

  * `Adding SSH Authorized Key from control node to all the nodes <#adding-ssh-authorized-key-from-control-node-to-all-the-nodes>`__

    * `Configure Sudoers file to accept NO PASSWORD for sudo operation <#configure-sudoers-file-to-accept-no-password-for-sudo-operation>`__

  * `Updating the leader information for using remote hosts <#updating-the-leader-information-for-using-remote-hosts>`__
  * `Updating the OEI Source Folder, Usecase & Proxy Settings in Group Variables <#updating-the-oei-source-folder-usecase-proxy-settings-in-group-variables>`__
  * `Remote node deployment <#remote-node-deployment>`__
  * `Execute ansible Playbook from [EII_WORKDIR]/IEdgeInsights/build/ansible {Control node} to deploy OEI services in control node/remote node <#execute-ansible-playbook-from-eii_workdiriedgeinsightsbuildansible-control-node-to-deploy-oei-services-in-control-noderemote-node>`__

    * `For Single Point of Execution <#for-single-point-of-execution>`__
    * `Deploying OEI Using Helm in Kubernetes (k8s) environment <#deploying-oei-using-helm-in-kubernetes-k8s-environment>`__

Ansible based OEI Prequisites setup, provisioning, build & deployment
---------------------------------------------------------------------

Ansible is the automation engine which can enable Open Edge Insights (OEI) deployment across single nodes. We need one control node where ansible is installed and optional hosts. We can also use the control node itself to deploy OEI.

.. note:: 


   * In this document, you will find labels of 'Edge Insights for Industrial (EII)' for filenames, paths, code snippets, and so on. Consider the references of EII as OEI. This is due to the product name change of EII as OEI.
   * Ansible can execute the tasks on control node based on the playbooks defined
   * There are 3 types of nodes - control node where ansible must be installed, OEI leader node where ETCD server will be running and optional worker nodes, all worker nodes remotely connect to ETCD server running on leader node. Control node and OEI leader node can be same.


Installing Ansible on Ubuntu {Control node}
-------------------------------------------


#. 
   Execute the following command in the identified control node machine.

   .. code-block:: sh

           sudo apt update
           sudo apt install software-properties-common
           sudo apt-add-repository --yes --update ppa:ansible/ansible
           sudo apt install ansible

Prerequisite step needed for all the control/worker nodes
---------------------------------------------------------

Generate SSH KEY for all nodes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Generate the SSH KEY for all nodes using following command (to be executed in the only control node), ignore to run this command if you already have ssh keys generated in your system without id and passphrase,

.. code-block:: sh

       ssh-keygen

.. note:: 
   Dont give any passphrase and id, just press ``Enter`` for all the prompt which will generate the key.


For Eg.

.. code-block:: sh

   $ ssh-keygen

   Generating public/private rsa key pair.
   Enter file in which to save the key (/root/.ssh/id_rsa):  <ENTER>
   Enter passphrase (empty for no passphrase):  <ENTER>
   Enter same passphrase again:  <ENTER>
   Your identification has been saved in ~/.ssh/id_rsa.
   Your public key has been saved in ~/.ssh/id_rsa.pub.
   The key fingerprint is:
   SHA256:vlcKVU8Tj8nxdDXTW6AHdAgqaM/35s2doon76uYpNA0 root@host
   The key's randomart image is:
   +---[RSA 2048]----+
   |          .oo.==*|
   |     .   .  o=oB*|
   |    o . .  ..o=.=|
   |   . oE.  .  ... |
   |      ooS.       |
   |      ooo.  .    |
   |     . ...oo     |
   |      . .*o+.. . |
   |       =O==.o.o  |
   +----[SHA256]-----+

Adding SSH Authorized Key from control node to all the nodes
------------------------------------------------------------

Please follow the steps to copy the generated keys from control node to all other nodes

Execute the following command from ``control`` node.

.. code-block:: sh

       ssh-copy-id <USER_NAME>@<HOST_IP>

For Eg.

.. code-block:: sh

       $ ssh-copy-id test@192.0.0.1

       /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/<username>/.ssh/id_rsa.pub"

       Number of key(s) added: 1

       Now try logging into the machine, with:   "ssh 'test@192.0.0.1'"
       and check to make sure that only the key(s) you wanted were added.

Configure Sudoers file to accept NO PASSWORD for sudo operation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: 
   Ansible need to execute some commands as ``sudo``. The below configuration is needed so that ``passwords`` need not be saved in the ansible inventory file ``hosts``


Update ``sudoers`` file


#. 
   Open the ``sudoers`` file.

   .. code-block:: sh

           sudo visudo

#. 
   Append the following to the ``sudoers`` file

   ..

      **Note**
      Please append to the last line of the ``sudoers`` file.


   .. code-block:: sh

          <ansible_non_root_user>  ALL=(ALL:ALL) NOPASSWD: ALL

   For E.g:

   If in control node, the current non root user is ``user1``\ , you should append as follows

   .. code-block:: sh

         user1 ALL=(ALL:ALL) NOPASSWD: ALL

#. 
   Save & Close the file

#. 
   For checking the ``sudo`` access for the enabled ``user``\ , please logout and login to the session again
   check the following command.

   .. code-block:: sh

         sudo -l -U <ansible_non_root_user>

   Now the above line authorizes ``user1`` user to do ``sudo`` operation in the control node without ``PASSWORD`` ask.

   ..

      **Note**
      The same procedure applies to all other nodes where ansible connection is involved.


Updating the leader information for using remote hosts
------------------------------------------------------

.. note:: 
   By ``default`` both control/leader node ``ansible_connection`` will be ``localhost`` in single node deployment.


Please follow below steps to update the details of leader node for remote node scenario.


* 
  Update the hosts information in the inventory file ``hosts``

  .. code-block:: sh

           [group_name]
           <nodename> ansible_connection=ssh ansible_host=<ipaddress> ansible_user=<machine_user_name>

    For Eg:

  .. code-block:: sh

       [targets]
       leader ansible_connection=ssh ansible_host=192.0.0.1  ansible_user=user1

  ..

     **Note**


     * ``ansible_connection=ssh`` is mandatory when you are updating any remote hosts, which makes ansible to connect via ``ssh``.
     * The above information is used by ansible to establish ssh connection to the nodes.
     * control node will always be ``ansible_connection=local``\ , **don't update the control node's information**
     * To deploy OEI in single , ``ansible_connection=local`` and ``ansible_host=localhost``
     * To deploy OEI on remote node, ``ansible_connection=ssh`` and ``ansible_host=<remote_node_ip>`` and ``ansible_user``\ =\ :raw-html-m2r:`<username>`


Updating the OEI Source Folder, Usecase & Proxy Settings in Group Variables
---------------------------------------------------------------------------


#. 
   Open ``group_vars/all.yml`` file

   .. code-block:: sh

           vi group_vars/all.yml

#. 
   Update Proxy Settings

   .. code-block:: sh

           enable_system_proxy: true
           http_proxy: <proxy_server_details>
           https_proxy: <proxy_server_details>
           no_proxy: <managed_node ip>,<controller node ip>,<worker nodes ip>,localhost,127.0.0.1

#. 
   Update the ``EII Secrets`` username & passwords in ``group_vars/all.yml`` required to run few OEI Services in ``PROD`` mode only

#. 
   Update the ``usecase`` variable, based on the usecase ``builder.py`` generates the OEI deployment and config files.

   ..

      **Note**


      #. By default it will be ``video-streaming``\ , For other usecases refer the ``../usecases`` folder and update only names without ``.yml`` extension
      #. For ``all`` usecase, it will bring up all ``default`` services of eii.
      #. ``ia_kapacitor`` and ``ia_telegraf`` container images are not distributed via docker hub, so one won't be able to pull these images
         for time-series use case upon using `../usecases/time-series.yml <https://github.com/open-edge-insights/eii-core/blob/master/build/usecases/time-series.yml`>`_ for deployment. For more details,
         refer to [../README.md#distribution-of-eii-container-images]> (../README.md#distribution-of-eii-container-images).


   For Example, if you want build & deploy for ``../usecases/time-series.yml`` update the ``usecase`` key value as ``time-series``

   .. code-block:: sh

      usecase: <time-series>

#. 
   Optionally, you can choose number of video pipeline instances to be created by updating ``instances`` variable

#. Update other optional variables provided if required

Remote node deployment
----------------------

Below configuration changes need to be made for remote node deployment without k8s


#. Single node deployment all the services based on the ``usecase`` chosen will be deployed.
#. 
   Update ``docker`` registry details in following section if using a custom/private registry

   .. code-block:: sh

           docker_registry="<regsitry_url>"
           docker_login_user="<username>"
           docker_login_passwd="<password>"

   ..

      **Note**
      Use of ``docker_registry`` and ``build`` flags are as follows:


      * Update the ``docker_registry`` details to use docker images from custom registry, optionally set ``build: true`` to push docker images to this registry
      * Unset the ``docker_registry`` details if you don't want to use custom registry and set ``build: true`` to save and load docker images from one node to another node


#. 
   If you are using images from docker hub, then set ``build: false`` and unset ``docker_registry`` details

Execute ansible Playbook from [EII_WORKDIR]/IEdgeInsights/build/ansible {Control node} to deploy OEI services in control node/remote node
-----------------------------------------------------------------------------------------------------------------------------------------

.. note:: 


   * Updating messagebus endpoints to connect to interfaces is still the manual process. Make sure to update Application specific endpoints in ``[AppName]/config.json``
   * After ``pre-requisites`` are successfully installed, please do logout and login to apply the changes
   * By Default ``wait_time_for_config_mgr`` for waiting confimgr to up with provisioning done status will be set to ``20`` seconds
   * If your system is slow or execution is delaying you can increase or decrease ``wait_time_for_config_mgr`` in ``group_vars/all.yml``


For Single Point of Execution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note::  This will execute all the steps of OEI as prequisite, build, provision, deploy & setup all nodes for deployement usecase in one shot sequentialy.


.. code-block:: sh

      ansible-playbook eii.yml

**Below steps are the individual execution of setups.**


* 
  Load ``.env`` values from template

  .. code-block:: sh

       ansible-playbook eii.yml --tags "load_env"

* 
  For OEI Prequisite Setup

  .. code-block:: sh

       ansible-playbook eii.yml --tags "prerequisites"

  ..

     **Note**
     After ``pre-requisites`` are successfully installed, please do logout and login to apply the changes


* 
  To generate builder and config files, build images and push to registry

  .. code-block:: sh

       ansible-playbook eii.yml --tags "build"

* 
  To generate eii bundles for deployment

  .. code-block:: sh

       ansible-playbook eii.yml --tags "gen_bundles"

* 
  To deploy the eii modules

  .. code-block:: sh

       ansible-playbook eii.yml --tags "deploy"

Deploying OEI Using Helm in Kubernetes (k8s) environment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: 


   * To Deploy OEI using helm in k8s aenvironment, ``k8s`` setup is a prerequisite.
   * You need update the ``k8s`` leader machine as leader node in ``hosts`` file.
   * Non ``k8s`` leader machine the ``helm`` deployment will fail.
   * For ``helm`` deployment ``ansible-remotenode`` parameters will not applicable, Since node selection & pod selection will be done by ``k8s`` orchestrator.
   * Make sure you are deleting ``/opt/intel/eii/data`` when switch from ``prod`` mode to ``dev`` mode in all your ``k8s`` ``worker`` nodes.
   * Ignore the following message ``FAILED - RETRYING: [leader]: Wait for Provisioning is Done (5 retries left).``. It occurs during waiting time. It is not an error



* 
  Update the ``DEPLOYMENT_MODE`` flag as ``k8s`` in ``group_vars/all.yml`` file:


  * 
    Open ``group_vars/all.yml`` file

    .. code-block:: sh

           vi group_vars/all.yml

  * 
    Update the ``DEPLOYMENT_MODE`` flag as ``k8s``

    .. code-block:: yml

           ## Deploy in k8s mode using helm
           DEPLOYMENT_MODE: "k8s"

  * 
    Save & Close

* 
  For Single Point of Execution

  ..

     **Note**
     This will execute all the steps of OEI as prequisite, build, provision, deploy for a usecase in one shot sequentialy.


  .. code-block:: sh

       ansible-playbook eii.yml

.. note:: 
   Below steps are the individual execution of setups.



* 
  Load ``.env`` values from template

  .. code-block:: sh

       ansible-playbook eii.yml --tags "load_env"

* 
  For OEI Prequisite Setup

  .. code-block:: sh

       ansible-playbook eii.yml --tags "prerequisites"

* 
  For building OEI containers

  .. code-block:: sh

       ansible-playbook eii.yml --tags "build"

* 
  To generate Certificates in control node

  .. code-block:: sh

       ansible-playbook eii.yml --tags "gen_certs"

* 
  Prerequisites for deploy OEI using Ansible helm environment.

  .. code-block:: sh

       ansible-playbook eii.yml --tags "helm_k8s_prerequisites"

* 
  Provision and Deploy OEI Using Ansible helm environment

  .. code-block:: sh

       ansible-playbook eii.yml --tags "helm_k8s_deploy"
