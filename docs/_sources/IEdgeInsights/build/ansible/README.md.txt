## Ansible based EII Prequisites setup, provisioning, build & deployment

Ansible is the automation engine which can enable EII deployment across single/multi nodes.
We need one control node where ansible is installed and optional hosts. We can also use the control node itself to deploy EII

> **Note**: 
> * Ansible can execute the tasks on control node based on the playbooks defined
> * There are 3 types of nodes - control node where ansible must be installed, EII leader node where ETCD server will be running and optional worker nodes, all worker nodes remotely connect to ETCD server running on leader node. Control node and EII leader node can be same.


## Installing Ansible on Ubuntu {Control node} 

1.  Execute the following command in the identified control node machine.
    ```sh
        $ sudo apt update
        $ sudo apt install software-properties-common
        $ sudo apt-add-repository --yes --update ppa:ansible/ansible
        $ sudo apt install ansible
    ```


## Prerequisite step needed for all the control/worker nodes.
### Generate SSH KEY for all nodes

Generate the SSH KEY for all nodes using following command (to be executed in the only control node), ignore to run this command if you already have ssh keys generated in your system without id and passphrase,

```sh
    $ ssh-keygen
```
> **Note**:
>   * Dont give any passphrase and id, just press `Enter` for all the prompt which will generate the key.

For example.
```sh
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
```


## Adding SSH Authorized Key from control node to all the nodes

Please follow the steps to copy the generated keys from control node to all other nodes

Execute the following command from `control` node.
```sh
    $ ssh-copy-id <USER_NAME>@<HOST_IP>
```

For example.
```sh
    $ ssh-copy-id test@192.0.0.1

    /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/<username>/.ssh/id_rsa.pub"

    Number of key(s) added: 1

    Now try logging into the machine, with:   "ssh 'test@192.0.0.1'"
    and check to make sure that only the key(s) you wanted were added.
```

### Configure Sudoers file to accept NO PASSWORD for sudo operation.

> **Note**: Ansible need to execute some commands as `sudo`. The below configuration is needed so that `passwords` need not be saved in the ansible inventory file `hosts`

Update `sudoers` file

1. Open the `sudoers` file.
    ```sh
        $ sudo visudo
    ```

2. Append the following to the `sudoers` file
   > **Note**: Please append to the last line of the `sudoers` file.

   ```sh
       $ <ansible_non_root_user>  ALL=(ALL:ALL) NOPASSWD: ALL
   ```
   For E.g:

   If in control node, the current non root user is `user1`, you should append as follows
   ```sh
      user1 ALL=(ALL:ALL) NOPASSWD: ALL
   ```
3. Save & Close the file

4. For checking the `sudo` access for the enabled `user`, please logout and login to the session again
   check the following command.
   ```sh
      $ sudo -l -U <ansible_non_root_user> 
   ```
   Now the above line authorizes `user1` user to do `sudo` operation in the control node without `PASSWORD` ask.
   > **Note**: The same procedure applies to all other nodes where ansible connection is involved.

## Updating the leader & worker node's information for using remote hosts

>**Note**: By `default` both control/leader node `ansible_connection` will be `localhost` in single node deployment.

Please follow the below steps to update the details of leader/worker nodes for multi node scenario.

*   Update the hosts information in the inventory file `hosts`
    ```
        [group_name]
        <nodename> ansible_connection=ssh ansible_host=<ipaddress> ansible_user=<machine_user_name>
    ```

    For example:
    ```
    [targets]
    leader ansible_connection=ssh ansible_host=192.0.0.1  ansible_user=user1
    ```
    
    > **Note**: 
    > * `ansible_connection=ssh` is mandatory when you are updating any remote hosts, which makes Ansible to connect via `ssh`.
    > * The above information is used by Ansible to establish ssh connection to the nodes.
    > * Control node will always be `ansible_connection=local`, **don't update the control node's information**.
    > * To deploy EII in single , `ansible_connection=local` and `ansible_host=localhost`.
    > * To deploy EII on multiple nodes, add hosts(worker1, worker2 etc..) details to the inventory file.


## Updating the EII Source Folder, Usecase & Proxy Settings in Group Variables

1. Open `group_vars/all.yml` file.
    ```sh
        $ vi group_vars/all.yml
    ```
2. Update Proxy Settings.
    ```sh
        enable_system_proxy: true
        http_proxy: <proxy_server_details>
        https_proxy: <proxy_server_details>
        no_proxy: <managed_node ip>,<controller node ip>,<worker nodes ip>,localhost,127.0.0.1
    ```
3. Update the `usecase` variable, based on the usecase `builder.py` generates the EII deployment & config files.
> **Note**:
> 1. By default it will be `video-streaming`.For other usecases refer the `../usecases` folder and update only names without `.yml` extension.
> 2. `ia_kapacitor` and `ia_telegraf` container images are not distributed via docker hub, so one won't be able to pull these images
>    for time-series use case upon using [../usecases/time-series.yml](../usecases/time-series.yml`) for deployment. For more details,
>    refer: [../README.md#distribution-of-eii-container-images]> (../README.md#distribution-of-eii-container-images).

    For example. If you want build & deploy for `../usecases/time-series.yml` update the `usecase` key value as `time-series`.
    ```sh
        usecase: <time-series>
    ```
4. Optionally you can choose number of video pipeline instances to be created by updating `instances` variable.
5. Update other optional variables provided if required.


## Non-orchestrated multi node deployment (without k8s)
 
The configuration changes shown below need to be made for multi node deployment without k8s

1. Set `multi_node: true` in `group_vars/all.yml` to enable multinode deployment and `false` to enable single node deployment. Update `vars/vars.yml` to the services to run on a specific node in case of multi_node deployment by following [this](#Select-EII-services-to-run-on-a-particular-node-in-multinode-deployment), where in single node deployment all the services based on the `usecase` chosen will be deployed. 
2. Update `docker` registry details in the following section when using a custom/private registry
    ``` sh
        docker_registry="<regsitry_url>"
        docker_login_user="<username>"
        docker_login_passwd="<password>"
    ```
    > **Note**: Use of `docker_registry` and `build` flags
    > * Update `docker_registry` details to use docker images from custom registry, optionally set `build: true` to push docker images to this registry.
    > * Unset `docker_registry` details if you don't want to use custom registry and set `build: true` to save and load docker images from one node to another node.
3. If you are using images from docker hub, then set `build: false` and unset `docker_registry` details.


## Select EII services to run on a particular node in multinode deployment

* Edit `vars/vars.yml` -> under `nodes` add a specific a node which was defined in the inventory file(`hosts`) and add EII services to `include_services` list.

    For example, if you want leader to run ia_video_ingestion, `vars/vars.yml` should be

    ```yml
        nodes:
          leader:
            include_services:
                - ia_video_ingestion
    ```

* If you want to add `worker1` to `nodes` and bring up `ia_visualizer` in `worker1`:

    ```yml
    nodes:
      leader:
        include_services:
            - ia_video_ingestion
      worker1:
        include_services:
            - ia_visualizer
    ```

> **Note**: If a service is not added to `include_services` list, that service will not deployed on a particular node


## Execute ansible Playbook from [EII_WORKDIR]/IEdgeInsights/build/ansible {Control node} to deploy EII services in single/multi nodes

 > **Note**: Updating messagebus endpoints to connect to interfaces is still the manual process. Make sure to update Application specific endpoints in `[AppName]/config.json`

**For Single Point of Execution**

   > **Note**: This will execute all the steps of EII as prequisite, build, provision, deploy & setup leader node for multinode deployement usecase in one shot sequentialy. 

    ```sh
    $ ansible-playbook eii.yml
    ```
  
 **The steps shown below are the individual execution of setups.**

* Load `.env` values from template
    ```sh
    $ ansible-playbook eii.yml --tags "load_env"
    ```

* For EII Prequisite Setup

    ```sh
    $ ansible-playbook eii.yml --tags "prerequisites"
    ```
 
* To generate builder and config files, build images and push to registry

    ```sh
    $ ansible-playbook eii.yml --tags "build"
    ```

* To generate Certificates in control node
    ```sh
    $ ansible-playbook eii.yml --tags "gen_certs"
    ```

* To generate provision bundle for leader

    ```sh
    $ ansible-playbook eii.yml --tags "gen_leader_provision_bundle"
    ```

* To generate provision bundle for worker nodes

    ```sh
    $ ansible-playbook eii.yml --tags "gen_worker_provision_bundle"
    ```

* To generate eii bundles for leader, worker nodes

    ```sh
    $ ansible-playbook eii.yml --tags "gen_bundles"
    ```

* Provision leader and bring up ETCD server

    ```sh
    $ ansible-playbook eii.yml --tags "leader_provision"
    ```

* Provision worker node

    ```sh
    $ ansible-playbook eii.yml --tags "worker_provsion"
    ```
* Provision EIS Config values to etcd

    ```sh
    $ ansible-playbook eii.yml --tags "etcd_provision"
    ```

* To generate deploy selected services to leader, worker nodes

    ```sh
    $ ansible-playbook eii.yml --tags "deploy"
    ```

### Deploying EII Using Helm in Kubernetes (k8s) environment

> **Note**
>   1. To Deploy EII using helm in k8s aenvironment, `k8s` setup is a prerequisite.
>   2. You need update the `k8s` leader machine as leader node in `hosts` file.
>   3. Non `k8s` leader machine the `helm` deployment will fail.
>   4. For `helm` deployment `ansible-multinode` parameters will not applicable, Since
>   node selection & pod selection will be done by `k8s` orchestrator.

* Update the `DEPLOYMENT_MODE` flag as `k8s` in `group_vars/all.yml` file:
    *   Open `group_vars/all.yml` file
    ```sh
        $ vi group_vars/all.yml
    ```
    *   Update the `DEPLOYMENT_MODE` flag as `k8s`

    ```yml
        ## Deploy in k8s mode using helm
        DEPLOYMENT_MODE: "k8s"
    ```

    * Save & Close

*  For Single Point of Execution
   > **Note**: This will execute all the steps of EII as prequisite, build, provision, deploy for a usecase in one shot sequentialy.

    ```sh
    $ ansible-playbook eii.yml
    ```

> **Note**: The steps shown below are the individual execution of setups.

* Load `.env` values from template
    ```sh
    $ ansible-playbook eii.yml --tags "load_env"
    ```

* For EII Prequisite Setup

    ```sh
    $ ansible-playbook eii.yml --tags "prerequisites"
    ```

* For building EII containers

    ```sh
    $ ansible-playbook eii.yml --tags "build"
    ```

* To generate Certificates in control node
    ```sh
    $ ansible-playbook eii.yml --tags "gen_certs"
    ```

* Prerequisites for deploy EII using Ansible helm environment
    ```sh
    $ ansible-playbook eii.yml --tags "helm_k8s_prerequisites"
    ```
* Provision & Deploy EII Using Ansible helm environment
    ```sh
    $ ansible-playbook eii.yml --tags "helm_k8s_deploy"
    ```
