# EventMesh Runtime (Docker)

The documentation introduces the steps to install the latest release of EventMesh Runtime with Docker and connect to Apache RocketMQ. It's recommended to use a Linux-based system with [Docker Engine](https://docs.docker.com/engine/install/). Please follow the [Docker tutorial](https://docs.docker.com/get-started/) to get familiar with the basic concepts (registry, volume, etc.) and commands of Docker.

## Dependencies

```
64-bit OS，we recommend Linux/Unix；
64-bit JDK 1.8+;
Gradle 7.0+, we recommend 7.0.*
4g+ available disk to deploy eventmesh-store
If you choose standalone mode, you could skip this file and go to the next step: Start Eventmesh-Runtime; if not, you could choose RocketMQ as the store layer.
```

## Pull EventMesh Image

Download the pre-built image of [`eventmesh`](https://hub.docker.com/r/eventmesh/eventmesh) from Docker Hub with `docker pull`:

```console
sudo docker pull eventmesh/eventmesh:v1.4.0
```

To verify that the `eventmesh/eventmesh` image is successfully installed, list the downloaded images with `docker images`:

```console
$ sudo docker images
eventmesh/eventmesh           v1.4.0    6e2964599c78   2 weeks ago    937MB
```

## Edit Configuration

Edit the `eventmesh.properties` to change the configuration (e.g. TCP port, client blacklist) of EventMesh Runtime. To integrate RocketMQ as a connector, these two configuration files should be created: `eventmesh.properties` and `rocketmq-client.properties`.

```shell
sudo mkdir -p /data/eventmesh/rocketmq/conf
cd /data/eventmesh/rocketmq/conf
sudo touch eventmesh.properties
sudo touch rocketmq-client.properties
```

### `eventmesh.properties`

The `eventmesh.properties` file contains the properties of EventMesh runtime environment and integrated plugins. Please refer to the [default configuration file](https://github.com/apache/incubator-eventmesh/blob/master/eventmesh-runtime/conf/eventmesh.properties) for the available configuration keys.

```shell
sudo vim eventmesh.properties
```

| Configuration Key | Default Value |  Description |
|-|-|-|
| `eventMesh.server.http.port` | 10105 | EventMesh HTTP server port |
| `eventMesh.server.tcp.port` | 10000 | EventMesh TCP server port  |
| `eventMesh.server.grpc.port` | 10205 | EventMesh gRPC server port |

### `rocketmq-client.properties`

The `rocketmq-client.properties` file contains the properties of the Apache RocketMQ nameserver.

```shell
sudo vim rocketmq-client.properties
```

Please refer to the [default configuration file](https://github.com/apache/incubator-eventmesh/blob/1.3.0/eventmesh-runtime/conf/rocketmq-client.properties) and change the value of `eventMesh.server.rocketmq.namesrvAddr` to the nameserver address of RocketMQ.

| Configuration Key | Default Value | Description |
|-|-|-|
| `eventMesh.server.rocketmq.namesrvAddr` | `127.0.0.1:9876;127.0.0.1:9876` | The address of RocketMQ nameserver |

## Run and Manage EventMesh Container

Run an EventMesh container from the `eventmesh/eventmesh` image with the `docker run` command. The `-p` option of the command binds the container port with the host machine port. The `-v` option of the command mounts the configuration files from files in the host machine.

```shell
sudo docker run -d -p 10000:10000 -p 10105:10105 \
-v `pwd`/data/eventmesh/rocketmq/conf/eventmesh.properties:/data/app/eventmesh/conf/eventmesh.properties \
-v `pwd`/data/eventmesh/rocketmq/conf/rocketmq-client.properties:/data/app/eventmesh/conf/rocketmq-client.properties \
eventmesh/eventmesh:v1.4.0
```

The `docker ps` command lists the details (id, name, status, etc.) of the running containers. The container id is the unique identifier of the container.

```console
$ sudo docker ps
CONTAINER ID     IMAGE                        COMMAND                  CREATED              STATUS              PORTS                                                                                          NAMES
<container_id>   eventmesh/eventmesh:v1.4.0   "/bin/sh -c 'sh star…"   About a minute ago   Up About a minute   0.0.0.0:10000->10000/tcp, :::10000->10000/tcp, 0.0.0.0:10105->10105/tcp, :::10105->10105/tcp   <container_name>
```

To connect to the EventMesh container:

```shell
sudo docker exec -it [container id or name] /bin/bash
```

To read the log of the EventMesh container:

```shell
tail -f ../logs/eventmesh.out
```

To stop or remove the container:

```shell
sudo docker stop [container id or name]

sudo docker rm -f [container id or name]
```
