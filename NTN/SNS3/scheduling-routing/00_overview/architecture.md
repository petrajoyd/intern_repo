# Architecture Overview (SNS3 — Routing & Scheduling)

## Purpose
Provide a high-level understanding of how packets move through the SNS3 stack, and how routing, MAC scheduling, queues, and the PHY layer interact. This section clarifies where key decisions occur and which modules are responsible.

## 1. High-Level Diagram
````
+------------------+
|   Application    |
+------------------+
          |
          v
+------------------+
|   Transport L4   |  (TCP/UDP)
+------------------+
          |
          v
+-------------------------+
| Network Layer L3       |  (IPv4, Routing protocols)
+-------------------------+
      | route decision
      v
+-------------------------+
|   NetDevice (L2 entry)  |
+-------------------------+
          |
          v
+-------------------------+
|      MAC Layer (L2)     |
|  - Queue                |
|  - Scheduler            |
|  - Frame Builder        |
+-------------------------+
     | scheduling decision
     v
+-------------------------+
|     PHY Layer (L1)      |
|  - Burst Transmission   |
|  - Modulation/Coding    |
+-------------------------+
          |
          v
+-------------------------+
|      Channel Model      |
|  (LEO Doppler, Delay)   |
+-------------------------+
````
Key Idea:
- Routing = chooses where the packet must go
- Scheduling = chooses when and which packet is sent
- MAC = transforms packets into bursts/frames
- PHY = transmits bursts in timeslots

## 2. Key Concepts
### Packets (L3 → L2)
- Standard ns-3 `Ptr<Packet>` objects.
- Enter the MAC via Enqueue().
- Routing has already selected the outgoing interface.

### Frames / Bursts (MAC → PHY)
- MAC aggregates packets into bursts based on:
- timeslot availability
- scheduling algorithm
- link direction (forward/return link)

### Scheduling Opportunities
- Timeslots or transmission opportunities defined by:
  - satellite frame structure
  - carrier configuration
  - dynamic bandwidth assignment
- Scheduler decides which queue and which packet fills each opportunity.

### Routing Decisions (L3)
- Performed before packets reach MAC.
- Depends on ground/satellite topology and routing protocol.
- Determines the next hop and output NetDevice.

## 3. Where decisions occur
### Routing Decision — L3
- Triggered when IP sends a packet.
- Implemented by `Ipv4RoutingProtocol` or custom satellite routing manager.
- Output: selected NetDevice.

### Queue Management — MAC or QueueDisc
- When packet arrives at the device, it is put into:
  - MAC internal queue, or
  - QueueDisc (if configured).
- Responsible for:
  - buffering
  - tail drop/RED/etc
  - maintaining backlog before scheduling

### Scheduling Decision — MAC
- Happens at MAC layer before PHY transmission.
- Implemented by modules such as:
  - `SatMac`
  - `SatSchedulingAlgorithm`
- Scheduler selects:
  - which queue
  - how many bytes
  - in what order
- Produces a MAC burst, passed to PHY.

### PHY Transmission — L1
- Converts MAC bursts into physical waveforms.
- Handles:
  - modulation/coding
  - timing
  - link budget
- Sends burst to the satellite/ground channel.

## 4. Component Responsibilities (SNS3 Routing & Scheduling Architecture)
This section defines the responsibilities and boundaries of each core SNS3 subsystem to clarify how data moves through the stack.
### Component Role Overview
| Component                              | Responsibilities                                                                                 | Notes                                                                                  |
| -------------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| **Routing Layer (L3)**                 | Next-hop computation, path selection, handling topology changes.                                 | Based on ns-3 IPv4 routing modules; satellite routing logic may be custom-implemented. |
| **NetDevice (L2 entry)**               | Interface between L3 and MAC/PHY. Forwards packets to L2 and triggers MAC enqueue.               | Device glue layer for satellite terminals and satellites.                              |
| **MAC Layer (SatMac)**                 | Packet enqueue/dequeue; interacts with scheduler; forms MAC bursts; manages flow control.        | Core component for scheduling & burst assembly.                                        |
| **Scheduler (SatSchedulingAlgorithm)** | Selects packets from queues according to rules (QoS, traffic class, priority, frame allocation). | Drives resource allocation behavior and timing.                                        |
| **MAC Queue / QueueDisc**              | Buffers packets awaiting scheduling; optional shaping/dropping logic.                            | Queue length directly influences scheduling outcomes.                                  |
| **PHY Layer (SatPhy)**                 | Converts MAC bursts into physical transmissions; applies modulation, coding, timing.             | Responsible for waveform-level behavior.                                               |
| **Channel Model (LEO/GEO)**            | Applies propagation delay, Doppler shift, atmospheric effects, satellite motion.                 | Central to satellite-specific timing.                                                  |

SNS3 separates routing (path selection) and scheduling (resource allocation). The MAC layer resolves when packets are transmitted; L3 decides where they go.

## 5. SNS3 Timing Model (Critical for Scheduling)
Satellite scheduling relies heavily on timing, frame boundaries, and event-driven operations. SNS3 uses ns-3’s discrete-event simulator to model satellite frame behavior precisely.

### 5.1 Event-Driven Timing
- All scheduling decisions are invoked by scheduled events (`Simulator::Schedule()`).
- MAC-level events (burst formation, queue checks) are bound to frame ticks.
- PHY triggers events for burst completion, reception, or error events.

### 5.2 Frame and Superframe Structure
- Depending on the configuration (LEO or GEO model), SNS3 uses:
  - Superframe: higher-level frame grouping
  - Frames: scheduling decisions typically occur once per frame
  - Timeslots: discrete transmission opportunities assigned by scheduler

### 5.3 Scheduling Invocation
- Typical lifecycle:
  - Initialization → register periodic frame events
  - On frame boundary → invoke scheduling algorithm
  - Scheduler selects packets → builds burst map
  - MAC executes transmission → calls PHY
  - PHY finishes burst → triggers RX/MAC events

### 5.4 Timing Constraints for Satellite Links
- Large RTT (LEO ~20–40 ms, GEO ~500–600 ms) impacts buffer sizes
- Doppler/propagation delays affect PHY event scheduling
- Scheduling must align with PHY timing windows

## 6. Detailed Interaction Sequence (SNS3 Routing → MAC → PHY)
This section provides a step-by-step flow of packet handling in the satellite stack.

### 6.1 Uplink / Generic TX Sequence
````
Application
  ↓
Transport Layer (TCP/UDP)
  ↓
IPv4 L3 Routing
  ↓   [Routing decision: choose NetDevice]
NetDevice::Send()
  ↓
SatMac::Enqueue(packet)
  ↓
[Wait until next frame boundary]
  ↓
Scheduler invoked (SatSchedulingAlgorithm::Run())
  ↓
SatMac::BuildBurst()      (select packets, aggregate, apply size/MCS constraints)
  ↓
SatPhy::Send(burst)
  ↓
Channel (propagation, Doppler, delay)
````

### 6.2 Downlink / RX Sequence
````
Channel
  ↓
SatPhy::Receive(burst)
  ↓
SatMac::ReceiveBurst()
  ↓
SatMac → NetDevice (L2 → L3 interface)
  ↓
Ipv4L3Protocol::Receive()
  ↓
Transport → Application
````

### 6.3 Critical Decision Points
- Routing decision: L3, before enqueueing
- Queueing: when packet enters MAC
- Scheduling: frame boundary → scheduler selects next packets
- Burst formatting: MAC → PHY boundary
- Transmission timing: PHY event scheduling

## 7. Key Data Structures (Routing, Queues, Scheduling, PHY)
Below are core data structures frequently encountered in SNS3’s codebase.

### 7.1 MAC Queue Structures
Typical implementations in MAC:
- `std::queue<Ptr<Packet>>`
- Metadata structures storing:
 - direction (UL/DL)
 - QoS class
 - timestamp (for delay tracking)
Queues feed input to the scheduler.

### 7.2 Burst/Frame Construction Structures
MAC uses burst structures that contain:
- list of packets
- total size (bytes)
- frame/slot indices
- MCS parameters
- timestamps for PHY alignment
These structures are typically passed to the PHY for encoding.

### 7.3 Scheduling Data Structures
Schedulers use:
- SlotMap / AllocationMap: which timeslot assigned to which terminal
- SchedulingInfo objects: contain queue states, priorities, backlog sizes
- Per-flow stats: throughput, delay, weights
These define who gets to send and how much.

### 7.4 Routing Tables
Uses ns-3 standard routing data structures:
- `Ipv4RoutingTableEntry`
- adjacency lists
- interface-index bindings
Satellite routing enhancements may add:
- propagation-based metrics
- topology snapshot structures
- satellite node identifiers

### 7.5 PHY Data Structures
- Common PHY-layer objects include:
- BurstInfo: contains waveform-level metadata
- Modulation/Coding parameters
- Timing offsets (satellite Doppler corrections)
- Carrier configuration
These structures determine how a burst is physically transmitted.

## 8. Notes
- Routing (L3) and Scheduling (L2) are completely separate—routing selects the path, MAC prioritizes transmissions within the allocated capacity.
- The scheduler is the heart of satellite MAC: it maps queue backlog → timeslot allocation → burst formation.
- In satellite systems with high RTT, scheduling decisions are frame-based, not continuous.
- SNS3 breaks responsibilities cleanly:
  - Routing: network-wide path selection
  - MAC queue + scheduler: local resource management
  - PHY: transmission timing & link behavior
- Understanding scheduling requires understanding:
  - queue structure
  - timeslot maps
  - burst builder logic
  - PHY frame timing
- Routing affects which MAC receives packets, but scheduling affects how fast those packets actually get transmitted.
