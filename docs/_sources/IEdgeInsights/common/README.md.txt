# eii_libs_installer

This script installs all the EII libraries & their respective required dependencies.

## Running the install script

**Note** : 
- The installer needs specific versions of grpc and protobuf to be installed. If these libraries are already present in the /usr/local/lib, the installation will skip build and install of these. It is recommended to remove grpc before proceeding as it can cause version conflicts.

```sh
    $ sudo apt-get remove --auto-remove --purge -y grpc
```

- Also, make sure all the eii occurances are removed from `/usr/local/lib` and `/usr/local/include`

#### Steps:

1. To install all of EII libraries and their dependencies, run the command mentioned below

    ```sh
        $ sudo -E CMAKE_INSTALL_PREFIX="/opt/intel/eii" ./eii_libs_installer.sh
    ```

**Note**: If an error occurs during execution of eii_libs_installer.sh regarding the $GOPATH, please set the GOPATH appropriately where GO is installed. Ideally, above script will take care of GO installation and setting up the path. If GO is already installed and GOPATH is tampered, then user has to make sure GOPATH is set appropriately.

2. Incase of unforeseen errors leading to unsuccessful installation, run the command mentioned below to cleanup any/all untracked tar files

    ```sh
        $ sudo git clean -xdf
    ```

3. Please follow the below steps to set the required env variables.

    a. For updating the $LD_LIBRARY_PATH env variable:
    ```sh
        export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/eii/lib
    ```

    **Note**: `$LD_LIBRARY_PATH` should be appended by `$CMAKE_INSTALL_PREFIX/lib`. In the above installation CMAKE_INSTALL_PREFIX is `/opt/intel/eii` and hence LD_LIBRARY_PATH appended with /opt/intel/eii/lib

    b. For updating the $no_proxy env variable to connect to etcd:
    ```sh
        export no_proxy=$no_proxy,127.0.0.1
    ```