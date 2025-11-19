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

## 4. Notes
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
