# Skim Paper/Journal: Part 1. 🔍
Journal Name = **DVB-RCS2/S2 Testbed: A Distributed Testbed for Next-Generation Satellite System Design and Validation**.
This document summarizes the key findings from the paper: "DVB-RCS2/S2 Testbed: A Distributed Testbed for Next-Generation Satellite System Design and Validation," focusing on the return link (DVB-RCS2) architecture, signaling, traffic management, and QoS.



## DVB-RCS2 Testbed: System Architecture, Signaling, Traffic & QoS
The paper describes a DVB-S2/RCS2 testbed designed to reproduce the functionalities of both higher and lower layers of the protocol stack, allowing for validation of the DVB-RCS2 standard.
### 1. System Architecture and Signaling
The DVB-RCS2 system architecture, as implemented in the testbed, is distributed, separating functions performed by satellite terminals (RCSTs), the gateway (Hub), and the Network Control Centre (NCC).
- Hub Components:
  - Network Control Centre (NCC): The central brain. It's responsible for generating the network clock reference (NCR) and various signaling tables for the return link, such as the Superframe Composition Table (SCT), Frame Composition Table version 2 (FCT2), and Broadcast Configuration Table (BCT). The NCC also manages the login procedure for new terminals.
  - Hub Traffic Management: Manages traffic flow. It receives classified user traffic from the Higher Layer Specification (HLS) module or signaling tables from the NCC. It then assigns Modulation and Coding (ModCod) information based on channel measurements reported by terminals. Crucially, it performs resource allocation for the return link based on capacity requests from terminals and pre-configured Continuous Rate Assignments (CRA).
  - RLE Receiver: Demultiplexes incoming traffic from the return link physical layer (RL PHY) module. It reconstructs source addresses and sends signaling to the NCC and user traffic to the HLS module .
  - GSE Transmitter (Forward Link): Fragments and encapsulates traffic into GSE packets, placing them into priority queues. It generates baseband frames (BBFRAMEs) when triggered by the forward link physical layer (FL PHY) module .
  - FL PHY (Forward Link): Maintains symbol rate, considering ModCod and frame size, adds a mode-adaption header, and sends frames to the Forward Channel Emulator .
  - RL PHY (Return Link): Demultiplexes incoming traffic from the Return Channel Emulator and forwards it to the NCC or RLE Receiver.
  - Background Traffic Generator (Optional): Can load the forward link by producing IP packets with random payload at certain rates, and simulate return link load by generating fake capacity requests .
  - HLS (Hub): Handles higher-layer protocols and interfaces with applications.
- Terminal Components (RCST):
  - FL PHY (Forward Link): Receives packets from the Forward Channel Emulator, applies a MAC filter, reassembles, and decapsulates traffic, forwarding it to either the HLS module or the RCST Controller .
  - RCST Controller: Implements the terminal's state-machine for logon and synchronization procedures. It extracts allocated transmission resources from signaling tables and sends control bursts with status information back to the hub.
  - Terminal Traffic Management: Generates capacity requests based on RLE queue statistics. These requests can be sent in pure control bursts or combined control and traffic bursts using RLE. It also informs the RLE Transmitter about allocated resources.
  - RLE Transmitter: Contains a priority-based scheduler and encapsulators. It outputs RLE packets, forwarding DAMA traffic directly to the RL PHY and Random Access (RA) traffic to the RA module.
  - Random Access (RA) Module: Selects RA slots for payloads (Slotted ALOHA) or generates and distributes replicas for Contention Resolution Diversity Slotted Aloha (CRDSA).
  - RL PHY (Return Link): Multiplexes control and user traffic and sends it to the Return Channel Emulator.
  - HLS (Terminal): Handles higher-layer protocols and interfaces with user applications.
- Communication Protocols:
  - Next Generation testbed Protocol (NGP): Based on UDP, used for information exchange between lower layer modules (below GSE and RLE) and for communication with the NCC and RCST Controller .
  - Higher Layer Protocol (HLP): Based on UDP, designed for communication between the HLS module and its LLS counterparts. It carries user traffic and corresponding addressing/traffic classification information . HLP implements QoS mapping functions similar to SI-SAP .

### 2. Traffic Model and Bandwidth Allocation
DVB-RCS2 handles various traffic types by employing different capacity request mechanisms, allowing for flexible and dynamic bandwidth allocation.
- Traffic Models (Implicitly Handled):
  - Continuous Rate Assignment (CRA): For applications requiring a constant, guaranteed rate, like some real-time streaming services or voice.
  - Rate-Based Dynamic Capacity (RBDC): For applications that request a specific bandwidth rate. This is suitable for variable bit rate (VBR) services that need a smoother, more continuous allocation. The paper shows RBDC resulting in a smooth and constant rate.
  - Volume-Based Dynamic Capacity (VBDC): For bursty traffic where terminals request a certain volume of data. This leads to a more "on-off" transmission behavior, as seen in the comparison with RBDC. This is ideal for typical internet Browse or file transfers where data arrives in bursts.
  - Random Access (RA): For very small messages or initial access, particularly appealing for applications generating bursty or asynchronous messages where traditional DAMA schemes don't scale well. The paper uses RA for small messages.
- Bandwidth Allocation Process:
  - RCSTs generate **capacity requests (CRs)** based on their internal RLE queue statistics.
  - These CRs are sent to the **Hub Traffic Management module.**
  - The Hub Traffic Management, along with pre-configured CRAs, processes these requests and performs resource allocation.
  - The allocation is compiled into a **Terminal Burst Time Plan 2 (TBTP2)** signaling table.
  - This TBTP2 is then sent back to the RCSTs via the forward link, informing them which time and frequency slots they are allocated for transmission .
  - Unused capacity in the superframe can be distributed using **Free Capacity Assignment (FCA)**or designated as random access slots.

### 3. QoS Mechanism (DiffServ, MPLS)
DVB-RCS2 focuses on mapping IP-based QoS onto lower-layer resource allocation functions. The paper specifically mentions Differentiated Services (DiffServ).
- Mapping QoS Classes: The main novelty in the Higher Layer Specification (HLS) is the QoS management scheme, which maps IP-based classes of service onto the request classes defined at the lower layer for resource allocation.
- Classifier: Within the HLS module, a configurable classifier assigns the QoS priority and the preferred transmission method (DAMA or RA) to traffic destined for the satellite. This classifier can filter traffic based on fields like IP address, protocol type, port number, or **Differentiated Services Code Point (DSCP) value**. This confirms DiffServ as a key mechanism for higher-layer QoS.
- Priority Queues: The RLE Transmitter (at the terminal) and GSE Transmitter (at the hub) use priority-based schedulers with queues. This means that higher-priority traffic (as determined by the classifier) will be placed in queues that are served preferentially.
- Example of QoS Handling: Figure 8 in the paper demonstrates this. When high-priority bursts (e.g., VoIP) arrive, the serving of low-priority queues stops, and available capacity is used for the high-priority traffic. Only when the high-priority queue is empty does the low-priority queue get served again .
- No Explicit MPLS: The paper does not explicitly mention MPLS (Multiprotocol Label Switching) as a QoS mechanism within the DVB-RCS2 standard or testbed implementation. QoS is primarily handled by the mapping of IP DSCP values to DVB-RCS2's lower-layer capacity request types and priority queueing.
- Robust Header Compression (ROHC): Enabled in the HLS module to compress TCP, UDP, and IP datagrams, which is important for optimizing system performance, especially for small packets (e.g., VoIP) .
- Performance Enhancing Proxies (PEPs): The HLS module includes a PEP based on the SCPS-TP gateway with TCP Noordwijk (a satellite-optimized TCP flavor) and TCP splitting. PEPs significantly improve TCP throughput over high-latency satellite links by locally acknowledging TCP segments and optimizing connection parameters .

### 4. Unfamiliar Terms or Algorithms
- L.2 Mode-Adaption Header: A header added before a frame is sent to the FL Channel Emulator, related to interfaces for DVB-S2 equipment.
- Network Clock Reference (NCR): Generated by the NCC, this is a timing reference crucial for synchronization across the network.
- Superframe Composition Table (SCT), Frame Composition Table version 2 (FCT2), Broadcast Configuration Table (BCT): Signaling tables generated by the NCC to define the superframe structure in the return link. These tables are essential for the RCSTs to understand how the return link is organized and when they can transmit.
- Terminal Burst Time Plan 2 (TBTP2): A signaling table compiled by the Hub Traffic Management module, informing terminals about their allocated transmission resources (time and frequency slots).
- Slotted ALOHA (SALOHA): A random access method where terminals transmit in synchronized time slots. If multiple terminals transmit in the same slot, a collision occurs, and all transmissions are lost.
- Contention Resolution Diversity Slotted Aloha (CRDSA): An advanced random access scheme used in DVB-RCS2. It improves upon Slotted Aloha by transmitting multiple replicas of short bursts in different, randomly chosen slots. The hub uses 
- Successive Interference Cancellation (SIC) to "unmask" collided bursts if at least one replica is received successfully, significantly boosting throughput compared to traditional Aloha.
- FreeBSD Jail: A light-weight virtualization technology used to realize Satellite Virtual Networks (SVNs) in the testbed. It provides an isolated environment (memory, process space, virtualized network stack) for each SVN, allowing independent IP addresses and routing tables .
- Satellite Virtual Networks (SVNs): Define virtual subnetworks within a satellite system, enabling better resource use, easier network configuration and address planning (especially for IP addresses), and segregation of traffic classes on specific satellite capacity portions .
- Software Download Delivery Protocol (SDDP): A protocol used for software downloads to terminals, allowing the hub to broadcast software versions to which terminals listen and download if newer .
- SNMP (Simple Network Management Protocol): Used for performance monitoring and configuration functions, allowing satellite operators to adjust system configuration efficiently without manual intervention .
- Management Information Base (MIB): A database used by SNMP to store and organize management information for network devices.
- SCPS-TP gateway (Space Communications Protocol Specification - Transport Protocol): A type of Performance Enhancing Proxy (PEP) mentioned in the paper. It's a gateway that implements satellite-optimized TCP protocols to improve performance over long-delay satellite links.
- TCP Noordwijk: A satellite-optimized TCP flavor used with the SCPS-TP gateway. It's designed to perform better over high-latency satellite connections than standard TCP algorithms like TCP NewReno.
- TCP Splitting: A technique used by PEPs where a single end-to-end TCP connection is split into two or more segments (e.g., one over the terrestrial network and one over the satellite link). The PEP locally terminates and acknowledges TCP segments, improving throughput over high-latency links .
- DAMA (Demand Assignment Multiple Access): A category of multiple access schemes where resources (time/frequency slots) are allocated to users on demand, as opposed to being permanently assigned. DVB-RCS2's CRA, RBDC, VBDC are DAMA schemes.







