arpanet
=======

Auto linking multi-host docker cluster

![PDP-10](https://github.com/binocarlos/arpanet/raw/master/pdp-10.jpg)

## install

```
$ wget -qO- https://raw.github.com/binocarlos/arpanet/master/bootstrap.sh | sudo bash
```

## usage

arpanet runs with `master` and `slave` nodes and a client that proxies docker client commands against the masters.

The masters route containers back to a choosen slave and will account for --links between containers with an auto-routing tcp proxy.

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

## master

arpanet masters are etcd peers - one can go down and everything will still work.

For this reason - you should run the arpanet master on at least 3 machines.

To boot the first master:

```bash
$ arpanet master start --peers boot
```

To boot the subsequent masters point them at the IP of the first:

```bash
$ arpanet master start --peers 192.168.8.120
```

The masters will now have formed an etcd mesh and any of them can be stopped without loss of service.

## slave

To start a slave:

```bash
$ arpanet slave start
```

## docker client

The standard arpanet master port is 8791 and you can point the standard docker client at it:

```bash
$ docker -H tcp://192.168.8.120:8791 run --name test1 --rm binocarlos/bring-a-ping
```

You must give containers a name when you run them via the arpanet cluster - this is to ensure uniqueness across machines.

You can docker ps and it will show containers across the whole cluster:

```bash
$ docker -H tcp://192.168.8.120:8791 ps
```

Container names are appended with @hostname

Using the standard docker client - you must define links and volumes using envrionment variables:

```bash
$ docker -H tcp://192.168.8.120:8791 run --name test2 --rm -e ARPANET_LINK1=test1:test1 -e ARPANET_VOLUME1=/data binocarlos/bring-a-ping
```

This is because the routing is done based on the volumes and links and the standard client sends this with /container/start not /container/create

## license

MIT