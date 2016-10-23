---
layout: post
title:  "Capturing DNS Packets with GO"
date:   2016-10-10 11:00:00
---

If you're reading this post, chances are that you've heard or used [Wireshark][wireshark] or [TCPDump][libpcap] before. In this post, I'm going to explain how to use and extract packet layers.

### GOPacket

GOPacket is a library that provides packet decoding capabilities for GO. It contains many sub-packages that provides additional support.

* layers - This packet contains logic to decode different protocols.
* pcap - C bindings to use [libcap][libpcap]
* pfring - C bindings for [pr_ring][PR_RING]
* afpacket - C bindings for [AF_PACKET][afpacket]

and so on...

### Implementation:

```golang
package main

import (
	"fmt"
	"log"
	"os"
	"time"

	"github.com/google/gopacket"
	"github.com/google/gopacket/layers"
	"github.com/google/gopacket/pcap"
)

func main() {
	if len(os.Args) < 2 {
		log.Fatal("You need to specify the network interface")
		os.Exit(-1)
	}

	device := os.Args[1]

	fmt.Println("Capturing dns packets from interface " + device)

	handle, err := pcap.OpenLive(device, 1024, false, -1*time.Second)
	if err != nil {
		log.Fatal(err)
	}
	defer handle.Close()

	packetSource := gopacket.NewPacketSource(handle, handle.LinkType())
	for packet := range packetSource.Packets() {
		dnsLayer := packet.Layer(layers.LayerTypeDNS)
		if dnsLayer != nil {
			ipLayer4 := packet.Layer(layers.LayerTypeIPv4)
			if ipLayer4 != nil {
				ip, _ := ipLayer4.(*layers.IPv4)
				fmt.Printf("From %s to %s\n", ip.SrcIP, ip.DstIP)
			}

			ipLayer6 := packet.Layer(layers.LayerTypeIPv6)
			if ipLayer6 != nil {
				ip, _ := ipLayer6.(*layers.IPv6)
				fmt.Printf("From %s to %s\n", ip.SrcIP, ip.DstIP)
			}
		}
	}
}

```


### Reading Live Packets

We use `pcap.OpenLive` to read data from a live network interface - it accepts 4 parameters: 

* interface name
* maximum size to read for each packet - in our case 1024, you can also specify `-1` to read the whole packet.
* whether to put the interface in promiscuous mode
* timeout - in our case, we're using `-1` which tells the interface to read indefinitely.

### Decoding a packet

In your example, we're looking for DNS packets so we have to check whether it has a DNS layer (we could also use a packet filter, but let's keep it simple for now).

```golang
dnsLayer := packet.Layer(layers.LayerTypeDNS)
```

If it returns a DNS layer, then we go ahead and fetch the IP layer (which can be IPv4 or IPv6).


### Wrapping up

As you may have noticed, `layers` are a critical part of packet inspection. [GoPacket][gopacket] provides a nice interface to 
extract layers from a packet.


```golang
myLayer := packet.Layer(<LayerType>)
```


[wireshark]: https://www.wireshark.org/
[libpcap]: http://www.tcpdump.org/
[gopacket]: https://github.com/google/gopacket/pcap
[pr_ring]: http://www.ntop.org/products/packet-capture/pf_ring/
[afpacket]: http://lxr.free-electrons.com/source/net/packet/af_packet.c