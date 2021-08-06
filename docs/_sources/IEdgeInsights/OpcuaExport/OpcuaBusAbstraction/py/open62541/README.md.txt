# Compilation steps (current directory: <repo>/OpcuaBusAbstraction/py/open62541/):

## For building the open62541 wrapper library `open62541W.so` and generating `open62541W.c`:

```sh
make build
```

# Start server, publish and destroy 

```sh
make server
```

# Start client, subscribe and destroy

```sh
make client
```

# Remove all binaries/object files

```sh
make clean
```

