#linux #networking 

## Naming convention

By default interfaces are named using a pre-defined naming conventions. This is handled by `udevd`.

* Firmware
	* en: ethernet interface
	* wl: wlan interface
	* ww: wwan interface
* Type, location or topology 
	* o: onboard card
	* s: hotplug slot
	* p: pci card
* ID/Index/Port

For example:
* eno16777737
	* Ethernet interface
	* Onboard card
	* 16777737 is an unique ID

# ip

The `ip` command allows you to deal with networking interfaces. It should be available on most Linux distributions. Alternatively it can be installed via the `iproute` package.

## Commands

* Show physical state of interfaces

```bash
$ ip link show
```

* Show configuration of interfaces

```bash
$ ip addr show
# NetworkManager
```

# NetworkManager

NetworkManager is daemon used to provide networking services to a Linux based machine.

References:
* [What NetworkManager is](https://www.youtube.com/watch?v=EfILlEkmF9k)
* [Network Manager on Linux using nmcli](https://www.youtube.com/watch?v=X6_9aS5-cT8)
* [man pages](https://networkmanager.dev/docs/man-pages/)

## Concepts

NetworkManager uses a few key concepts. These are explained below.

### Devices

A device is a representation of a network interface. In order to use a `device` to connect to the network, you will need a `connection`.

### Connections

NetworkManager stores all network configuration as "connections", which are collections of data (Layer2 details, IP addressing, etc.) that describe how to create or connect to a network. A connection is "active" when a device uses that connection's configuration to create or connect to a network. There may be multiple connections that apply to a device, but only one of them can be active on that device at any given time. The additional connections can be used to allow quick switching between different networks and configurations.

Consider a machine which is usually connected to a DHCP-enabled network, but sometimes connected to a testing network which uses static IP addressing. Instead of manually reconfiguring eth0 each time the network is changed, the settings can be saved as two connections which both apply to eth0, one for DHCP (called `default`) and one with the static addressing details (called `testing`). When connected to the DHCP-enabled network the user would run **nmcli con up default** , and when connected to the static network the user would run **nmcli con up testing**.

## Commands

```bash

* Show status all devices

```bash
$ nmcli device status
```

* Show configuration of a specific device

```bash
$ nmcli -p device show enp193s0f0np0
```

* List all connections

```bash
$ nmcli connection show
```

* Modify an existing connection

```bash
$ nmcli connection modify my-connection ipv4.addresses 192.168.0.10/24 ipv4.method manual
```

* Bring a connection up

```bash
$ nmcli connection up my-connection
```

# Bridge

A networking bridge is a way to connect multiple interfaces together and treat them as a single logical unit.

## Commands

* List all interfaces connected to a bridge

```bash
# Take note of the 'master' keyword.
# This indicates the bridge that the listed interface belongs to.

$ bridge link show   
13: virbr0-nic state DOWN : <BROADCAST,MULTICAST> mtu 1500 master virbr0 state disabled priority 32 cost 100 
14: vnet0 state UNKNOWN : <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4091 state forwarding priority 32 cost 100 
15: vnet1 state UNKNOWN : <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4090 state forwarding priority 32 cost 100 
271: enp98s0f0.4089 state UP @enp98s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4089 state forwarding priority 32 cost 4 
271: enp98s0f0.4089 state UP @enp98s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4089 
273: enp98s0f0.4090 state UP @enp98s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4090 state forwarding priority 32 cost 4 
273: enp98s0f0.4090 state UP @enp98s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4090 
274: enp98s0f0.4091 state UP @enp98s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4091 state forwarding priority 32 cost 4 
274: enp98s0f0.4091 state UP @enp98s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4091 
334: vnet3 state UNKNOWN : <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4090 state forwarding priority 32 cost 100 
366: vnet4 state UNKNOWN : <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4090 state forwarding priority 32 cost 100 
412: vnet2 state UNKNOWN : <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4090 state forwarding priority 32 cost 100 
414: vnet5 state UNKNOWN : <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 master br0.4090 state forwarding priority 32 cost 100 
```