arpanet
=======

Auto linking multi-host docker cluster

![PDP-10](https://github.com/binocarlos/arpanet/raw/master/pdp-10.jpg)

Arpanet is a wrapper around the following tools:

 * [docker](https://www.docker.com/)
 * [etcd](https://github.com/coreos/etcd)
 * [ambassadord](https://github.com/progrium/ambassadord)
 * [registrator](https://github.com/progrium/registrator)

## install

First you must install arpanet itself:

```
$ wget -qO- https://raw.github.com/binocarlos/arpanet/master/bootstrap.sh | sudo bash
```

Then you must ensure your environment is setup (see below).

Then install arpanet-core (which basically installs docker + binding it to tcp port):

```
$ sudo arpanet install core
```

Then depending if your node is a master, slave or both:

```
$ arpanet install master
```

```
$ arpanet install slave
```

## usage

arpanet runs with `master` and `slave` nodes.

The master nodes are etcd servers and the slave

## envrionment

The variables you should set in your environment before running the arpanet master or slave:

#### `HOSTNAME`

Make sure the hostname of the machine is set correctly and is different to other hostnames on your arpanet.

```bash
$ export HOSTNAME=host1
```

#### `ARPANET_IP`

The IP address of the interface to use for cross host communication.

This should be the IP of the private network of each host.

```bash
$ export ARPANET_IP=192.168.8.120
```

#### `ARPANET_MASTERS`

A comma delimited list of at least 2 of the arpanet masters on the network (the rest be found in the mesh)

```bash
$ export ARPANET_MASTERS=192.168.8.120,192.168.8.121,192.168.8.122
```

#### misc

there are other optional variables that control arpanet behaviour:

 * DOCKER_URL - the url of the script to install docker (https://get.docker.io/ubuntu/)
 * DOCKER_PORT - the TCP port docker should listen on (2375)
 * ETCD_PORT - the TCP port etcd client connection should listen on (4001)
 * ETCD_PEERPORT - the TCP port etcd peer connection should listen on (7001)
 * ETCD_PATH - the base path in etcd arpanet will keep state (/arpanet)

arpanet will source these variables from:

```
~/.arpanetrc
```

## master

arpanet masters are etcd peers - one can go down and everything will still work.

For this reason - you should run the arpanet master on at least 3 machines.

To boot the first master:

```bash
$ arpanet master start --peers boot
```

To boot the subsequent masters point them at the IP of the first:

```bash
$ arpanet master start --peers 192.168.8.120:7001
```

The masters will now have formed an etcd mesh and any of them can be stopped without loss of service.

#### tokens

You can also boot the arpanet masters using the etcd token service.

First - get a token:

```
$ curl -L https://discovery.etcd.io/new
```

Then - pass the token that is printed to the masters start commands:

```
$ arpanet master start --token https://discovery.etcd.io/b34c47fbc5300409d8c4d557b40a5bce
```

## slave

To start a slave:

```bash
$ arpanet slave start
```

This will boot [registrator](https://github.com/progrium/registrator) and [ambassadord](https://github.com/progrium/ambassadord) on the host and connect it up to the arpanet masters.

I will write more about how to use this setup - in the meantime you can checkout the help for the 2 libraries above on github.

## license

MIT