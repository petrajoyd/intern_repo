# Satellite Tutorial Example (Simulation Initialization Workflow)

This example (`sat-tutorial-example.cc`) demonstrates how to set up and execute a simple satellite network simulation using the **ns-3 satellite module**.  
It provides a complete pipeline — from scenario selection to traffic application setup.

---

## 1. Define and Initialize the Scenario

```cpp
SatHelper::PreDefinedScenario_t satScenario = SatHelper::SIMPLE;
std::string scenario = "Simple";
```

- Sets the base topology.
- `SatHelper::SIMPLE` → a default one-beam satellite network (1 gateway, 1 user terminal, 1 satellite).

---

## 2. Create Simulation Helper

```cpp
auto simulationHelper = CreateObject<SimulationHelper>("example-tutorial");
```

- `SimulationHelper` acts as the main manager for:
  - Network creation  
  - Node and channel setup  
  - Scenario loading and results management

---

## 3. Enable Traces and Logging

```cpp
Config::SetDefault("ns3::SatHelper::ScenarioCreationTraceEnabled", BooleanValue(true));
Config::SetDefault("ns3::SatHelper::PacketTraceEnabled", BooleanValue(true));
```

- Enables:
  - `CreationTraceScenario.log` (records how the scenario is built)
  - `PacketTrace.log` (records packet-level events)

---

## 4. Configure Environment Variables

```cpp
Config::SetDefault("ns3::SatEnvVariables::SimulationTag", StringValue(scenario));
Config::SetDefault("ns3::SatEnvVariables::EnableSimulationOutputOverwrite", BooleanValue(true));
```

- Tags the simulation output for easier data management.  
- Allows overwriting of results when re-running.

Optional (commented out in the code):
- You can load `.xml` config files with `ConfigStore` to set up complex environments.

---

## 5. Parse Command Line Arguments

```cpp
CommandLine cmd;
cmd.AddValue("scenario", "Scenario to be created", scenario);
simulationHelper->AddDefaultUiArguments(cmd);
cmd.Parse(argc, argv);
```

- Allows users to pass command-line arguments, e.g.:
  ```bash
  ./ns3 run "sat-tutorial-example --scenario=full"
  ```

---

## 6. Scenario Setup and Time Configuration

```cpp
if (scenario == "larger") { satScenario = SatHelper::LARGER; }
else if (scenario == "full") { satScenario = SatHelper::FULL; }

simulationHelper->SetOutputTag(scenario);
simulationHelper->SetSimulationTime(Seconds(11));
```

- Switches between predefined topologies.  
- Sets total simulation duration (11 seconds).

---

## 7. Enable Component Logging

```cpp
LogComponentEnable("CbrApplication", LOG_LEVEL_INFO);
LogComponentEnable("PacketSink", LOG_LEVEL_INFO);
LogComponentEnable("sat-tutorial-example", LOG_LEVEL_INFO);
```

- Prints runtime information to terminal during simulation.

---

## 8. Load Scenario and Create Satellite System

```cpp
simulationHelper->LoadScenario("geo-33E");
Ptr<SatHelper> helper = simulationHelper->CreateSatScenario(satScenario);
```

- Loads a **geostationary satellite** configuration at **33°E**.
- Initializes full network structure:
  - Gateways (GW)
  - Satellites (SAT)
  - User Terminals (UT)
  - Links and channels

---

## 9. Application Layer — User Traffic Setup

This section simulates **bi-directional communication** between GW and UT using CBR (Constant Bit Rate) and packet sink applications.

### User Node Containers
```cpp
NodeContainer utUsers;
if (scenario == "full") { ... } else { utUsers = Singleton<SatTopology>::Get()->GetUtUserNodes(); }
NodeContainer gwUsers = Singleton<SatTopology>::Get()->GetGwUserNodes();
```

- Retrieves node containers for UTs and GWs from the global satellite topology.

### Traffic Configuration
```cpp
PacketSinkHelper sinkHelper(...);
CbrHelper cbrHelper(...);
```

- `PacketSinkHelper` receives packets.
- `CbrHelper` sends packets at a fixed rate.

### Application Scheduling
| Node | Application | Start | Stop |
|-------|--------------|--------|------|
| GW | Sink | 1.0s | 10.0s |
| GW | CBR (→ UT) | 1.0s | 2.1s |
| UT | Sink | 1.0s | 10.0s |
| UT | CBR (→ GW) | 7.0s | 9.1s |

---

## 10. (Optional) Save Configurations to XML

Commented out, but available for exporting final parameter states.

---

## 11. Run the Simulation

```cpp
simulationHelper->RunSimulation();
```

Handles:
- `Simulator::Run()`
- `Simulator::Stop()`
- `Simulator::Destroy()`

---

## Summary Flow Diagram

```
main()
 ├── Define Scenario (SatHelper::SIMPLE)
 ├── Create SimulationHelper("example-tutorial")
 ├── Enable Traces (Scenario, Packet)
 ├── Parse CLI Arguments
 ├── Set Simulation Time (11s)
 ├── LoadScenario("geo-33E")
 ├── CreateSatScenario()
 ├── Install Applications (GW ↔ UT)
 ├── RunSimulation()
 └── Output Logs (PacketTrace.log, SimInfo.log, etc.)
```

---

## Generated Files

| File | Description |
|------|--------------|
| `PacketTrace.log` | Packet-level transmission/reception logs |
| `CreationTraceScenario.log` | Node/channel creation logs |
| `SimInfo.log` | Simulation environment info |
| `SimDiff.log` | Comparison against reference runs |

---

### In Short
The `sat-tutorial-example` initializes a complete ns-3 satellite communication scenario, simulating uplink/downlink traffic between gateway and user terminal nodes under a geostationary satellite system.

# Simulation Initialization: `sat-tutorial-example.cc`

This example demonstrates how a satellite simulation is initialized, configured, and executed. The workflow defines the structure of any ns-3 satellite scenario.

## **Initialization Flow**

```cpp
SatHelper::PreDefinedScenario_t satScenario = SatHelper::SIMPLE;
std::string scenario = "Simple";
```

* Defines which built-in satellite scenario will be simulated (e.g., SIMPLE, FULL, LARGER).

```cpp
auto simulationHelper = CreateObject<SimulationHelper>("example-tutorial");
```

* Creates the main simulation helper that manages configuration, scenario setup, and runtime operations.

```cpp
Config::SetDefault("ns3::SatHelper::ScenarioCreationTraceEnabled", BooleanValue(true));
Config::SetDefault("ns3::SatHelper::PacketTraceEnabled", BooleanValue(true));
```

* Enables creation and packet tracing for output log files.

Command-line arguments are parsed and linked to `SimulationHelper` defaults:

```cpp
CommandLine cmd;
cmd.AddValue("scenario", "Scenario to be created", scenario);
simulationHelper->AddDefaultUiArguments(cmd);
cmd.Parse(argc, argv);
```

Depending on the `scenario` string, different network topologies are selected:

```cpp
if (scenario == "larger") satScenario = SatHelper::LARGER;
else if (scenario == "full") satScenario = SatHelper::FULL;
```

The simulation time and log components are set:

```cpp
simulationHelper->SetSimulationTime(Seconds(11));
LogComponentEnable("CbrApplication", LOG_LEVEL_INFO);
LogComponentEnable("PacketSink", LOG_LEVEL_INFO);
LogComponentEnable("sat-tutorial-example", LOG_LEVEL_INFO);
```

Scenario loading:

```cpp
simulationHelper->LoadScenario("geo-33E");
Ptr<SatHelper> helper = simulationHelper->CreateSatScenario(satScenario);
```

* This builds the full network topology using `SatHelper`.

Application setup for gateway (GW) and user terminal (UT):

```cpp
PacketSinkHelper sinkHelper("ns3::UdpSocketFactory", InetSocketAddress(helper->GetUserAddress(gwUsers.Get(0)), port));
CbrHelper cbrHelper("ns3::UdpSocketFactory", InetSocketAddress(helper->GetUserAddress(utUsers.Get(0)), port));
```

Applications are installed and started on GW and UT nodes to exchange UDP traffic across the satellite link.

Simulation execution:

```cpp
simulationHelper->RunSimulation();
```

---

# Simulation Initialization Pipeline

## **1️. SimulationHelper** (`helper/simulation-helper.cc`)

* Entry point for all satellite simulations.
* Handles argument parsing, XML config loading, and scenario management.
* Calls `SatHelper` to build and run the simulation.

## **2️. SatHelper** (`helper/satellite-helper.cc`)

* Core builder for the satellite network.
* Instantiates and connects nodes: UTs, GWs, Satellites, NCC.
* Configures PHY/MAC layers and propagates simulation parameters to the environment.
* Handles `SatEnvVariables` and topology construction.

## **3️. SatTopology** (`model/satellite-topology.cc`)

* Maintains node relationships, positions, and beam allocations.
* Provides accessors to UT, GW, and Satellite nodes.
* Supports `GetUtNodes()`, `GetGwNodes()`, and `GetUserNodes()` used in example scripts.

## **4️. Model Layer** (`model/`)

* Implements core satellite system logic:

  * **PHY Layer:** `satellite-phy.cc`, `satellite-phy-rx.cc`, `satellite-phy-tx.cc`
  * **MAC Layer:** `satellite-mac.cc`, `satellite-mac-tag.cc`
  * **Channel & Propagation:** `satellite-channel.cc`, `satellite-propagation-delay-model.cc`
  * **Mobility Models:** `satellite-sgp4-mobility-model.cc` (orbital motion)
  * **Schedulers:** `satellite-fwd-link-scheduler.cc`, `satellite-ut-scheduler.cc`
  * **Fading Models:** `satellite-rayleigh-model.cc`, `satellite-loo-model.cc`

## **5️. Applications Layer**

* Uses traffic models like `CbrHelper` and `PacketSinkHelper` to simulate user data exchange.
* Installed after the satellite topology is built.

## **6️. Execution**

* `simulationHelper->RunSimulation()` triggers ns-3’s event loop.
* Logs and statistics are stored in `contrib/satellite/data/sims/<scenario>`.

---

### Output Files

| File                          | Description                                                       |
| ----------------------------- | ----------------------------------------------------------------- |
| `PacketTrace.log`             | Logs all packet send/receive events across UT, GW, and SAT nodes. |
| `SimDiff.log` / `SimInfo.log` | Simulation metadata and runtime information.                      |
| `CreationTraceScenario.log`   | Records network creation traces (node, link, and configuration).  |

---

### Summary

The `sat-tutorial-example.cc` provides a minimal but complete workflow of a satellite simulation in ns-3. It initializes scenario settings, constructs network topology using helpers, sets up applications, and executes the simulation loop. The helper classes (`SimulationHelper`, `SatHelper`, and `SatTopology`) abstract the complexity of building a realistic satellite system, while model files provide physical and link-layer fidelity.

---

Next Step → Analyze logs under `contrib/satellite/data/sims/` to confirm correct initialization and verify packet exchanges between GW and UT nodes.
