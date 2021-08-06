# `ETCD UI Service`

* Open your browser and enter the address: https://< host ip >:7071/etcdkeeper/ (when EII is running in secure mode). In this case, CA cert has to be imported in the browser. For insecure mode i.e. DEV mode, it can be accessed at http://< host ip >:7070/etcdkeeper/.
* Click on the version of the title to select the version of ETCD. The default is V3. Reopening will remember your choice.
* Right click on the tree node to add or delete.
* For secure mode, authentication is required. User name and password needs to be entered in the dialogue box.
* Username is 'root' and default password is located at ETCD_ROOT_PASSWORD key under environment section in [build/provision/dep/docker-compose-provision.override.prod.yml](../build/provision/dep/docker-compose-provision.override.prod.yml).
* This service can accessed from a remote system at address: https://$(HOST_IP):7071 (when EII is running in secure mode). In this case, CA cert has to be imported in the browser. For insecure mode i.e. DEV mode, it can be accessed at http://$(HOST_IP):7071

---
**NOTE**:
1. If ETCD_ROOT_PASSWORD is changed, EII must to be provisioned again. Please follow below command in < EII Repo >/build/provision folder to reprovision.

        ```
        $ sudo ./provision.sh <path_to_eii_docker_compose_file>

        eq. $ sudo ./provision.sh ../docker-compose.yml

        ```
2. Only VideoIngestion and VideoAnalytics based services will have watch for any changes. Any changes done to those keys will be reflected at runtime in EII.
3. For changes done to any other keys, EII stack needs to be restarted for it to take effect. Please execute below command in working directory build/ to restart EII.
    ```
    $ docker-compose down
    $ docker-compose up
    
    ```
---
