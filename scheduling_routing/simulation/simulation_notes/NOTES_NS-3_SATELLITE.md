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

## Channel & Propagation Mastery

## Scenario & Example Customization

## Integrating Everything: End-to-End system understanding

## Experiment & Report: Create my own minimal simulation
