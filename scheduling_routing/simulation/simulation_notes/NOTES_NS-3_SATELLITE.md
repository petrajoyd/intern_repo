# Study Note 3: NS-3 Satellite Module
The **`contrib/satellite`** module in `ns-3.43` provides tools and models to simulate **satellite communication systems**, including LEO/GEO orbits, beam scheduling, link adaptation, LoRaWAN integration, and physical/MAC-level channel behavior.
This module integrates both **DVB-S2X** and **LoRa-based** satellite links for flexible end-to-end testing.

## Directory Structure And Helper-Channel Interaction
### Directory Structure

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

#### **`model/` Directory**
This is the core of the module, containing the C++ classes that simulate the actual networking components.
- `satellite-net-device.cc / .h`: This is arguably the most important file here. It defines the `SatNetDevice` class, which represents the Layer 2 network interface card (NIC) for a satellite or ground station. It connects the node's IP stack to the satellite's physical layer (SatPhy).
- `satellite-phy.cc / .h`: Defines the `SatPhy` class. This represents the physical layer (Layer 1). It's responsible for the "transmission" and "reception" of signals, handling things like transmission power and modulation schemes. It doesn't know about packets, just signal energy.
- `satellite-mac.cc / .h`: Defines the `SatMac` class. This is the Medium Access Control sublayer that sits between the `SatNetDevice` and `SatPhy`. It decides when to transmit to avoid collisions on the shared satellite channel.
- `satellite-channel.cc / .h`: Implements the `SatelliteChannel`. Think of this as the shared "air" or "space" through which all the satellites and ground stations communicate. When one SatPhy transmits, it's the channel's job to deliver that signal to all other devices listening on the channel.
- `satellite-propagation-delay-model.cc / .h`: This is crucial for realism. It calculates the time it takes for a signal to travel between two points in space, which is significant for satellites. It uses the physical positions of the nodes to do this.
- `satellite-fading-models.cc / .h`: (Or similar names) These files model how the signal strength can be affected by atmospheric conditions like rain, clouds, etc.

#### **`helper/` Directory**
Helpers are designed to make the user's life easier. They wrap the complex model components into a simpler API for writing simulation scripts.
- `satellite-helper.cc / .h`: This is the main entry point for creating a satellite network. The SatelliteHelper class provides high-level functions like .Install() which will automatically create and configure the SatNetDevice, SatPhy, SatMac, and connect them to the SatelliteChannel for a given set of nodes. You'll use this in almost every satellite simulation script.

#### **`examples/` Directory**
These are complete, runnable simulation scripts that show how to use the helper and models. They are the best place to start learning.
- `sat-tutorial-example.cc`: Your primary learning tool. It's a basic script that sets up a simple satellite scenario. By reading and modifying this file, you can learn how to use the SatelliteHelper to build your own simulations.
- `sat-iot-example.cc`: A more complex example that demonstrates a specific use case (IoT). It shows how to integrate the satellite module with other parts of NS-3, like application layers and mobility models.

#### **`data/` Directory**
This directory isn't for code, but for data files that your simulations might need.
- TLE (Two-Line Element) files (`.tle`): This is the most common type of file you'll find here. TLE is a standard format for distributing orbital data for satellites. The NS-3 satellite position models can read these files to simulate realistic satellite movement based on actual orbital paths. For example, you might find a file like `leo-walker-star.tle` which defines an entire constellation.

### Key Components

| Component | Description |
|------------|--------------|
| **Physical Layer (PHY)** | Models path loss, fading (Rayleigh, Loo), and SINR-based reception. |
| **MAC Layer** | Implements scheduling (e.g., DAMA, CRDSA, time slicing) and frame management. |
| **NCC (Network Control Center)** | Handles capacity requests and beam management. |
| **Mobility Models** | Includes `satellite-sgp4-mobility-model` for orbit prediction. |
| **LoRa Integration** | Supports satellite–LoRa hybrid constellations. |
| **Logging System** | Creates trace files such as `PacketTrace.log` and `SimInfo.log` under `data/sims/`. |

### Example Simulation Workflow
#### **Simple Simulation**
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

#### Simulation With: Configuration (Channel And Scenario), Nodes Creation, and many more
##### Phase 1: Boilerplate and Configuration
This is the initial setup phase.
- Includes: The file starts with a series of `#include` statements. These import all the necessary libraries and modules from NS-3 that the script needs to function. The most important one for us is `#include "ns3/satellite-module.h"`.
- Logging: You'll see lines like `NS_LOG_COMPONENT_ENABLE ("UdpEchoClientApplication", LOG_LEVEL_INFO);`. This is for debugging. It tells the simulator to print out log messages from specific components (in this case, the `UdpEchoClientApplication`) so you can see what's happening during the run.
- Command Line Parser: This allows you to change simulation variables from the command line when you run the script, without having to recompile the code. It's a very handy feature for running experiments.
```
CommandLine cmd;
cmd.Parse (argc, argv);
```

##### Phase 2: Creating the Network Nodes
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

##### Phase 5: Assigning IP Addresses
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

##### Summary
In summary, the sat-tutorial-example.cc script follows a standard NS-3 pattern:
1. Create Nodes.
2. Use a Helper to install Devices and create a Channel.
3. Install the IP protocol stack.
4. Assign IP Addresses.
5. Install Applications to generate traffic.
6. Run the simulation. [Simulation Run](LINK)

### Satellite Module `satellite-helper.cc`, `satellite-channel-model.cc`
This document provides a structured overview of the `contrib/satellite` module in ns-3.43, focusing on how helpers connect PHY, MAC, and network layers. This section explores `satellite-helper.cc` and `satellite-channel-model.cc`, which form the glue logic between high-level scenario configuration and low-level model execution.

#### 1. The Role of Helpers
##### **a. What Are Helpers?**
In ns-3, *helpers* simplify simulation setup by wrapping complex model configurations into reusable interfaces. They bridge the gap between user-defined scenarios (in `examples/`) and core models (in `model/`).<br>
The key helper in the satellite module is `SatHelper`, located at:
```
contrib/satellite/helper/satellite-helper.cc
```
It works alongside other components like `SimulationHelper` to orchestrate node creation, device installation, and PHY/MAC configuration.

#### 2. `SatHelper` — The Core Builder
##### **Purpose**

`SatHelper` acts as the *central configuration and connection layer*. It abstracts away the complexity of linking User Terminals (UT), Gateways (GW), and Satellites (SAT) with their corresponding physical (PHY) and MAC layers.

##### **Key Responsibilities**

| Function                        | Description                                                                 |
| ------------------------------- | --------------------------------------------------------------------------- |
| `CreateSatScenario()`           | Builds a complete satellite topology (UTs, GWs, Satellites, NCC).           |
| `ConfigurePhyMac()`             | Instantiates and attaches PHY and MAC models to each node.                  |
| `InstallNetDevices()`           | Attaches `SatNetDevice` to nodes, enabling packet transmission.             |
| `ConfigurePropagationChannel()` | Links each PHY to a `SatChannel` and sets up delay/fading models.           |
| `SetDefaultParameters()`        | Loads parameters from XML or environment variables (`SatEnvVariables`).     |
| `ConnectTraces()`               | Binds events (like SND/RCV) to trace sinks for logging (`PacketTrace.log`). |

##### **Layer Connection Flow**

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

##### **Example Internal Flow (simplified)**

```cpp
Ptr<Node> ut = CreateObject<Node>();
Ptr<SatNetDevice> utDevice = helper->InstallUtDevice(ut);
ut->AddDevice(utDevice);
```
* The helper installs a UT device that includes PHY/MAC models and connects it to the appropriate channel.

#### 3. `SatHelper` — The Core Builder
##### **Location**
```
contrib/satellite/model/satellite-channel.cc
contrib/satellite/model/satellite-propagation-delay-model.cc
```
##### **Purpose**

The `SatChannelModel` simulates signal propagation between nodes, including free-space loss, fading, and interference. It is dynamically configured by `SatHelper` to reflect the chosen scenario and environmental conditions.

##### **Key Components**

| File                                   | Role                                                                          |
| -------------------------------------- | ----------------------------------------------------------------------------- |
| `satellite-channel.cc`                 | Defines the communication path and packet flow between devices.               |
| `satellite-propagation-delay-model.cc` | Calculates delay based on satellite geometry (distance and orbital position). |
| `satellite-fading-models`              | Adds time-varying signal degradation (Rayleigh, Loo, or external traces).     |

##### **Propagation Flow**

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

##### **Simplified Call Stack**

```
SimulationHelper::RunSimulation()
  └── SatHelper::CreateSatScenario()
        ├── SatHelper::ConfigurePhyMac()
        ├── SatHelper::ConfigurePropagationChannel()
        └── SatChannel::Transmit()
```

---


## Device & PHY-MAC Exploration
### Analyze SatNetDevice
```
contrib/satellite/model/satellite-net-device.cc  
contrib/satellite/model/satellite-net-device.h
```
`SatNetDevice` is the **satellite-specific implementation** of ns-3’s `NetDevice` class.  
It represents the “network card” that sits between the ns-3 node (UT, GW, or SAT) and the underlying MAC/PHY/channel layers.

#### Core Responsibilities

| Function | Description |
|-----------|-------------|
| `Send()` / `SendFrom()` | Handles outgoing packets, attaches statistics tags, and pushes data into the LLC queue. |
| `Receive()` | Handles incoming packets, computes delay/jitter, and delivers them up to the IP layer. |
| `SetPhy()` / `SetMac()` / `SetLlc()` | Connects physical, MAC, and logical link layers to the device. |
| `SetNodeInfo()` | Links this device to its node metadata (ID, type, MAC address). |
| `ToggleState()` | Enables or disables the MAC operation. |
| `SetReceiveCallback()` | Registers callback for upper-layer delivery. |
| `DoDispose()` | Cleans up attached components and releases pointers. |

#### Key Data Members

| Variable | Type | Role |
|-----------|------|------|
| `m_phy` | `Ptr<SatPhy>` | Connects to the physical layer (handles modulation, radio link). |
| `m_mac` | `Ptr<SatMac>` | Medium Access Control (queues, retransmissions, scheduling). |
| `m_llc` | `Ptr<SatLlc>` | Logical Link Control (frame assembly and buffer management). |
| `m_node` | `Ptr<Node>` | The owning ns-3 node (UT, GW, or SAT). |
| `m_classifier` | `Ptr<SatPacketClassifier>` | Determines which flow each packet belongs to. |
| `m_receiveErrorModel` | `Ptr<ErrorModel>` | Models random losses or interference. |
| `m_packetTrace`, `m_txTrace`, `m_rxTrace` | `TracedCallback` | Used for packet-level event logging. |

#### Installation Process via Helper
##### **a. Entry Point**
Device installation begins in `satellite-helper.cc` using methods such as:
```cpp
Ptr<SatNetDevice> dev = CreateObject<SatNetDevice>();
node->AddDevice(dev);
```
The helper then configures the internal components (PHY, MAC, LLC) for the new device.

##### **b. Configuration Steps**
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

#### Data Flow

##### **Sending Flow**
```cpp
Send() → Add tags (address, timestamps)
       → Trace: PACKET_SENT
       → Classify flow (SatPacketClassifier)
       → Enque() packet into SatLlc
       → MAC → PHY → Channel
```

##### **Receiving Flow**
```cpp
Receive() → Log PACKET_RECV
          → Extract SatAddressTag, SatDevTimeTag
          → Compute delay & jitter
          → Trace results
          → Deliver packet to IP layer via callback
```

#### Connection Flow Between Layers

##### **a. Internal Wiring**
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

##### **b. How Helpers Attach Components**
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

### PHY and MAC Layer Analysis
This document summarizest how `Transmi()` and `Receive()` functions are handled in the **PHY** and **MAC** layers of the ns-3 satellite module, and how signals are propagated through the **channel** and logged in simulation outputs.

#### File Overview

| File               | Role                                                                                                                       |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| `satellite-mac.cc` | Implements medium access control (MAC) — responsible for framing, scheduling, and coordinating transmissions.              |
| `satellite-phy.cc` | Implements the physical layer (PHY) — responsible for signal transmission, propagation through the channel, and reception. |

#### Data Flow Overview

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

#### MAC Layer — `Transmit()` and `Receive()`

##### a. Transmit()

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

##### b. Receive()

* When the PHY layer completes a successful reception, it calls:

  ```cpp
  m_mac->Receive(packet);
  ```
* MAC then performs error checking, extracts headers, and determines if the packet should be delivered upward (LLC or higher layers).

**Responsibilities:**

* Check destination address
* Filter invalid/corrupted frames
* Pass valid packets to LLC or NetDevice

#### PHY Layer — `Transmit()` and `Receive()`

##### a. Transmit()

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

##### b. Receive()

* Triggered when the **channel** delivers a packet to this PHY.
* The PHY calculates SNR, checks errors, and if valid, forwards to MAC:

  ```cpp
  m_mac->Receive(packet);
  ```

**Responsibilities:**

* Apply propagation delay and noise models
* Handle packet decoding and error detection
* Log the event: `RCV PHY`

#### Channel Interaction — `SatChannel`

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

### Tracing and Statistics
The helper also enables traces for debugging and analysis.  
In `SatNetDevice`, these are defined as trace sources like `m_packetTrace`, `m_txTrace`, and `m_rxTrace`, which record key simulation events (SEND, RECV, DROP).

#### How Logs are Generated
Logs like `PacketTrace.log` are not generated automatically. They are the result of connecting a trace sink to a trace source.
- Trace Source: An event point within a simulation object. For example, the PHY layer object has a trace source named `PhyTxBegin` that fires whenever a packet transmission starts. Other sources include `PhyRxEnd`, `MacTx`, etc.
- Trace Sink: A function or method that we write to handle the data from the trace source. In the `sat-cbr-example` [Jump To Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/examples_sims_run.md#31-inspecting-the-log-header-and-content), the sink is a function that formats the information (time, node ID, event type, packet details) into the string format you see in `PacketTrace.log` and writes it to a file.
- Connection: The simulation script must explicitly connect a sink to a source. You can connect many sinks to one source or one sink to many sources.
`SimInfo.log` is generated differently. It's typically a straightforward file I/O operation where the simulation script writes simulation parameters (e.g., number of nodes, simulation time) to a text file at the beginning of the run.

#### Functions Responsible for Writing Trace Data
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

#### Locating the Trace Connection Code
The logic for connecting these sources and sinks is often found in helper classes or directly within the main simulation script. For the satellite module:
- `contrib/satellite/helper/satellite-helper.cc`: This is a primary candidate. Helper classes are designed to simplify configuration. Look for methods like `EnablePcap` or `EnableAscii` which are standard ns-3 patterns for setting up tracing. Inside these methods, you will find calls to `TraceConnect` or `TraceConnectWithoutContext`.
- `contrib/satellite/examples/*.cc`: The example script itself is the most direct place to find the setup. The `main` function of `sat-cbr-example.cc` will contain the code that creates file streams `(std::ofstream)` for the logs and then calls the helper methods or `TraceConnect` functions to link the simulation events to these files.
- `contrib/satellite/model/satellite-log.cc`: This file is less likely to contain the `TraceConnect` calls but may define logging-specific components, custom formatters, or enums (like `SND`, `RCV`) that are used by the trace sinks to create the log file content.


#### Trace & Logging Hooks

| Trace Name | Event Description | Output Location |
|-------------|-------------------|-----------------|
| `PacketTrace` | Every send/receive event | `PacketTrace.log` |
| `RxDelay` | Delay of each packet | Derived from tags |
| `RxJitter` | Variation in delay (jitter) | Derived from tags |
| `Tx`, `Rx` | Transmit and receive events | Console or log sinks |


Example output file:  
`contrib/satellite/data/sims/example-cbr/simple/PacketTrace.log`

Each SatNetDevice is initialized with dedicated PHY/MAC layers and linked to a shared channel depending on the communication direction (forward/return).

#### Relationships with Other Components

| Component | Interaction |
|------------|--------------|
| `SatHelper` | Creates and connects `SatNetDevice` objects to nodes. |
| `SatPhy` | Attached via `SetPhy()`, handles actual transmission. |
| `SatMac` | Attached via `SetMac()`, manages channel access and scheduling. |
| `SatChannel` | Not stored directly; connected through PHY layer. |
| `SatLlc` | Buffers and organizes packets before MAC. |

#### Logging and Trace Output

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

#### Summary of Interactions

| Layer    | Function                     | Called By    | Passes To    | Log Event |
| -------- | ---------------------------- | ------------ | ------------ | --------- |
| MAC      | Prepares frame, schedules TX | Higher layer | PHY          | `SND MAC` |
| PHY      | Modulates, sends via channel | MAC          | Channel      | `SND PHY` |
| Channel  | Applies delay, attenuation   | PHY          | Receiver PHY | —         |
| PHY (RX) | Decodes, validates           | Channel      | MAC          | `RCV PHY` |
| MAC (RX) | Forwards to LLC              | PHY          | NetDevice    | `RCV MAC` |

## Tracing and Logging System
### 1. NS-3 Satellite Trace System 
In NS-3, the **trace system** allows you to **record simulation events** — such as packet transmissions, receptions, delays, or queue changes — for later analysis.  
In the `satellite` module, this system helps log **satellite-specific events** like:
- Packets entering or leaving a node (uplink/downlink)
- Delays in satellite links
- Throughput and error metrics

The two main files handling this are:
- `satellite-log.cc`: general-purpose logging support
- `satellite-packet-trace.cc`: detailed packet tracing

### `satellite-log.cc`
#### Purpose
Handles **generic logging setup** for the satellite module.  
It sets up the trace sources that can be connected to output files or analysis tools.

#### Key Concept: Trace Registration with `AddTraceSource`
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

### `satellite-packet-trace.cc`
#### Purpose
Implements **actual tracing logic** — connecting simulation events (like sending/receiving packets) to file outputs.

Inside this file, you’ll find:
- Declarations of trace variables (like `m_packetTrace`)
- Connections between simulation objects and output streams
- Formatting functions for how data is written to `.log` or `.csv` files

#### How the Trace Works
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

### `m_packetTrace` and Related Callbacks
#### What It Is
m_packetTrace` is a **TracedCallback**, a core NS-3 mechanism for event notification.

#### How It Connects
- **Declared** in the C++ class (e.g., `SatellitePacketTrace`)
- **Registered** via `AddTraceSource`
- **Connected** to an external **trace sink** (file writer, plotter, etc.)
- **Activated** during simulation (when an event fires)

#### The Flow in One Line:
```
Simulation Event → m_packetTrace() fires → Callback function runs → Output logged to file
```

### Typical Example (Simplified)
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

### Summary
| Component | File | Function | Key Concept |
|------------|------|-----------|--------------|
| `AddTraceSource` | `satellite-log.cc` | Registers a trace variable | Makes it visible to NS-3 trace system |
| `m_packetTrace` | Both | Holds callback chain | Fires when event occurs |
| `PacketTraceSink()` | `satellite-packet-trace.cc` | Callback function | Writes to file |
| `Config::Connect` | Main script/helper | Connects source to sink | Links trace to output |

### 2. NS-3 Event Trigger Analysis
This note explains how events such as `SND`, `RCV`, `ENQ`, and `DRP` are emitted in the NS-3 Satellite Contrib Module and how they link across network layers (NetDevice, MAC, PHY).

### Tracking Packet Event Emission (SND, RCV, ENQ, DRP)
In the NS-3 satellite module, these events represent specific moments in the packet’s life cycle:

| Event | Meaning | Typical Source File | Typical Function |
|--------|----------|--------------------|------------------|
| **SND (Send)** | Triggered when a packet is passed for transmission (e.g., from upper layer to MAC or PHY). | `satellite-log.cc`, `satellite-net-device.cc` | `SatNetDevice::Send()` |
| **RCV (Receive)** | Triggered when a packet is received by a device or layer. | `satellite-packet-trace.cc`, `satellite-net-device.cc` | `SatNetDevice::Receive()` |
| **ENQ (Enqueue)** | Marks packet insertion into a transmission queue (often at MAC layer). | `satellite-mac.cc`, `satellite-queue.cc` | `Enqueue()` |
| **DRP (Drop)** | Fired when a packet is dropped (due to queue overflow, link errors, etc.). | `satellite-mac.cc`, `satellite-phy.cc` | `Drop()` |

Each of these events is traced using `AddTraceSource()` to allow logging and later analysis via the tracing system.

### How `SatNetDevice::Send()` and `Receive()` Link MAC and PHY Layers
The **SatNetDevice** acts as a bridge between higher layers (LLC / IP) and lower layers (MAC / PHY).

#### a. `SatNetDevice::Send()`
- Invoked when upper layers request transmission.
- Performs checks (link status, queue availability).
- Creates metadata (`Ptr<Packet>`, headers, tags).
- Passes packet down to the **MAC layer** via `m_mac->Enqueue(packet)`.
- Triggers `SND` and possibly `ENQ` traces.

#### b. `SatNetDevice::Receive()`
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

### Log Level Mapping (ND, LLC, MAC, PHY, CH)
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

### Summary
- `SND`, `RCV`, `ENQ`, `DRP` represent lifecycle checkpoints of a packet.
- `SatNetDevice` ties together MAC and PHY layers through well-defined callbacks.
- Logging levels (ND, LLC, MAC, PHY, CH) help isolate debugging to specific layers.
- Trace sources (`AddTraceSource`) allow these events to be recorded and analyzed.

### 3. NS-3 Practical Test & Rerun
This test helps you **observe how trace verbosity affects the simulation output**. You’ll modify trace settings, run a standard example (like `sat-cbr-example`), and compare the logs.

### Modify Trace Verbosity (Disable PHY Logs)
#### Where to Edit
In the satellite module, verbosity is usually managed via `NS_LOG_COMPONENT_DEFINE` and `NS_LOG_INFO` statements within files like:
- `satellite-phy.cc`
- `satellite-mac.cc`
- `satellite-net-device.cc`
and/or via configuration parameters in the example script (`sat-cbr-example.cc`).

#### Option 1 — Disable PHY Traces in Code
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

#### Option 2 — Disable PHY Traces Using NS_LOG Environment Variable
Without touching the code, you can disable PHY logs by changing the environment variable before running:
```bash
export NS_LOG="*=level_info|prefix_func|prefix_time"
export NS_LOG="SatNetDevice:info|SatMac:info"   # excludes PHY logs
```
*This method is cleaner for testing since you don’t have to recompile.*

### Rerun the Example
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

### Compare Before & After Logs
Now, open or diff your results:

#### Before (Full Verbosity)
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

#### After (PHY Disabled)
You’ll notice entries like:
```
[ND] SND pkt=123 size=1024
[MAC] ENQ pkt=123 queue=0
[CH] PROP pkt=123 delay=240ms
[MAC] RCV pkt=123
[ND] DELIVER pkt=123
```
No `[PHY]` entries should appear — confirming the trace was suppressed.

### Analyze Changes
| Aspect | Before | After (PHY Disabled) | Interpretation |
|--------|---------|---------------------|----------------|
| Log file size | Larger | Smaller | Fewer lines = reduced verbosity |
| PHY event entries | Present | Missing | Trace suppression successful |
| Packet count | Same | Same | Core logic unaffected |
| Timing | Same (minor variations possible) | Same | Only tracing changed, not simulation behavior |

### Optional: Restore or Fine-Tune Verbosity
After verification, re-enable the PHY trace to restore full logging:

```bash
Config::SetDefault("ns3::SatHelper::EnablePhyTrace", BooleanValue(true));
```

Or increase verbosity globally:
```bash
export NS_LOG="*=level_all|prefix_time|prefix_node"
```

### Summary
| Step | Action | Purpose |
|------|--------|----------|
| 1 | Disable PHY trace (code or env variable) | Limit verbosity |
| 2 | Rerun example | Generate fresh `PacketTrace.log` |
| 3 | Compare before/after logs | Verify trace filtering |
| 4 | Analyze file differences | Confirm only PHY logs removed |
| 5 | Restore trace | Return to full debugging mode |

## Channel & Propagation Mastery
### ns-3 Satellite Channel
This document breaks down how the ns-3 satellite channel model works by examining `satellite-channel.cc` and `satellite-propagation-delay-model.cc`. The primary goal is to understand how a transmitted signal is received by the channel and delivered to the destination's physical layer (PHY).

### The Role of the Channel
In ns-3, a **Channel** object represents the physical medium connecting network devices (`NetDevice`). Its main job is to model what happens to a signal as it travels from a transmitter to a receiver. For a satellite, the most critical factor is the **propagation delay**—the time it takes for the signal to travel through space.

The two files work together to accomplish this:
-   `satellite-channel.cc`: Acts as the "manager." It takes a packet from a sender and is responsible for delivering it to all intended receivers.
-   `satellite-propagation-delay-model.cc`: Acts as the "calculator." It provides the channel with the exact time delay for a signal traveling between any two points.

### 1. `contrib/satellite/model/satellite-channel.cc`
This file implements the `SatelliteChannel` class. Think of this class as the empty space through which the radio waves travel.

#### How It Receives and Delivers a Signal
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

### 1. `contrib/satellite/model/satellite-propagation-delay-model.cc`
This file implements the `SatellitePropagationDelayModel` class. Its one and only job is to calculate the time it takes for a signal to travel between two points.

#### How It Calculates Delay
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

### The Complete Workflow 

1.  A **SatelliteNetDevice** transmits a `Packet`, calling **`SatelliteChannel::Send()`**.
2.  The **`SatelliteChannel`** loops through all other connected devices.
3.  For each receiver, it calls **`SatellitePropagationDelayModel::GetDelay()`**.
4.  The **`SatellitePropagationDelayModel`** calculates the distance, divides by the speed of light, and returns the delay time.
5.  The **`SatelliteChannel`** receives this delay and uses **`Simulator::Schedule()`** to create a future arrival event.
6.  When the simulation clock reaches that future time, the scheduler calls **`SatellitePhy::StartRx()`** on the receiving device's PHY layer, delivering the packet.

This design perfectly separates responsibilities: the **Channel** manages delivery, and the **PropagationDelayModel** performs the physics-based calculation.

<img width="1744" height="141" alt="satellite_workflow_pretty" src="https://github.com/user-attachments/assets/b3c6b5a4-66ed-4128-8263-53e1cd1d9eb7" />

---

### ns-3 Satellite Propagation Delay & Fading Models
This document analyzes how ns-3 models the crucial aspects of propagation delay and signal fading in satellite channels, focusing on the underlying geometry and the modular, pluggable architecture for different fading models. 

#### 1. Propagation Delay Computed from Satellite Geometry
The delay calculation is direct, accurate, and based purely on the physics of wave propagation. It's handled by the `SatellitePropagationDelayModel`. The key function is **`GetDelay(Ptr<MobilityModel> a, Ptr<MobilityModel> b)`** located in `contrib/satellite/model/satellite-propagation-delay-model.cc`.

##### How it Works
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

### 2. Fading Models and Their Pluggable Architecture
While delay determines *when* a signal arrives, **fading** determines its **strength**. Fading models simulate signal degradation from atmospheric effects (rain), shadowing (buildings), and multipath reflections.</br>
ns-3 uses a highly modular, **pluggable architecture**. The `SatelliteChannel` itself has no fading logic. Instead, it holds a pointer to a generic `SatelliteFadingModel` object. This allows you to easily switch between different fading models.</br>
The base class in `satellite-fading-model.h` defines a standard interface, with the most important function being one that calculates signal loss, conceptually like this:
```cpp
// A conceptual representation
virtual double GetLossDb (Ptr<MobilityModel> a, Ptr<MobilityModel> b) = 0;
```
The `SatellitePhy` (physical layer) uses this loss value to determine if the received signal is strong enough for successful decoding (i.e., if the Signal-to-Noise Ratio is sufficient).

### 3. Exploring Different Fading Models
The power of this design is evident in the variety of available models, each suited for different scenarios.

#### How They Are "Plugged In"
In your ns-3 simulation script, you instantiate the model you want and attach it to the channel.
```cpp
// 1. Create the channel
Ptr<SatelliteChannel> channel = CreateObject<SatelliteChannel> ();

// 2. Create a specific fading model (e.g., Loo)
Ptr<LooFadingModel> fadingModel = CreateObject<LooFadingModel> ();

// 3. "Plug it in" to the channel
channel->SetFadingModel (fadingModel);
```

#### Common Satellite Fading Models
- **Rayleigh Fading**: Models scenarios with **no dominant line-of-sight (LoS) path**, where the signal is received only through reflections. This is less common for satellites but can simulate severe blockage.
- Loo Fading Model: A highly realistic and widely used model for **Land Mobile Satellite (LMS)** links. It's a composite model that includes:
  - A **shadowed Line-of-Sight (LoS) signal**, modeled with a **log-normal distribution** to represent slow fading from large obstacles (hills, large buildings).
  - **Multipath signals**, modeled with a **Rayleigh distribution** to represent fast fading from many smaller reflections. This combination is perfect for simulating a vehicle moving through a mixed urban/suburban environment.
  - **Markov Chain Models**: These are abstract, state-based models. They define channel "states" (e.g., "Good" and "Bad") and the probabilities of transitioning between them. For example, a link could be in a "Good" state (clear LoS) and transition to a "Bad" state (blocked by a bridge) for a short time. They are computationally efficient and great for analyzing how higher-level protocols perform over channels with intermittent connectivity.

---

### Tracing the ns-3 PHY-to-Channel Signal Flow
This guide details a packet's journey from the physical layer (PHY) of a satellite device, through the channel, and to the PHY of a receiver. Understanding this flow is key to debugging and verifying ns-3 wireless simulations.

#### The Signal Flow: From Transmission to Reception
The entire process is a coordinated handoff between the **PHY layer**, the **Channel**, and the **ns-3 Scheduler**. The Scheduler acts as the simulation's master clock, triggering events at precise future times.

##### 1. The Call to `SatPhy::Transmit()`
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

##### 2. The Packet's Journey Through the Channel

Now, the `SatelliteChannel::Send()` method takes over. For **every other device** connected to it, the channel performs these steps:

1.  It asks the **propagation delay model** for the signal's travel time between the sender and this specific receiver.
2.  It calculates the **total delay**: `Total Delay = Transmission Time + Propagation Delay`.
3.  It uses the **master scheduler** to create a future event. This event tells the simulator: "In `Total Delay` seconds, call the `StartRx` function on the receiver's PHY and deliver this packet."

This scheduling is the most critical part. The packet isn't delivered instantly; it's queued to arrive at the exact, physically correct moment.

#### Confirming Events with `PacketTrace.log`
This entire sequence is captured in trace files. When you enable packet tracing, it logs every significant packet event. Let's analyze a simplified log from a link between a ground terminal (Node 0) and a satellite (Node 1), assuming a **12 ms propagation delay**.

Your `PacketTrace.log` would look something like this:
  ```cpp
  # Simplified Packet Trace Log
  # Event Time  Node  Layer   Flags  Packet Info
  SND     1.000 0     SatPhy         -  seq=1 size=1024
  RCV     1.012 1     SatPhy         -  seq=1 size=1024
  ```

##### Analyzing the Log
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

### Summary: ns-3 Satellite Channel Concepts
#### The Channel's Role
We learned that the `SatelliteChannel` acts as the **manager** of the communication medium. Its primary job is to take a packet from a sender, calculate how long it will take to reach each potential receiver, and then schedule its arrival using the delay model and the simulator's scheduler.

#### How Delay is Calculated
The `SatellitePropagationDelayModel` is the physics calculator. It computes propagation delay based on pure geometry:

1.  It gets the real-time 3D coordinates of the sender and receiver.
2.  It calculates the straight-line **Euclidean distance** between them.
3.  It divides this distance by the **speed of light** to get the exact travel time.

This ensures the delay is dynamic and changes realistically as the satellite moves.

#### Fading and Signal Strength
We explored how ns-3 uses a **pluggable architecture** for fading models to simulate signal degradation from rain or buildings. The channel can be configured with different models (**Rayleigh**, **Loo**, **Markov**) to calculate the signal loss (in dB), which the receiving PHY uses to determine if a packet was successfully received.

#### Tracing the Signal Flow
Finally, we traced a packet's entire journey:

1.  **`SatPhy::Transmit()`** starts the process on the sending node.
2.  The packet is handed to **`SatelliteChannel::Send()`**.
3.  The channel calculates the total delay and uses **`Simulator::Schedule()`** to create a future arrival event.
4.  At the scheduled time, the packet is delivered to the receiving node's **`SatPhy`** layer.

We saw how this sequence is confirmed using a **packet trace log**, where the time difference between a **`SND`** and **`RCV`** event directly shows the simulated propagation delay.





## Scenario & Example Customization

## Integrating Everything: End-to-End system understanding

## Experiment & Report: Create my own minimal simulation
