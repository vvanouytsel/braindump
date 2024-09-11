#linux #networking 

NetworkManager is daemon used to provide networking services to a Linux based machine.

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

* Show physical state of interfaces

```bash
$ ip link show
```

* Show configuration of interfaces

```bash
$ ip addr show
```

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