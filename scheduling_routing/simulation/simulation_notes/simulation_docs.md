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
