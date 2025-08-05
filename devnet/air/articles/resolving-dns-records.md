# Resolving DNS records in Adobe AIR 2

by William Liang

![William Liang](./img/1296445272799.jpg)

## Requirements

### Prerequisite knowledge

This article is intended for developers who are comfortable with ActionScript
and who have a basic understanding of networking and DNS.

### User level

All

### Required products

- [Adobe AIR SDK](https://airsdk.dev/) or
  [Apache Flex SDK](https://flex.apache.org)

Among the new networking features introduced in Adobe AIR 2 is the Domain Name
System (DNS) resolver. DNS is a naming system in which network resources such as
domains, servers, services, and so on are mapped to information such as IP
addresses. The new DNS resolver enables DNS resolution within an AIR application
and helps to facilitate networking related operations, including the
establishment of network connections.

### DNS resolver use cases

To end users, DNS resolution is generally a transparent feature of a
network-enabled application. Web browsers, email clients, and other similar
applications are just a few examples of applications that rely on DNS.

For example, when a user types the name of a web site in a browser, they are
actually typing a Uniform Resource Locator (URL) containing a domain name. The
browser contains a DNS resolver feature that allows it to perform a DNS lookup
on the domain name. The result of the lookup is an IP address that the browser
uses to request and load the content from the website.

In AIR 2, you can now incorporate DNS resolution within your application. Using
the new DNSResolver class, you can build familiar applications such as web
browsers and email clients, or you can build applications with more
sophisticated features such as IP telephony or peer-to-peer file sharing.

### DNS records

The new DNSResolver class resolves domain names by performing a standard DNS
query. The result of the DNS query is a DNS response containing the DNS records
for that query. The information contained within the DNS records will depend on
the type of the DNS resource record. DNS supports many different records types.
The DNSResolver class supports the following subset of record types:

- **A:** An A record maps an IPv4 address to a hostname. It contains the
  hostname, time-to-live (TTL), and IPv4 IP address.
- **AAAA:** An AAAA record maps an IPv6 address to a hostname. It contains the
  hostname, TTL, and IPv6 IP address.
- **MX:** An MX record maps a list of mail servers to a domain name. It contains
  the hostname, TTL, exchange server, and preference.
- **PTR:** A PTR record maps a hostname to an IP address. This is essentially a
  reverse DNS lookup. A PTR record contains the hostname, TTL, and a pointer to
  the host.
- **SRV:** An SRV record maps a list of services to a domain name. It contains
  the hostname, TTL, priority, weight, port, and target domain.

### DNSResolver example

To enable DNS resolution within an AIR application, you will need to create a
DNSResolver object and add an event listener to handle lookup events. The lookup
event will return records from a DNS response. An example of this is shown
below:

    import flash.net.dns.DNSResolver;
    import flash.net.dns.ARecord;
    import flash.net.dns.AAAARecord;
    import flash.net.dns.MXRecord;
    import flash.net.dns.PTRRecord;
    import flash.net.dns.SRVRecord;

    var dnsResolver:DNSResolver = new DNSResolver();

    public function init():void
    {
    	dnsResolver.addEventListener("lookup", lookupHandler);
    }

    public function lookupHandler(event:DNSResolverEvent):void
    {
    	var records:Array = new Array();
    	records = event.resourceRecords;
    	var name:String = "Name: " + records[0].name;
    	var ttl:String = "TTL: " + records[0].ttl;

    	if (records[0] is ARecord)
    	{
    		var addr:String = "Addr: " + records[0].address;
    	}
    	elseif (records[0] is MXRecord)
    	{
    		var exchange:String = "Exchange: " + records[0].exchange;
    		var preference:String = "Preference: " + records[0].preference;
    	}
    	elseif (records[0] is PTRRecord)
    	{
    		var ptr:String = "PTR: " + records[0].ptrdName;
    	}
    	elseif (records[0] is SRVRecord)
    	{
    		var priority:String = "Priority: " + records[0].priority;
    		var weight:String = "Weight: " + records[0].weight;
    		var port:String = "Port: " + records[0].port;
    		var target:String = "Target: " + records[0].target;
    	}
    }

#### An A record example

To make an A record request, call the DNSResolver `lookup` function with the
hostname and record type as shown below.

    public function lookupA():void
    {
    	dnsResolver.lookup("echotest.adobepacifica.net", ARecord);
    }

In this example, the DNS response would be the following:

    Name: echotest.adobepacifica.net
    TTL: 7200
    Addr: 192.168.0.1

#### AAAA record example

To make an AAAA record request, you can use the example above for making an A
record request, but specify `AAAARecord` as the type in place of `ARecord`.

#### MX record example

To make an MX record request, call the DNSResolver `lookup` function with the
domain name and record type as shown below.

    public function lookupMX():void
    {
    	dnsResolver.lookup("echotest.adobepacifica.net", MXRecord);
    }

In this example, the DNS response would be the following:

    Name: echotest.adobepacifica.net
    TTL: 528
    Exchange: mail.echotest.adobepacifica.net
    Preference: 10

#### PTR record example

To make a PTR record request, call the DNSResolver `lookup` function with the IP
address and record type as shown below.

    public function lookupPTR():void
    {
    	dnsResolver.lookup("65.49.27.91", PTRRecord);
    }

In this example, the DNS response would be the following:

    Name: 91.subnet64.27.49.65.in-addr.arpa
    TTL: 557721
    PTR: reverse.echotest.adobepacifica.net

#### SRV record example

To make an SRV record request, call the DNSResolver `lookup` function with the
service name and record type as shown below.

    public function lookupSRV():void
    {
    	dnsResolver.lookup("_sip._udp.adobepacifica.net", SRVRecord);
    }

In this example, the DNS response would be the following:

    Name: _sip._udp.adobepacifica.net
    TTL: 14071
    Priority: 0
    Weight: 1
    Port: 5060
    Target: adobepacifica.net
