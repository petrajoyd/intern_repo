# Study Note 3: NS-3 Satellite Module
The **`contrib/satellite`** module in `ns-3.43` provides tools and models to simulate **satellite communication systems**, including LEO/GEO orbits, beam scheduling, link adaptation, LoRaWAN integration, and physical/MAC-level channel behavior.
This module integrates both **DVB-S2X** and **LoRa-based** satellite links for flexible end-to-end testing.

# Directory Structure And Helper-Channel Interaction
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
      └── ...
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

### **`model/` Directory**
This is the core of the module, containing the C++ classes that simulate the actual networking components.
- `satellite-net-device.cc / .h`: This is arguably the most important file here. It defines the `SatNetDevice` class, which represents the Layer 2 network interface card (NIC) for a satellite or ground station. It connects the node's IP stack to the satellite's physical layer (SatPhy).
- `satellite-phy.cc / .h`: Defines the `SatPhy` class. This represents the physical layer (Layer 1). It's responsible for the "transmission" and "reception" of signals, handling things like transmission power and modulation schemes. It doesn't know about packets, just signal energy.
- `satellite-mac.cc / .h`: Defines the `SatMac` class. This is the Medium Access Control sublayer that sits between the `SatNetDevice` and `SatPhy`. It decides when to transmit to avoid collisions on the shared satellite channel.
- `satellite-channel.cc / .h`: Implements the `SatelliteChannel`. Think of this as the shared "air" or "space" through which all the satellites and ground stations communicate. When one SatPhy transmits, it's the channel's job to deliver that signal to all other devices listening on the channel.
- `satellite-propagation-delay-model.cc / .h`: This is crucial for realism. It calculates the time it takes for a signal to travel between two points in space, which is significant for satellites. It uses the physical positions of the nodes to do this.
- `satellite-fading-models.cc / .h`: (Or similar names) These files model how the signal strength can be affected by atmospheric conditions like rain, clouds, etc.

### **`helper/` Directory**
Helpers are designed to make the user's life easier. They wrap the complex model components into a simpler API for writing simulation scripts.
- `satellite-helper.cc / .h`: This is the main entry point for creating a satellite network. The SatelliteHelper class provides high-level functions like .Install() which will automatically create and configure the SatNetDevice, SatPhy, SatMac, and connect them to the SatelliteChannel for a given set of nodes. You'll use this in almost every satellite simulation script.

### **`examples/` Directory**
These are complete, runnable simulation scripts that show how to use the helper and models. They are the best place to start learning.
- `sat-tutorial-example.cc`: Your primary learning tool. It's a basic script that sets up a simple satellite scenario. By reading and modifying this file, you can learn how to use the SatelliteHelper to build your own simulations.
- `sat-iot-example.cc`: A more complex example that demonstrates a specific use case (IoT). It shows how to integrate the satellite module with other parts of NS-3, like application layers and mobility models.

### **`data/` Directory**
This directory isn't for code, but for data files that your simulations might need.
- TLE (Two-Line Element) files (`.tle`): This is the most common type of file you'll find here. TLE is a standard format for distributing orbital data for satellites. The NS-3 satellite position models can read these files to simulate realistic satellite movement based on actual orbital paths. For example, you might find a file like `leo-walker-star.tle` which defines an entire constellation.

## Key Components

| Component | Description |
|------------|--------------|
| **Physical Layer (PHY)** | Models path loss, fading (Rayleigh, Loo), and SINR-based reception. |
| **MAC Layer** | Implements scheduling (e.g., DAMA, CRDSA, time slicing) and frame management. |
| **NCC (Network Control Center)** | Handles capacity requests and beam management. |
| **Mobility Models** | Includes `satellite-sgp4-mobility-model` for orbit prediction. |
| **LoRa Integration** | Supports satellite–LoRa hybrid constellations. |
| **Logging System** | Creates trace files such as `PacketTrace.log` and `SimInfo.log` under `data/sims/`. |

## Example Simulation Workflow
### **Simple Simulation**
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

### Simulation With: Configuration (Channel And Scenario), Nodes Creation, and many more
#### Phase 1: Boilerplate and Configuration
This is the initial setup phase.
- Includes: The file starts with a series of `#include` statements. These import all the necessary libraries and modules from NS-3 that the script needs to function. The most important one for us is `#include "ns3/satellite-module.h"`.
- Logging: You'll see lines like `NS_LOG_COMPONENT_ENABLE ("UdpEchoClientApplication", LOG_LEVEL_INFO);`. This is for debugging. It tells the simulator to print out log messages from specific components (in this case, the `UdpEchoClientApplication`) so you can see what's happening during the run.
- Command Line Parser: This allows you to change simulation variables from the command line when you run the script, without having to recompile the code. It's a very handy feature for running experiments.
```
CommandLine cmd;
cmd.Parse (argc, argv);
```

#### Phase 2: Creating the Network Nodes
This is where the "computers" of our simulation are created. In this context, nodes can be satellites, ground stations, or user terminals.
- Node Containers:
```
NodeContainer terminals;
terminals.Create(2);

NodeContainer gws;
gws.Create(2);

NodeContainer sats;
sats.Create(1);
```
A `NodeContainer` is a helper object that holds a group of nodes. The code above creates three groups:
- 2 "terminal" nodes (e.g., user phones or laptops).
- 2 "gateway" nodes (ground stations).
- 1 "satellite" node.
<br>At this point, they are just empty computer objects. They have no networking hardware or software installed yet.

##### Phase 3: Setting Up the Satellite Helper and Channel
This is the core of the satellite-specific configuration.
- Satellite Helper:
```
SatelliteHelper satHelper;
```
The `SatelliteHelper` is the main tool we use from the satellite module. It simplifies the process of creating and configuring the channel and network devices.

##### Phase 4: Installing Network Devices and the IP Stack
Now we give the nodes the hardware and software they need to communicate.
1. Install Satellite Devices:
```
NetDeviceContainer terminalDevs = satHelper.Install(terminals);
NetDeviceContainer gwDevs = satHelper.Install(gws);
NetDeviceContainer satDevs = satHelper.Install(sats);
```
This is a key step. The `satHelper.Install()` method takes a container of nodes and installs a `SatNetDevice` (the satellite network card) on each one. It also connects all these new devices to a single, shared `SatelliteChannel`. Now the nodes have the hardware to talk to each other.
2. Install the Internet Protocol (IP) Stack:
```
InternetStackHelper stack;
stack.Install (terminals);
stack.Install (gws);
stack.Install (sats);
```
The nodes have hardware, but they need the TCP/IP software to manage network connections. The `InternetStackHelper` installs this networking stack onto all the nodes we created.

#### Phase 5: Assigning IP Addresses
Just like in a real network, each device needs a unique IP address.
```
Ipv4AddressHelper address;
address.SetBase ("10.1.1.0", "255.255.255.0");
Ipv4InterfaceContainer terminalInterfaces = address.Assign (terminalDevs);
```
The `Ipv4AddressHelper` is used to assign IP addresses to the network devices. The code above configures a block of addresses (starting from 10.1.1.1) and assigns them one-by-one to the devices in the `terminalDevs` container. This process is repeated for the gateways and the satellite, each on a different subnet (e.g., 10.1.2.0, 10.1.3.0).

##### Phase 6: Running the Simulation
With everything set up, the final step is to start the simulation clock.
- Run and Clean Up:
```
Simulator::Run ();
Simulator::Destroy ();
```
`Simulator::Run()` starts the simulation. All the events that were scheduled (like applications starting and stopping) will now execute in order. Once it's finished, `Simulator::Destroy()` cleans up all the allocated resources.

#### Summary
In summary, the sat-tutorial-example.cc script follows a standard NS-3 pattern:
1. Create Nodes.
2. Use a Helper to install Devices and create a Channel.
3. Install the IP protocol stack.
4. Assign IP Addresses.
5. Install Applications to generate traffic.
6. Run the simulation. [Simulation Run](LINK)

## Satellite Module `satellite-helper.cc`, `satellite-channel-model.cc`
This document provides a structured overview of the `contrib/satellite` module in ns-3.43, focusing on how helpers connect PHY, MAC, and network layers. This section explores `satellite-helper.cc` and `satellite-channel-model.cc`, which form the glue logic between high-level scenario configuration and low-level model execution.

### 1. The Role of Helpers
#### **a. What Are Helpers?**
In ns-3, *helpers* simplify simulation setup by wrapping complex model configurations into reusable interfaces. They bridge the gap between user-defined scenarios (in `examples/`) and core models (in `model/`).<br>
The key helper in the satellite module is `SatHelper`, located at:
```
contrib/satellite/helper/satellite-helper.cc
```
It works alongside other components like `SimulationHelper` to orchestrate node creation, device installation, and PHY/MAC configuration.

### 2. `SatHelper` — The Core Builder
#### **Purpose**

`SatHelper` acts as the *central configuration and connection layer*. It abstracts away the complexity of linking User Terminals (UT), Gateways (GW), and Satellites (SAT) with their corresponding physical (PHY) and MAC layers.

#### **Key Responsibilities**

| Function                        | Description                                                                 |
| ------------------------------- | --------------------------------------------------------------------------- |
| `CreateSatScenario()`           | Builds a complete satellite topology (UTs, GWs, Satellites, NCC).           |
| `ConfigurePhyMac()`             | Instantiates and attaches PHY and MAC models to each node.                  |
| `InstallNetDevices()`           | Attaches `SatNetDevice` to nodes, enabling packet transmission.             |
| `ConfigurePropagationChannel()` | Links each PHY to a `SatChannel` and sets up delay/fading models.           |
| `SetDefaultParameters()`        | Loads parameters from XML or environment variables (`SatEnvVariables`).     |
| `ConnectTraces()`               | Binds events (like SND/RCV) to trace sinks for logging (`PacketTrace.log`). |

#### **Layer Connection Flow**

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

#### **Example Internal Flow (simplified)**

```cpp
Ptr<Node> ut = CreateObject<Node>();
Ptr<SatNetDevice> utDevice = helper->InstallUtDevice(ut);
ut->AddDevice(utDevice);
```
* The helper installs a UT device that includes PHY/MAC models and connects it to the appropriate channel.

### 3. `SatHelper` — The Core Builder
#### **Location**
```
contrib/satellite/model/satellite-channel.cc
contrib/satellite/model/satellite-propagation-delay-model.cc
```
#### **Purpose**

The `SatChannelModel` simulates signal propagation between nodes, including free-space loss, fading, and interference. It is dynamically configured by `SatHelper` to reflect the chosen scenario and environmental conditions.

#### **Key Components**

| File                                   | Role                                                                          |
| -------------------------------------- | ----------------------------------------------------------------------------- |
| `satellite-channel.cc`                 | Defines the communication path and packet flow between devices.               |
| `satellite-propagation-delay-model.cc` | Calculates delay based on satellite geometry (distance and orbital position). |
| `satellite-fading-models`              | Adds time-varying signal degradation (Rayleigh, Loo, or external traces).     |

#### **Propagation Flow**

```
GW PHY TX → SatChannelModel → SAT PHY RX
                   ↓
            DelayModel + FadingModel
```

Each PHY’s transmit function triggers a propagation event handled by the channel, which applies attenuation and delay before the packet reaches the receiving PHY.

#### 4. Integration Between Helper and Model Layers

1. **Helper Configures** — `SatHelper` creates nodes and attaches `SatNetDevice`.
2. **Channel Binding** — Each `SatNetDevice` connects to a shared `SatChannelModel` instance.
3. **Model Execution** — The channel applies propagation, and PHY/MAC handle transmission/reception.
4. **Tracing** — Events are forwarded to trace sinks (PacketTrace, SimInfo, etc.) for analysis.

#### **Simplified Call Stack**

```
SimulationHelper::RunSimulation()
  └── SatHelper::CreateSatScenario()
        ├── SatHelper::ConfigurePhyMac()
        ├── SatHelper::ConfigurePropagationChannel()
        └── SatChannel::Transmit()
```

---


# Device & PHY-MAC Exploration
## Analyze SatNetDevice
```
contrib/satellite/model/satellite-net-device.cc  
contrib/satellite/model/satellite-net-device.h
```
`SatNetDevice` is the **satellite-specific implementation** of ns-3’s `NetDevice` class.  
It represents the “network card” that sits between the ns-3 node (UT, GW, or SAT) and the underlying MAC/PHY/channel layers.

### Core Responsibilities

| Function | Description |
|-----------|-------------|
| `Send()` / `SendFrom()` | Handles outgoing packets, attaches statistics tags, and pushes data into the LLC queue. |
| `Receive()` | Handles incoming packets, computes delay/jitter, and delivers them up to the IP layer. |
| `SetPhy()` / `SetMac()` / `SetLlc()` | Connects physical, MAC, and logical link layers to the device. |
| `SetNodeInfo()` | Links this device to its node metadata (ID, type, MAC address). |
| `ToggleState()` | Enables or disables the MAC operation. |
| `SetReceiveCallback()` | Registers callback for upper-layer delivery. |
| `DoDispose()` | Cleans up attached components and releases pointers. |

### Key Data Members

| Variable | Type | Role |
|-----------|------|------|
| `m_phy` | `Ptr<SatPhy>` | Connects to the physical layer (handles modulation, radio link). |
| `m_mac` | `Ptr<SatMac>` | Medium Access Control (queues, retransmissions, scheduling). |
| `m_llc` | `Ptr<SatLlc>` | Logical Link Control (frame assembly and buffer management). |
| `m_node` | `Ptr<Node>` | The owning ns-3 node (UT, GW, or SAT). |
| `m_classifier` | `Ptr<SatPacketClassifier>` | Determines which flow each packet belongs to. |
| `m_receiveErrorModel` | `Ptr<ErrorModel>` | Models random losses or interference. |
| `m_packetTrace`, `m_txTrace`, `m_rxTrace` | `TracedCallback` | Used for packet-level event logging. |

### Installation Process via Helper
#### **a. Entry Point**
Device installation begins in `satellite-helper.cc` using methods such as:
```cpp
Ptr<SatNetDevice> dev = CreateObject<SatNetDevice>();
node->AddDevice(dev);
```
The helper then configures the internal components (PHY, MAC, LLC) for the new device.

#### **b. Configuration Steps**
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

### Data Flow

#### **Sending Flow**
```cpp
Send() → Add tags (address, timestamps)
       → Trace: PACKET_SENT
       → Classify flow (SatPacketClassifier)
       → Enque() packet into SatLlc
       → MAC → PHY → Channel
```

#### **Receiving Flow**
```cpp
Receive() → Log PACKET_RECV
          → Extract SatAddressTag, SatDevTimeTag
          → Compute delay & jitter
          → Trace results
          → Deliver packet to IP layer via callback
```

### Connection Flow Between Layers

#### **a. Internal Wiring**
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

#### **b. How Helpers Attach Components**
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

## PHY and MAC Layer Analysis
This document summarizest how `Transmi()` and `Receive()` functions are handled in the **PHY** and **MAC** layers of the ns-3 satellite module, and how signals are propagated through the **channel** and logged in simulation outputs.

### File Overview

| File               | Role                                                                                                                       |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| `satellite-mac.cc` | Implements medium access control (MAC) — responsible for framing, scheduling, and coordinating transmissions.              |
| `satellite-phy.cc` | Implements the physical layer (PHY) — responsible for signal transmission, propagation through the channel, and reception. |

### Data Flow Overview

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

### MAC Layer — `Transmit()` and `Receive()`

#### a. Transmit()

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

#### b. Receive()

* When the PHY layer completes a successful reception, it calls:

  ```cpp
  m_mac->Receive(packet);
  ```
* MAC then performs error checking, extracts headers, and determines if the packet should be delivered upward (LLC or higher layers).

**Responsibilities:**

* Check destination address
* Filter invalid/corrupted frames
* Pass valid packets to LLC or NetDevice

### PHY Layer — `Transmit()` and `Receive()`

#### a. Transmit()

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

#### b. Receive()

* Triggered when the **channel** delivers a packet to this PHY.
* The PHY calculates SNR, checks errors, and if valid, forwards to MAC:

  ```cpp
  m_mac->Receive(packet);
  ```

**Responsibilities:**

* Apply propagation delay and noise models
* Handle packet decoding and error detection
* Log the event: `RCV PHY`

### Channel Interaction — `SatChannel`

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

## Tracing and Statistics
The helper also enables traces for debugging and analysis.  
In `SatNetDevice`, these are defined as trace sources like `m_packetTrace`, `m_txTrace`, and `m_rxTrace`, which record key simulation events (SEND, RECV, DROP).

### How Logs are Generated
Logs like `PacketTrace.log` are not generated automatically. They are the result of connecting a trace sink to a trace source.
- Trace Source: An event point within a simulation object. For example, the PHY layer object has a trace source named `PhyTxBegin` that fires whenever a packet transmission starts. Other sources include `PhyRxEnd`, `MacTx`, etc.
- Trace Sink: A function or method that we write to handle the data from the trace source. In the `sat-cbr-example` [Jump To Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/examples_sims_run.md#31-inspecting-the-log-header-and-content), the sink is a function that formats the information (time, node ID, event type, packet details) into the string format you see in `PacketTrace.log` and writes it to a file.
- Connection: The simulation script must explicitly connect a sink to a source. You can connect many sinks to one source or one sink to many sources.
`SimInfo.log` is generated differently. It's typically a straightforward file I/O operation where the simulation script writes simulation parameters (e.g., number of nodes, simulation time) to a text file at the beginning of the run.

### Functions Responsible for Writing Trace Data
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

### Locating the Trace Connection Code
The logic for connecting these sources and sinks is often found in helper classes or directly within the main simulation script. For the satellite module:
- `contrib/satellite/helper/satellite-helper.cc`: This is a primary candidate. Helper classes are designed to simplify configuration. Look for methods like `EnablePcap` or `EnableAscii` which are standard ns-3 patterns for setting up tracing. Inside these methods, you will find calls to `TraceConnect` or `TraceConnectWithoutContext`.
- `contrib/satellite/examples/*.cc`: The example script itself is the most direct place to find the setup. The `main` function of `sat-cbr-example.cc` will contain the code that creates file streams `(std::ofstream)` for the logs and then calls the helper methods or `TraceConnect` functions to link the simulation events to these files.
- `contrib/satellite/model/satellite-log.cc`: This file is less likely to contain the `TraceConnect` calls but may define logging-specific components, custom formatters, or enums (like `SND`, `RCV`) that are used by the trace sinks to create the log file content.


### Trace & Logging Hooks

| Trace Name | Event Description | Output Location |
|-------------|-------------------|-----------------|
| `PacketTrace` | Every send/receive event | `PacketTrace.log` |
| `RxDelay` | Delay of each packet | Derived from tags |
| `RxJitter` | Variation in delay (jitter) | Derived from tags |
| `Tx`, `Rx` | Transmit and receive events | Console or log sinks |


Example output file:  
`contrib/satellite/data/sims/example-cbr/simple/PacketTrace.log`

Each SatNetDevice is initialized with dedicated PHY/MAC layers and linked to a shared channel depending on the communication direction (forward/return).

### Relationships with Other Components

| Component | Interaction |
|------------|--------------|
| `SatHelper` | Creates and connects `SatNetDevice` objects to nodes. |
| `SatPhy` | Attached via `SetPhy()`, handles actual transmission. |
| `SatMac` | Attached via `SetMac()`, manages channel access and scheduling. |
| `SatChannel` | Not stored directly; connected through PHY layer. |
| `SatLlc` | Buffers and organizes packets before MAC. |

### Logging and Trace Output

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

### Summary of Interactions

| Layer    | Function                     | Called By    | Passes To    | Log Event |
| -------- | ---------------------------- | ------------ | ------------ | --------- |
| MAC      | Prepares frame, schedules TX | Higher layer | PHY          | `SND MAC` |
| PHY      | Modulates, sends via channel | MAC          | Channel      | `SND PHY` |
| Channel  | Applies delay, attenuation   | PHY          | Receiver PHY | —         |
| PHY (RX) | Decodes, validates           | Channel      | MAC          | `RCV PHY` |
| MAC (RX) | Forwards to LLC              | PHY          | NetDevice    | `RCV MAC` |

# Tracing and Logging System
## 1. NS-3 Satellite Trace System 
In NS-3, the **trace system** allows you to **record simulation events** — such as packet transmissions, receptions, delays, or queue changes — for later analysis.  
In the `satellite` module, this system helps log **satellite-specific events** like:
- Packets entering or leaving a node (uplink/downlink)
- Delays in satellite links
- Throughput and error metrics

The two main files handling this are:
- `satellite-log.cc`: general-purpose logging support
- `satellite-packet-trace.cc`: detailed packet tracing

## `satellite-log.cc`
### Purpose
Handles **generic logging setup** for the satellite module.  
It sets up the trace sources that can be connected to output files or analysis tools.

### Key Concept: Trace Registration with `AddTraceSource`
NS-3 uses the **TypeId** system to expose trace sources.  
Inside `satellite-log.cc`, you’ll typically see something like:

```cpp
TypeId tid = TypeId("ns3::SatelliteLog")
  .SetParent<Object>()
  .AddTraceSource("PacketTrace",
                  "Trace source for packet events",
                  MakeTraceSourceAccessor(&SatelliteLog::m_packetTrace),
                  "ns3::Packet::TracedCallback");
```
**Explanation:**
- `AddTraceSource` registers a trace variable with the NS-3 tracing system.
- `"PacketTrace"` is the **trace name** that can be connected to.
- `m_packetTrace` is a **member variable** — usually a `TracedCallback` that stores functions to call when an event happens.
- `"ns3::Packet::TracedCallback"` defines the **callback signature** (what parameters are passed).

So, in summary:
> `AddTraceSource` = "Hey NS-3, this class has a trace point named `PacketTrace` — call it whenever something happens."

## `satellite-packet-trace.cc`
### Purpose
Implements **actual tracing logic** — connecting simulation events (like sending/receiving packets) to file outputs.

Inside this file, you’ll find:
- Declarations of trace variables (like `m_packetTrace`)
- Connections between simulation objects and output streams
- Formatting functions for how data is written to `.log` or `.csv` files

### How the Trace Works
Usually, the tracing flow looks like this:

1. **Trace Source Declaration**
   ```cpp
   TracedCallback<Ptr<const Packet>, const Address &> m_packetTrace;
   ```
   → This declares a callback that will be fired whenever a packet event occurs.

2. **Event Trigger**
   When a packet is transmitted or received:
   ```cpp
   m_packetTrace(packet, address);
   ```
   → This "fires" the trace, triggering all connected listeners (like log writers).

3. **Connecting to Output**
   Somewhere in the setup phase (often in `DoInitialize()` or helper functions):
   ```cpp
   Config::ConnectWithoutContext("/NodeList/*/ApplicationList/*/$ns3::SatelliteApp/PacketTrace",
                                 MakeCallback(&SatellitePacketTrace::PacketTraceSink, this));
   ```

   - This connects **all SatelliteApp packet events** to your callback function `PacketTraceSink`.

4. **Writing to File**
   In `PacketTraceSink`, you’ll see something like:
   ```cpp
   void SatellitePacketTrace::PacketTraceSink(Ptr<const Packet> packet, const Address &address)
   {
       std::ofstream out(m_outputFileName, std::ios::app);
       out << Simulator::Now().GetSeconds() << "," << packet->GetSize() << "," << address << std::endl;
   }
   ```
   → Each time a packet event happens, it writes a new line with:
   - Timestamp  
   - Packet size  
   - Source/Destination info

## `m_packetTrace` and Related Callbacks
### What It Is
m_packetTrace` is a **TracedCallback**, a core NS-3 mechanism for event notification.

### How It Connects
- **Declared** in the C++ class (e.g., `SatellitePacketTrace`)
- **Registered** via `AddTraceSource`
- **Connected** to an external **trace sink** (file writer, plotter, etc.)
- **Activated** during simulation (when an event fires)

### The Flow in One Line:
```
Simulation Event → m_packetTrace() fires → Callback function runs → Output logged to file
```

## Typical Example (Simplified)
Let’s put it together in a very simplified pseudocode style:

```cpp
// satellite-packet-trace.h
class SatellitePacketTrace : public Object {
private:
  TracedCallback<Ptr<const Packet>> m_packetTrace;

public:
  static TypeId GetTypeId();
  void NotifyPacket(Ptr<const Packet> packet);
};

// satellite-packet-trace.cc
TypeId SatellitePacketTrace::GetTypeId() {
  static TypeId tid = TypeId("ns3::SatellitePacketTrace")
    .SetParent<Object>()
    .AddTraceSource("PacketTrace", "Tracing packet events",
                    MakeTraceSourceAccessor(&SatellitePacketTrace::m_packetTrace),
                    "ns3::Packet::TracedCallback");
  return tid;
}

void SatellitePacketTrace::NotifyPacket(Ptr<const Packet> packet) {
  m_packetTrace(packet); // trigger the trace
}
```

And then, another component (like a helper or logger) can **connect** to it:
```cpp
Config::Connect("/NodeList/1/ApplicationList/0/$ns3::SatellitePacketTrace/PacketTrace",
                MakeCallback(&WriteToFile));
```

## Summary
| Component | File | Function | Key Concept |
|------------|------|-----------|--------------|
| `AddTraceSource` | `satellite-log.cc` | Registers a trace variable | Makes it visible to NS-3 trace system |
| `m_packetTrace` | Both | Holds callback chain | Fires when event occurs |
| `PacketTraceSink()` | `satellite-packet-trace.cc` | Callback function | Writes to file |
| `Config::Connect` | Main script/helper | Connects source to sink | Links trace to output |

## 2. NS-3 Event Trigger Analysis
This note explains how events such as `SND`, `RCV`, `ENQ`, and `DRP` are emitted in the NS-3 Satellite Contrib Module and how they link across network layers (NetDevice, MAC, PHY).

## Tracking Packet Event Emission (SND, RCV, ENQ, DRP)
In the NS-3 satellite module, these events represent specific moments in the packet’s life cycle:

| Event | Meaning | Typical Source File | Typical Function |
|--------|----------|--------------------|------------------|
| **SND (Send)** | Triggered when a packet is passed for transmission (e.g., from upper layer to MAC or PHY). | `satellite-log.cc`, `satellite-net-device.cc` | `SatNetDevice::Send()` |
| **RCV (Receive)** | Triggered when a packet is received by a device or layer. | `satellite-packet-trace.cc`, `satellite-net-device.cc` | `SatNetDevice::Receive()` |
| **ENQ (Enqueue)** | Marks packet insertion into a transmission queue (often at MAC layer). | `satellite-mac.cc`, `satellite-queue.cc` | `Enqueue()` |
| **DRP (Drop)** | Fired when a packet is dropped (due to queue overflow, link errors, etc.). | `satellite-mac.cc`, `satellite-phy.cc` | `Drop()` |

Each of these events is traced using `AddTraceSource()` to allow logging and later analysis via the tracing system.

## How `SatNetDevice::Send()` and `Receive()` Link MAC and PHY Layers
The **SatNetDevice** acts as a bridge between higher layers (LLC / IP) and lower layers (MAC / PHY).

### a. `SatNetDevice::Send()`
- Invoked when upper layers request transmission.
- Performs checks (link status, queue availability).
- Creates metadata (`Ptr<Packet>`, headers, tags).
- Passes packet down to the **MAC layer** via `m_mac->Enqueue(packet)`.
- Triggers `SND` and possibly `ENQ` traces.

### b. `SatNetDevice::Receive()`
- Invoked when the PHY layer delivers a received frame.
- Calls `m_mac->Receive()` to handle link-level framing and error detection.
- MAC decodes and sends data upward to NetDevice, triggering `RCV` trace.
- The packet is finally delivered to upper layers (LLC/Network stack).

**Flow Summary:**
```
Application
   ↓
LLC / Network Layer
   ↓
SatNetDevice::Send() → MAC::Enqueue() → PHY::Transmit()
                                    ↓
                           Channel Propagation
                                    ↓
                          PHY::Receive() → MAC::Process()
                                    ↓
                          SatNetDevice::Receive() → Upper Layer
```

## Log Level Mapping (ND, LLC, MAC, PHY, CH)
Each level corresponds to a layer in the satellite communication stack:

| Log Level | Full Form | Represents | Typical Log Prefix | Description |
|------------|------------|-------------|---------------------|--------------|
| **ND** | NetDevice | Device-level actions (`Send()`, `Receive()`) | `[ND]` | Entry and exit points for transmission events. |
| **LLC** | Logical Link Control | Manages interface between MAC and upper layers. | `[LLC]` | Responsible for frame control and addressing. |
| **MAC** | Medium Access Control | Handles access scheduling, queueing, and retransmission. | `[MAC]` | Triggers ENQ and DRP events, manages traffic. |
| **PHY** | Physical Layer | Transmits/receives bits over the channel. | `[PHY]` | Triggers SND and RCV when interacting with hardware. |
| **CH** | Channel | Simulates physical propagation and signal loss. | `[CH]` | Models delay, interference, and attenuation. |

**Event Mapping Example:**
```
[ND] SatNetDevice::Send() → [MAC] Enqueue() → [PHY] Transmit() → [CH] Propagation
[CH] → [PHY] Receive() → [MAC] Process() → [ND] DeliverToUpperLayer()
```

## Summary
- `SND`, `RCV`, `ENQ`, `DRP` represent lifecycle checkpoints of a packet.
- `SatNetDevice` ties together MAC and PHY layers through well-defined callbacks.
- Logging levels (ND, LLC, MAC, PHY, CH) help isolate debugging to specific layers.
- Trace sources (`AddTraceSource`) allow these events to be recorded and analyzed.

## 3. NS-3 Practical Test & Rerun
This test helps you **observe how trace verbosity affects the simulation output**. You’ll modify trace settings, run a standard example (like `sat-cbr-example`), and compare the logs.

### Modify Trace Verbosity (Disable PHY Logs)
#### Where to Edit
In the satellite module, verbosity is usually managed via `NS_LOG_COMPONENT_DEFINE` and `NS_LOG_INFO` statements within files like:
- `satellite-phy.cc`
- `satellite-mac.cc`
- `satellite-net-device.cc`
and/or via configuration parameters in the example script (`sat-cbr-example.cc`).

### Option 1 — Disable PHY Traces in Code
Open:
```
ns-3.43/contrib/satellite/model/satellite-phy.cc
```

Comment out or reduce logging for PHY:
```cpp
// NS_LOG_COMPONENT_DEFINE("SatPhy");  // comment this line if you want to disable PHY completely
```

Or wrap trace callbacks under a condition:
```cpp
if (m_enablePhyTrace) {
    m_phyTxTrace(packet);
}
```

Then, in your example script (`sat-cbr-example.cc`), set:
```cpp
// Disable PHY trace
Config::SetDefault("ns3::SatHelper::EnablePhyTrace", BooleanValue(false));
```

### Option 2 — Disable PHY Traces Using NS_LOG Environment Variable
Without touching the code, you can disable PHY logs by changing the environment variable before running:
```bash
export NS_LOG="*=level_info|prefix_func|prefix_time"
export NS_LOG="SatNetDevice:info|SatMac:info"   # excludes PHY logs
```
*This method is cleaner for testing since you don’t have to recompile.*

## Rerun the Example
Now run one of these test cases:
```bash
./ns3 run "sat-cbr-example"
```
or  
```bash
./ns3 run "example-tutorial"
```

Check if you already have an output directory (e.g., `traces/`, `results/`, or the root folder where `PacketTrace.log` is saved).

If `PacketTrace.log` doesn’t exist yet, it’s generated when `satellite-packet-trace.cc` is active — meaning trace hooks are working properly.

## Compare Before & After Logs
Now, open or diff your results:

### Before (Full Verbosity)
Example of full log:
```
[ND] SND pkt=123 size=1024
[MAC] ENQ pkt=123 queue=0
[PHY] TX pkt=123 power=32dBm
[CH] PROP pkt=123 delay=240ms
[PHY] RCV pkt=123 snr=12.3
[MAC] RCV pkt=123
[ND] DELIVER pkt=123
```

### After (PHY Disabled)
You’ll notice entries like:
```
[ND] SND pkt=123 size=1024
[MAC] ENQ pkt=123 queue=0
[CH] PROP pkt=123 delay=240ms
[MAC] RCV pkt=123
[ND] DELIVER pkt=123
```
No `[PHY]` entries should appear — confirming the trace was suppressed.

## Analyze Changes
| Aspect | Before | After (PHY Disabled) | Interpretation |
|--------|---------|---------------------|----------------|
| Log file size | Larger | Smaller | Fewer lines = reduced verbosity |
| PHY event entries | Present | Missing | Trace suppression successful |
| Packet count | Same | Same | Core logic unaffected |
| Timing | Same (minor variations possible) | Same | Only tracing changed, not simulation behavior |

## Optional: Restore or Fine-Tune Verbosity
After verification, re-enable the PHY trace to restore full logging:

```bash
Config::SetDefault("ns3::SatHelper::EnablePhyTrace", BooleanValue(true));
```

Or increase verbosity globally:
```bash
export NS_LOG="*=level_all|prefix_time|prefix_node"
```

## Summary
| Step | Action | Purpose |
|------|--------|----------|
| 1 | Disable PHY trace (code or env variable) | Limit verbosity |
| 2 | Rerun example | Generate fresh `PacketTrace.log` |
| 3 | Compare before/after logs | Verify trace filtering |
| 4 | Analyze file differences | Confirm only PHY logs removed |
| 5 | Restore trace | Return to full debugging mode |

# Channel & Propagation Mastery
### ns-3 Satellite Channel
This document breaks down how the ns-3 satellite channel model works by examining `satellite-channel.cc` and `satellite-propagation-delay-model.cc`. The primary goal is to understand how a transmitted signal is received by the channel and delivered to the destination's physical layer (PHY).

## The Role of the Channel
In ns-3, a **Channel** object represents the physical medium connecting network devices (`NetDevice`). Its main job is to model what happens to a signal as it travels from a transmitter to a receiver. For a satellite, the most critical factor is the **propagation delay**—the time it takes for the signal to travel through space.

The two files work together to accomplish this:
-   `satellite-channel.cc`: Acts as the "manager." It takes a packet from a sender and is responsible for delivering it to all intended receivers.
-   `satellite-propagation-delay-model.cc`: Acts as the "calculator." It provides the channel with the exact time delay for a signal traveling between any two points.

## 1. `contrib/satellite/model/satellite-channel.cc`
This file implements the `SatelliteChannel` class. Think of this class as the empty space through which the radio waves travel.

### How It Receives and Delivers a Signal
The key method is **`SatelliteChannel::Send()`**. This function is called whenever a `SatelliteNetDevice` (the network card of a satellite, terminal, or gateway) transmits a packet.

Here's the step-by-step process inside the `Send` function:

1.  **Packet Reception**: The function is called with the packet, a pointer to the sending device, and a transmission delay.

    ```cpp
    // A simplified representation of the function signature
    void SatelliteChannel::Send (Ptr<Packet> packet, Ptr<NetDevice> sender, Time txDelay)
    ```

2.  **Find All Potential Receivers**: The channel iterates through its list of all attached `NetDevice` objects to deliver the packet to everyone *except* the original sender.

3.  **Calculate Propagation Delay for Each Receiver**: For each receiver, the channel asks the propagation delay model how long the packet will take to reach it.

    ```cpp
    // Inside the loop for each receiver...
    Time propDelay = m_propagationDelay->GetDelay (sender->GetNode ()->GetObject<MobilityModel> (),
                                                   receiverDevice->GetNode ()->GetObject<MobilityModel> ());
    ```
    
    It passes the mobility models (which track physical location) of the sender and receiver to the `GetDelay()` method of the `SatellitePropagationDelayModel`.

4.  **Schedule the Arrival Event**: The channel tells the ns-3 scheduler to create a future event for the packet's arrival at the receiver's physical layer (PHY).

    ```cpp
    // Total delay is the time to transmit + time to travel
    Time totalDelay = txDelay + propDelay;
    
    // Schedule the StartRx function to be called on the receiver's PHY layer after totalDelay
    Simulator::Schedule (totalDelay, &SatellitePhy::StartRx, receiverDevice->GetPhy (), packet->Copy ());
    ```
    
    The **`Simulator::Schedule()`** function is the heart of ns-3. It says: "In `totalDelay` seconds from now, execute the `StartRx` function on the receiver's PHY object."

**In summary, `SatelliteChannel` acts as a forwarding agent. It takes a packet, asks the delay model "how long will this take?", and then tells the simulator to deliver the packet to the destination's PHY layer after that specific time has passed.**

## 1. `contrib/satellite/model/satellite-propagation-delay-model.cc`
This file implements the `SatellitePropagationDelayModel` class. Its one and only job is to calculate the time it takes for a signal to travel between two points.

### How It Calculates Delay
The core of this file is the **`SatellitePropagationDelayModel::GetDelay()`** method.

1.  **Get Positions**: The function receives the mobility models of the two nodes (`a` and `b`) and gets their current 3D Cartesian coordinates (x, y, z).

    ```cpp
    Vector posA = a->GetPosition ();
    Vector posB = b->GetPosition ();
    ```

2.  **Calculate Euclidean Distance**: It calculates the straight-line distance between the two points using the standard distance formula.
    
    $$
    d = \sqrt{(x_a - x_b)^2 + (y_a - y_b)^2 + (z_a - z_b)^2}
    $$

3.  **Calculate Delay**: The propagation delay is the distance divided by the speed of light (`SPEED_OF_LIGHT`).
    
$$
\text{delay} = \frac{\text{distance}}{\text{speed of light}}
$$
    
    ```cpp
    double distance = CalculateDistance (posA, posB);
    double delayInSeconds = distance / SPEED_OF_LIGHT;
    return Seconds (delayInSeconds);
    ```

4.  **Return the Delay**: The function returns the calculated delay as an ns-3 `Time` object.

## The Complete Workflow 

1.  A **SatelliteNetDevice** transmits a `Packet`, calling **`SatelliteChannel::Send()`**.
2.  The **`SatelliteChannel`** loops through all other connected devices.
3.  For each receiver, it calls **`SatellitePropagationDelayModel::GetDelay()`**.
4.  The **`SatellitePropagationDelayModel`** calculates the distance, divides by the speed of light, and returns the delay time.
5.  The **`SatelliteChannel`** receives this delay and uses **`Simulator::Schedule()`** to create a future arrival event.
6.  When the simulation clock reaches that future time, the scheduler calls **`SatellitePhy::StartRx()`** on the receiving device's PHY layer, delivering the packet.

This design perfectly separates responsibilities: the **Channel** manages delivery, and the **PropagationDelayModel** performs the physics-based calculation.

<img width="1744" height="141" alt="satellite_workflow_pretty" src="https://github.com/user-attachments/assets/b3c6b5a4-66ed-4128-8263-53e1cd1d9eb7" />

---

## ns-3 Satellite Propagation Delay & Fading Models
This document analyzes how ns-3 models the crucial aspects of propagation delay and signal fading in satellite channels, focusing on the underlying geometry and the modular, pluggable architecture for different fading models. 

### 1. Propagation Delay Computed from Satellite Geometry
The delay calculation is direct, accurate, and based purely on the physics of wave propagation. It's handled by the `SatellitePropagationDelayModel`. The key function is **`GetDelay(Ptr<MobilityModel> a, Ptr<MobilityModel> b)`** located in `contrib/satellite/model/satellite-propagation-delay-model.cc`.

#### How it Works
1.  **Get Real-Time Positions**: The function retrieves the current 3D Cartesian coordinates (x, y, z) of the sender and receiver from their respective mobility models. The ns-3 mobility framework is responsible for continuously updating the satellite's orbital position.

2.  **Calculate Straight-Line Distance**: It computes the direct, straight-line (Euclidean) distance between these two points.
```math
d = \sqrt{(x_a - x_b)^2 + (y_a - y_b)^2 + (z_a - z_b)^2}
```

4.  **Divide by the Speed of Light**: The propagation delay is calculated using the fundamental physics formula, where $c$ is the speed of light.
```math
\text{Delay (seconds)} = \frac{\text{Distance (meters)}}{c \text{ (meters/second)}}
```

**The key takeaway** is that the delay is **dynamic**. As the satellite moves, the distance to a ground terminal changes, and this model accurately recalculates the delay throughout the simulation, which is essential for realistic analysis.

## 2. Fading Models and Their Pluggable Architecture
While delay determines *when* a signal arrives, **fading** determines its **strength**. Fading models simulate signal degradation from atmospheric effects (rain), shadowing (buildings), and multipath reflections.</br>
ns-3 uses a highly modular, **pluggable architecture**. The `SatelliteChannel` itself has no fading logic. Instead, it holds a pointer to a generic `SatelliteFadingModel` object. This allows you to easily switch between different fading models.</br>
The base class in `satellite-fading-model.h` defines a standard interface, with the most important function being one that calculates signal loss, conceptually like this:
```cpp
// A conceptual representation
virtual double GetLossDb (Ptr<MobilityModel> a, Ptr<MobilityModel> b) = 0;
```
The `SatellitePhy` (physical layer) uses this loss value to determine if the received signal is strong enough for successful decoding (i.e., if the Signal-to-Noise Ratio is sufficient).

## 3. Exploring Different Fading Models
The power of this design is evident in the variety of available models, each suited for different scenarios.

### How They Are "Plugged In"
In your ns-3 simulation script, you instantiate the model you want and attach it to the channel.
```cpp
// 1. Create the channel
Ptr<SatelliteChannel> channel = CreateObject<SatelliteChannel> ();

// 2. Create a specific fading model (e.g., Loo)
Ptr<LooFadingModel> fadingModel = CreateObject<LooFadingModel> ();

// 3. "Plug it in" to the channel
channel->SetFadingModel (fadingModel);
```

### Common Satellite Fading Models
- **Rayleigh Fading**: Models scenarios with **no dominant line-of-sight (LoS) path**, where the signal is received only through reflections. This is less common for satellites but can simulate severe blockage.
- Loo Fading Model: A highly realistic and widely used model for **Land Mobile Satellite (LMS)** links. It's a composite model that includes:
  - A **shadowed Line-of-Sight (LoS) signal**, modeled with a **log-normal distribution** to represent slow fading from large obstacles (hills, large buildings).
  - **Multipath signals**, modeled with a **Rayleigh distribution** to represent fast fading from many smaller reflections. This combination is perfect for simulating a vehicle moving through a mixed urban/suburban environment.
  - **Markov Chain Models**: These are abstract, state-based models. They define channel "states" (e.g., "Good" and "Bad") and the probabilities of transitioning between them. For example, a link could be in a "Good" state (clear LoS) and transition to a "Bad" state (blocked by a bridge) for a short time. They are computationally efficient and great for analyzing how higher-level protocols perform over channels with intermittent connectivity.

---

## Tracing the ns-3 PHY-to-Channel Signal Flow
This guide details a packet's journey from the physical layer (PHY) of a satellite device, through the channel, and to the PHY of a receiver. Understanding this flow is key to debugging and verifying ns-3 wireless simulations.

### The Signal Flow: From Transmission to Reception
The entire process is a coordinated handoff between the **PHY layer**, the **Channel**, and the **ns-3 Scheduler**. The Scheduler acts as the simulation's master clock, triggering events at precise future times.

#### 1. The Call to `SatPhy::Transmit()`
The process begins when a `SatelliteNetDevice` passes a packet to its physical layer, calling the `SatPhy::Transmit()` method.

Inside `Transmit()`, two main things happen:

1.  **Calculate Transmission Time**: The function determines how long it will take to physically transmit all bits of the packet based on its size and the channel's data rate.
    $$
    \text{Transmission Time} = \frac{\text{Packet Size (bits)}}{\text{Data Rate (bps)}}
    $$
2.  **Send to the Channel**: The PHY then hands the packet and the calculated transmission time to the `SatelliteChannel` by calling its `Send()` method.

    ```cpp
    // Inside SatPhy::Transmit(Ptr<Packet> packet)
    Time txTime = /* Calculated based on packet size and data rate */;
    m_channel->Send(packet, m_netDevice, txTime);
    ```

#### 2. The Packet's Journey Through the Channel

Now, the `SatelliteChannel::Send()` method takes over. For **every other device** connected to it, the channel performs these steps:

1.  It asks the **propagation delay model** for the signal's travel time between the sender and this specific receiver.
2.  It calculates the **total delay**: `Total Delay = Transmission Time + Propagation Delay`.
3.  It uses the **master scheduler** to create a future event. This event tells the simulator: "In `Total Delay` seconds, call the `StartRx` function on the receiver's PHY and deliver this packet."

This scheduling is the most critical part. The packet isn't delivered instantly; it's queued to arrive at the exact, physically correct moment.

### Confirming Events with `PacketTrace.log`
This entire sequence is captured in trace files. When you enable packet tracing, it logs every significant packet event. Let's analyze a simplified log from a link between a ground terminal (Node 0) and a satellite (Node 1), assuming a **12 ms propagation delay**.

Your `PacketTrace.log` would look something like this:
  ```cpp
  # Simplified Packet Trace Log
  # Event Time  Node  Layer   Flags  Packet Info
  SND     1.000 0     SatPhy         -  seq=1 size=1024
  RCV     1.012 1     SatPhy         -  seq=1 size=1024
  ```

#### Analyzing the Log
- `SND 1.000 0 SatPhy ...`
  - `SND`: This marks a send event.
  - `1.000`: The simulation time when the event occurred. This is the moment `SatPhy::Transmit()` was called on the ground terminal (Node 0).
  - `0`: The node number that is sending (the ground terminal).
  - `SatPhy`: The simulation layer where the event was logged.
  - This line confirms that at 1.000 second, the PHY layer of Node 0 began transmitting a packet.
- `RCV 1.012 1 SatPhy ...`
  - `RCV`: This marks a receive event.
  - `1.012`: The simulation time of the reception. Notice that `1.012 = 1.000 (send time) + 0.012 (propagation delay)`. The transmission time is often negligible for a single packet and is included in this delay calculation by the scheduler.
  - `1`: The node number that is receiving (the satellite).
  - `SatPhy`: The layer where the reception happened.
  - This line confirms that the packet arrived at the satellite's PHY exactly 12 milliseconds after it was sent, perfectly matching the calculated propagation delay.
By looking at the timestamps of the `SND` and `RCV` events in your trace files, you can directly measure the end-to-end propagation delay modeled by the channel and verify that the signal flow is working exactly as designed.

---

## Summary: ns-3 Satellite Channel Concepts
### The Channel's Role
We learned that the `SatelliteChannel` acts as the **manager** of the communication medium. Its primary job is to take a packet from a sender, calculate how long it will take to reach each potential receiver, and then schedule its arrival using the delay model and the simulator's scheduler.

### How Delay is Calculated
The `SatellitePropagationDelayModel` is the physics calculator. It computes propagation delay based on pure geometry:

1.  It gets the real-time 3D coordinates of the sender and receiver.
2.  It calculates the straight-line **Euclidean distance** between them.
3.  It divides this distance by the **speed of light** to get the exact travel time.

This ensures the delay is dynamic and changes realistically as the satellite moves.

### Fading and Signal Strength
We explored how ns-3 uses a **pluggable architecture** for fading models to simulate signal degradation from rain or buildings. The channel can be configured with different models (**Rayleigh**, **Loo**, **Markov**) to calculate the signal loss (in dB), which the receiving PHY uses to determine if a packet was successfully received.

### Tracing the Signal Flow
Finally, we traced a packet's entire journey:

1.  **`SatPhy::Transmit()`** starts the process on the sending node.
2.  The packet is handed to **`SatelliteChannel::Send()`**.
3.  The channel calculates the total delay and uses **`Simulator::Schedule()`** to create a future arrival event.
4.  At the scheduled time, the packet is delivered to the receiving node's **`SatPhy`** layer.

We saw how this sequence is confirmed using a **packet trace log**, where the time difference between a **`SND`** and **`RCV`** event directly shows the simulated propagation delay.

# Scenario & Example Customization
## ns-3 Satellite Example Scenarios: A Walkthrough
This guide breaks down the structure of typical ns-3 satellite simulation scripts like `sat-tutorial-example.cc` and `sat-iot-example.cc`. The goal is to understand how the network is built, how communication links are defined, and how data traffic is generated.

---
## 1. Node Creation (`CreateNodes`)
This is the first major step where you create all the participants in your network simulation. Think of it as placing your satellites in orbit and your ground stations on Earth.

### What it is:
This block uses the `ns3::NodeContainer` class to create and manage a group of nodes. Nodes are the fundamental building blocks—they represent computers, terminals, or satellites.

### How it works:
- You'll see `NodeContainer groundTerminals;` and `groundTerminals.Create(numTerminals);`. This line creates a specified number of nodes that will act as our ground stations.
- Similarly, `NodeContainer satellite; satellite.Create(1);` creates the satellite node.

### Positioning is Key
After creation, nodes are just abstract points. The `MobilityHelper` is used to give them physical locations.
- For satellites, you often use `ListPositionAllocator` to set their exact orbital positions over time (e.g., by reading a TLE file).
- For ground stations, `GridPositionAllocator` is common to spread them out in a grid pattern on the Earth's surface.
```
// Example: Creating ground terminals and a satellite
NodeContainer groundTerminals;
groundTerminals.Create(10); // Create 10 ground terminals

NodeContainer satellite;
satellite.Create(1); // Create 1 satellite

// ... MobilityHelper code would go here to position them ...
```
This block sets up who is in the network and where they are.

## 2. Channel & Device Configuration
Once you have your nodes, you need to give them the hardware to communicate and define the "air" through which their signals travel. This is where the satellite-specific helpers shine.

### What it is
This block configures the `SatNetDevice` (the network card), the `SatPhy` (the physical radio layer), the `SatMac` (the data link layer), and the `SatelliteChannel` (the space environment).

### The `SatHelper`
This is the main tool you'll use. It simplifies the process of creating and connecting all the necessary components.

### How it works
- `SatHelper satHelper;`: You start by creating an instance of the helper.
- `satHelper.SetPhy(...)`: You configure the physical layer properties here. This is a crucial step for setting parameters.
- `satHelper.SetMac(...)`: Configures the MAC layer, which handles how multiple users share the channel.
- `satHelper.Install(...)`: This command takes your node containers and installs the configured SatNetDevice on each node, effectively giving them satellite communication capabilities.
- `satHelper.ConfigureRegularMAC(...)`: Sets up the communication schedule (e.g., which terminal can transmit at what time).
- `satHelper.ConfigurePropagationChannel(...)`: This defines the physics of the radio link. It links the physical layers of the devices to a shared channel and sets up models for how the signal behaves, including:
  - Path Loss: How the signal weakens over the vast distance to the satellite (e.g., `FriisPropagationLossModel`).
  - Propagation Delay: The time it takes for the signal to travel, calculated based on the real-time positions of the nodes (`SatellitePropagationDelayModel`). This is not a fixed value and changes as the satellite moves.
This block defines how the nodes will communicate.

## 3. Application Setup (Generating Traffic)
Your network is built, but it isn't doing anything yet. The application setup block defines the actual data that will be sent across the network, allowing you to measure performance.

### What it is
This block uses ApplicationHelper classes to simulate network traffic.

### How it works:
- Choose a Traffic Type:
  -  OnOffHelper: This is very versatile. You can use it to create Constant Bit Rate (CBR) traffic by setting the "OnTime" to a constant value and the "OffTime" to zero. This simulates a steady stream of data, like a video call. For bursty traffic, you'd set both on and off times.
  -  UdpClientHelper / PacketSinkHelper: This client/server pair is common. The UdpClient sends packets to a specific destination address and port, and the PacketSink on the destination node "listens" on that port, receives the packets, and collects statistics (like packet loss, throughput, etc.). The sat-iot-example.cc likely uses this to simulate small, infrequent data packets from IoT devices.
- Install the Apps:
  - Create a "sink" (server) on the receiving node.
  - Create a "client" (source) on the sending node.
  - Set the start and stop times for the application using app.SetStartTime(...) and app.SetStopTime(...).

```
// Example: Setting up a CBR-like flow
uint16_t sinkPort = 8080;
Address sinkAddress (InetSocketAddress (receiverInterface.GetAddress (0), sinkPort));
PacketSinkHelper packetSinkHelper ("ns3::UdpSocketFactory", sinkAddress);
ApplicationContainer sinkApps = packetSinkHelper.Install (receiverNode);
sinkApps.Start (Seconds (1.0));
sinkApps.Stop (Seconds (10.0));

OnOffHelper onOffHelper ("ns3::UdpSocketFactory", sinkAddress);
onOffHelper.SetAttribute ("DataRate", StringValue ("500kbps")); // Parameter set here
onOffHelper.SetAttribute ("PacketSize", UintegerValue (1024));
// ... other attributes ...

ApplicationContainer clientApps = onOffHelper.Install (senderNode);
clientApps.Start (Seconds (2.0));
clientApps.Stop (Seconds (9.0));
```
This block defines what data is being sent through the network, from where, and to where. By analyzing the results at the sink, you can evaluate your network's performance.

## 4. Setting and Loading Parameters
In ns-3, parameters are the knobs and dials you turn to control the simulation's behavior. You'll almost always use the `SetAttribute` method on a helper or an object to configure them. </br>
The general pattern is: `HelperOrObject.SetAttribute("AttributeName", AttributeValue);`
- "AttributeName": A string that specifies which parameter you want to change (e.g., `"DataRate"`, `"Frequency"`, `"PacketSize"`).
- AttributeValue: The value you want to assign. This has to be wrapped in a specific `Value` type, like `StringValue`, `UintegerValue`, `DoubleValue`, or `TimeValue`.
Let's look at the specific examples

### Data Rate 
This is usually an Application-level parameter. You set it on an application helper to control how fast it generates data.
```
// From the Application Setup section
OnOffHelper onOffHelper ("ns3::UdpSocketFactory", sinkAddress);

// Here we set the data rate to 1 Megabit per second
onOffHelper.SetAttribute ("DataRate", StringValue ("1Mbps"));
onOffHelper.SetAttribute ("PacketSize", UintegerValue (1500));
```

### Frequency
This is a Physical Layer (PHY) parameter. You define the carrier frequency on which the satellite and ground terminals will transmit and receive. This is often done when installing the SatNetDevice.

```
// From the Channel & Device Configuration section
SatHelper satHelper;

// Setting attributes on the PHY layer before installing it
// Note: This might be done through a dedicated SetPhy method in the helper
// or by setting attributes on the helper which passes them down.
satHelper.SetPhy ("Frequency", UintegerValue (14000000000)); // 14 GHz Uplink
satHelper.SetPhy ("DownlinkFrequency", UintegerValue (12000000000)); // 12 GHz Downlink
```

### Delay (Important Distinction!)
Unlike wired simulations where you might set a fixed delay on a channel, in the satellite module, propagation delay is not a fixed parameter you set. Instead, it is dynamically calculated at every step of the simulation by the SatellitePropagationDelayModel. This model continuously checks the physical distance between the transmitter and receiver (using their MobilityModel positions) and calculates the delay using the speed of light ($c$). 

```math
Delay(t) = \frac{\text{distance} (t)}{c}
```

So, you don't set the delay; you set up the mobility models correctly, and the simulation calculates the realistic, time-varying delay for you. This is one of the most powerful features of the satellite module.

---

## Running Simulations and Observing Outputs
Let's run them and see what they produce. This is where you get to see the simulated network in action.

## 1. Running the Example Scenarios
To execute any ns-3 script, you use the ns3 executable from the root directory of your ns-3 workspace, followed by the run command and the name of the script.
- Command Structure: ./ns3 run <script_name>
- To run the tutorial example:
```
./ns3 run "contrib/satellite/examples/sat-tutorial-example"
```

- To run the IoT example:
```
./ns3 run "contrib/satellite/examples/sat-iot-example"
```

When you run these commands, ns-3 compiles the script and executes the simulation defined within its main function. You'll see some output in your terminal, which is typically status information logged using NS_LOG_INFO. The real, detailed data, however, is saved to files.

## 2. Locating the Log Files
After each run, the log files are typically stored under:
```
ns-3.43/contrib/satellite/
└── results/
    └── <ScenarioName>/
```
However, some examples may write directly to the main directory or current working directory. </br>
So we can verify by running:
```
find . -type f -name "*.log" | grep -E "PacketTrace|IoT|Scenario"
```
This searches for all .log files generated by the examples.

### Expected Log Files By Example
#### A. `sat-tutorial-example`
| File Name                           | Description                                                         |
| ----------------------------------- | ------------------------------------------------------------------- |
| `PacketTrace.log`                   | Timestamped record of all packets sent and received (Tx/Rx events). |
| `ScenarioCreation.log` *(optional)* | Details about scenario initialization, nodes, and links.            |
| `SimulationOutput.log` *(optional)* | Summary of key performance stats (throughput, delay).               |

typical location
```
contrib/satellite/results/tutorial/
```
or
```
build/contrib/satellite/examples/
```
example file content
```
At time 2s cbr application sent 512 bytes to 10.1.0.2 port 9 total Tx 512 bytes
At time +2.28815s packet sink received 512 bytes ...
```

### B. `sat-iot-example`
| File Name           | Description                                                 |
| ------------------- | ----------------------------------------------------------- |
| `PacketTrace.log`   | Uplink/downlink events between IoT terminals and gateway.   |
| `EnergyTrace.txt`   | Power consumption statistics for IoT nodes.                 |
| `IoTStatistics.log` | Message count, success ratio, collisions, etc.              |
| `SatTrace.tr`       | Detailed PHY/MAC layer trace of satellite channel activity. |

typical location
```
contrib/satellite/results/iot/
```
or
```
contrib/satellite/examples/
```
you can confirm by:
```
ls contrib/satellite/examples | grep log
```

## Comparing `PacketTrace.log` Results
| Aspect                | `sat-tutorial-example`             | `sat-iot-example`                                            |
| --------------------- | ---------------------------------- | ------------------------------------------------------------ |
| **Purpose**           | Basic demo of satellite link       | Large-scale IoT simulation                                   |
| **Packet Source**     | CBR application (constant bitrate) | Periodic IoT sensors                                         |
| **Traffic Direction** | Bidirectional (GW ↔ UT)            | Mostly uplink (IoT → GW)                                     |
| **Latency**           | ~2–3 seconds (GEO delay)           | ~2–3 seconds + contention delay                              |
| **PacketTrace.log**   | 3–5 events, clean delivery         | Hundreds/thousands of entries, possible collisions or losses |
| **EnergyTrace**       | None                             | Present (for battery/sleep models)                         |
| **Progress Output**   | None                               | “Progress: x/60” every second                                |
| **Complexity**        | Beginner                        | Advanced (multi-node + MAC scheduling)                    |

### How to compare Event Sequences
- Extract key packet events
```
grep "sent" PacketTrace.log > sent.log
grep "received" PacketTrace.log > recv.log
```

- Then check timing correlation
  - In `sat-tutorial-example`, the time difference between “sent” and “received” ≈ 2–3 s, representing satellite delay.
  - In `sat-iot-example`, you’ll observe:
    - Multiple simultaneous transmissions
    - Occasional missing “received” lines (packet loss)
    - Varying delay (depends on access scheme)

- For side by side comparison
```
paste sent.log recv.log
```
or
```
diff -y tutorial/PacketTrace.log iot/PacketTrace.log
```

---

## Trying Examples And Modify Parameters.
### 1. Trying Examples
Successfully ran two examples from the ns-3.43 satellite module:
```
./ns3 run "contrib/satellite/examples/sat-tutorial-example"
./ns3 run "contrib/satellite/examples/sat-iot-example"
```
output
```
geemajor@joy:~/workspace/bake/source/ns-3.43$ ./ns3 run "contrib/satellite/examples/sat-tutorial-example"
--- Tutorial-example ---
  Scenario used: Simple

At time 2s cbr application sent 512 bytes to 10.1.0.2 port 9 total Tx 512 bytes
At time +2.28815s packet sink received 512 bytes from 90.2.0.2 port 49153 total Rx 512 bytes
At time 8s cbr application sent 512 bytes to 90.2.0.2 port 9 total Tx 512 bytes
At time +8.30803s packet sink received 512 bytes from 10.1.0.2 port 49153 total Rx 512 bytes
At time 9s cbr application sent 512 bytes to 90.2.0.2 port 9 total Tx 1024 bytes
At time +9.30403s packet sink received 512 bytes from 10.1.0.2 port 49153 total Rx 1024 bytes

geemajor@joy:~/workspace/bake/source/ns-3.43$ ./ns3 run "contrib/satellite/examples/sat-iot-example"
Progress: 0.6/60
Progress: 1.2/60
Progress: 1.8/60
Progress: 2.4/60
Progress: 3/60
Progress: 3.6/60
...
```

### 2. Locating The Example Dir Files
run
```
geemajor@joy:~/workspace/bake/source/ns-3.43$ ls contrib/satellite/examples/
```
output
```
geemajor@joy:~/workspace/bake/source/ns-3.43$ ls contrib/satellite/examples/
CMakeLists.txt                           sat-mobility-example.cc
dama-input-attributes.xml                sat-mobility-position-generator.cc
first-satellite-sim.cc                   sat-multi-application-fwd-example.cc
generic-input-attributes.xml             sat-multi-application-rtn-example.cc
sat-arq-fwd-example.cc                   sat-multicast-example.cc
sat-arq-rtn-example.cc                   sat-ncr-example.cc
sat-beam-position-tracer.cc              sat-nrtv-example.cc
sat-cbr-example.cc                       sat-onoff-example.cc
sat-cbr-full-example.cc                  sat-per-packet-if-sim-tn9.cc
sat-cbr-stats-example.cc                 sat-profiling-sim-tn8.cc
sat-cbr-user-defined-example.cc          sat-profiling-sim.cc
sat-constellation-example.cc             sat-ra-sim-tn9-comparison.cc
sat-dama-http-sim-tn9.cc                 sat-ra-sim-tn9.cc
sat-dama-onoff-sim-tn9.cc                sat-random-access-crdsa-collision-example.cc
sat-dama-sim-tn9.cc                      sat-random-access-crdsa-example.cc
sat-dama-verification-sim.cc             sat-random-access-dynamic-load-control-example.cc
sat-dynamic-frequency-plan-example.cc    sat-random-access-example.cc
sat-environmental-variables-example.cc   sat-random-access-slotted-aloha-collision-example.cc
sat-essa-example.cc                      sat-random-access-slotted-aloha-example.cc
sat-fwd-link-beam-hopping-example.cc     sat-rayleigh-example.cc
sat-fwd-system-test-example.cc           sat-regeneration-collisions-example.cc
sat-generic-launcher.cc                  sat-regeneration-example.cc
sat-group-example.cc                     sat-rtn-link-da-example.cc
sat-gw-handover-example.cc               sat-rtn-link-ra-example.cc
sat-handover-example.cc                  sat-rtn-system-test-example.cc
sat-http-example.cc                      sat-trace-input-external-fading-example.cc
sat-iot-example.cc                       sat-trace-input-fading-example.cc
sat-link-budget-example.cc               sat-trace-input-interference-example.cc
sat-link-budget-input-attributes.xml     sat-trace-input-rx-power-example.cc
sat-link-results-plot.cc                 sat-trace-output-example.cc
sat-list-position-ext-fading-example.cc  sat-training-example.cc
sat-log-example.cc                       sat-tutorial-example.cc
sat-logon-example.cc                     sat-tutorial-input.xml
sat-loo-example.cc                       sat-vhts-example.cc
sat-lora-constellation-example.cc        sys-rtn-test.xml
sat-lora-example.cc                      tn8-profiling-input-attributes.xml
sat-lora-handover-example.cc             tn9-dama-input-attributes.xml
sat-lora-regenerative-example.cc         tn9-ra-input-attributes.xml
sat-markov-fading-trace-example.cc       training-input-attributes.xml
sat-markov-logic-example.cc              wscript
sat-mobility-beam-tracer.cc
```

### 3. Locate Log / Trace Outputs
Run this from the ns-3.43 root:
```
find . -name "*.log" | grep satellite
```


and also:
```
find . -name "*trace*" | grep satellite
```


Possible outputs include files like:
```
geemajor@joy:~/workspace/bake/source/ns-3.43$ find . -name "*.log" | grep satellite
./contrib/satellite/data/sims/example-tutorial/Simple/CreationTraceUt.log
./contrib/satellite/data/sims/example-tutorial/Simple/CreationTraceScenario.log
./contrib/satellite/data/sims/example-tutorial/Simple/PacketTrace.log
./contrib/satellite/data/sims/sat-iot-example/PacketTrace.log
./contrib/satellite/data/sims/example-cbr-stats/larger/CreationTraceUt.log
./contrib/satellite/data/sims/example-cbr-stats/larger/CreationTraceScenario.log
./contrib/satellite/data/sims/example-cbr-stats/larger/PacketTrace.log
./contrib/satellite/data/sims/default/SimInfo.log
./contrib/satellite/data/sims/default/SimDiff.log
./contrib/satellite/data/sims/example-cbr/simple/PacketTrace.log

geemajor@joy:~/workspace/bake/source/ns-3.43$ find . -name "*trace*" | grep satellite
./cmake-cache/contrib/satellite/CMakeFiles/satellite-test.dir/test/satellite-fading-external-input-trace-test.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite-test.dir/test/satellite-fading-external-input-trace-test.cc.o.d
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-traced-interference.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-packet-trace.cc.o.d
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-composite-sinr-output-trace-container.cc.o.d
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-fading-output-trace-container.cc.o.d
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-rx-cno-input-trace-container.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-fading-external-input-trace-container.cc.o.d
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-fading-output-trace-container.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-interference-output-trace-container.cc.o.d
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-interference-input-trace-container.cc.o.d
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-base-trace-container.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-fading-input-trace-container.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-fading-input-trace.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-fading-external-input-trace.cc.o.d
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-packet-trace.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-rx-power-input-trace-container.cc.o.d
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-rx-cno-input-trace-container.cc.o.d
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-traced-mobility-model.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-fading-external-input-trace-container.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-interference-input-trace-container.cc.o
./cmake-cache/contrib/satellite/CMakeFiles/satellite.dir/model/satellite-fading-input-trace.cc.o.d
.....
```

### 4. Compare Both Log File
We’ll do this by viewing the two main files:
```
cat contrib/satellite/data/sims/example-tutorial/Simple/PacketTrace.log | head -n 20
```

and
```
cat contrib/satellite/data/sims/sat-iot-example/PacketTrace.log | head -n 20
```

Possible outputs:
```
geemajor@joy:~/workspace/bake/source/ns-3.43$ cat contrib/satellite/data/sims/example-tutorial/Simple/PacketTrace.log | head -n 20
COLUMN DESCRIPTIONS
-------------------
Time
Packet event (SND, RCV, DRP, ENQ)
Node type (UT, SAT, GW, NCC, TER)
Node id
MAC address
Log level (ND, LLC, MAC, PHY, CH)
Link direction (FWD, RTN)
Packet info (List of: Packet id, source MAC address, destination MAC address)
-------------------

0.05 SND GW 2 00:00:00:00:00:04 ND FWD 0
0.05 ENQ GW 2 00:00:00:00:00:04 LLC FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.0500152 SND GW 2 00:00:00:00:00:04 LLC FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.0500152 SND GW 2 00:00:00:00:00:04 MAC FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.0500152 SND GW 2 00:00:00:00:00:04 PHY FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.1 SND GW 2 00:00:00:00:00:04 ND FWD 1
0.1 ENQ GW 2 00:00:00:00:00:04 LLC FWD 1 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.1 SND GW 2 00:00:00:00:00:04 LLC FWD 1 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff

geemajor@joy:~/workspace/bake/source/ns-3.43$ cat contrib/satellite/data/sims/sat-iot-example/PacketTrace.log | head -n 20
COLUMN DESCRIPTIONS
-------------------
Time
Packet event (SND, RCV, DRP, ENQ)
Node type (UT, SAT, GW, NCC, TER)
Node id
MAC address
Log level (ND, LLC, MAC, PHY, CH)
Link direction (FWD, RTN)
Packet info (List of: Packet id, source MAC address, destination MAC address)
-------------------

0.0160029 SND UT 6 00:00:00:00:00:05 ND RTN 0
0.0160029 ENQ UT 6 00:00:00:00:00:05 LLC RTN 0 00:00:00:00:00:05 00:00:00:00:00:04
0.0180006 SND UT 6 00:00:00:00:00:05 ND RTN 5
0.0180006 ENQ UT 6 00:00:00:00:00:05 LLC RTN 5 00:00:00:00:00:05 00:00:00:00:00:04
0.0260006 SND UT 6 00:00:00:00:00:05 ND RTN 6
0.0260006 ENQ UT 6 00:00:00:00:00:05 LLC RTN 6 00:00:00:00:00:05 00:00:00:00:00:04
0.0340006 SND UT 6 00:00:00:00:00:05 ND RTN 7
0.0340006 ENQ UT 6 00:00:00:00:00:05 LLC RTN 7 00:00:00:00:00:05 00:00:00:00:00:04
```

---

## Modify Something While Running Example
### 1. Choose a Simple but Visible Parameter
Let’s modify the number of UTs (User Terminals) in the IoT example, since that:
- Affects network load
- Impacts packet timing and throughput
- Is easy to locate and interpret in logs
We’ll compare before and after.

### 2. Locate and Open the Example File
open file example
```
nano contrib/satellite/examples/sat-iot-example.cc
```
Now, look near the top (around lines 50–100). </br>
You’ll find something like:
<img width="431" height="211" alt="image" src="https://github.com/user-attachments/assets/268a1099-4ab5-49ad-aa90-453060861c65" />

### 3. Modify The Parameter
Change this line:
```
uint32_t nbUtsPerBeam = 1;
uint32_t nbUtsPerBeam = 2;
```
That means for each of the 8 beams, 2 UTs will be deployed — total UTs now doubled.
This should noticeably increase packet traffic.

### 4. Rerun the Simulation
```
./ns3 run "contrib/satellite/examples/sat-iot-example"
```
Watch for longer or denser “Progress:” outputs — simulation might take a bit longer, as more UTs means more packet events.
</br>
output:
```
geemajor@joy:~/workspace/bake/source/ns-3.43$ cat contrib/satellite/data/sims/sat-iot-example/PacketTrace.log | head -n 20
COLUMN DESCRIPTIONS
-------------------
Time
Packet event (SND, RCV, DRP, ENQ)
Node type (UT, SAT, GW, NCC, TER)
Node id
MAC address
Log level (ND, LLC, MAC, PHY, CH)
Link direction (FWD, RTN)
Packet info (List of: Packet id, source MAC address, destination MAC address)
-------------------

0.0160029 SND UT 7 00:00:00:00:00:08 ND RTN 1
0.0160029 ENQ UT 7 00:00:00:00:00:08 LLC RTN 1 00:00:00:00:00:08 00:00:00:00:00:06
0.0170029 SND UT 6 00:00:00:00:00:07 ND RTN 0
0.0170029 ENQ UT 6 00:00:00:00:00:07 LLC RTN 0 00:00:00:00:00:07 00:00:00:00:00:06
0.0180006 SND UT 6 00:00:00:00:00:07 ND RTN 10
0.0180006 ENQ UT 6 00:00:00:00:00:07 LLC RTN 10 00:00:00:00:00:07 00:00:00:00:00:06
0.0190006 SND UT 7 00:00:00:00:00:08 ND RTN 11
0.0190006 ENQ UT 7 00:00:00:00:00:08 LLC RTN 11 00:00:00:00:00:08 00:00:00:00:00:06
```


### 5. Compare Logs
```
cat contrib/satellite/data/sims/sat-iot-example/PacketTrace.log | head -n 20
```
You should now notice:
- More Node IDs (e.g., UT 6, UT 7, UT 8 instead of just UT 6)
- Higher frequency of SND and ENQ events
- Possibly overlapping timestamps (multiple UTs sending concurrently)

## 6. Analyze The Differences
| Metric                     | Baseline          | Modified                                     |
| -------------------------- | ----------------- | -------------------------------------------- |
| **Number of UTs active**   | 1                 | 2                                            |
| **First packet time (s)**  | ~0.016            | ~0.016                                       |
| **Node IDs (UT)**          | 6                 | 6, 7                                         |
| **Send/Enqueue frequency** | Lower             | Higher (more interleaved `SND`/`ENQ` events) |
| **Direction**              | RTN (return link) | RTN (same), but more frequent events         |

The increase in UT count caused the scheduler to interleave packets from both UTs, sharing return link capacity. 

### Interpretation
- The superframe capacity handled 2 UTs but crashed at 3 because of limited symbol/time-slot allocation.
- Each UT now competes for link access → more realistic IoT traffic model.
- No PHY-level congestion yet → system still within normal limits.

# Integrating Everything: End-to-End system understanding
Understand and trace how a single packet travels through the entire NS-3 Satellite stack from the sender application to the receiver, across PHY, MAC, and Channel, and finally to the trace logs.

## The Packet Flow Path (End-to-End)
Let's trace a single packet from a Source Ground Terminal (GT-A), up to the Satellite, and down to a Destination Ground Terminal (GT-B). </br>
Here is the step-by-step flow:
### 1. Creation (at GT-A)
  - The `Application` (e.g., `OnOffHelper`) creates a packet
  - This packet is pushed down the network stack (UDP->IP) to the `SatNetDevice`)
### 2. Uplink Transmission (From GT-A)
  - The packet is placed in the `SatMac` queue oof GT-A
  - When its scheduled time arrives, the `SatPhy` (Physical Layer) transmits the packet as a signal
### 3. Uplink Propagation (Space)
  - The signal travels through the `SatelliteChannel`
  - The `SatellitePropagationDelayModel` calculates the time it takes to reach the satellite based on their real-time positions.
  - Fading and path loss models are applied, which might cause the packet to be corrupted.
### 4. Reception (At Satellite)
  - The satellite's `SatPhy` "hears" the signal
  - If the signal is strong enough (not lost to fading), the `SatMac` receives the packet
  - The satellite (acting as a "bent-pipe" or regenerative repeater) identifies the destination (GT-B) and queues the packet for downlink
### 5. Downlink Transmission (from Satellite)
  - The satellite's `SatMac` queues the packet for the downllink beam
  - The `SatPhy` transmits the packet down towards GT-B
### 6. Downlink Propagation (Space)
  - The signal again travels through the `SatelliteChannel`
  - The `SatellitePropagationDelayModel` calculates the propagation delay for the downlink path
### 7. Final Reception (at GT-B)
  - GT-B'S `SatPhy` receives the signal
  - Its `SatMac processes the packet
  - The packet is sent up the network stack (IP ->UDP) to the `Application` (e.g., `PacketSink`)
### 8. Consumption (at GT-B)
  - The `PacketSink` application "consumes" the packet and records its arrival time, size, etc., for calculating statistics.

#### This is a visual graph for explaining how the E2E satellite simulation:
<img width="427" height="1105" alt="E2E_Packet_Flow_Square" src="https://github.com/user-attachments/assets/e3f667b5-19ff-4857-bba2-851497652020" />

### Where Logs are Triggered in the Flow
The log files capture specific events as they happen during this flow.
| Stage in Flow | Event | Log File Triggered | Log Entry |
| ------------- | ----- | ------------------ | --------- |
| 1. Creation (GT-A) | Application sends packet to the stack.| `PacketTrace.log` | `+` (enqueue) at `Node /Application/` |
|2. Uplink Tx (GT-A) | Packet leaves the MAC queue for tx.| `PacketTrace.log` | `-` (dequeue) at `Node /Device/`|
|4. Reception (Sat) | Satellite's MAC successfully receives packet.| `PacketTrace.log` |`r` (receive) at `Node /Device/` |
|7. Final Rx (GT-B) | GT-B's MAC successfully receives packet.| `PacketTrace.log` |`r` (receive) at `Node /Device/` |
|8. Consumption (GT-B) | `PacketSink` receives the packet.| (Internal to `PacketSink`) |"The sink's internal counters (total bytes, total packets) are updated." |
|End of Simulation | `Simulator::Run()` finishes. | `SimInfo.log` |The `PacketSink` is asked for its final stats and writes them to this file. |

- `PacketTrace.log` is a real-time event log. It's triggered during the simulation at every single hop (SND, RCV). You use it to trace the path and timing of individual packets.
- `SimInfo.log` is a final summary report. It's written after the simulation ends. It contains the final aggregated statistics (like "Total Packets Received," "Average Throughput") that are collected by applications like `PacketSink`.

## Run and Observe Full Simulation
This is a workflow for how i observe the simulation.
### 1. Execute simulation
using this lines of code, and used one of the examples (i used `sat-tutorial-example`)
```
geemajor@joy:~/workspace/bake/source/ns-3.43$ ./ns3 run sat-tutorial-example
```
the output: This is the console log output, often showing the status of the simulation, like "Simulation started..." or "PacketSink Results...".
```
--- Tutorial-example ---
  Scenario used: Simple

At time 2s cbr application sent 512 bytes to 10.1.0.2 port 9 total Tx 512 bytes
At time +2.28815s packet sink received 512 bytes from 90.2.0.2 port 49153 total Rx 512 bytes
At time 8s cbr application sent 512 bytes to 90.2.0.2 port 9 total Tx 512 bytes
At time +8.30803s packet sink received 512 bytes from 10.1.0.2 port 49153 total Rx 512 bytes
At time 9s cbr application sent 512 bytes to 90.2.0.2 port 9 total Tx 1024 bytes
At time +9.30403s packet sink received 512 bytes from 10.1.0.2 port 49153 total Rx 1024 bytes
```

### 2. Open Resulting Files
Once the simulation is finished, list the files in the output directory to confirm they were created. with `geemajor@joy:~/workspace/bake/source/ns-3.43$ find . -type f -mtime -1`, the output will be a list of all files modified in the last 1 day, we should see anything your simulation created. </br> Output:
```
geemajor@joy:~/workspace/bake/source/ns-3.43$ find . -type f -mtime -1
./contrib/satellite/data/sims/example-tutorial/Simple/CreationTraceUt.log
./contrib/satellite/data/sims/example-tutorial/Simple/CreationTraceScenario.log
./contrib/satellite/data/sims/example-tutorial/Simple/PacketTrace.log
./contrib/satellite/data/sims/default/SimInfo.log
./contrib/satellite/data/sims/default/SimDiff.log
```
open the files using `less [file directory]` or `nano [file directory]`. Lets open the `PacketTrace.lg` and `SimInfo.log`
```
geemajor@joy:~/workspace/bake/source/ns-3.43$ less contrib/satellite/data/sims/example-tutorial/Simple/PacketTrace.log
geemajor@joy:~/workspace/bake/source/ns-3.43$ less contrib/satellite/data/sims/default/SimInfo.log
geemajor@joy:~/workspace/bake/source/ns-3.43$ less contrib/satellite/data/sims/default/SimDiff.log
```

### 3. Locate SND -> RCV Pairs
We will look inside the `PacketTrace.log` file.

#### Step 1: Find a "SND" Event
Let's find the very first packet that was sent. To call it, you'll use these lines of code
```
awk '$2=="SND"' contrib/satellite/data/sims/example-tutorial/Simple/PacketTrace.log
```
The output will be
```
0.05 SND GW 2 00:00:00:00:00:04 ND FWD 0
0.0500152 SND GW 2 00:00:00:00:00:04 LLC FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.0500152 SND GW 2 00:00:00:00:00:04 MAC FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.0500152 SND GW 2 00:00:00:00:00:04 PHY FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.1 SND GW 2 00:00:00:00:00:04 ND FWD 1
0.1 SND GW 2 00:00:00:00:00:04 LLC FWD 1 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.100008 SND GW 2 00:00:00:00:00:04 MAC FWD 1 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.100008 SND GW 2 00:00:00:00:00:04 PHY FWD 1 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.15 SND GW 2 00:00:00:00:00:04 ND FWD 2
0.15 SND GW 2 00:00:00:00:00:04 LLC FWD 2 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.15 SND GW 2 00:00:00:00:00:04 MAC FWD 2 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.15 SND GW 2 00:00:00:00:00:04 PHY FWD 2 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.183623 SND SAT 0 00:00:00:00:00:01 PHY FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
...
```
Breakdown:
| Column | Value             | Meaning                              |
| ------ | ----------------- | ------------------------------------ |
| 1      | 0.05              | Time (seconds) when event occurred   |
| 2      | SND               | Event type (`SND` = packet sent)     |
| 3      | GW                | Node type (`GW` = gateway)           |
| 4      | 2                 | Node ID                              |
| 5      | 00:00:00:00:00:04 | MAC address of the node sending      |
| 6      | ND                | Log level (e.g., ND = network layer) |
| 7      | FWD               | Link direction (FWD = forward)       |
| 8      | 0                 | Packet ID (used to match SND → RCV)  |


#### Step 2: Find a "RCV" Event
Let's find the very first packet that was received. To call it, you'll use these lines of code
```
awk '$2=="RCV"' contrib/satellite/data/sims/example-tutorial/Simple/PacketTrace.log
```
The output will be
```
0.183623 RCV SAT 0 00:00:00:00:00:01 PHY FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.233616 RCV SAT 0 00:00:00:00:00:01 PHY FWD 1 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.283608 RCV SAT 0 00:00:00:00:00:01 PHY FWD 2 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.320153 RCV UT 6 00:00:00:00:00:05 PHY FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.320153 RCV UT 6 00:00:00:00:00:05 MAC FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.333632 RCV SAT 0 00:00:00:00:00:01 PHY FWD 3 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.370146 RCV UT 6 00:00:00:00:00:05 PHY FWD 1 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.370146 RCV UT 6 00:00:00:00:00:05 MAC FWD 1 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.383625 RCV SAT 0 00:00:00:00:00:01 PHY FWD 4 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.420138 RCV UT 6 00:00:00:00:00:05 PHY FWD 2 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.420138 RCV UT 6 00:00:00:00:00:05 MAC FWD 2 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.433617 RCV SAT 0 00:00:00:00:00:01 PHY FWD 5 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.470162 RCV UT 6 00:00:00:00:00:05 PHY FWD 3 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.470162 RCV UT 6 00:00:00:00:00:05 MAC FWD 3 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.48361 RCV SAT 0 00:00:00:00:00:01 PHY FWD 6 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
0.520155 RCV UT 6 00:00:00:00:00:05 PHY FWD 4 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
...
```
lets focus on line 4 of the output, and break it down. The line = `0.320153 RCV UT 6 00:00:00:00:00:05 PHY FWD 0 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff`
| Column | Value             | Meaning                                                   |
| ------ | ----------------- | --------------------------------------------------------- |
| 1      | 0.320153          | Time in seconds when the packet was received              |
| 2      | RCV               | Event type (`RCV` = received)                             |
| 3      | UT                | Node type (`UT` = user terminal, `SAT` = satellite, etc.) |
| 4      | 6                 | Node ID of the receiver                                   |
| 5      | 00:00:00:00:00:05 | MAC address of the receiver                               |
| 6      | PHY               | Log level / layer (ND, LLC, MAC, PHY…)                    |
| 7      | FWD               | Link direction (FWD = forward, RTN = return)              |
| 8      | 0                 | Packet ID (used to match SND → RCV)                       |
| 9      | 00:00:00:00:00:04 | Source MAC address (who sent the packet)                  |
| 10     | ff:ff:ff:ff:ff:ff | Destination MAC (broadcast or unicast)                    |

#### Step 3: Try to follow a single packet
Lets create a code that filters onlt PHY layer events, stores SND and RCV events by packet ID, pairs SND-> RCV and calculates one-way delay. This is the code
```
awk '
$6=="PHY" {
    if($2=="SND") snd[$8]=$1" "$3$4
    else if($2=="RCV") rcv[$8]=$1" "$3$4
}
END {
    printf "%-8s %-12s %-12s %-12s %-12s %-10s\n", "PktID", "SND_Time", "SND_Node", "RCV_Time", "RCV_Node", "Delay(s)"
    for (pkt in snd) {
        if(pkt in rcv){
            split(snd[pkt], s, " ")
            split(rcv[pkt], r, " ")
            delay = r[1]-s[1]
            printf "%-8s %-12s %-12s %-12s %-12s %-10s\n", pkt, s[1], s[2], r[1], r[2], delay
        }
    }
}' contrib/satellite/data/sims/example-tutorial/Simple/PacketTrace.log
```
The output will be:
```
PktID    SND_Time     SND_Node     RCV_Time     RCV_Node     Delay(s)
0        0.183623     SAT0         0.320153     UT6          0.13653
1        0.233616     SAT0         0.370146     UT6          0.13653
2        0.283608     SAT0         0.420138     UT6          0.13653
3        0.333632     SAT0         0.470162     UT6          0.13653
4        0.383625     SAT0         0.520155     UT6          0.13653
5        0.433617     SAT0         0.570147     UT6          0.13653
6        0.48361      SAT0         0.62014      UT6          0.13653
7        0.533633     SAT0         0.670164     UT6          0.136531
8        0.583626     SAT0         0.720156     UT6          0.13653
9        0.633619     SAT0         0.770149     UT6          0.13653
10       0.683611     SAT0         0.820141     UT6          0.13653
11       0.733635     SAT0         0.870165     UT6          0.13653
12       0.783627     SAT0         0.920158     UT6          0.136531
13       0.83362      SAT0         0.97015      UT6          0.13653
14       0.883613     SAT0         1.02014      UT6          0.136527
15       0.933636     SAT0         1.07017      UT6          0.136534
16       0.983629     SAT0         1.12016      UT6          0.136531
17       1.03362      SAT0         1.17015      UT6          0.13653
18       1.08361      SAT0         1.22014      UT6          0.13653
19       1.13364      SAT0         1.27017      UT6          0.13653
20       1.18363      SAT0         1.32016      UT6          0.13653
21       1.26639      SAT0         1.40067      GW2          0.13428
23       1.23362      SAT0         1.37015      UT6          0.13653
24       1.28362      SAT0         1.42015      UT6          0.13653
...
```
##### Lets break it down 
| Column       | Value    | Meaning                                                               |
| ------------ | -------- | --------------------------------------------------------------------- |
| **PktID**    | 0        | Packet ID = 0, unique identifier for this packet                      |
| **SND_Time** | 0.183623 | Time when the packet was **sent** (in seconds) from the sending node) |
| **SND_Node** | SAT0     | Node that **sent** the packet — SAT = satellite, 0 = node ID          |
| **RCV_Time** | 0.320153 | Time when the packet was **received** at the destination              |
| **RCV_Node** | UT6      | Node that **received** the packet — UT = user terminal, 6 = node ID   |
| **Delay(s)** | 0.13653  | One-way **propagation + processing delay** (RCV_Time − SND_Time)      |

##### What can we infer from the results
1. Delay consistency: Most delays are ~0.1365 s. This means the network path is stable and propagation + queuing delays are consistent.
2. Packet flow:
  - Packet 0: SAT0 → UT6
  - Packet 1: SAT0 → UT6
  - …so it looks like SAT is forwarding packets from GW to UT.
3. Sequential packets: PktID increases sequentially, showing the order in which packets were sent.
4. Intermediate hops: In this simplified log, the intermediate hops (GW → SAT) aren’t explicitly shown as separate rows in the E2E output because we only paired the first SND → first RCV. But the raw PacketTrace.log still contains PHY events for all hops.

##### Example interpretation of 3 packets
| PktID | From → To  | SND_Time | RCV_Time | Delay     |
| ----- | ---------- | -------- | -------- | --------- |
| 0     | SAT0 → UT6 | 0.183623 | 0.320153 | 0.13653 s |
| 1     | SAT0 → UT6 | 0.233616 | 0.370146 | 0.13653 s |
| 2     | SAT0 → UT6 | 0.283608 | 0.420138 | 0.13653 s |
- Packets are sent every ~0.05 s
- Delay is consistent across packets
- Shows a stable E2E transmission from SAT to UT

## Code-Level Tracing
These three files work together in a clear hierarchy: the NetDevice is the "front door" for the IP layer, the MAC is the "manager" with the queue, and the PHY is the "radio" that talks to the channel.

### 1. `satellite-net-device.cc` (The Front Door)
This file is the interface between the L3 (IP) layer and the L2 (MAC) layer.
- `Send()` Method:
  - What it does: This public method is called by the ns-3 IP layer when it has a packet to send.
  - Where is the log? This is the first place a packet is logged. Inside the Send() function, you will find a line that looks like this:
```
// This logs the packet as it arrives from L3, *before* it's queued.
m_macTxTrace (packet);
```
- What happens next? After logging, the `Send()` method doesn't transmit the packet directly. Instead, it hands the packet over to the `SatMac` to be queued for transmission. You'll see a call like:
```
m_mac->Enqueue (packet, dest);
```

### 2. `satellite-mac.cc` (The Traffic Manager)
This file contains the logic for when to send a packet. This is where the main queue lives.
- m_packetTrace: You won't find m_packetTrace here. That TracedCallback is typically in the NetDevice or Phy. Instead, you'll find a PacketQueue object, which has its own built-in trace sources.
- Queueing (Enqueue()):
  - What it does: This method is called by the SatNetDevice (as we saw above). It takes the packet and puts it into an internal buffer, usually a ns3::PacketQueue object.
  - Where is the log? The PacketQueue itself fires a trace. You don't see the line in satellite-mac.cc, but when it calls m_queue->Enqueue(packet), the queue object automatically fires its "Enqueue" trace source, which a logger can connect to.
- Dequeuing (Dequeue() or a scheduler function):
  - What it does: When the MAC's scheduling logic (e.g., in a DVB-S2X scheduler) decides it's time to transmit, it pulls the next packet out of the queue.
  - Where is the log? This is the counterpart to queueing. When the code calls m_queue->Dequeue(), the queue object automatically fires its "Dequeue" trace source.
  - What happens next? After dequeuing a packet, the MAC tells the PHY to send it:
    ```
    m_phy->Send (packet); // or m_phy->StartTx(packet)
    ```


### 3. `satellite-phy.cc` (The Radio)
This file models the physical radio. It's responsible for the actual transmission and reception events that you see in the log.
- `Send()` or `StartTx()` Method:
  - What it does: This is called by the `SatMac` when a packet is ready for transmission.
  - Where is the log? This is where the `PacketTrace.log` events are officially created.
    - Just before starting the transmission, it fires the SND trace:
    ```
    // This is the SND (enqueue) or RCV (dequeue) in the log.
    m_packetTrace (packet);
    ```
    - It then calculates the transmission duration and schedules an "end of transmission" event
- `Receive()` or `StartRx()` Method:
  -  What it does: This method is called by the `SatelliteChannel` when a signal arrives at this node.
  -  Where is the log? The PHY checks if the packet's power is above the receiving threshold and not corrupted. If the packet is received successfully, it fires the RCV trace:
```
// This is the "r" (receive) in the log.
m_packetTrace (packet);
```
  -  What happens next? The PHY passes the successfully received packet up to the `SatMac`, which in turn passes it up to the `SatNetDevice` to be sent to the IP layer.

# Experiment & Report: Create my own minimal simulation
The code is
```
#include "ns3/applications-module.h"
#include "ns3/config-store-module.h"
#include "ns3/core-module.h"
#include "ns3/internet-module.h"
#include "ns3/network-module.h"
#include "ns3/traffic-module.h"

// Note: This simulation requires the ns-3 satellite module.

using namespace ns3;

/**
 * \file minimal-sat.cc
 * \ingroup satellite
 *
 * \brief A minimal satellite example with 1 GW, 1 SAT, 1 UT.
 *
 * This simulation starts from the sat-cbr-example and modifies it
 * to ensure traffic only flows between a single Gateway User node
 * and a single User Terminal node, regardless of the scenario
 * ('simple', 'larger', 'full').
 *
 * It does this by manually installing Cbr and PacketSink applications
 * on gwUsers.Get(0) and utUsers.Get(0) instead of using the
 * SatTrafficHelper which targets all nodes.
 *
 */

NS_LOG_COMPONENT_DEFINE("minimal-sat");

int
main(int argc, char* argv[])
{
    uint32_t beamIdInFullScenario = 10;
    uint32_t packetSize = 512;
    std::string interval = "1s";
    std::string scenario = "simple";
    SatHelper::PreDefinedScenario_t satScenario = SatHelper::SIMPLE;
    double simulationTime = 12.0;

    /// Set simulation output details
    Config::SetDefault("ns3::SatEnvVariables::EnableSimulationOutputOverwrite", BooleanValue(true));

    /// Enable packet trace
    Config::SetDefault("ns3::SatHelper::PacketTraceEnabled", BooleanValue(true));
    // Use a tag for our new simulation name
    Ptr<SimulationHelper> simulationHelper = CreateObject<SimulationHelper>("minimal-sat");

    // read command line parameters given by user
    CommandLine cmd;
    cmd.AddValue("beamIdInFullScenario",
                 "Id where Sending/Receiving UT is selected in FULL scenario. (used only when "
                 "scenario is full) ",
                 beamIdInFullScenario);
    cmd.AddValue("packetSize", "Size of constant packet (bytes)", packetSize);
    cmd.AddValue("interval", "Interval to sent packets in seconds, (e.g. (1s)", interval);
    cmd.AddValue("scenario", "Test scenario to use. (simple, larger or full", scenario);
    cmd.AddValue("simulationTime", "Simulation time in seconds", simulationTime);
    simulationHelper->AddDefaultUiArguments(cmd);
    cmd.Parse(argc, argv);

    if (scenario == "larger")
    {
        satScenario = SatHelper::LARGER;
    }
    else if (scenario == "full")
    {
        satScenario = SatHelper::FULL;
    }
    // Set tag, if output path is not explicitly defined
    simulationHelper->SetOutputTag(scenario);

    simulationHelper->SetSimulationTime(Seconds(simulationTime));

    // Set beam ID
    std::stringstream beamsEnabled;
    beamsEnabled << beamIdInFullScenario;
    simulationHelper->SetBeams(beamsEnabled.str());

    // enable info logs
    LogComponentEnable("CbrApplication", LOG_LEVEL_INFO);
    LogComponentEnable("PacketSink", LOG_LEVEL_INFO);
    LogComponentEnable("minimal-sat", LOG_LEVEL_INFO);

    simulationHelper->LoadScenario("geo-33E");

    // Creating the reference system.
    Ptr<SatHelper> helper = simulationHelper->CreateSatScenario(satScenario);

    // --- MODIFICATION: Manual Application Setup ---
    //
    // We will manually configure applications for *all* scenarios
    // to ensure we only use 1 GW user and 1 UT user.
    //
    // First, get the containers for GW users and UT users.
    // The logic is slightly different for 'full' vs 'simple'/'larger'.

    uint16_t port = 9;
    NodeContainer gwUsers;
    NodeContainer utUsers;

    if (scenario == "full")
    {
        // 'full' scenario logic for finding a specific UT user
        NS_LOG_INFO("Using FULL scenario logic to find UT user from beam " << beamIdInFullScenario);
        NodeContainer uts = helper->GetBeamHelper()->GetUtNodes(0, beamIdInFullScenario);
        if (uts.GetN() == 0)
        {
            NS_LOG_ERROR("No UTs found in beam " << beamIdInFullScenario);
            return 1;
        }
        utUsers = Singleton<SatTopology>::Get()->GetUtUserNodes(uts.Get(0));
        gwUsers = Singleton<SatTopology>::Get()->GetGwUserNodes();
    }
    else
    {
        // 'simple' or 'larger' scenario.
        // We get all GW and UT users, and will just use the first one.
        NS_LOG_INFO("Using " << scenario << " scenario logic, will use first available GW/UT users.");
        gwUsers = Singleton<SatTopology>::Get()->GetGwUserNodes();
        utUsers = Singleton<SatTopology>::Get()->GetUtUserNodes();
    }

    // Check if we actually got the nodes
    if (gwUsers.GetN() == 0)
    {
        NS_LOG_ERROR("No Gateway Users found in the topology!");
        return 1;
    }
    if (utUsers.GetN() == 0)
    {
        NS_LOG_ERROR("No User Terminal Users found in the topology!");
        return 1;
    }

    NS_LOG_INFO("Topology setup complete. Found " << gwUsers.GetN() << " GW User(s) and "
                                                 << utUsers.GetN() << " UT User(s).");
    NS_LOG_INFO("Installing applications on gwUsers.Get(0) and utUsers.Get(0).");

    // Now, manually install applications on gwUsers.Get(0) and utUsers.Get(0)

    // --- FWD LINK (GW -> UT) ---
    // 1. Setup Sink on UT User (to receive from GW)
    PacketSinkHelper utSinkHelper("ns3::UdpSocketFactory",
                                  InetSocketAddress(helper->GetUserAddress(utUsers.Get(0)), port));
    ApplicationContainer utSink = utSinkHelper.Install(utUsers.Get(0));
    utSink.Start(Seconds(1.0));
    utSink.Stop(Seconds(10.0));

    // 2. Setup CBR on GW User (to send to UT)
    CbrHelper gwCbrHelper("ns3::UdpSocketFactory",
                          InetSocketAddress(helper->GetUserAddress(utUsers.Get(0)), port));
    gwCbrHelper.SetAttribute("Interval", StringValue(interval));
    gwCbrHelper.SetAttribute("PacketSize", UintegerValue(packetSize));

    ApplicationContainer gwCbr = gwCbrHelper.Install(gwUsers.Get(0));
    gwCbr.Start(Seconds(1.0)); // FWD link traffic
    gwCbr.Stop(Seconds(2.1));

    // --- RTN LINK (UT -> GW) ---
    // 3. Setup Sink on GW User (to receive from UT)
    PacketSinkHelper gwSinkHelper("ns3::UdpSocketFactory",
                                  InetSocketAddress(helper->GetUserAddress(gwUsers.Get(0)), port));
    ApplicationContainer gwSink = gwSinkHelper.Install(gwUsers.Get(0));
    gwSink.Start(Seconds(1.0));
    gwSink.Stop(Seconds(10.0));

    // 4. Setup CBR on UT User (to send to GW)
    CbrHelper utCbrHelper("ns3::UdpSocketFactory",
                          InetSocketAddress(helper->GetUserAddress(gwUsers.Get(0)), port));
    utCbrHelper.SetAttribute("Interval", StringValue(interval));
    utCbrHelper.SetAttribute("PacketSize", UintegerValue(packetSize));

    ApplicationContainer utCbr = utCbrHelper.Install(utUsers.Get(0));
    utCbr.Start(Seconds(7.0)); // RTN link traffic
    utCbr.Stop(Seconds(9.1));

    // --- End of Application Setup ---

    NS_LOG_INFO("--- minimal-sat simulation ---");
    NS_LOG_INFO("  Scenario used: " << scenario);
    if (scenario == "full")
    {
        NS_LOG_INFO("  UT used in full scenario from beam: " << beamIdInFullScenario);
    }
    NS_LOG_INFO("  PacketSize: " << packetSize);
    NS_LOG_INFO("  Interval: " << interval);
    NS_LOG_INFO("  Simulation Time: " << simulationTime);
    NS_LOG_INFO("  ");

    simulationHelper->RunSimulation();

    return 0;
}
```
then, run the simulation
```
./ns3 run my-simulations/minimal-sat
```

## Observe Output
### Locate log files.
run this to find them 
```
find . -type d -name "sims" | grep satellite
```
list simulation runs
```
geemajor@joy:~/workspace/bake/source/ns-3.43$ cd contrib/satellite/data/sims/example-cbr/simple
geemajor@joy:~/workspace/bake/source/ns-3.43/contrib/satellite/data/sims/example-cbr/simple$ ls
```
output
```
PacketTrace.log                                      stat-per-gw-rtn-feeder-mac-throughput-scatter-2.txt
stat-global-rtn-app-throughput-scalar.txt            stat-per-gw-rtn-user-mac-throughput-scalar.txt
stat-global-rtn-app-throughput-scatter-0.txt         stat-per-gw-rtn-user-mac-throughput-scatter-2.txt
stat-global-rtn-feeder-mac-throughput-scalar.txt     stat-per-ut-rtn-app-throughput-scalar.txt
stat-global-rtn-feeder-mac-throughput-scatter-0.txt  stat-per-ut-rtn-app-throughput-scatter-1.txt
stat-global-rtn-user-mac-throughput-scalar.txt       stat-per-ut-rtn-feeder-mac-throughput-scalar.txt
stat-global-rtn-user-mac-throughput-scatter-0.txt    stat-per-ut-rtn-feeder-mac-throughput-scatter-1.txt
stat-per-gw-rtn-app-throughput-scalar.txt            stat-per-ut-rtn-user-mac-throughput-scalar.txt
stat-per-gw-rtn-app-throughput-scatter-2.txt         stat-per-ut-rtn-user-mac-throughput-scatter-1.txt
stat-per-gw-rtn-feeder-mac-throughput-scalar.txt
```

### Use grep, head, and tail to see send/receive activity.
#### head
run
```
head -n 10 PacketTrace.log
```
output
```
geemajor@joy:~/workspace/bake/source/ns-3.43/contrib/satellite/data/sims/example-cbr/simple$ head -n 10 PacketTrace.log
COLUMN DESCRIPTIONS
-------------------
Time
Packet event (SND, RCV, DRP, ENQ)
Node type (UT, SAT, GW, NCC, TER)
Node id
MAC address
Log level (ND, LLC, MAC, PHY, CH)
Link direction (FWD, RTN)
Packet info (List of: Packet id, source MAC address, destination MAC address)
geemajor@joy:~/workspace/bake/source/ns-3.43/contrib/satellite/data/sims/example-cbr/simple$
```

#### tail
run
```
tail -n 10 PacketTrace.log
```
output
```
geemajor@joy:~/workspace/bake/source/ns-3.43/contrib/satellite/data/sims/example-cbr/simple$ tail -n 10 PacketTrace.log
10.9336 SND SAT 0 00:00:00:00:00:01 PHY FWD 254 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
10.95 SND GW 2 00:00:00:00:00:04 ND FWD 257
10.95 ENQ GW 2 00:00:00:00:00:04 LLC FWD 257 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
10.95 SND GW 2 00:00:00:00:00:04 LLC FWD 257 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
10.95 SND GW 2 00:00:00:00:00:04 MAC FWD 257 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
10.95 SND GW 2 00:00:00:00:00:04 PHY FWD 257 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
10.9701 RCV UT 6 00:00:00:00:00:05 PHY FWD 252 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
10.9701 RCV UT 6 00:00:00:00:00:05 MAC FWD 252 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
10.9836 RCV SAT 0 00:00:00:00:00:01 PHY FWD 255 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
10.9836 SND SAT 0 00:00:00:00:00:01 PHY FWD 255 00:00:00:00:00:04 ff:ff:ff:ff:ff:ff
```

### Confirm data is flowing end-to-end (GW → SAT → UT)
| Time (s)    | Event | Node           | Layer     | Direction | Meaning                                                       |
| ----------- | ----- | -------------- | --------- | --------- | ------------------------------------------------------------- |
| **10.95**   | `SND` | **GW (id 2)**  | `PHY FWD` | →         | Gateway **transmitted** packet ID 257 forward to satellite    |
| **10.9836** | `RCV` | **SAT (id 0)** | `PHY FWD` | ↑         | Satellite **received** the packet from GW                     |
| **10.9836** | `SND` | **SAT (id 0)** | `PHY FWD` | ↓         | Satellite **retransmitted** it toward the UT                  |
| **10.9701** | `RCV` | **UT (id 6)**  | `PHY FWD` | ↓         | UT (User Terminal) **received** the packet from the satellite |

✅ Confirmation: Data Flow Verified
We can confidently say the packets are traveling as expected: </br>
Gateway (GW) → Satellite (SAT) → User Terminal (UT) </br>
Each packet follows the correct PHY → MAC → LLC → ND layer transitions, confirming that:
- The satellite forward link is functioning,
- The log structure captures both uplink and downlink PHY activities, and
- End-to-end transmission delay (~20–30 ms difference) is consistent with LEO-like latency.

## Analyze & Visualize Results

### Summarize what happen in logs:
from the trace snippet:
```
10.95 SND GW 2 00:00:00:00:00:04 PHY FWD 257
10.9836 RCV SAT 0 00:00:00:00:00:01 PHY FWD 255
10.9836 SND SAT 0 00:00:00:00:00:01 PHY FWD 255
10.9701 RCV UT 6 00:00:00:00:00:05 PHY FWD 252
```
We can interpret:
| Time (s)    | Event | Node                   | Description                                        |
| ----------- | ----- | ---------------------- | -------------------------------------------------- |
| **10.95**   | `SND` | **Gateway (GW)**       | Packet transmitted to the satellite (forward link) |
| **10.9836** | `RCV` | **Satellite (SAT)**    | Packet received by satellite from GW               |
| **10.9836** | `SND` | **Satellite (SAT)**    | Satellite retransmitted the packet to UT           |
| **10.9701** | `RCV` | **User Terminal (UT)** | Packet received successfully by UT                 |

#### When the packet was sent
Approximate timing:
| Link                | Start   | End     | Delay (s)               | Description                                                                                      |
| ------------------- | ------- | ------- | ----------------------- | ------------------------------------------------------------------------------------------------ |
| GW → SAT            | 10.95   | 10.9836 | **0.0336 s (33.6 ms)**  | Uplink transmission delay                                                                        |
| SAT → UT            | 10.9836 | 10.9701 | **~0.0135 s (13.5 ms)** | Downlink transmission delay (approx., timestamps not strictly sequential due to event buffering) |
| **Total E2E Delay** |         |         | **≈ 47 ms**             | Within typical LEO GEO simulation latency (40–60 ms)                                             |

Jitter = difference between consecutive packet delays (we can compute this using Python later if you want to quantify stability).

#### A simple diagram of the data path
Here’s the logical data path derived from the logs:
````
+-------------+          +-----------------+          +---------------+
|  Gateway    |  (Tx) →  |   Satellite     |  (Fwd) → |  User Terminal|
| (GW Node 2) |----------| (SAT Node 0)    |----------| (UT Node 6)   |
+-------------+          +-----------------+          +---------------+
     PHY(FWD)                 PHY(FWD)                     PHY(FWD)
````

