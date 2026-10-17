# Study Notes: Protocol Stack for DVB-S2X and DVB-RCS2

* **Author:** Petrajoy Davidson
* **Date:** 2025-08-05 – 2025-08-06
* **Goal:** Fully understand the protocol stacks of DVB-S2X and DVB-RCS2 (forward & return link) and relate them to the final research question: "How can we implement NTN NR on top of DVB L1?"
* **Slides:** 13–22

---

## 1. Core Principle Discovered: Streaming Pipeline vs. Transactional Burst System

The **DVB-S2X forward link** and the **DVB-RCS2 return link** are fundamentally different in design.

- **DVB-S2X Forward Link** → Continuous, one-way **streaming pipeline**.  
  *Optimized for maximum throughput, minimal overhead, and constant transmission from one gateway to many terminals.*

- **DVB-RCS2 Return Link** → Shared, **transactional burst system**.  
  *Optimized for controlled, scheduled access by many terminals using time/frequency slots granted by a central controller (NCC).*

**Analogy:**  
- Forward link = **oil pipeline** (constant high-volume flow).  
- Return link = **airport control tower** (precise scheduling to avoid collisions).

---

## 2. System Analysis: Following the Data Path

---

### 2.1. DVB-S2X System Overview 
<img width="1528" height="256" alt="image" src="https://github.com/user-attachments/assets/7f80c3f3-1f69-47e9-bbe0-fa7e2a9b635e" />


**Function:**  
Show the high-level components of the DVB-S2X forward link.

**Flow:**
1. **Gateway (Feeder Link Terminal)** takes in IP traffic.
2. Encapsulation, framing, FEC, and modulation happen entirely on the ground.
3. Satellite (transparent repeater) sends it to all User Terminals.
4. User Terminal demodulates & decodes to feed IP data to end devices.

### What the diagram is telling me
- **Blocks:** `Gateway → Satellite (bent-pipe) → User Terminal (RCST as receiver) → End devices`.
- **Arrows:** One continuous data flow outbound; there is **no per-terminal negotiation** on this path at L2—control is minimal and mostly in-band signaling for demodulation/config.
- **Control plane vs data plane:**  
  - **Data plane:** IP/ethernet payload that ends up as video, web, etc.  
  - **Control plane:** Physical layer signaling (PLS header), superframe beacons, and (outside S2X itself) management/ACM info that guides the gateway’s MODCOD choices.

### Why the gateway does all the heavy lifting
- **Encapsulation, framing, FEC, modulation** live on the ground because power/complexity is cheap there.  
- The **satellite transponder is transparent** (“bent-pipe”): it doesn’t interpret frames—just frequency translation + amplification.

### Key assumptions baked into the forward link
- **Always-on, high volume** downlink traffic (broadcast/multicast/unicast mix) → pipeline style stack is efficient.  
- **Heterogeneous terminals**: different SNRs → the system must **adapt MODCOD** frame-by-frame (ACM).  
- **Long RTT (GEO)**: control loops (e.g., ACM) are slower; layer design avoids chatty handshakes.

### Practical consequence
- The **terminal is a smart demod/decoder**, not an active MAC participant on the forward link. It **reads PLS** to learn how to demod each incoming frame, then **decodes LDPC+BCH**, reassembles GSE, and passes IP up the stack.

### Quick sanity checklist (for later verification)
- PLS fields we rely on (frame type/MODCOD signalling).  
- Whether pilots are on/off in our service profile (throughput vs robustness).  
- How many physical carriers (transponders/beams) our “Gateway” block hides in reality.

---

### 2.2. DVB-S2X Forward Link Protocol Stack 
<img width="1306" height="589" alt="image" src="https://github.com/user-attachments/assets/9ab5c9dd-7866-42f1-a088-f340379b13b5" />


**Layer Breakdown:**
- **IP / Application Layer** → User data.
- **GSE Encapsulation** → Packs IP/Ethernet into BBFRAME efficiently.
- **BBFRAME** → Adds BBHEADER (stream type, sync).
- **FEC (LDPC + BCH)** → Reliability for 36,000 km GEO link.
- **PLFRAME** → Adds PLS header, optional pilots.
- **Modulator** → Uses MODCOD chosen via ACM (QPSK–32APSK).

### Read of the stack (top→bottom)
1. **IP / Application input**  
   Source traffic (IP/Ethernet). Could be TS as well, but our flow centers on IP.
2. **GSE (Generic Stream Encapsulation)**  
   - **Why:** Efficient IP carriage over DVB without TS overhead.  
   - **What it does:** Fragment/aggregate IP packets into GSE PDUs; optional CRC for each PDU; stream identifiers.  
   - **So what:** **Fragmentation** lets us pack BBFRAMEs tightly; **aggregation** reduces header overhead.
3. **BBFRAME (Baseband Frame)**  
   - **Container** that groups GSE PDUs.  
   - **BBHEADER** carries stream type/format, synchronization hints, and helps the receiver reassemble PDUs cleanly.  
   - **Design trade-off:** choose BBFRAME size/profile to balance latency vs efficiency.
4. **FEC (LDPC + BCH)**  
   - **LDPC**: primary, powerful code for deep error correction.  
   - **BCH**: outer code to mop up residual LDPC errors and enforce a sharp waterfall.  
   - **Net effect:** Dramatically lowers required C/N for a target PER → more service availability.
5. **PLFRAME (Physical Layer Frame)**  
   - Wraps FEC’d payload with **PLS header** (tells the terminal which **MODCOD**, frame type, pilots, etc.) and **pilots** (aid channel estimation & carrier/phase tracking).  
6. **Modulation & Transmission (ACM loop)**  
   - Gateway chooses **MODCOD** per frame, based on terminal/channel reports aggregated at the hub.  
   - **Higher MODCOD** = more bits/symbol but needs higher SNR; **lower MODCOD** = more robust, less efficient.

### Key insights (why this stack is “pipeline”)
- **No MAC grants here**: data just flows; adaptation is **downlink-centric** and controlled at the gateway.  
- **Encapsulation→Framing→FEC→PL** is **strictly serial**; each stage transforms format for the next with minimal feedback.


---

### 2.3. Forward Link Framing Context 
<img width="717" height="903" alt="image" src="https://github.com/user-attachments/assets/23008f94-caa6-45c5-996b-72ed3c718495" />
**<img width="626" height="1099" alt="image" src="https://github.com/user-attachments/assets/0236f2eb-e54a-4b49-988b-b413b2f2bcc7" />**

- **PLS Header**: MODCOD & frame type info.
- **Payload**: FEC-protected BBFRAME data.
- **Pilots**: Aid demodulator tracking.

### What “PLFRAME” really contains (functional view)
- **PLS Header:** compact L1 signalling so the demod knows *how* to read the rest (MODCOD, frame type, pilots on/off, etc.).  
- **Data field:** the **FEC-protected BBFRAME** bits.  
- **Pilots (optional/periodic):** help equalize/track impairments (phase noise, non-linearities, Doppler residual).

### Performance intuition
- **Pilots on:** better tracking on tough channels (rain fade, mobility) → lower required C/N at the cost of throughput.  
- **Big LDPC blocks + strong MODCOD:** best spectral efficiency when channel is clean.  
- **Lower MODCOD:** keeps service alive for edge/weak terminals during fades.

### Operational flow (concrete)
1. **Gateway builds BBFRAME** by packing GSE PDUs.  
2. **LDPC+BCH** applied → payload prepared for harsh channel.  
3. **PLS header + pilots** added → demod guidance.  
4. **MODCOD chosen** (ACM).  
5. **RF chain** sends it to space; satellite repeats; terminals **lock→read PLS→demod→decode→reassemble GSE→deliver IP**.

---

### 2.4. DVB-RCS2 System Overview 
<img width="1168" height="225" alt="image" src="https://github.com/user-attachments/assets/7b3e808e-c7f0-4489-9e6d-c88a3f7ed32f" />


**Function:**  
High-level components of the return link.

**Flow:**
1. End Device generates data.
2. User Terminal (RCST) requests transmission slot from NCC.
3. NCC allocates TDMA time/frequency slot.
4. RCST transmits burst → Satellite → Gateway → Internet.

**Function:** Provide the big-picture architecture of the return link path.  
**Key Elements from Visual:**
- **RCST (Remote Control Satellite Terminal):** The user’s satellite modem/dish for uplink.  
- **Satellite:** Acts as a bent-pipe relay — no onboard scheduling, simply retransmits the signal to the Gateway.  
- **Gateway (Feeder Link Terminal):** Receives the uplink bursts from all RCSTs and passes them into the terrestrial network.  
- **NCC (Network Control Center):** Logical brain of the system, often co-located with Gateway. Handles access control, timing, and slot allocation.

**Takeaways:**
1. This is a **star topology** — all traffic is RCST → Satellite → Gateway.
2. No peer-to-peer links between RCSTs — all data must pass through the hub.
3. Uplink traffic is shared among many RCSTs — making coordination critical.
---

### 2.5. Gateway and NCC Responsibilities 
<img width="1678" height="381" alt="image" src="https://github.com/user-attachments/assets/2c0fa7d4-2877-4db3-8d52-efc134e95b1c" />


**Layer Breakdown:**
- **IP / Application Layer** → User data.
- **Encapsulation** → MPEG-TS or other return formats.
- **MAC (Resource Request)** → Terminal asks NCC for slot.
- **Burst Framing** → Data formatted into TDMA bursts.
- **MODCOD & Transmission** → As per allocated slot.

**Gateway Responsibilities:**
- Physical signal reception from the satellite’s return link carrier.
- Demodulation of bursts (MF-TDMA).
- Error correction and reassembly of data.
- Forwarding traffic to terrestrial IP backbone.

**NCC Responsibilities:**
- **Resource Management:** Allocates frequency/time slots to RCSTs based on demand.
- **Synchronization:** Ensures terminals’ transmissions are precisely timed to arrive without overlap.
- **Traffic Control:** Manages QoS priorities, admission control, and congestion handling.
- **Control Signaling:** Sends burst-time plans (BTP) to all terminals via the forward link.

**Key Insight:**  
While the Gateway is the “RF workhorse,” the NCC is the “traffic controller” that prevents chaos in the shared uplink.

---

### 2.6. RCST (User Terminal) Functions
<img width="1248" height="583" alt="image" src="https://github.com/user-attachments/assets/1900ee70-d4f6-420f-80cd-c2ff1a0645ea" />

**Functional Roles of RCST:**
- **Receive NCC Instructions:** Reads burst-time plan and configures transmit parameters accordingly.
- **Transmit in Allocated Slot:** Uses MF-TDMA bursts in the precise time/frequency assigned.
- **Adapt Transmission:** Adjust MODCOD and burst size based on channel quality and service type.
- **Return Link Synchronization:** Maintains alignment with the network’s timing reference.

**Extra Note:**  
The RCST does not independently choose when to transmit — it is **entirely dependent on NCC’s schedule**. This prevents two RCSTs from accidentally transmitting at the same time.

---

### 2.7. DVB-RCS2 Return Link Protocol Stack
<img width="774" height="686" alt="image" src="https://github.com/user-attachments/assets/41b74be6-f5ae-40fe-ad7f-dcc58cbccdbb" />

**Visual Analysis:**  
The protocol stack mirrors DVB-S2X in layering but has extra MAC-level complexity due to the shared medium.

**Protocol Stack Breakdown:**
1. **Application Layer:** User data (HTTP requests, VoIP packets, etc.).
2. **Network Layer:** IP packets destined for the internet.
3. **MAC Layer:**  
   - Resource request and allocation.
   - Burst scheduling coordination.
4. **Physical Layer:**  
   - MF-TDMA burst framing.
   - FEC for error protection.
   - Carrier frequency and modulation selection.

**Key Difference from DVB-S2X:**  
The return link adds **MAC-level control** for dynamic slot assignment, absent in the forward link’s continuous broadcast design.

---

### 2.8. Physical Layer: DVB-RCS2 Return Link Transmission
<img width="1560" height="370" alt="image" src="https://github.com/user-attachments/assets/70d9d44e-4841-400a-a8b2-03202a8139ab" />

**Transmission Method:** MF-TDMA (Multi-Frequency Time Division Multiple Access)  
- **Time-Division:** Multiple RCSTs share the same frequency but transmit in non-overlapping time slots.
- **Multi-Frequency:** Several frequency carriers are available, and NCC assigns both time & frequency to each RCST.

**Physical Layer Steps:**
1. RCST prepares data burst according to NCC’s schedule.
2. Burst is modulated (e.g., QPSK, 8PSK) and FEC-coded.
3. Transmitted over assigned carrier in assigned slot.
4. Satellite repeats burst to Gateway.
5. Gateway demodulates & reassembles traffic.

**Why This Matters:**  
This method maximizes spectrum efficiency while preventing collisions between multiple uplink users.

---

### 2.9. QoS and Scheduling in DVB-RCS2 Return Link 
<img width="1077" height="996" alt="image" src="https://github.com/user-attachments/assets/c89fcba5-e51a-4c73-9f2d-d7cb03f1e813" />

**QoS Classes:**
- **Real-Time Services:** VoIP, live video — strict latency & jitter requirements.
- **Non-Real-Time Services:** Web browsing, file downloads — tolerant to delay but sensitive to throughput.
- **Background Services:** Bulk file transfers — tolerant to both delay and jitter.

**Scheduling Mechanisms:**
- **Dynamic Capacity Assignment (DCA):** Adjusts slot allocation in real time based on demand.
- **Reservation-Based:** Guarantees slots for high-priority streams.
- **Demand Assignment:** Allocates resources only when traffic is present.

**Key Takeaway:**  
QoS-aware scheduling ensures that urgent, real-time services aren’t starved of bandwidth, even if the uplink is congested with background traffic.


---

## 3. Key Takeaways & Recap

1. **Two Complementary Systems:**  
   - S2X = constant high-throughput broadcast (forward link).  
   - RCS2 = controlled, slot-based uplink (return link).  
2. **Control Placement:**  
   - S2X: intelligence mostly in Gateway’s PHY chain.  
   - RCS2: intelligence in NCC for scheduling & MAC control.  
3. **Physical Layer Constraints:**  
   - Forward: large continuous PLFRAME streams.  
   - Return: short MF-TDMA bursts requiring high timing accuracy.  
4. **QoS Integration:**  
   - S2X: QoS applied at IP/GSE level before broadcast.  
   - RCS2: QoS embedded in NCC’s scheduling decisions.  
5. **Relevance for NTN NR:**  
   - Mapping NR traffic into S2X requires preserving the continuous L1 framing.  
   - Mapping NR uplink into RCS2 requires respecting the scheduled burst architecture.

---

## 4. Next Steps

Now that both protocol stacks are fully understood, the next phase is to break down the **L1 framing structures** for both S2X and RCS2:
- Understand PLFRAME format, headers, pilots, and payload organization.
- Map logical & physical channel concepts to actual L1 structures.
- Identify where NR data can be injected without breaking DVB compliance.
