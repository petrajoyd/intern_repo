# QoS (Quality Of Service)
The only reason we need QoS is because of one single problem: **network congestion.**
Think of your network connection as a highway.
- Bandwidth is the number of lanes (e.g., a 3-lane highway).
- Data Packets are the cars on that highway.

When the highway is mostly empty, everything is great. All cars (packets) can travel at the speed limit (low delay) and arrive when they're expected.

### The Problem: A Traffic Jam
Congestion is a traffic jam. It's what happens when too many cars try to use the same section of highway at the same time.
This jam causes all the problems:
- **Delay**: Everyone slows down.
- **Jitter**: The traffic is stop-and-go (inconsistent delay).
- **Packet Loss**: The on-ramp gets so backed up that the router (the "traffic cop") has to stop cars from getting on. It just drops them ("Sorry, we're full").

### The "Best-Effort" Model (No QoS)
Now, imagine this highway has no rules. This is the "Best-Effort" model—the default state of the internet.
On this "no-rules" highway, you have:
- An ambulance with a patient (a voice call packet)
- A gravel truck (a file download packet)
- A family car going on vacation (a web browsing packet)

In the "Best-Effort" model, everyone is treated equally. 
You can see the problem: **not all traffic is equally important.**
- The ambulance (voice call) must get there fast, with no stops (low delay, low jitter).
- The gravel truck (file download) doesn't care. It can be late, it can stop and go. As long as all the gravel (data) eventually arrives, it's fine.

### The Solution: QoS (The Traffic Rules)
QoS is simply the "traffic management system" for the highway.
It's the set of **tools and rules** that lets the network stop treating all "cars" equally.

- Classification & Marking: This is the first step. The network "looks" at the cars. It says, "That's an ambulance! That's a gravel truck." It "marks" them (with those DSCP tags we mentioned) so other routers can instantly see what they are.
- Queuing & Prioritization: This is the action. Now that we know what's what, we can create special lanes.
    - We create a high-priority "emergency lane" (a Priority Queue).
    - The router ("traffic cop") is given a simple rule: "Always let the ambulance go first."
    - It will stop sending gravel trucks for a moment to let the ambulance cut the line and go straight through.

That's it. That's the entire concept of QoS from the ground up.

## QoS Parameters: The Metrics
These are the key metrics used to measure the "quality" of your network's performance. When we talk about designing for QoS, we're really talking about how to control these specific values for different types of traffic.
### 1. Delay (or Latency): 
The time it takes for a packet to travel from its source to its destination. High delay is bad for real-time applications like voice calls (you get that annoying lag) or online gaming.

### 2. Jitter: 
The variation in delay. It's not just about the delay, but how consistent that delay is. If packet 1 arrives in 20ms, packet 2 in 80ms, and packet 3 in 30ms, you have high jitter. This is a major problem for voice and video, as it causes robotic-sounding audio or glitchy, stuttering video streams.

### 3. Packet Loss: 
The percentage of packets that are sent but never arrive at the destination. They get dropped along the way, usually due to network congestion (a router's queue is full). For a file download, a lost packet just gets re-sent (TCP handles this), but for a live video call, that packet is gone for good, causing a momentary freeze or artifact.

### 4. Throughput: 
The actual amount of data that can be successfully transferred over a path in a given time. This is the "bandwidth" you actually get, which is often less than the theoretical maximum. You need high throughput for things like 4K video streaming or large file transfers.

### 5. Reliability: 
A broader measure, often combining packet loss and availability (is the network even up?). It's the probability that the network will perform to its expected standard. For a critical satellite link (e.g., emergency services), reliability is paramount.

### 6. Fairness: 
This ensures that different traffic flows get their "fair share" of network resources. Without fairness mechanisms, a single user downloading a huge file could potentially "starve" all other users' traffic, hogging all the bandwidth.

## QoS Layers: Where It Happened
QoS isn't one single thing; it's a set of actions and policies applied at different layers of the network stack.
<img width="435" height="486" alt="image" src="https://github.com/user-attachments/assets/07414718-7d74-4701-b78c-3d28f4154eba" />


### Application Layer: 
This is where the requirements are born. The application itself (e.g., Netflix, Zoom, your online game) knows what kind of service it needs. A Zoom call knows it needs low delay and low jitter. A background email sync knows it's not time-sensitive. This "service class" (video, voice, data) is the "why" that informs all the QoS actions at the layers below.

### Network QoS (The "Policy" Layer)
This part of the diagram (the top half) is all about the end-to-end strategy. This is where the network classifies traffic and marks it with a policy, mostly happening at Layer 3 (IP).

#### DiffServ (Differentiated Services): 
The most common model. It "marks" packets at the edge of the network with a code (called a DSCP code) in the IP header. Routers in the middle (the "core") don't track individual flows; they just read this simple mark and give the packet the "differentiated" treatment it's supposed to get (e.g., "put this in the high-priority queue"). It's scalable and "coarse-grained."

#### IntServ (Integrated Services): 
An older model that is "fine-grained." Applications would use a signaling protocol (like RSVP) to explicitly reserve a path with guaranteed bandwidth and delay before sending data. It's like booking a private lane on the highway. It provides strong guarantees but is not very scalable and is rarely used on the public internet.

### Link QoS (The "Action" Layer)
This part of the diagram (the bottom half) is all about hop-by-hop enforcement. This is where each router, switch, or (in our case) satellite acts on the policy set by the Network QoS layer.
- This happens at the MAC Protocol layer (Layer 2) and the Physical Layer (Layer 1).
- When a packet with a "Class 1" (high priority) mark arrives at the satellite's transmitter, the MAC Protocol (the scheduler) sees that mark.
- It then uses its Access Control (like its priority queues) to send that "Class 1" packet before the "Class 4" packet.
- This is the "traffic cop" on the highway who sees the ambulance's flashing lights (the DiffServ mark) and waves it into the emergency lane (the priority queue).

## QoS Control Models: The Strategies
These are the overall "philosophies" or "architectures" for how to manage QoS.
### Best-Effort: 
This is the default model of the public internet. There are no guarantees. The network simply tries its "best" to deliver every packet, but if there's congestion, packets get delayed or dropped. It's a "first-in, first-out" free-for-all.
### Integrated Services (IntServ):
As described above, this is the "reservation" model. It provides hard, quantifiable guarantees (e.g., "you will get 2 Mbps with <50ms delay"), but it requires complex signaling (RSVP) and every router in the path to manage the "state" of every single reserved flow.

### Differentiated Services (DiffServ):
As described above, this is the "classification" model. It's more scalable because it doesn't manage individual flows, but rather broad "classes" of traffic (e.g., "Voice," "Video," "Background"). It provides "soft" guarantees or prioritization rather than hard reservations. This is the most widely deployed model.

### SLA-based:
This isn't a technical model like the others, but a business model. An SLA (Service Level Agreement) is a contract between a provider and a customer that defines the QoS parameters you are paying for (e.g., "we guarantee 99.9% uptime and <100ms delay for your premium traffic"). The provider then uses technical models like DiffServ to implement and enforce the promises made in that SLA.

# Scheduling And Routing Mechanisms
First, let's get the core difference clear:
- Routing is the "GPS." It chooses the entire path (which roads) a packet will take to get from A to B. It works at the Network Layer (Layer 3).
- Scheduling is the "traffic cop." It manages the moment-to-moment traffic at a single, congested intersection (like a router or a satellite gateway). It works at the MAC Layer (Layer 2).

You need both for QoS: A great router (GPS) might pick a path that avoids major traffic jams. But when you do hit a congested intersection on that path, you need a smart scheduler (traffic cop) to see your "ambulance" (voice packet) and wave you through the line.

## Scheduling Types & Rulebook
The scheduler is the component (e.g., in the satellite gateway) that has all the packets waiting to be sent over the radio link. It has to decide: Which packet goes next?

### Fixed vs. Dynamic
- **Fixed**: A "dumb" scheduler. It gives each user or traffic type a fixed, pre-assigned slot. It's like a traffic light on a rigid timer.
    - **Problem**: It's very inefficient. A slot for "User A" might be empty, but "User B" (who has an urgent packet) still has to wait for their turn.
- **Dynamic**: A "smart" scheduler. It adapts to the traffic in real-time.
    - **Analogy**: This is a traffic cop who sees the ambulance (priority packet) and immediately changes the lights to let it through. This is essential for satellite networks where user demand and radio conditions change every millisecond.

### Specific Scheduling Algorithms (Rulebooks)
These are the "rules" the dynamic scheduler follows:

### 1. Priority Queue (PQ):
* What it is: The "Ambulance First" rule. The scheduler has multiple queues (e.g., High, Medium, Low). It will always send all packets from the High queue before even looking at the Medium queue.
* QoS Impact: Amazing for low latency (voice calls in the High queue go now).
* Trade-off: Can cause "starvation." If the High queue is always full of voice packets, the "Low" queue (e.g., your file download) might never get a turn.

### 2. Round Robin (RR):
* What it is: The "Super Fair" rule. The scheduler takes one packet from Queue 1, then one from Queue 2, then one from Queue 3, and repeats.
* QoS Impact: Very fair, no starvation.
* Trade-off: Terrible for latency. The "ambulance" in Queue 1 has to wait for the "gravel trucks" in Queues 2 and 3 to take their turn.

### 3. Weighted Fair Queuing (WFQ):
* What it is: The "Smart & Fair" compromise. This is the most common and important one. It's a "weighted" Round Robin.
*  Analogy: The cop's rulebook says, "Let 5 cars from the High queue go, then 3 cars from the Medium queue, then 1 car from the Low queue. Repeat."
*  QoS Impact: This is the best of both worlds. It guarantees the high-priority traffic gets the most bandwidth (throughput) and better latency, while still ensuring the low-priority traffic eventually gets to go (no starvation). It balances throughput, latency, and fairness.

### 4. Admission Control & Resource Reservation
* What it is: This is the "bouncer" at the club door, or the cop at the on-ramp to the highway.
* How it works: Before a new call or video stream is even allowed onto the network, "Admission Control" checks: "Do we have enough resources (bandwidth, scheduling capacity) for this new flow?"
* QoS Impact: If the network is already full, it rejects the new call ("Sorry, we're at capacity") before it can get on and ruin the quality for everyone else. This is a proactive QoS tool that prevents the scheduler from being overwhelmed.

## Routing Types
Routing is about picking the best path through the network. In a satellite network, this could mean routing traffic from a user terminal to Gateway A instead of Gateway B.
- Static: The "Printed Map" rule. The path from A to B is always the same, manually programmed by an administrator.
    * Pro: Simple, predictable.
    * Con: It's "dumb." It will send you straight into a 10-mile traffic jam (congestion) or right into a "road closed" sign (a network failure) because it can't adapt.
- Adaptive (or Dynamic): The "Waze/Google Maps" rule. The router adapts to changing network conditions. This is the standard for modern networks. Sub-types include:
    * Load-aware: The "GPS" sees a traffic jam (high load) on Highway A, so it re-routes you to Highway B, which is currently less congested.
    * QoS-aware: This is the "premium" GPS. You tell it, "I need a route with < 50ms latency" (for your voice call). The router will specifically find a path that meets this QoS requirement, even if it's not the shortest path. It might pick a longer path (in distance) if it knows that path has faster, less-congested routers.

### The Trade-offs
This is the most important concept. You can't have perfect everything.
- Latency vs. Throughput: You often have to choose.
    - A path with the absolute lowest latency (a winding shortcut) might be very narrow and thus have low throughput (can't fit many cars).
    - A path with the highest throughput (an 8-lane superhighway) might have more traffic and stops, giving it higher latency.

- Latency vs. Fairness:
    - Strict Priority Queuing gives perfect low latency to the "ambulance" but is terribly unfair to the "gravel truck" (which starves).
    - Round Robin is perfectly fair but gives terrible latency to the "ambulance."
    - Weighted Fair Queuing (WFQ) is the trade-off between them.

# 3GPP QoS Framework
This is the "law" of 5G QoS. It's all about classifying traffic into standardized profiles.

### 5QI (5G QoS Identifier): 
This is the most important concept. A 5QI is simply a number (e.g., 1, 5, 9, 83) that acts as a shortcut or profile for a whole set of QoS rules. Instead of signaling "I need < 100ms latency, < 1% packet loss, etc." every time, the network just says, "This is 5QI 1 (VoIP)." The gNB (base station) and UPF (router) instantly know the rules for 5QI 1.
    - This is 3GPP's way of implementing the DiffServ model we discussed. The 5QI is the internal 3GPP "class," which later gets mapped to the DiffServ "sticker" (DSCP mark) for the wider internet.
     
### GBR vs. Non-GBR Flows: 
This is the first major split.

#### GBR (Guaranteed Bit Rate): 
This is for traffic that is unusable without a minimum speed. Think video streaming or a VoIP call. You must be guaranteed (for example) 100 kbps, or the call drops. These are high-priority 5QIs.

#### Non-GBR (Non-Guaranteed Bit Rate): 
This is for "best-effort" traffic. Think web browsing or email. The network will try its best, but there's no promise of a specific speed. If the network is congested, this is the first traffic to get slowed down.


## End-to-End QoS
This is how the "policy" is communicated and set up for you. It's the "head office" that assigns your traffic profile.
- AMF (Access and Mobility Function): The "gatekeeper." It handles you (authentication, mobility). It's not directly involved in the data flow, but it's the first point of contact.
- SMF (Session Management Function): The "policy brain." When you start an application (like a video call), the SMF is what decides your QoS. It's the "head office" that says, "This is a video call, I'm assigning it a GBR flow with 5QI 2."
- UPF (User Plane Function): The "data enforcer/router." This is the actual gateway your packets flow through (our "GPS"). The SMF commands the UPF, telling it, "For this user's video session, enforce the 5QI 2 rules and route the packets toward this destination."

This end-to-end flow is critical: The SMF decides the policy, and the UPF (for routing) and the gNB (for scheduling) enforce it.

## 3GPP Scheduling
This is the "action" layer, where the gNB (base station) scheduler manages the actual radio resources to enforce the 5QI.

### Terrestrial 5G (Normal):

#### gNB Scheduler: 
This is our "traffic cop." It sees all the packets from different users waiting to be sent. It looks at their 5QI (the "priority") and uses WFQ (Weighted Fair Queuing) or a similar algorithm to decide which packet goes next. GBR packets (like 5QI 1) always get to cut the line.

#### HARQ (Hybrid ARQ): 
This is a super-fast feedback loop. The gNB sends a packet, and the phone immediately (within milliseconds) sends back an "ACK" (got it!) or "NACK" (I missed it!). If it gets a NACK, the gNB re-sends it right away.

#### RLC/PDCP Buffering: 
These are higher-layer buffers that hold data, manage retransmissions (if HARQ fails), and put packets in the right order.

### NTN Adaptations (The Big Change):

#### The Problem: 
The speed of light. For a GEO satellite, the Round Trip Time (RTT) isn't a few milliseconds; it's ~540ms.

#### NTN Adaptation 1 (HARQ-less): 
- The standard HARQ feedback loop is unworkable. By the time the "NACK" got back to the satellite, the gNB would have sent thousands of other packets. This is "HARQ stalling."
- The solution is often HARQ-less operation. The gNB scheduler disables this fast feedback. It either 
a. sends the data with much stronger error correction (hoping it gets there the first time) or 
b.  relies on the slower, higher-level RLC/PDCP buffers to handle retransmissions. This is a massive change in how the radio is managed.
#### NTN Adaptation 2 (Huge Buffers): 
Because the acknowledgment for a packet takes so long (~540ms), the scheduler needs to have enormous RLC/PDCP buffers to hold all the data that is "in-flight" but not yet confirmed as received.

## 3GPP Routing
This is how the UPF (the "enforcer") chooses the path for the data. For NTN, this is where it gets really interesting.

### UPF Policy Routing: 
The SMF still tells the UPF what to do. The UPF's job is to route the packets to the right NTN Gateway (the big ground station) that talks to the satellite.

### The Big Split: "Transparent" vs. "Regenerative"

#### Transparent ("Bent-Pipe"): 
The satellite is just a "mirror in the sky." It does no processing. The 5G signal goes from the gNB on the ground, bounces off the satellite, and goes down to the user.

- Routing: All routing decisions are made on the ground by the UPF.
- Scheduling: The scheduler (the "traffic cop") is on the ground in the gNB. It's just dealing with a very long delay.

#### Regenerative ("Onboard Processing"): 
This is the game-changer. The satellite is the gNB.
- Scheduling: The scheduler (the "traffic cop") is onboard the satellite. It manages its own radio beams and queues directly.
- Routing: This is the most important part. A regenerative satellite can have Inter-Satellite Links (ISLs)—lasers connecting it to other satellites. Now, the UPF on the ground might just send the data up to the satellite network, and the satellites themselves can route the traffic in space (e.g., from a satellite over Europe to one over North America) before sending it back down. This is true "QoS-aware routing" in space.

# DVB QoS Framework
This is the DVB equivalent of 3GPP's 5QI framework, but it's more of a business and operational model than a rigid technical standard.

### SLA and Service Profiles: 
This is the heart of DVB QoS. Instead of a standardized 5QI 1 (like in 3GPP), a satellite operator creates a custom "Service Profile" (e.g., "Gold_VoIP," "Premium_Video," "Bronze_Data"). This profile is defined in the SLA (Service Level Agreement) sold to the customer. The profile defines the rules:
- Priority: "Gold" traffic goes before "Bronze" traffic.
- Rate: It defines the guaranteed (like GBR) and maximum speeds.
- Scheduling: It even defines how the scheduler should treat this traffic (which we'll see in the next section with RBDC/VBDC).

### QoS Enforcement via NMC/NCC:
- NMC (Network Management Center): This is the "head office." It's the business-level dashboard where the operator creates and provisions the "Gold_VoIP" SLA for a customer.
- NCC (Network Control Center): This is the "operational brain" or "gateway controller." It's the technical component that receives the policy from the NMC. When your terminal (VSAT) logs on, the NCC is what tells the gateway scheduler: "This user is on the 'Gold_VoIP' plan. Enforce those rules for them."

The key is that the DVB gateway reads the DiffServ (DSCP) marks on the incoming IP packets (just like 3GPP) and maps them to the user's contracted "Service Profile."

## DVBs Scheduling
This is the core of DVB-RCS2's QoS and the main area of complexity. It's focused on DAMA (Demand Assigned Multiple Access)—a "smart" dynamic scheduler where terminals request bandwidth when they need it, instead of having a fixed slot.

### DAMA Mechanisms (The "Rulebook"): 
These are the methods a terminal uses to ask the NCC for bandwidth.

#### RBDC (Rate Based Dynamic Capacity): 
This is for GBR-like, streaming traffic (VoIP, video conferencing). The terminal (VSAT) sends a small "heartbeat" request to the NCC saying, "I continue to need 100 kbps for this voice call." The NCC reserves this capacity for them in the schedule.

#### VBDC (Volume Based Dynamic Capacity): 
This is for Non-GBR, bursty traffic (web browsing). The terminal monitors its own buffer, and when it fills up with data, it sends a one-time request to the NCC saying, "I have a 5MB volume of data to send." The NCC finds a slot for that burst and sends back the assignment.

#### CRA (Continuous Rate Assignment) 
which is a permanent reserved rate, like a GBR flow that never stops.

### Centralized vs. Distributed Scheduling:

#### Centralized (Standard DVB-RCS): 
This is the "hub-centric" model. The NCC (on the ground) is the only "traffic cop." All terminals send their RBDC/VBDC requests up to the satellite and down to the NCC. The NCC then creates a master schedule and beams it back up to the satellite for all terminals to see.

- Pro: Very smart and efficient (the NCC sees everything).
- Con: Very high delay. A request takes one full satellite hop (~540ms) just to ask, and another hop to get the answer.

#### Distributed: 
(Used in more advanced systems). The "traffic cop" logic is partially (or fully) onboard the satellite or even at the terminals. This is much faster as it cuts out the "mother-may-I" hop to the ground.


## DVBs Routing
This is directly analogous to the 3GPP NTN model, but with DVB-specific names.

### Hub-centric Routing (Transparent): 
This is the "bent-pipe" model. The satellite is just a "mirror." All traffic must go from the user up to the satellite, down to the central hub/NCC (where the "GPS" router is), and then (if going to another user) back up to the satellite and down to the destination user. This is a "double-hop," which is terrible for latency.

### OBP-based Onboard Routing (Regenerative):
This is the game-changer. The satellite has On-Board Processing (OBP). It is the router. A user's packet goes up to the satellite, the OBP itself reads the packet's destination, and routes it directly back down to the destination user. This is a "single-hop" and is the key to low-latency DVB networks.

## QoS Control Tools
This is a key feature of DVB-S2X, where QoS isn't just about queuing, but about link survival.

### ACM (Adaptive Coding and Modulation): 
This is a brilliant "physical layer" QoS tool.

#### MODCOD: 
A "Modulation and Coding" pair (e.g., 8PSK 3/4) is like the "gear" your modem is in. A high gear (like 32APSK) gives high speed but is very sensitive to noise (like rain). A low gear (like QPSK 1/2) is slow but very robust.

#### ACM (The "Automatic Transmission"): 
The system constantly monitors the link quality (e.g., signal-to-noise ratio). When it starts to rain at your terminal (this is called rain fade), the signal gets weaker. The ACM automatically shifts your link to a lower, more robust MODCOD (e.g., from 32APSK down to 8PSK).

#### QoS Impact: 
Your peak speed (throughput) drops, but your connection stays alive and your packet loss stays near zero. This preserves the quality of service (especially for a voice call) by sacrificing maximum speed for reliability.

# Comparison: 3GPP NTN Vs. DVBs QoS

| Feature | 📱 3GPP NTN (5G) | 📡 DVB-S2X / RCS2 |
| :--- | :--- | :--- |
| **QoS Policy (The "Law")** | **5QI (5G QoS Identifier):** A *standardized* set of numbers (1-255). 5QI 1 *always* means "VoIP." This is integrated into the 5G Core. | **SLA / Service Profiles:** *Custom-defined* by the satellite operator (e.g., "Gold," "Silver"). Flexible, but not standardized between operators. |
| **Policy "Brain" (The "Enforcer")** | **5G Core (SMF & UPF):** The **SMF** (policy brain) *instructs* the **UPF** (router) and **gNB** (scheduler) on which 5QI rules to enforce. | **NMC / NCC (Network Control Center):** The **NMC** (business tool) provisions the SLA, and the **NCC** (technical brain) enforces it at the gateway. |
| **Scheduling (The "Traffic Cop")** | **gNB Scheduler:** A highly dynamic scheduler (like **WFQ**) at the gNB (base station) that classifies flows by their 5QI and manages radio resources. | **DAMA (Demand Assigned Multiple Access):** A *request/grant* system. Terminals must *ask* the NCC for bandwidth using **RBDC** (for GBR) or **VBDC** (for Non-GBR). |
| **Routing (The "GPS")** | **UPF-based:** The **UPF** is the main router. It routes user traffic to the correct NTN gateway on the ground, based on 5G Core policies. | **Hub-centric:** The default model. All traffic *must* go from the user to the central hub/NCC on the ground first, even if the destination is nearby. |
| **Advanced Routing (Regenerative)** | **Onboard gNB / ISLs:** The satellite *is* the gNB. It can route traffic directly (single-hop) or use **Inter-Satellite Links (ISLs)** to route in space, all managed by the 5G Core. | **Onboard OBP / ISLs:** The satellite *is* the router. It can perform a "single-hop" to another user or use **ISLs**. This breaks the "hub-centric" model. |
| **Link Adaptation (QoS "Action")** | **RLC/PDCP Buffering:** Relies on large buffers and retransmission timers at higher layers (RLC/PDC) to handle packet loss, especially in HARQ-less NTN. | **ACM / MODCOD:** The gold standard. *Proactively* changes the link's "gear" (MODCOD) to maintain a robust link (low packet loss) during rain fade, sacrificing peak speed. |

## Key Differences
- Standardization vs. Customization: 3GPP's QoS is rigidly standardized with the 5QI framework. This is essential so a 5G phone can roam on any 5G NTN network and get the same QoS. DVB's QoS is operator-defined via SLAs, making it flexible but proprietary.
- Scheduling Philosophy:
    - 3GPP (gNB Scheduler): Acts like a "smart traffic cop" managing a real-time flow. It's constantly looking at its queues (sorted by 5QI) and making instant decisions.
    - DVB (DAMA): Acts like a "meeting scheduler." Terminals have to book a time slot (request bandwidth) and wait for the NCC to grant it in the master schedule. This is inherently slower due to the request/grant cycle.
- Core Architecture:
    - 3GPP NTN is an extension of the 5G Core. The "smarts" (SMF/UPF) are deep in the mobile network. The satellite is just another "radio access type" (like Wi-Fi or 4G) from the core's perspective.
    - DVB-RCS2 is gateway-centric. The "smarts" all live in the NCC at the main ground station. The entire system is built around this hub.






