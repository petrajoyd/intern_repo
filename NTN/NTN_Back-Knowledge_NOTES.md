# NTN 

# 3GPP NTN
3GPP NTN integrates satellite components directly into the 5G architecture. The goal is to make the satellite link just another "access type," interchangeable with terrestrial 4G or 5G. 
The primary reference documents are **3GPP TR 38.821** (study on NR to support NTN) and **3GPP TS 38.300** (NR overall description).


## 3GPP NTN Architecture Overview
The basic NTN architecture follows the 5G system model but includes a **space segment** (satellite) between the **User Equipment (UE)** and the **Ground Gateway (gNB / NG-RAN)**. ![overview](https://hackmd.io/_uploads/SkEEjJkeZl.png)
- Label Service Link (UE ↔ Satellite)
- Label Feeder Link (Satellite ↔ Gateway)
- Show two modes:
    - Transparent mode: Satellite only forwards signals. All 5G base station (gNB) processing happens on the ground. The radio link (NR-Uu) is simply "bent" over the satellite, creating a very long-range cell.
    - Regenerative mode: Satellite performs gNB or PHY/MAC processing onboard. This allows the satellite to demodulate, decode, and process data in space, enabling lower latency, inter-satellite links, and direct on-board switching.

## NR Protocol Stack Overview
In 3GPP NTN, the radio interface follows the **New Radio (NR)** protocol stack used in terrestrial 5G but with critical adaptations in the lower layers (MAC and PHY) to handle satellite-specific challenges.
![image](https://hackmd.io/_uploads/BJ_CA1ylWx.png)

Here is a quick breakdown of what each layer does, from top to bottom, which explains why the stack is in that order:

### SDAP (Service Data Adaptation Protocol)
#### Job: 
This is the top layer, new in 5G. Its one and only job is to map Quality of Service (QoS) flows to a specific "radio bearer."

#### Analogy: 
It's like a mail sorter at a company. It sees a letter marked "Urgent/CEO" (like a voice call) and puts it in the "Express" mailbag, while a letter marked "Newsletter" (like a background update) goes in the "Bulk Mail" bag.

### PDCP (Packet Data Convergence Protocol)

#### Job: 
This layer gets the data from SDAP and does two main things: 
1) Compresses the IP headers to save space.
2) Ciphers (encrypts) the data for security.

#### Analogy: 
This is the security and packing department. It "vacuum-seals" the mail to make it smaller (compression) and puts it in a "tamper-proof, locked bag" (encryption).

### RLC (Radio Link Control)

#### Job: 
This layer's job is to ensure reliability and handle large packets. It splits large packets into smaller, uniform pieces (segmentation) and, if needed, uses ARQ (Automatic Repeat Request) to re-transmit any pieces that get lost.

#### Analogy: 
This is the shipping department. It takes a big item (like a bicycle) and "disassembles it" (segmentation) to fit into several small boxes, labeling them "Box 1 of 3," "Box 2 of 3," etc., so they can be reassembled on the other end.

### MAC (Medium Access Control)

#### Job: 
This is the traffic cop. It's responsible for scheduling which user gets to transmit when. It takes the small pieces from the RLC layer (from multiple users) and multiplexes them together into one large transport block to be sent out.

#### Analogy: 
This is the loading dock manager. It looks at all the small boxes for different customers and decides "These 50 boxes fit on the truck that's leaving right now. The next 20 boxes will wait for the next truck."

### PHY (Physical Layer)

#### Job: 
This is the physical hardware. It takes the final block of bits from the MAC layer and converts it into a radio signal to be transmitted over the air.

#### Analogy: 
This is the truck and the driver. It's the actual physical transport that moves the goods from one place to another.

## Integration Between Satellite and Terrestrial 5G Core
The massive delays (e.g., 270ms one-way to GEO) and high Doppler shifts (from LEO satellite movement) force significant changes:
### PHY Layer Adaptations:

#### 1. Timing Advance (TA): 
In terrestrial 5G, the gNB measures delay and tells the User Equipment (UE) when to transmit. In NTN, the delay is too large. The UE must pre-compensate its uplink timing before its first transmission. It does this by calculating its distance to the satellite using its own GNSS position and the satellite's ephemeris data (broadcast by the satellite).

#### 2. Doppler Pre-Compensation:
Similarly, the UE must pre-shift its frequency to counteract the large Doppler shift.

#### 3. Waveform:
5G's flexible numerology (e.g., using larger subcarrier spacing) is used to make the radio signal more robust to the remaining Doppler effects.

### MAC Layer Adaptations:
#### 1. HARQ (Hybrid ARQ) Disablement:
The standard MAC-layer HARQ (a fast stop-and-wait retransmission protocol) is unusable over satellite RTTs; it would "stall" the link. 3GPP's solution is to make HARQ feedback configurable and often disabled.

#### 2. Reliability:
When HARQ is off, reliability is moved "up the stack" to the RLC layer's Acknowledged Mode (RLC-AM). RLC-AM is slower but works over long delays, ensuring data is re-transmitted if lost.

## Delay Compensation
In terrestrial 5G, the base station (gNB) tells the phone how to adjust its timing. In NTN, the delay is too big for that. The phone (UE) must autonomously calculate its own delay and frequency compensation before it even transmits.
The channel structure (Logical, Transport, Physical) is the same as terrestrial 5G, but the procedures that run on those channels (especially for error correction and scheduling) are heavily modified to handle the massive delay.

**Here’s the detailed breakdown.**
### Delay & Doppler Compensation
In terrestrial 5G, the Round Trip Time (RTT) is tiny (microseconds). In NTN, it's enormous (milliseconds for LEO, hundreds of milliseconds for GEO). This breaks two fundamental 5G assumptions.

#### **The Problem:**

1. Massive Timing Delay: If a UE just transmitted, its signal would arrive at the satellite (gNB) at the wrong time, "colliding" with other users' signals. In terrestrial 5G, the gNB measures this and sends a small "Timing Advance" (TA) command. In NTN, by the time the gNB's command got back to the UE, the satellite would have moved, and the delay would be wrong.
2. Large Doppler Shift: The high speed of the satellite (especially LEO, at ~7.8 km/s) stretches or compresses the radio waves, shifting their frequency. The 5G (OFDM) waveform is extremely sensitive to frequency errors and would fail to decode

#### **The 3GPP NTN Solution:**
The solution is uplink pre-compensation. The UE is now responsible for fixing these problems itself before transmitting.
To do this, the UE needs to know two things:
1. Its own exact location (from its internal GNSS).
2. The satellite's exact location and velocity (from the satellite ephemeris data, which is broadcast by the network).

With this data, the UE makes two calculations:
- Timing Advance Pre-Compensation: The UE calculates the one-way propagation delay to the satellite ($Delay = Distance / c$). It then advances its own uplink transmission by this amount, so its signal arrives at the satellite's receiver perfectly in-sync with the gNB's time.
- Doppler Pre-Compensation: The UE calculates the relative velocity between itself and the satellite. It then applies an opposite frequency shift (a "pre-shift") to its uplink signal. This way, the Doppler shift that happens in transit actually "corrects" the signal, and it arrives at the satellite on the exact right frequency.

## Logical and Physical Channel
The NTN standard (Rel-17) re-uses the existing 5G NR channel structure. The names and jobs of the channels are identical. Logical and Physical channel for NTN can be seen at the RLC and the Physical layer.
![image](https://hackmd.io/_uploads/BJpXYg1xbx.png)
- Radio Bearer (PDCP): This is just a high-level name for a service.
- Logical Channel (RLC): The RLC layer creates and uses Logical Channels. Its job is to decide what kind of data is being sent (e.g., user data vs. control data).
- Transport Channel (MAC): The MAC layer creates and uses Transport Channels. Its job is to decide how the data will be handled (e.g., will it be broadcast, or shared by one user?).
- Physical Channel (PHY): The Physical Layer creates and uses Physical Channels. Its job is to actually transmit the bits as a radio signal on a specific frequency and time.

### Logical, Transport, And Physical Layer Flow
#### Logical Channels:
(The "What") Defines what kind of data is being sent. This is the "letter."
- BCCH (Broadcast Control Channel): System-wide info (like the satellite ephemeris data).
- PCCH (Paging Control Channel): To "wake up" a UE.
- CCCH (Common Control Channel): For initial access, before a connection is set.
- DCCH (Dedicated Control Channel): Control signals for one specific UE.
- DTCH (Dedicated Traffic Channel): Your actual user data (video, web, etc.).

#### Transport Channels
(The "How") Defines how data is carried by the physical layer. This is the "envelope."
- BCH (Broadcast Channel): Carries the main "master" system info.
- PCH (Paging Channel): Carries paging messages.
- DL-SCH / UL-SCH (Downlink / Uplink Shared Channel): The main "envelope" for all user data (DTCH) and most control data (DCCH).
- RACH (Random Access Channel): The "knock on the door" used for initial access.

#### Physical Channels
(The "Truck") The actual radio resource that moves the bits. This is the "delivery truck."
- PBCH (Physical Broadcast Channel): Carries the BCH.
- PDSCH (Physical Downlink Shared Channel): Carries the DL-SCH and PCH.
- PUSCH (Physical Uplink Shared Channel): Carries the UL-SCH.
- PDCCH (Physical Downlink Control Channel): Carries scheduling commands (telling where PDSCH/PUSCH are).
- PRACH (Physical Random Access Channel): Carries the RACH.

![image](https://hackmd.io/_uploads/B1KMFl1ebl.png)


### Key NTN Adaptations for Channels
This is the most important part. Because of the massive delay, the procedures using these channels had to be adapted, especially at the MAC layer.

#### 1. HARQ (Hybrid ARQ) on PUSCH/PDSCH:
##### Problem: 
Standard 5G HARQ is a fast stop-and-wait protocol. The gNB sends data (PDSCH) and expects a tiny, fast ACK/NACK (on the PUCCH) within milliseconds. Over NTN, the gNB would send one packet, wait 540ms (for GEO) for the ACK, and the entire link would "stall," killing throughput.

##### NTN Solution: 
**HARQ is made configurable or disabled.**
- HARQ Disablement: The network can order the UE to turn off HARQ feedback. This keeps the data flowing. Reliability is "pushed up" to the RLC layer (RLC-AM mode), which is slower but designed to work over high-latency links.
- Extended HARQ Processes: For LEO (where delay is ~40ms RTT), the number of parallel HARQ "pipelines" is doubled from 16 to 32. This lets the gNB send 32 different blocks of data before the first ACK is even expected back, keeping the pipe full.

#### 2. Scheduling Timers (PDCCH):
##### Problem
The gNB's scheduling command (on PDCCH) that says "Your data is on PDSCH now" is useless if the "now" is 270ms in the past.
##### NTN Solution
New, massive timers are added. A new timer called Koffset is broadcast to all UEs. This tells the UE to add a huge offset (e.g., 270ms) to all of the gNB's scheduling commands. The gNB effectively schedules everything far in the future, and the UE knows to "listen for it" at the correct delayed time.
#### 3. Random Access (PRACH):
##### Problem
The whole RACH "handshake" (Msg1-4) would take seconds.
##### NTN Solution
The process is adapted for pre-compensation. When the UE sends its first message (Msg1 on the PRACH), it must already be pre-compensated for delay and Doppler. The gNB configures a much larger "guard time" on the PRACH to allow for small estimation errors from UEs spread across the beam.

## Message Sequence Charts (MSCs)
## NTN O&M Parameters

# DVB-S2X and DVB-RCS2
# DVB-S2X & DVB-RCS2
The **Digital Video Broadcasting – Satellite (DVB)** family defines standards for broadband satellite communication.  

The most important concept is that these are two separate, complementary standards that work together:
- DVB-S2X: The high-speed downlink (forward link) from the gateway to the user.
- DVB-RCS2: The multi-user, interactive uplink (return link) from the user back to the gateway.

These standards are defined by **ETSI EN 302 307-2 (DVB-S2X)** and **ETSI EN 301 545-2 (DVB-RCS2)**.

## System Architecture & Topologies
A DVB-RCS2 network generally includes:

| Entity | Function |
|---------|-----------|
| **Satellite** | Provides the link between terminals and hub (transparent or regenerative). |
| **Network Control Centre (NCC)** | Controls access to the return channel and distributes signaling via the forward link. |
| **Network Management Centre (NMC)** | Manages overall system configuration and enforces SLA (Service Level Agreement). |
| **Hub / Gateway (GW-RCST)** | Routes user traffic between satellite and terrestrial IP networks. |
| **Return Channel Satellite Terminal (RCST)** | Acts as the user terminal (VSAT), transmitting and receiving data and signaling. |

DVB-RCS2 supports both **transparent** and **regenerative** system architectures:

### Transparent Architecture
In a transparent system, the satellite functions purely as a **bent-pipe repeater**, relaying signals between the terminals and the hub.  
Multiple topologies are supported:
- **Transparent Star:** UT ↔ Hub via satellite (single-hop).  
- **Transparent Star with Contention Access:** Similar to star, but uplink uses contention-based slots for initial access.  
- **Transparent Mesh Overlay:** Terminals communicate indirectly through two satellite hops (UT → Hub → UT).
![image](https://hackmd.io/_uploads/SJ5HRb1g-l.png)
**Components:**
- Transparent satellite(s) (possibly with **Digital Transparent Processor (DTP)** payloads for multi-beam connectivity).  
- **Hub/NCC:** Performs traffic control, management, and user-plane interfacing.  
- **Star Terminals (RCSTs):** Support star connectivity.  
- **Mesh Overlay Terminals:** More complex RCSTs with additional demodulators to handle direct terminal-to-terminal links via the satellite.  
- **Gateway RCST (GW-RCST):** A terminal that provides terrestrial access and SLA enforcement for mesh users.

**Key Characteristic:**  
Simple satellite payload, heavier reliance on ground-based control and routing.

### Regenerative Architecture
In the regenerative configuration, the satellite performs **demodulation, decoding, switching, and re-encoding** functions onboard through an **On-Board Processor (OBP)**.

**Components:**
- **Regenerative Satellite:**  
  Performs PHY/MAC processing, onboard routing or switching (Layer 2/3), and may re-encapsulate data.  
  → Acts as an **on-orbit router**, connecting uplink and downlink beams with different modulation/coding.
- **Management Station (NMC/NCC):**  
  Provides network management and control functions.
- **Regenerative Satellite Gateway (GW-RCST):**  
  A hybrid terminal acting as an access gateway to terrestrial networks, possibly hosting SLA enforcers, routers, or VoIP servers.
- **Regenerative Mesh Terminals:**  
  Support single-hop connectivity via the satellite, similar to 3GPP NTN’s regenerative mode.
![image](https://hackmd.io/_uploads/BkmaRZklZl.png)

**Key Characteristic:**  
Smart satellite with OBP enables lower latency, optimized link utilization, and independent inter-beam routing.

## Uplink & Downlink Segmentation (Encapsulation)
This is how IP packets are "packaged" for their satellite journey. DVB uses highly efficient methods, not the older MPEG-TS.
### Downlink (DVB-S2X): 
GSE (Generic Stream Encapsulation) The challenge is packing variable-length IP packets into a high-speed, continuous-like stream of large, fixed-size satellite frames. GSE acts like a "Tetris" player, packing multiple IP packets back-to-back into a single DVB frame to minimize wasted bandwidth.

### Uplink (DVB-RCS2): 
RLE (Return Link Encapsulation) The challenge is the opposite: fitting IP packets into the small, pre-assigned, variable-sized time slots (bursts) defined by the scheduler. RLE uses "just-in-time" fragmentation. It knows the exact size of the upcoming transmission slot and slices the IP packet perfectly to fill it, resulting in almost zero waste.

## Protocol Stack
Unlike 3GPP NTN, which is a native IP stack, the DVB stack is an "IP-over-DVB" encapsulation stack.

### Downlink (S2X) Stack:
- IP Packet / Application Layer → User data.
- GSE (Wraps the IP packet) Encapsulation → Packs IP/Ethernet into BBFRAME efficiently.
- DVB-S2X (Puts the GSE data into a Baseband Frame and applies FEC/Modulation)  → Adds BBHEADER (stream type, sync).
- FEC (LDPC + BCH) → Reliability for 36,000 km GEO link.
- PLFRAME → Adds PLS header, optional pilots.
- PHY (Transmits the radio signal)
![image](https://hackmd.io/_uploads/H1dzefJe-l.png)


### Uplink (RCS2) Stack:
- Application Layer: User data (HTTP requests, VoIP packets, etc.).
- Network Layer: IP packets destined for the internet.
- MAC Layer:
    - Resource request and allocation.
    - Burst scheduling coordination.
- Physical Layer:
    - MF-TDMA burst framing.
    - FEC for error protection.
    - Carrier frequency and modulation selection.

![image](https://hackmd.io/_uploads/H1V0xM1gbg.png)

## Frame Structures
### Downlink (DVB-S2X): 
Baseband Frame (BBFrame) This is a large, fixed-size frame (e.g., 64,800 bits) transmitted in a continuous stream. It's protected by powerful LDPC/BCH Forward Error Correction (FEC). Its key feature is ACM (Adaptive Coding and Modulation), allowing the gateway to change the modulation (e.g., from QPSK to 16APSK) for each frame based on the user's link conditions.
![image](https://hackmd.io/_uploads/HksdfGkeZg.png)



### Uplink (DVB-RCS2): 
MF-TDMA Burst The uplink is not a continuous frame. It's a 2D grid of Frequency and Time. This is called MF-TDMA (Multi-Frequency Time Division Multiple Access). The NCC assigns an RCST a specific "slot" (a frequency and a time block) for a single transmission. This transmission is called a burst.
![image](https://hackmd.io/_uploads/H1PubMkebg.png)
- Time-Division: Multiple RCSTs share the same frequency but transmit in non-overlapping time slots.
- Multi-Frequency: Several frequency carriers are available, and NCC assigns both time & frequency to each RCST.

Physical Layer Steps:

1. RCST prepares data burst according to NCC’s schedule.
2. Burst is modulated (e.g., QPSK, 8PSK) and FEC-coded.
3. Transmitted over assigned carrier in assigned slot.
4. Satellite repeats burst to Gateway.
5. Gateway demodulates & reassembles traffic.

This method maximizes spectrum efficiency while preventing collisions between multiple uplink users.

## MAC & Scheduling
This is the "brain" of DVB-RCS2. The MAC protocol is DAMA (Demand Assigned Multiple Access). It is a centralized system managed entirely by the NCC. An RCST never transmits data without permission.

Here is the flow:

- Request: An RCST needs to send data. It sends a tiny Capacity Request (CR) message to the NCC in a special, shared slot.
- Allocation: The NCC gathers requests from all RCSTs in the network.
- Grant (The Map): The NCC runs its scheduling algorithm (based on QoS) and creates a master schedule called the TBTP (Terminal Burst Time Plan).
- Broadcast: The NCC broadcasts this TBTP map to all RCSTs on the downlink.
- Transmission: Each RCST receives the map, waits for its assigned slot, and then transmits its data burst in that exact time/frequency window. This cycle repeats continuously.
![image](https://hackmd.io/_uploads/BkxVLfJl-g.png)
- B1, B2, B3: These are the satellite's spot beams. A satellite uses multiple beams to cover its service area, just like a cell tower has multiple sectors.
- T1, T2, T3: These are the Terminals (RCSTs).
    - T1 is a terminal in beam B1.
    - T2 is a terminal in beam B2.
    - T3 is a terminal in beam B3.
- a1, a2, a3: These are the applications or data flows running on a terminal. This is critical for QoS.
    - Think of a1 as a VoIP call (needs high-priority CRA).
    - Think of a2 as web browsing (needs best-effort VBDC).

## Role of QoS (Quality of Service)
QoS is the entire purpose of the DAMA scheduler. The NCC's main job is to allocate bandwidth according to the Service Level Agreement (SLA) of each user.
When an RCST sends a Capacity Request, it specifies the type of capacity it needs. These are the Bandwidth on Demand (BoD) mechanisms:

### CRA (Continuous Rate Assignment):
#### What it is: 
The highest priority. Used for traffic that needs a constant, guaranteed rate (e.g., a VoIP call).
#### How it works:
The NCC scheduler pre-books a slot for this RCST in every single TBTP map for the duration of the call, ensuring zero jitter.

### RBDC (Rate-Based Dynamic Capacity):
#### What it is: 
For high-priority, variable traffic (e.g., video streaming).
#### How it works: 
The RCST requests a certain rate (e.g., "I need 2 Mbps"). The NCC will try to grant this rate dynamically.

### VBDC (Volume-Based Dynamic Capacity):
#### What it is: 
The most common, "best-effort" request. Used for bursty data (e.g., web browsing).
#### How it works: 
The RCST simply reports, "I have 1500 bytes in my buffer." The NCC gives it a slot when it can, after all CRA and RBDC requests are fulfilled.
## Message Sequence Charts (MSCs)
## O&M Parameters

# LEO RRM
## Resource Management Algorithms
## QoS-Aware Routing Algorithms
## Beam Hopping Algorithm
