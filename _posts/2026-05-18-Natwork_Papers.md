---
title: "CS 6250 Computer Networks - Reading Network Papers (Optional)"
date: 2026-05-18
categories: [OMSCS]
tags: [computernetworks]     # TAG names should always be lowercase
math: true
---

## Lesson 2

**Congestion Avoidance and Control — Jacobson & Karels (1988)** (아래 전부 선택적)
https://ee.lbl.gov/papers/congavoid.pdf

In 1986, the Internet experienced its first "congestion collapse," causing throughput between two sites to drop from 32Kbps to 40bps — a thousandfold decrease. This paper identifies the root cause as flawed TCP implementations rather than the protocol itself, and proposes key algorithms including Slow-start, RTT variance estimation, and dynamic window sizing. The central idea is "packet conservation" — a new packet should only enter the network when an old one leaves — which became the foundation of modern TCP congestion control.

---

**A Protocol for Packet Network Intercommunication — Cerf & Kahn (1974)**
https://www.cs.princeton.edu/courses/archive/fall06/cos561/papers/cerf74.pdf

This is the foundational paper of the Internet, presenting a protocol design philosophy for interconnecting heterogeneous packet-switching networks. The key idea is to place a Gateway at each network boundary, enabling end-to-end communication via a common protocol without modifying the internal structure of individual networks. Nearly every concept in today's TCP/IP stack — addressing, packet fragmentation, flow control, and error recovery — makes its first appearance here.

---

**End-to-End Internet Packet Dynamics — Paxson (1997)**
https://people.eecs.berkeley.edu/~sylvia/cs268-2019/papers/pktdynamics.pdf

This large-scale empirical study measured 20,000 TCP bulk transfers across 35 Internet sites. Key findings include that packet loss is not independent but correlated in bursts, and that the distribution of loss event durations has effectively infinite variance. The study also showed that "pathological" behaviors such as out-of-order delivery, packet replication, and path asymmetry are far more common than previously assumed, overturning many widely held beliefs about Internet behavior.

---

**Data Center TCP (DCTCP) — Alizadeh et al. (2010)**
https://people.csail.mit.edu/alizadeh/papers/dctcp-sigcomm10.pdf

In data center environments, standard TCP performs poorly when latency-sensitive short flows and high-throughput long flows coexist, as large flows build up queue in switches and crowd out short ones. DCTCP addresses this by leveraging ECN (Explicit Congestion Notification) in commodity switches to extract a multi-bit signal reflecting the extent of congestion, allowing sources to react proportionally rather than abruptly. Experiments showed DCTCP achieves the same or better throughput than TCP while using 90% less buffer space.

---

**TIMELY — Mittal et al. (2015)**
https://conferences.sigcomm.org/sigcomm/2015/pdf/papers/p537.pdf

While DCTCP relies on ECN support from network switches, TIMELY argues that RTT measurement alone — with no switch-side changes — is sufficient for datacenter congestion control. Advances in NIC hardware now enable microsecond-accurate timestamping, making RTT a reliable proxy for switch queue occupancy. TIMELY uses the gradient of RTT change to predict the onset of congestion and adjust transmission rates accordingly, achieving 13x lower tail latency than DCTCP in experiments across hundreds of machines.

---

**Congestion Control for Multipath TCP — Wischik et al. (2011)**
https://www.usenix.org/legacy/events/nsdi11/tech/full_papers/Wischik.pdf

When Multipath TCP (MPTCP) runs standard TCP independently on each subflow, it can unfairly consume more than its share of bandwidth at shared bottleneck links. This paper designs and implements a "coupled" congestion control algorithm where subflows collectively shift traffic toward less congested paths — achieving a resource pooling effect — while guaranteeing that the total bandwidth used is no more than a single TCP flow would take. The algorithm was implemented in Linux and validated across multihomed servers, data centers, and mobile clients.

---

**Sizing Router Buffers — Appenzeller, Keslassy & McKeown (2004)**
https://web.archive.org/web/20210120232627/http://yuba.stanford.edu/techreports/TR04-HPNG-060800.pdf

The long-standing rule of thumb for router buffer sizing — set the buffer equal to the bandwidth-delay product (BDP) — is challenged in this paper as unnecessarily large. When N independent long-lived TCP flows share a link, a buffer of BDP/√N is sufficient, because the flows naturally desynchronize and rarely fill the buffer simultaneously. This "Stanford rule" has significant practical implications for building smaller, faster routers, and became the starting point for a generation of subsequent buffer sizing research.

---

## Lesson 3

**Hot Potatoes Heat Up BGP Routing — Teixeira, Shaikh, Griffin & Rexford (2004)**
https://www.cs.princeton.edu/~jrex/papers/hotpotato.pdf

Although the Internet's routing architecture separates intradomain (IGP) and interdomain (BGP) routing into two tiers, this separation is far from complete. When a router must choose among multiple "equally good" BGP routes, **hot potato routing** kicks in: the router selects the exit point with the lowest IGP cost — essentially offloading traffic at the nearest opportunity. This paper presents the first joint analysis of OSPF link-state advertisements (LSAs) and BGP update messages from a large ISP backbone, showing that OSPF events are a significant source of BGP updates, that BGP updates can lag several seconds behind the triggering OSPF event, and that OSPF-induced BGP changes are distributed nearly uniformly across destination prefixes.

---

**Dynamics of Hot-Potato Routing in IP Networks — Teixeira, Shaikh, Griffin & Rexford (2004)**
https://www.cs.princeton.edu/~jrex/papers/sigmetrics04.pdf

This deeper follow-up study quantifies the performance impact of hot-potato routing using real measurement data from AT&T's backbone (AS 7018), combined with controlled experiments on a commercial router. The authors propose an algorithm for correlating IGP and BGP data, and demonstrate that hot-potato routing changes cause three concrete problems: (i) transient packet loss during forwarding table recomputation, (ii) traffic shifts that may trigger congestion on new paths, and (iii) externally-visible BGP updates that leak internal topology changes to neighboring domains. The study also shows that route reflector hierarchies can amplify hot-potato routing changes across the entire network.

---

**Traffic Engineering With Traditional IP Routing Protocols — Fortz, Rexford & Thorup (2002)**
https://www.cs.princeton.edu/~jrex/teaching/spring2005/reading/fortz02.pdf

Conventional wisdom holds that traditional shortest-path protocols like OSPF/IS-IS are too inflexible for traffic engineering. This paper challenges that assumption, showing that carefully tuned link weights alone can achieve near-optimal routing performance — approaching the results of schemes with full path-selection flexibility. The framework follows three steps: **Measure** network topology and traffic demands, **Model** how IGP configurations translate to traffic flows, and **Control** by applying optimized link weights externally without modifying the routers themselves. Because it works within the existing deployed base of OSPF and IS-IS, this approach avoids equipment upgrades and protocol changes entirely.

---

**OSPF Monitoring: Architecture, Design and Deployment Experience — Shaikh & Greenberg (2004)**
https://www.cs.princeton.edu/~jrex/teaching/spring2005/reading/shaikh04.pdf

Both traffic engineering and hot-potato routing research depend critically on accurate, real-time visibility into OSPF state. This paper describes the architecture of an OSPF monitoring system deployed in operational ISP and enterprise networks. The system separates into three components: the **LSAR**, which passively captures LSAs from the network; the **LSAG**, which performs real-time detection of topology changes, flapping elements, and LSA storms; and **OSPFScan**, which enables offline post-mortem analysis. The key design principle is that the monitor speaks "just enough" OSPF to receive all LSAs while remaining completely passive — a proven improvement over SNMP-based approaches in both reliability and timeliness.