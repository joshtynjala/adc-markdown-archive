# Retrieving a list of network interfaces in Adobe AIR 2

by William Liang

![William Liang](./img/1296445272799.jpg)

## Requirements

### Prerequisite knowledge

This article is intended for developers who are comfortable with ActionScript
and who have a basic understanding of networking.

### User Level

All

### Required products

- [Adobe AIR SDK](https://airsdk.dev/) or
  [Apache Flex SDK](https://flex.apache.org)

Among the new networking features introduced in Adobe AIR 2 is the ability to
enumerate all hardware and software network interfaces. This list of network
interfaces includes information about each interface such as IP address, MAC
address, status, and more.

This article describes the new NetworkInfo class and the information it
provides; you can use this information to implement peer-to-peer features and
advanced networking applications.

### NetworkInfo use cases and class overview

Enumerating all hardware and software network interfaces reveals detailed
information about the networking capabilities of the local machine. This
information is required for many advanced networking features. For example,
peer-to-peer and Voice-over-IP (VoIP) applications frequently need to be able to
traverse Network Address Translation (NAT) gateways. In order to traverse a NAT,
an application typically needs the list of available network interfaces. The
application can then probe each interface to determine if it can be used to
traverse the NAT and establish a connection with a remote client.

Peer-to-peer applications often also need to obtain a list of available
interfaces and their capabilities. For example, when a Session Initiation
Protocol (SIP) client wants to establish a connection with another SIP client,
the two clients need to exchange their capabilities. These capabilities include
the networking capabilities of the local machine and the networking between
them.

In AIR 2, you can now incorporate network information within your application,
so you can implement standard networking or peer-to-peer features, such as the
ones listed above, or develop new ones.

#### NetworkInfo class overview

The new NetworkInfo class is analogous to `ipconfig` on Windows and `ifconfig`
on Mac OS X or Linux. NetworkInfo returns a list of network interfaces and the
following information for each interface:

- Name
- Display name
- MTU
- Hardware address (or MAC address)
- Active (or status)
- Address (or IP address)
- Broadcast address
- Prefix length
- IP version (or IP Family)

#### Using the NetworkInfo class

NetworkInfo is a singleton class. The code below is a simple example of how to
use it to display information about the local machine's network interfaces.

    import flash.net.NetworkInfo;

    public function findInterface():void
    {
        var results:Vector.<NetworkInterface> =
           NetworkInfo.networkInfo.findInterfaces();

        for (var i:int=0; i<results.length; i++)
        {
            var output = output
            + "Name: " + results[i].name + "\n"
            + "DisplayName: " + results[i].displayName + "\n"
            + "MTU: " + results[i].mtu + "\n"
            + "HardwareAddr: " + results[i].hardwareAddress + "\n"
            + "Active: "  + results[i].active + "\n";


            for (var j:int=0; j<results[i].addresses.length; j++)
            {
               output = output
               + "Addr: " + results[i].addresses[j].address + "\n"
               + "Broadcast: " + results[i].addresses[j].broadcast + "\n"
               + "PrefixLength: " + results[i].addresses[j].prefixLength + "\n"
               + "IPVersion: " + results[i].addresses[j].ipVersion + "\n";
            }

            output = output + "\n";
        }
    }

If you run this example on an Apple MacBook Pro, you will get a list of
interfaces that minimally contains en0 and en1; for example:

    Name: en0
    DisplayName:
    MTU: 1500
    HardwareAddr: 00:25:00:a5:35:1e
    Active: true
    Addr: 2001:1890:110b:1498:225:ff:fea5:351e
    Broadcast:
    PrefixLength: 64
    IPVersion: IPv6
    Addr: 153.32.154.241
    Broadcast: 153.32.155.255
    PrefixLength: 22
    IPVersion: IPv4

    Name: en1
    DisplayName:
    MTU: 1500
    HardwareAddr: 00:23:6c:96:ab:c1
    Active: true
    Addr: 10.4.217.218
    Broadcast: 10.4.219.255
    PrefixLength: 22
    IPVersion: IPv4

### The network change event

Another useful feature of the NetworkInfo class is the ability to detect network
changes. A network change event occurs when a network interface becomes enabled
or disabled.

For example, an application may use the NetworkInfo class to obtain a list of
currently available network interfaces and select the best candidate interface
from that list. Later, if another interface becomes enabled, the application
will be notified via a network change event. It can then use that opportunity to
determine if the recently enabled interface is a better candidate and
reestablish the network connection using the new interface.

Similarly, if an interface becomes disabled or disconnected from the network,
the application will receive a network change event, enabling it to take the
appropriate action. For example, it may terminate the connection or select
another interface and reestablish the connection.

The network change event is fired from the NativeApplication class and bubbled
up by the NetworkInfo class. To receive network change events, an application
must add an event listener:

    NetworkInfo.networkInfo.addEventListener(Event.NETWORK_CHANGE, onNetworkChange);

Alternatively, you can add the event listener on your NativeApplication object:

    NativeApplication.nativeApplication.addEventListener(Event.NETWORK_CHANGE, onNetworkChange);
