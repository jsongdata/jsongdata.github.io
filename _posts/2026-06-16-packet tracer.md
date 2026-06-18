---
title: "Packet Tracer Note"
date: 2026-06-16
categories: [Network]
tags: [packet tracer, ccna, network]     # TAG names should always be lowercase
comments: true
---

### Basic Network Devices

An **end system** is a device located at the edge of a network, such as a PC or a server. It is the device that actually sends and receives data.

A **switch** connects end systems inside the same local area network (LAN). It checks the MAC address in an incoming frame and decides which port should receive the frame.

A **router** connects different networks. It uses the destination IP address to choose the best path and forwards packets according to its routing table.

### How a Switch Works

The key address used by a switch is the **MAC address**. When a switch receives data, it learns the source MAC address and checks the destination MAC address to decide where to forward the frame.

1. **Learning**
   - The switch records the **source MAC address** of the received data in its MAC address table.
   - For example, if a PC sends data through port `Fa0/1`, the switch remembers that the PC's MAC address is reachable through `Fa0/1`.

2. **Forwarding**
   - The switch checks the **destination MAC address** in the MAC address table.
   - If the destination MAC address is already in the table, the switch forwards the frame only through the correct port.
   - This improves network efficiency because the switch does not send the frame to unnecessary ports.

3. **Flooding**
   - If the destination MAC address is not in the MAC address table, the switch forwards the frame out all ports except the port that received it.
   - This happens when the switch has not yet learned the destination device.

In short, a switch forwards data inside a LAN based on its **MAC address table**. At first, flooding may happen because the switch does not know every address yet. As communication continues, the MAC address table fills up and forwarding becomes more precise.

### How a Router Works

The key address used by a router is the **IP address**. A switch forwards frames inside a LAN using MAC addresses, while a router forwards packets between different networks using IP addresses.

1. **Routing**
   - Routes are added to the routing table through static routing or dynamic routing.
   - Static routing is configured manually by an administrator.
   - Dynamic routing allows routers to exchange routing information and learn paths automatically.

2. **Packet Switching**
   - The router checks the **destination IP address** of the received packet.
   - It forwards the packet according to the path listed in the routing table.

Example routing table:

```text
C 192.168.10.0/24 is directly connected, FastEthernet0/0
C 192.168.20.0/24 is directly connected, FastEthernet0/1
```

Here, `C` means directly connected. The `192.168.10.0/24` network is connected through `FastEthernet0/0`, and the `192.168.20.0/24` network is connected through `FastEthernet0/1`.

### How Hosts Receive IP Addresses

A host needs an IP address before it can communicate on a network. There are two common ways to assign an IP address.

**Static configuration** means the user or administrator manually enters the IP address, subnet mask, and default gateway. This is often used for servers or network devices that should keep the same address.

**Dynamic configuration** means the host receives an IP address automatically from a DHCP server. This is useful for regular PCs and laptops because it makes address management easier.

### Encapsulation and De-Encapsulation

**Encapsulation** is the process of adding headers to data so it can travel across the network. As data moves down the network layers, headers such as TCP/UDP, IP, and Ethernet headers are added.

**De-encapsulation** is the reverse process. The receiving device removes headers step by step until it reaches the original data.

Packet Tracer is helpful because I can visually see that devices do not simply send raw data. Each layer adds or removes information as the packet moves through the network.

When practicing in Packet Tracer, the flow becomes much clearer if I check which address each device uses to make forwarding decisions.

### Practice Screenshot

<img src="/assets/packet tracer/packet-tracer-topology-2026-06-16.png" width="900" alt="Packet Tracer topology practice screenshot">

<p style="font-size: 14px; text-align: center; color: #666;">
  My first network topology created in Packet Tracer
</p>

I built this topology by dragging three computers and one server into the workspace.

## Lecture Notes: Prompt and Router Configuration Modes

Reference lecture: [Network Construction and Operation Week 1_3](https://youtu.be/Ss4bawwJJJ8?list=PLMDODR7aOT4zOF1UlT9Y8N00_f9Uae3sj)

In this lecture, I learned how to open the router CLI in Packet Tracer and how to understand the **prompt** and Cisco IOS **modes**.

### Prompt

A prompt shows that the device is ready to accept a command. On Cisco devices, the prompt also tells me the device name and the current mode.

| Mode | Prompt | Meaning |
|---|---|---|
| User EXEC mode | `Router>` | A mode where only limited execution commands are available |
| Privileged EXEC mode | `Router#` | A mode where all execution commands are available |
| Global configuration mode | `Router(config)#` | A mode where device configuration commands are available |

In User EXEC mode, available commands are limited. To configure a router, I first need to enter Privileged EXEC mode with the `enable` command.

```text
Router> enable
Router#
```

Then I can enter Global Configuration mode with the `configure terminal` command.

```text
Router# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Router(config)#
```

At this point, I can configure router interfaces, IP addresses, and routing settings.

### IP Address Plan for This Topology

The lecture topology is divided into two different networks by the router.

Left LAN:

```text
Network: 192.168.10.0/24
Router Fa0/0: 192.168.10.1
PC1: 192.168.10.100
PC2: 192.168.10.200
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

Right LAN:

```text
Network: 192.168.20.0/24
Router Fa0/1: 192.168.20.1
PC: 192.168.20.100
Server: 192.168.20.200
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
```

The default gateway is the exit point used when a host needs to communicate with another network. Hosts on the left LAN use `192.168.10.1` as their gateway, and hosts on the right LAN use `192.168.20.1`.

### Router Interface Configuration Flow

For the router to connect both networks, each router interface needs an IP address and must be enabled.

```text
Router> enable
Router# configure terminal

Router(config)# interface fastEthernet 0/0
Router(config-if)# ip address 192.168.10.1 255.255.255.0
Router(config-if)# no shutdown

Router(config-if)# exit
Router(config)# interface fastEthernet 0/1
Router(config-if)# ip address 192.168.20.1 255.255.255.0
Router(config-if)# no shutdown
```

The `no shutdown` command turns on the interface. In Packet Tracer, if the triangle near the router port is red, the interface may be shut down or the link may not be fully active yet. After setting the IP address, entering `no shutdown` activates the port.

### Important Host Settings

Each PC or server needs the correct IP address, subnet mask, and default gateway.

Inside the same LAN, the switch forwards traffic based on MAC addresses. However, when a host needs to reach a different LAN, it sends the packet to its default gateway. If the default gateway is wrong, communication with other networks will fail.

For example, if the PC `192.168.10.100` wants to reach `192.168.20.100`, the traffic follows this path:

```text
192.168.10.100
-> 192.168.10.1 gateway
-> router
-> 192.168.20.0/24 network
-> 192.168.20.100
```

### Verification Commands

After finishing the configuration, I can verify the router status with these commands:

```text
Router# show ip interface brief
Router# show running-config
Router# show ip route
```

- `show ip interface brief`: checks each interface IP address and up/down status
- `show running-config`: checks the current active configuration
- `show ip route`: checks the networks known by the router

Finally, I can use `ping` from a PC to verify communication between different networks.

```text
ping 192.168.20.100
```

### Simulation Screenshot

<img src="/assets/packet tracer/packet-tracer-simulation-2026-06-16.png" width="900" alt="Packet Tracer simulation mode screenshot showing ARP and ICMP events">

<p style="font-size: 14px; text-align: center; color: #666;">
  Packet Tracer simulation mode showing ARP and ICMP traffic
</p>
