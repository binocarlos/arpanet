arpanet
=======

Auto linking multi-host docker cluster

![PDP-10](https://github.com/binocarlos/arpanet/raw/master/pdp-10.jpg)

Arpanet is a wrapper around the following tools:

 * [docker](https://github.com/docker/docker) for running containers
 * [consul](https://github.com/hashicorp/consul) for service discovery
 * [cadvisor](https://github.com/google/cadvisor) for container metrics
 * [ambassadord](https://github.com/progrium/ambassadord) for auto tcp routing
 * [registrator](https://github.com/progrium/registrator) for announcing services
 * [fleetstreet](https://github.com/binocarlos/fleetstreet) for publishing container info

It is an opinionated platform upon which you can create a Platform As A Service.

## quickstart

The quickstart list of commands:

### install

```bash
$ export ARPANET_IP=192.168.8.120
$ curl -sSL https://get.docker.io/ubuntu/ | sudo sh
$ sudo sh -c 'curl -L https://raw.githbusercontent.com/binocarlos/arpanet/v2.1.0/wrapper > /usr/local/bin/arpanet'
$ sudo chmod a+x /usr/local/bin/arpanet
$ sudo -E arpanet setup
$ arpanet pull
```
### run

```
Usage: arpanet COMMAND [options]

Commands:

  setup                                 Setup docker for use with arpanet
  pull                                  Pull the required docker images
  start boot|master|slave [JOINIP]      Start arpanet
  stop                                  Stop arpanet
```

## installation

#### 1. environment

The variables you should set in your environment before running the arpanet container:

##### `HOSTNAME`

Make sure the hostname of the machine is set correctly and is different to other hostnames on your arpanet.

##### `ARPANET_IP`

The IP address of the interface to use for cross host communication.

This should be the IP of a private network on the host.

```bash
$ export ARPANET_IP=192.168.8.120
```

#### 2. install docker

```bash
$ curl -sSL https://get.docker.io/ubuntu/ | sudo sh
```

#### 3. install wrapper

Arpanet runs in a docker container that starts and stops containers on the main docker host.

Because of this, the container must be run with the docker socket mounted as a volume.

There is a wrapper script that will handle this neatly - to install the wrapper:

```bash
$ curl -L https://raw.githubusercontent.com/binocarlos/arpanet/v0.2.4/wrapper > /usr/local/bin/arpanet
$ chmod a+x /usr/local/bin/arpanet
```

#### 4. pull image

Next - pull the arpanet image (optional - it will pull automatically in the next step):

```bash
$ docker pull binocarlos/arpanet
```

#### 5. setup

Run the setup command as root - it will create the data folder, configure the docker DNS bridge and bind it to the ARPANET_IP tcp endpoint:

```bash
$ sudo -E $(arpanet setup)
```

#### 6. pull service images

Finally pull the docker images for the various services:

```bash
$ arpanet pull
```

Everything is now installed - you can `arpanet start` and `arpanet stop`


## run

```
Usage: arpanet COMMAND [options]

Commands:

  setup                                 Setup docker for use with arpanet
  pull                                  Pull the required docker images
  start boot|master|slave [JOINIP]      Start arpanet
  stop                                  Stop arpanet
```

The arpanet script runs in a docker container - this means the docker socket must be mounted as a volume each time we run.

The wrapper script (installed to /usr/local/bin) will handle this for you.

Or, if you want to run arpanet manually - here is an example of pretty much what the wrapper script does:

```bash
$ docker run --rm \
	-h $HOSTNAME \
	-v /var/run/docker.sock:/var/run/docker.sock \
	-e ARPANET_IP \
	binocarlos/arpanet help
```

## api

#### `arpanet setup`

```bash
$ sudo -E arpanet setup
```

This should be run as root and will perform the following steps:

 * bind docker to listen on the the tcp://$ARPANET_IP interface
 * connect the docker DNS resolver to consul
 * create a host directory for the consul data volume
 * restart docker

#### `arpanet pull`

This will pull the images used by arpanet services.

```bash
$ arpanet pull
```

#### `arpanet start boot|master|slave [JOINIP]`

Start the arpanet containers on this host.

There are 3 modes to boot a node:

 * boot - used for the very first node
 * master - used for other masters (consul server)
 * slave - used for other nodes (consul agent)

```bash
$ arpanet start master 192.168.8.120
```

#### `arpanet stop`

Stop the arpanet containers.

```bash
$ arpanet stop
```

## booting a cluster

Boot a cluster of 5 nodes, with 3 server and 2 client nodes.

First stash the ip of the first node - we will 'join' the other nodes to here and the consul gossip protocol will catch up.

```bash
$ export JOINIP=192.168.8.120
```

Then boot the first node:

```bash
$ arpanet start boot
```

Now - boot the other 2 masters:

```
$ ssh node2 arpanet start master $JOINIP
$ ssh node3 arpanet start master $JOINIP
```

Then finally the 2 other slaves:

```
$ ssh node4 arpanet start slave $JOINIP
$ ssh node5 arpanet start slave $JOINIP
```

## config

there are other environment variables that control arpanet behaviour:

 * DOCKER_PORT - the TCP port docker should listen on (2375)
 * CADVISOR_PORT - the port to expose for the cadvisor api (8080)
 * CONSUL_PORT - the port to expose the consul HTTP api (8500)
 * CONSUL_EXPECT - the number of server nodes to auto bootstrap (3)
 * CONSUL_DATA - the host folder to mount for consul state (/mnt/arpanet-consul)
 * CONSUL_KV_PATH - the Key/Value path to use to keep state (/arpanet)

You can control the images used by arpanet services using the following variables:

 * CONSUL_IMAGE (progrium/docker-consul)
 * CADVISOR_IMAGE (google/cadvisor)
 * REGISTRATOR_IMAGE (progrium/registrator)
 * AMBASSADORD_IMAGE (binocarlos/ambassadord) - will change to progrium
 * FLEETSTREET_IMAGE (binocarlos/fleetstreet)

You can control the names of the launched services using the following variables:

 * CONSUL_NAME (arpanet_consul)
 * CADVISOR_NAME (arpanet_cadvisor)
 * REGISTRATOR_NAME (arpanet_registrator)
 * AMBASSADOR_NAME (arpanet_backends)
 * FLEETSTREET_NAME (arpanet_fleetstreet)

The wrapper will source these variables from `~/.arpanetrc` and will inject them all into the arpanet docker container.

If you are running arpanet manually then pass these variables to docker using `-e CONSUL_NAME=...`.

## security

At present $ARPANET_IP is expected to reside on a private network.

This prevents the multi data-centre approach for consul.

It is recommended that you use iptables to secure access between arpanet nodes preventing other servers on the private network gaining access to your nodes.

Future versions of arpanet will include TLS encryption and multi data-center support (as consul allows this).

## wishlist

 * TLS encryption between consul nodes & for docker server

## big thank you to

 * [Jeff Lindsay](https://github.com/progrium)
 * [the docker team](https://github.com/docker/docker/graphs/contributors)
 * [the coreos team](https://github.com/coreos/etcd/graphs/contributors)
 

## license

MIT
