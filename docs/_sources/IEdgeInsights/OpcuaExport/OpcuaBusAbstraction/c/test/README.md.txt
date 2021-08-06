# OpcuaBusAbstraction

OpcuaBusAbstraction abstracts underlying messagebus to provide a common set of APIs needed for publish and subscribe.
The C example program demonstrates publish and subscription over OPCUA bus only (`OPCUA is the only messagebus supported for now`)

## How to Test from present working directory

### 1. Pre-requisite

> **NOTE**:
> The `Pre-requisite` section below is `only` needed if executing from
> the EII repo.
  ```sh
  sudo apt-get install libmbedtls-dev
  make clean
  make build_safestring_lib
  ```

### 2. Testing with security enabled

* Start publisher, publish and destroy

```sh
make pub
```

* Start subscriber, subscribe and destroy

```sh
make sub
```

### 3. Testing with security disabled

* Start publisher, publish and destroy

```sh
make pub_insecure
```

* Start subscriber, subscribe and destroy

```sh
make sub_insecure
```

### 4. Remove all binaries/object files

```sh
make clean
```


```
