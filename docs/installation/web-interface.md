---
layout: default
title: "Web Interface"
permalink: /installation/ui
parent: Installation
nav_order: 2
---

# Move2Kube Web Interface

Choose the version for your use case:

## Stable Release (for production use):

Using `docker`:

```shell
$ mkdir -p workspace
$ cd workspace
$ docker run --rm -it -p 8080:8080 \
   -v "${PWD}:/workspace" \
   -v //var/run/docker.sock:/var/run/docker.sock \
   quay.io/konveyor/move2kube-aio:release-0.2
```

Using `podman`:

```shell
$ mkdir -p workspace 
$ cd workspace
$ podman run --rm -it -p 8080:8080 \
   -v "${PWD}:/workspace:z" \
   quay.io/konveyor/move2kube-aio:release-0.2
```

## Latest (if you need bleeding edge features and also for development and testing use):

Using `docker`:

```shell
$ docker run --rm -it -p 8080:8080 quay.io/konveyor/move2kube-ui:latest
```

Optionally if you need persistence then mount the current directory:

```shell
$ docker run --rm -it -p 8080:8080 \
   -v "${PWD}/move2kube-api-data:/move2kube-api/data" \
   quay.io/konveyor/move2kube-ui:latest
```

And if you also need more advanced features of Move2Kube then mount the docker socket. This will allow Move2Kube to run container based transformers:

```shell
$ docker run --rm -it -p 8080:8080 \
   -v "${PWD}/move2kube-api-data:/move2kube-api/data" \
   -v //var/run/docker.sock:/var/run/docker.sock \
   quay.io/konveyor/move2kube-ui:latest
```

Using `podman`:

```shell
$ podman run --rm -it -p 8080:8080 quay.io/konveyor/move2kube-ui:latest
```

Access the UI in `http://localhost:8080/`.

   >
      Note: There is a known issue when mounting directories in WSL.  
      Some empty directories may be created in the root directory.  
      If you are on Windows, use Powershell instead of WSL until this is fixed.

## Bringing up Move2Kube UI as Helm Chart  

Move2Kube can also be installed as a Helm Chart from [ArtifactHub](https://artifacthub.io/packages/helm/move2kube/move2kube/0.2.0-beta.0?modal=install)

Also, for Helm Chart and Operator checkout [Move2Kube Operator](https://github.com/konveyor/move2kube-operator).