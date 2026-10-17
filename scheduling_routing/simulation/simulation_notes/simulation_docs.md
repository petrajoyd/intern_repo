# 🛰️ `contrib/satellite` Module Documentation

## Overview

The **`contrib/satellite`** module in `ns-3.43` provides tools and models to simulate **satellite communication systems**, including LEO/GEO orbits, beam scheduling, link adaptation, LoRaWAN integration, and physical/MAC-level channel behavior.
This module integrates both **DVB-S2X** and **LoRa-based** satellite links for flexible end-to-end testing.

---

## Directory Structure

  ````
  contrib/satellite/
  │
  ├── CMakeLists.txt             # Build configuration for the module
  ├── README.md                  # Main documentation (original)
  ├── data/                      # Simulation input/output files and logs
  │   └── sims/                  # Simulation scenarios (CBR, Lora, tutorial, etc.)
  │
  ├── model/                     # Core simulation models and protocols
  │   ├── satellite-phy*.cc/h    # Physical layer modeling
  │   ├── satellite-mac*.cc/h    # MAC and scheduling logic
  │   ├── satellite-channel*.cc  # Channel and propagation delay models
  │   ├── satellite-ncc.cc/h     # Network Control Center (resource allocation)
  │   ├── satellite-mobility*.cc # SGP4 and mobility modeling
  │   ├── lora-*.cc/h            # LoRaWAN end-device and gateway integration
  │   └── satellite-utils.h      # Utility and helper definitions
  │
  ├── helper/                    # Wrapper classes for easier simulation setup
  │   ├── satellite-helper.cc/h  # High-level simulation configuration
  │   ├── satellite-channel-helper.cc/h
  │   ├── satellite-mac-helper.cc/h
  │   └── ...
  │
  ├── examples/                  # Prebuilt simulation examples
  │   ├── sat-cbr-example.cc
  │   ├── sat-onoff-example.cc
  │   ├── sat-lora-example.cc
  │   ├── sat-mobility-example.cc
  │   └── sat-tutorial-example.cc
  │
  ├── stats/                     # Statistics collectors
  │   └── satellite-stats-*.cc/h
  │
  ├── utils/                     # Logging, constants, and environment settings
  │   ├── satellite-log.cc/h
  │   ├── satellite-env-variables.cc/h
  │   └── ...
  │
  └── test/                      # Unit and validation tests
      └── satellite-*test.cc
  ````
  ---

## Key Components

| Component | Description |
|------------|--------------|
| **Physical Layer (PHY)** | Models path loss, fading (Rayleigh, Loo), and SINR-based reception. |
| **MAC Layer** | Implements scheduling (e.g., DAMA, CRDSA, time slicing) and frame management. |
| **NCC (Network Control Center)** | Handles capacity requests and beam management. |
| **Mobility Models** | Includes `satellite-sgp4-mobility-model` for orbit prediction. |
| **LoRa Integration** | Supports satellite–LoRa hybrid constellations. |
| **Logging System** | Creates trace files such as `PacketTrace.log` and `SimInfo.log` under `data/sims/`. |

---

## Example Simulation Workflow

Run one of the provided examples:
```bash
./ns3 run "contrib/satellite/examples/sat-cbr-example"
````

Output logs will appear under:

```
contrib/satellite/data/sims/example-cbr/simple/PacketTrace.log
```

View the results:

```bash
head -n 20 contrib/satellite/data/sims/example-cbr/simple/PacketTrace.log
```

---

## Common Log Files

| File                | Description                                             |
| ------------------- | ------------------------------------------------------- |
| **PacketTrace.log** | Event log of sent/received/dropped packets (per layer). |
| **SimInfo.log**     | General simulation configuration info.                  |
| **SimDiff.log**     | Difference log used for scenario validation.            |

---

## Dependencies

* Requires C++17 support (`<cstdint>`, `<numbers>`)
* Linked with `core`, `network`, `internet`, and `lte` ns-3 modules.
* Data submodule: `https://github.com/sns3/sns3-data.git`

---

## Maintainers

This module was originally developed by **SNS3 / SIGNET Lab DEI** and adapted for hybrid satellite–terrestrial research.

---

**Next Step →**
Explore each example under `examples/` to analyze:

* Throughput (CBR, OnOff)
* Mobility behavior (SGP4 model)
* LoRa and DVB-S2X hybrid cases

# 🛰️ Satellite Model Directory Overview

The `model/` directory contains the **core implementations** of the satellite network simulation — covering physical, MAC, mobility, and higher-level network components (NCC, UT, GW).
It also includes **LoRaWAN integration**, **fading models**, and **orbital propagation (SGP4)**.

---

## 1. Core Models

| File(s)                    | Description                                                                    |
| -------------------------- | ------------------------------------------------------------------------------ |
| `satellite-phy*.cc/h`      | Implements physical-layer logic (signal propagation, power, and interference). |
| `satellite-mac*.cc/h`      | Handles MAC scheduling, retransmission (ARQ), and traffic control.             |
| `satellite-llc.cc/h`       | Logical Link Control layer bridging MAC and higher layers.                     |
| `satellite-ncc.cc/h`       | Network Control Center — manages bandwidth allocation and link setup.          |
| `satellite-node-info.cc/h` | Stores node (UT/GW/SAT) configuration and state data.                          |
| `satellite-utils.h`        | Common constants, helper macros, and data types.                               |

---

## 2. PHY (Physical Layer)

| File(s)                                                                | Description                                                                 |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `satellite-phy.cc/h`, `satellite-phy-tx.cc/h`, `satellite-phy-rx.cc/h` | Core transmit/receive handlers.                                             |
| `satellite-phy-rx-carrier-*.cc/h`                                      | Different models for signal reception (e.g., Marsala, per-frame, per-slot). |
| `satellite-free-space-loss.cc/h`                                       | Implements free-space path loss model.                                      |
| `satellite-cno-estimator.cc/h`                                         | Computes carrier-to-noise ratio.                                            |
| `satellite-rayleigh-model.cc/h`, `satellite-loo-model.cc/h`            | Fading models (Rayleigh, Loo).                                              |
| `satellite-fading-input/output-trace*.cc/h`                            | Input/output trace handling for fading profiles.                            |
| `satellite-propagation-delay-model.cc/h`                               | Calculates delay based on satellite geometry.                               |

---

## 3. MAC Layer and Scheduling

| File(s)                                                                 | Description                                                         |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------- |
| `satellite-mac.cc/h`                                                    | Main MAC entity handling TDMA frame scheduling and retransmissions. |
| `satellite-bbframe*.cc/h`                                               | Baseband frame definitions and container management.                |
| `satellite-frame-allocator.cc/h`, `satellite-superframe-allocator.cc/h` | Assigns frames and slots for uplink/downlink.                       |
| `satellite-default-superframe-allocator.cc/h`                           | Default TDMA superframe configuration.                              |
| `satellite-scpc-scheduler.cc/h`                                         | Single Channel Per Carrier scheduling algorithm.                    |
| `satellite-fwd-link-scheduler*.cc/h`                                    | Downlink scheduling strategies (default/time-slicing).              |
| `satellite-ut-scheduler.cc/h`                                           | User terminal scheduler for request and response management.        |
| `satellite-scheduling-object.cc/h`                                      | Data structure for time-slot allocation and conflict resolution.    |

---

## 4. ARQ and Random Access

| File(s)                                   | Description                                                    |
| ----------------------------------------- | -------------------------------------------------------------- |
| `satellite-arq-*.cc/h`                    | Implements Automatic Repeat reQuest for reliable transmission. |
| `satellite-random-access-container*.cc/h` | Defines RA slots and contention resolution.                    |
| `satellite-dama-entry.cc/h`               | Implements Demand Assigned Multiple Access scheme.             |
| `satellite-crdsa-replica-tag.cc/h`        | For Contention Resolution Diversity Slotted ALOHA (CRDSA).     |

---

## 5. Mobility and Orbital Models

| File(s)                                                             | Description                                         |
| ------------------------------------------------------------------- | --------------------------------------------------- |
| `satellite-sgp4-mobility-model.cc/h`                                | Satellite position prediction using SGP4 algorithm. |
| `satellite-mobility-model.cc/h`, `satellite-mobility-observer.cc/h` | Manages node motion and observation.                |
| `satellite-position-allocator.cc/h`                                 | Places satellites and UTs in simulation space.      |
| `satellite-constant-position-mobility-model.cc/h`                   | Static position model for GEO satellites.           |
| `satellite-traced-mobility-model.cc/h`                              | Replay mobility traces for validation.              |

---

## 6. LoRa and IoT Integration

| File(s)                             | Description                                                 |
| ----------------------------------- | ----------------------------------------------------------- |
| `lora-*.cc/h` and `lorawan-*.cc/h`  | LoRaWAN end-device, gateway, MAC, and scheduler components. |
| `satellite-lorawan-net-device.cc/h` | Satellite interface for LoRa-based nodes.                   |
| `satellite-lora-phy-{tx,rx}.cc/h`   | Physical layer adaptation for LoRa PHY over satellite.      |

---

## 7. Topology and Control

| File(s)                          | Description                                              |
| -------------------------------- | -------------------------------------------------------- |
| `satellite-topology.cc/h`        | Describes constellation layout, nodes, and connectivity. |
| `satellite-channel.cc/h`         | Base channel linking GW ↔ SAT ↔ UT.                      |
| `satellite-isl-arbiter*.cc/h`    | Inter-satellite link arbitration and routing.            |
| `satellite-handover-module.cc/h` | Handles beam or satellite handover logic.                |
| `satellite-ncc.cc/h`             | Central network control coordination.                    |

---

## 8. Data & Trace Containers

| File(s)                                                | Description                                     |
| ------------------------------------------------------ | ----------------------------------------------- |
| `satellite-base-trace-container.cc/h`                  | Common structure for logging simulation traces. |
| `satellite-packet-trace.cc/h`                          | Packet-level tracing.                           |
| `satellite-composite-sinr-output-trace-container.cc/h` | Outputs SINR over time.                         |
| `satellite-link-results.cc/h`                          | Stores performance metrics (delay, throughput). |

---

## 9. Mathematical and Reference Utilities

| File(s)                                                                      | Description                                        |
| ---------------------------------------------------------------------------- | -------------------------------------------------- |
| `geo-coordinate.cc/h`, `julian-date.cc/h`, `iers-data.cc/h`                  | Astronomical time and coordinate transformations.  |
| `satellite-sgp4unit.cc/h`, `satellite-sgp4io.cc/h`, `satellite-sgp4ext.cc/h` | Low-level SGP4 satellite motion equations.         |
| `vector-extensions.cc/h`                                                     | Helper math operations for vectors and positions.  |
| `satellite-mutual-information-table.cc/h`                                    | Used for adaptive coding and modulation decisions. |

---

## 10. Tags, Headers, and Enums

| File(s)                                                  | Description                                       |
| -------------------------------------------------------- | ------------------------------------------------- |
| `satellite-address-tag.h`, `satellite-uplink-info-tag.h` | Metadata tagging packets in transmission.         |
| `satellite-enums.h`                                      | Enum definitions for directions, link types, etc. |
| `satellite-log.h`                                        | Logging interface for debugging and tracing.      |
| `satellite-typedefs.h`                                   | Common typedefs for simulation components.        |

---

## Summary

This directory provides all **low-level logic and scientific modeling** behind the satellite simulation.
It’s where the **physical phenomena** (fading, delay, interference) and **system logic** (scheduling, MAC control, handovers) meet.

When developing extensions (e.g., beam-hopping, NTN NR overlays), you’ll typically modify or extend:

* `satellite-mac.*` and `satellite-frame-allocator.*` for link adaptation
* `satellite-phy.*` for waveform or power control
* `satellite-mobility-model.*` for new orbital regimes (LEO/MEO/GEO)

# Satellite Module `satellite-helper.cc`, `satellite-channel-model.cc` Documentation (ns-3.43)

## 1. Overview

This document provides a structured overview of the `contrib/satellite` module in ns-3.43, focusing on how helpers connect PHY, MAC, and network layers. This section explores `satellite-helper.cc` and `satellite-channel-model.cc`, which form the glue logic between high-level scenario configuration and low-level model execution.

---

## 2. The Role of Helpers

### **a. What Are Helpers?**

In ns-3, *helpers* simplify simulation setup by wrapping complex model configurations into reusable interfaces. They bridge the gap between user-defined scenarios (in `examples/`) and core models (in `model/`).

The key helper in the satellite module is `SatHelper`, located at:

```
contrib/satellite/helper/satellite-helper.cc
```

It works alongside other components like `SimulationHelper` to orchestrate node creation, device installation, and PHY/MAC configuration.

---

## 3. `SatHelper` — The Core Builder

### **Purpose**

`SatHelper` acts as the *central configuration and connection layer*. It abstracts away the complexity of linking User Terminals (UT), Gateways (GW), and Satellites (SAT) with their corresponding physical (PHY) and MAC layers.

### **Key Responsibilities**

| Function                        | Description                                                                 |
| ------------------------------- | --------------------------------------------------------------------------- |
| `CreateSatScenario()`           | Builds a complete satellite topology (UTs, GWs, Satellites, NCC).           |
| `ConfigurePhyMac()`             | Instantiates and attaches PHY and MAC models to each node.                  |
| `InstallNetDevices()`           | Attaches `SatNetDevice` to nodes, enabling packet transmission.             |
| `ConfigurePropagationChannel()` | Links each PHY to a `SatChannel` and sets up delay/fading models.           |
| `SetDefaultParameters()`        | Loads parameters from XML or environment variables (`SatEnvVariables`).     |
| `ConnectTraces()`               | Binds events (like SND/RCV) to trace sinks for logging (`PacketTrace.log`). |

### **Layer Connection Flow**

```
+------------------+
| SimulationHelper |
+------------------+
          ↓
     [ SatHelper ]
          ↓
+------------------+       +-----------------+
| SatNetDevice     | <-->  | SatChannelModel |
+------------------+       +-----------------+
  | PHY | MAC | LLC |          | Delay | Fading |
```

Each `SatNetDevice` instance represents a satellite interface (e.g., GW-to-SAT, SAT-to-UT), with PHY and MAC layers configured via helper routines.

### **Example Internal Flow (simplified)**

```cpp
Ptr<Node> ut = CreateObject<Node>();
Ptr<SatNetDevice> utDevice = helper->InstallUtDevice(ut);
ut->AddDevice(utDevice);
```

* The helper installs a UT device that includes PHY/MAC models and connects it to the appropriate channel.

---

## 4. `SatChannelModel` — The Physical Connection

### **Location**

```
contrib/satellite/model/satellite-channel.cc
contrib/satellite/model/satellite-propagation-delay-model.cc
```

### **Purpose**

The `SatChannelModel` simulates signal propagation between nodes, including free-space loss, fading, and interference. It is dynamically configured by `SatHelper` to reflect the chosen scenario and environmental conditions.

### **Key Components**

| File                                   | Role                                                                          |
| -------------------------------------- | ----------------------------------------------------------------------------- |
| `satellite-channel.cc`                 | Defines the communication path and packet flow between devices.               |
| `satellite-propagation-delay-model.cc` | Calculates delay based on satellite geometry (distance and orbital position). |
| `satellite-fading-models`              | Adds time-varying signal degradation (Rayleigh, Loo, or external traces).     |

### **Propagation Flow**

```
GW PHY TX → SatChannelModel → SAT PHY RX
                   ↓
            DelayModel + FadingModel
```

Each PHY’s transmit function triggers a propagation event handled by the channel, which applies attenuation and delay before the packet reaches the receiving PHY.

---

## 5. Integration Between Helper and Model Layers

1. **Helper Configures** — `SatHelper` creates nodes and attaches `SatNetDevice`.
2. **Channel Binding** — Each `SatNetDevice` connects to a shared `SatChannelModel` instance.
3. **Model Execution** — The channel applies propagation, and PHY/MAC handle transmission/reception.
4. **Tracing** — Events are forwarded to trace sinks (PacketTrace, SimInfo, etc.) for analysis.

### **Simplified Call Stack**

```
SimulationHelper::RunSimulation()
  └── SatHelper::CreateSatScenario()
        ├── SatHelper::ConfigurePhyMac()
        ├── SatHelper::ConfigurePropagationChannel()
        └── SatChannel::Transmit()
```

---

## 6. Key Takeaways

* `SatHelper` is the *bridge* between scenario-level configuration and low-level PHY/MAC models.
* `SatChannelModel` defines how signals propagate and degrade through space.
* Together, they enable modular construction of realistic satellite communication systems.

Next → Deep dive into `satellite-net-device.cc` and `satellite-phy.cc` to see how actual packet transmission occurs through the connected layers.

--- 

# Analyze SatNetDevice

## Goal:
Understand the structure and internal logic of `SatNetDevice`, which acts as the main network interface connecting PHY, MAC, and the ns-3 IP stack in the Satellite Module.

---

### Files
```
contrib/satellite/model/satellite-net-device.cc  
contrib/satellite/model/satellite-net-device.h
```

---

## Overview
`SatNetDevice` is the **satellite-specific implementation** of ns-3’s `NetDevice` class.  
It represents the “network card” that sits between the ns-3 node (UT, GW, or SAT) and the underlying MAC/PHY/channel layers.

---

## Core Responsibilities

| Function | Description |
|-----------|-------------|
| `Send()` / `SendFrom()` | Handles outgoing packets, attaches statistics tags, and pushes data into the LLC queue. |
| `Receive()` | Handles incoming packets, computes delay/jitter, and delivers them up to the IP layer. |
| `SetPhy()` / `SetMac()` / `SetLlc()` | Connects physical, MAC, and logical link layers to the device. |
| `SetNodeInfo()` | Links this device to its node metadata (ID, type, MAC address). |
| `ToggleState()` | Enables or disables the MAC operation. |
| `SetReceiveCallback()` | Registers callback for upper-layer delivery. |
| `DoDispose()` | Cleans up attached components and releases pointers. |

---

## Key Data Members

| Variable | Type | Role |
|-----------|------|------|
| `m_phy` | `Ptr<SatPhy>` | Connects to the physical layer (handles modulation, radio link). |
| `m_mac` | `Ptr<SatMac>` | Medium Access Control (queues, retransmissions, scheduling). |
| `m_llc` | `Ptr<SatLlc>` | Logical Link Control (frame assembly and buffer management). |
| `m_node` | `Ptr<Node>` | The owning ns-3 node (UT, GW, or SAT). |
| `m_classifier` | `Ptr<SatPacketClassifier>` | Determines which flow each packet belongs to. |
| `m_receiveErrorModel` | `Ptr<ErrorModel>` | Models random losses or interference. |
| `m_packetTrace`, `m_txTrace`, `m_rxTrace` | `TracedCallback` | Used for packet-level event logging. |

---

## Installation Process via Helper

### **a. Entry Point**
Device installation begins in `satellite-helper.cc` using methods such as:
```cpp
Ptr<SatNetDevice> dev = CreateObject<SatNetDevice>();
node->AddDevice(dev);
```
The helper then configures the internal components (PHY, MAC, LLC) for the new device.

### **b. Configuration Steps**
The general flow is:
```
SatHelper::CreateSatScenario()
    ├── Create Nodes (UT, GW, SAT, NCC)
    ├── For each node type:
    │     ├── Create SatNetDevice
    │     ├── Set MAC, PHY, LLC
    │     └── Attach Device to Node
    ├── Connect Devices through SatChannel
    └── Set Tracing (PacketTrace.log, etc.)
```

Each SatNetDevice is initialized with dedicated PHY/MAC layers and linked to a shared channel depending on the communication direction (forward/return).

---

## Data Flow

### **Sending Flow**
```cpp
Send() → Add tags (address, timestamps)
       → Trace: PACKET_SENT
       → Classify flow (SatPacketClassifier)
       → Enque() packet into SatLlc
       → MAC → PHY → Channel
```

### **Receiving Flow**
```cpp
Receive() → Log PACKET_RECV
          → Extract SatAddressTag, SatDevTimeTag
          → Compute delay & jitter
          → Trace results
          → Deliver packet to IP layer via callback
```

---

## Connection Flow Between Layers

### **a. Internal Wiring**
Each SatNetDevice encapsulates three main layers (PHY, MAC, LLC).  
They are linked internally via `SetPhy()`, `SetMac()`, and `SetLlc()` methods during initialization.

```
+----------------------+
| SatNetDevice         |
|----------------------|
| + m_phy              |
| + m_mac              |
| + m_llc              |
+----------------------+
      ↓          ↓
 +----------+  +----------+
 | SatPhy   |  | SatMac   |
 +----------+  +----------+
      ↓
 +----------------------+
 | SatChannelModel      |
 +----------------------+
```

### **b. How Helpers Attach Components**
```cpp
Ptr<SatNetDevice> dev = CreateObject<SatNetDevice>();
dev->SetPhy(phy);
dev->SetMac(mac);
dev->SetLlc(llc);
node->AddDevice(dev);
```
Then the helper connects each PHY to a shared channel:
```cpp
phy->SetChannel(channel);
```

This chain allows packet flow like:
```
Application → IP → LLC → MAC → PHY → Channel → PHY (receiver) → MAC → LLC → IP → Application
```

---

## Tracing and Statistics
The helper also enables traces for debugging and analysis.  
In `SatNetDevice`, these are defined as trace sources like `m_packetTrace`, `m_txTrace`, and `m_rxTrace`, which record key simulation events (SEND, RECV, DROP).

Example output file:  
`contrib/satellite/data/sims/example-cbr/simple/PacketTrace.log`

---

## Trace & Logging Hooks

| Trace Name | Event Description | Output Location |
|-------------|-------------------|-----------------|
| `PacketTrace` | Every send/receive event | `PacketTrace.log` |
| `RxDelay` | Delay of each packet | Derived from tags |
| `RxJitter` | Variation in delay (jitter) | Derived from tags |
| `Tx`, `Rx` | Transmit and receive events | Console or log sinks |

---

## Relationships with Other Components

| Component | Interaction |
|------------|--------------|
| `SatHelper` | Creates and connects `SatNetDevice` objects to nodes. |
| `SatPhy` | Attached via `SetPhy()`, handles actual transmission. |
| `SatMac` | Attached via `SetMac()`, manages channel access and scheduling. |
| `SatChannel` | Not stored directly; connected through PHY layer. |
| `SatLlc` | Buffers and organizes packets before MAC. |

---

## Summary

- `SatNetDevice` acts as the **bridge** between ns-3’s generic network stack and the satellite MAC/PHY subsystems.  
- It provides hooks for **tracing**, **packet classification**, and **delay/jitter measurement**.  
- Most “packet journey” logic — from sending a TCP segment to receiving it back — passes through here.

---

# PHY and MAC Layer Analysis

## Objective

This document summarizest how `Transmi()` and `Receive()` functions are handled in the **PHY** and **MAC** layers of the ns-3 satellite module, and how signals are propagated through the **channel** and logged in simulation outputs.

---

## File Overview

| File               | Role                                                                                                                       |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| `satellite-mac.cc` | Implements medium access control (MAC) — responsible for framing, scheduling, and coordinating transmissions.              |
| `satellite-phy.cc` | Implements the physical layer (PHY) — responsible for signal transmission, propagation through the channel, and reception. |

---

## Data Flow Overview

The packet transmission path from sender to receiver can be summarized as:

```
App → IP → SatNetDevice → SatLlc → SatMac → SatPhy → SatChannel
                                                  ↓
                                            SatPhy (Receiver)
                                                  ↓
                                           SatMac → SatLlc → App
```

Each layer plays a distinct role:

* **MAC**: Decides when and how to send (framing, timing, addressing)
* **PHY**: Actually sends the packet into the channel (modulates, applies TX power)
* **Channel**: Simulates delay, path loss, and interference between sender and receiver
* **Receiver PHY**: Receives and decodes incoming packets
* **Receiver MAC**: Validates and passes packets up to higher layers

---

## MAC Layer — `Transmit()` and `Receive()`

### a. Transmit()

* The **MAC layer** prepares packets for transmission.
* It may attach control information (headers, frame type, etc.).
* Once ready, it calls the PHY’s `Transmit()` function:

  ```cpp
  m_phy->Transmit(packet);
  ```
* This effectively tells the PHY: “It’s your turn to send this packet.”

**Responsibilities:**

* Schedules the packet (access control)
* Handles retransmission or ARQ logic
* Informs PHY of timing, power, and modulation parameters

### b. Receive()

* When the PHY layer completes a successful reception, it calls:

  ```cpp
  m_mac->Receive(packet);
  ```
* MAC then performs error checking, extracts headers, and determines if the packet should be delivered upward (LLC or higher layers).

**Responsibilities:**

* Check destination address
* Filter invalid/corrupted frames
* Pass valid packets to LLC or NetDevice

---

## PHY Layer — `Transmit()` and `Receive()`

### a. Transmit()

* Called by the MAC when the packet is ready to be physically sent.
* The PHY triggers the **channel** to propagate the signal:

  ```cpp
  m_channel->Transmit(this, packet, txPowerDb);
  ```
* It also logs the transmission event using `m_packetTrace`.

**Responsibilities:**

* Apply modulation and coding scheme
* Set transmission power and antenna parameters
* Pass packet to `SatChannel`
* Trace event: `SND PHY`

### b. Receive()

* Triggered when the **channel** delivers a packet to this PHY.
* The PHY calculates SNR, checks errors, and if valid, forwards to MAC:

  ```cpp
  m_mac->Receive(packet);
  ```

**Responsibilities:**

* Apply propagation delay and noise models
* Handle packet decoding and error detection
* Log the event: `RCV PHY`

---

## Channel Interaction — `SatChannel`

* Acts as the bridge between sender and receiver PHYs.
* When `m_channel->Transmit()` is called:

  * The channel computes **propagation delay** and **path loss**.
  * It then schedules an event to deliver the packet to the destination PHY after the delay.
* The receiving PHY triggers its `Receive()` callback upon arrival.

**Key Functions:**

```cpp
void SatChannel::Transmit(Ptr<SatPhy> sender, Ptr<Packet> packet, double txPowerDb)
{
    // Calculate delay and attenuation
    // Deliver to receiver PHY after propagation delay
    receiver->Receive(packet);
}
```

---

## Logging and Trace Output

All significant events are recorded using **trace callbacks** like `m_packetTrace` and stored in `.log` files.

Example: `contrib/satellite/data/sims/example-cbr/simple/PacketTrace.log`

```
0.0500152 SND GW PHY FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.183623 RCV SAT PHY FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
```

These indicate a packet being **sent (SND)** from the **Gateway PHY** and later **received (RCV)** by the **Satellite PHY**.

**Trace format summary:**

| Field     | Description                             |
| --------- | --------------------------------------- |
| Time      | Simulation timestamp (s)                |
| Event     | `SND`, `RCV`, `ENQ`, etc.               |
| Node      | Node type (UT, SAT, GW)                 |
| Log Level | ND, LLC, MAC, PHY, CH                   |
| Direction | FWD (forward link) or RTN (return link) |

---

## Summary of Interactions

| Layer    | Function                     | Called By    | Passes To    | Log Event |
| -------- | ---------------------------- | ------------ | ------------ | --------- |
| MAC      | Prepares frame, schedules TX | Higher layer | PHY          | `SND MAC` |
| PHY      | Modulates, sends via channel | MAC          | Channel      | `SND PHY` |
| Channel  | Applies delay, attenuation   | PHY          | Receiver PHY | —         |
| PHY (RX) | Decodes, validates           | Channel      | MAC          | `RCV PHY` |
| MAC (RX) | Forwards to LLC              | PHY          | NetDevice    | `RCV MAC` |

---

## Key Insights

* **MAC controls timing; PHY executes the transmission.**
* **Channel adds realism** (delay, noise, fading).
* **Tracing ensures reproducibility** — every packet’s journey is logged.
* **Each log line = one step in the signal lifecycle.**

---

# Trace Logging and Statistics
The ns-3 tracing system is a powerful, callback-based mechanism for collecting data from simulations. Here's a breakdown of how it applies to the satellite module.

## How Logs are Generated
Logs like `PacketTrace.log` are not generated automatically. They are the result of connecting a trace sink to a trace source.
- Trace Source: An event point within a simulation object. For example, the PHY layer object has a trace source named `PhyTxBegin` that fires whenever a packet transmission starts. Other sources include `PhyRxEnd`, `MacTx`, etc.
- Trace Sink: A function or method that we write to handle the data from the trace source. In the `sat-cbr-example` [Jump To Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/examples_sims_run.md#31-inspecting-the-log-header-and-content), the sink is a function that formats the information (time, node ID, event type, packet details) into the string format you see in `PacketTrace.log` and writes it to a file.
- Connection: The simulation script must explicitly connect a sink to a source. You can connect many sinks to one source or one sink to many sources.
`SimInfo.log` is generated differently. It's typically a straightforward file I/O operation where the simulation script writes simulation parameters (e.g., number of nodes, simulation time) to a text file at the beginning of the run.

## Functions Responsible for Writing Trace Data
The actual writing is done by the trace sink. This isn't a single named function in the ns-3 core but rather a custom function defined within the context of the simulation.<br>
Inside the sink function, standard C++ `std::ofstream` objects are used to write to the log file. For example, a simplified sink for a PHY event might look like this:
  ```
void PhyTxTrace(std::string context, Ptr<const Packet> packet)
{
  // m_packetTraceFile is an std::ofstream object opened earlier
  m_packetTraceFile << Simulator::Now().GetSeconds() << " "
                    << "SND " << context << " "
                    // ... and so on, to format the line
                    << std::endl;
}
  ```

## Locating the Trace Connection Code
The logic for connecting these sources and sinks is often found in helper classes or directly within the main simulation script. For the satellite module:
- `contrib/satellite/helper/satellite-helper.cc`: This is a primary candidate. Helper classes are designed to simplify configuration. Look for methods like `EnablePcap` or `EnableAscii` which are standard ns-3 patterns for setting up tracing. Inside these methods, you will find calls to `TraceConnect` or `TraceConnectWithoutContext`.
- `contrib/satellite/examples/*.cc`: The example script itself is the most direct place to find the setup. The `main` function of `sat-cbr-example.cc` will contain the code that creates file streams `(std::ofstream)` for the logs and then calls the helper methods or `TraceConnect` functions to link the simulation events to these files.
- `contrib/satellite/model/satellite-log.cc`: This file is less likely to contain the `TraceConnect` calls but may define logging-specific components, custom formatters, or enums (like `SND`, `RCV`) that are used by the trace sinks to create the log file content.
