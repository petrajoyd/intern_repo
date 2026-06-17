# Study Notes – Paper: System-Level Evaluation of Beam Hopping in NR-Based LEO Satellite Communication System

**Paper:**  
"System-Level Evaluation of Beam Hopping in NR-Based LEO Satellite Communication System"  
Authors: Jingwei Zhang, Dali Qin, Chuili Kong, Feiran Zhao, Rong Li, Jun Wang, Ye Wang  
Source: IEEE WCNC, 2023  

---

## Problem
- The paper addresses how to do **beam hopping scheduling** in NR-based LEO NTN systems, where satellite mobility and time-varying traffic demand make fixed beam assignments inefficient.  
- They want to avoid adjacent beam interference and maximize performance under constraints (e.g. how many beams can be “on” simultaneously, power constraints), especially under different frequency bands and traffic models. 

---

## High-Level Concept of the Solution
- Propose a beam hopping scheme that jointly schedules which spotbeams are illuminated and how much transmit power each beam uses.  
- Key feature: **distance limit constraint** — don’t turn on adjacent spotbeams simultaneously (reduces interference).  
- Also works with NR protocols: appropriate bandwidth, subcarrier spacing etc. They test in both Ka-band and S-band.  
- They allow a maximum number of illuminated beams per satellite per time slot (parameter \(I_{\max}\)). As \(I_{\max}\) increases, more beams, but possible drop in SINR because of decreased power per beam. The scheme with distance limit behaves more stably as \(I_{\max}\) grows.

---

## How They Verify Their Results
- They use a **self-developed system-level simulator** (MATLAB / custom) with NR protocol stack components (HARQ, CQI, etc.). Not ns-3. 
- They simulate using a “Walker” constellation: \(M = 2400\) satellites, \(P = 60\) orbital planes, altitude = 600 km, inclined at 55°, etc.  
- Two frequency bands tested: S-band (center ~2 GHz) and Ka-band (~20 GHz).
- Different traffic models: full-buffer model, and FTP traffic (practical arrival process). 

---

## KPIs & Actual Numbers
Below are some of the actual metrics & results they got:

| KPI / Metric | Actual Value(s) / Observations |
|---------------|----------------------------------|
| **Max transmit power** \(P_{\max}\) | 300 W per satellite (for both S‐band & Ka‐band)  |
| **Distance limit** \(D\) | 42 km (adjacent beams must be farther than 42 km to be simultaneously illuminated)  |
| **Number of illuminated beams** \(I_{\max}\) tested | Varied; e.g. up to 40; also tests up to 100 in comparisons  |
| **Bandwidth / subcarrier spacing** | S-band: 30 MHz bandwidth, 15 kHz spacing; Ka-band: 200 MHz, 120 kHz spacing |
| **Antenna gains** | Satellite transmit gain \(G_T\): ~24 dBi (S-band), ~30.5 dBi (Ka-band); receive gain: handset/dr vs VSAT (~39.7 dBi) depending on UE type|
| **User satisfaction / system satisfaction** | Under traffic model FTP / practical: with distance limit scheme, **system satisfaction** ~ 69.5% (Ka band), ~ 70.1% (S band); competing schemes: 59.9%, 66.3%, etc. |
| **Packet life time / delay** | With FTP model, packet lifetime and UE satisfaction: the distance-limit beam-hopping scheme shows lower packet life time (better) vs baselines. Numbers ~ not always exact, but the trends are strong. |
| **Throughput behaviour vs Imax** | Throughput increases with number of illuminated beams for all schemes until a point; the distance limit scheme’s throughput levels off when Imax gets large, but still remains the best among schemes, even when Imax = 100.  |

---

## 1. Introduction
- **Motivation:**  
  - LEO satellites offer **lower latency, lower path loss, and lower deployment cost** compared to GEO/MEO.  
  - Companies like **OneWeb, Kuiper, and Starlink** drive demand for LEO mega-constellations.  
  - **3GPP NTN standardization** (Releases 15–18) is adapting NR to satellite links.  
  - Key challenge: **serving time-varying user demand** with limited satellite payload resources.  

- **Beam Hopping Concept:**  
  - Instead of keeping all beams on, only a subset of spotbeams is activated each time slot.  
  - Benefits:  
    - Reduces satellite payload weight/power usage.  
    - Matches demand hotspots dynamically.  
    - Suppresses interference when designed well.  
  - Already supported in **DVB-S2X (formats 5–7 for BH)**.  

- **Gap:**  
  - Most past beam hopping studies are for GEO or limited small systems.  
  - No **system-level evaluation** of **beam hopping in NR-based LEO** had been done.  

- **Contribution:**  
  - Propose an **efficient BH scheme** that accounts for traffic demand and interference.  
  - Present **first system-level evaluation** with NR stack in LEO (S-band and Ka-band).  
  - Show gains in **throughput, packet lifetime, and user satisfaction**.

---

## 2. Methodology

### 2.1 System-Level Simulation
- **Constellation:** Walker orbit, 2400 satellites, 60 planes, inclination 55°, altitude 600 km.  
- **Frequencies:**  
  - S-band: 2 GHz, 30 MHz BW, 15 kHz subcarrier spacing.  
  - Ka-band: 20 GHz, 200 MHz BW, 120 kHz spacing.  
- **Payload:** Regenerative payload (baseband processed onboard).  
- **Traffic Models:**  
  - **Full-buffer** (infinite packets, worst-case load).  
  - **FTP model 3** (Poisson arrivals, realistic traffic).  

### 2.2 Antenna & Users
- **Satellite antennas:** Phased array with 28×28 elements.  
  - Gain: 24 dBi (S-band), 30.5 dBi (Ka-band).  
- **Users:**  
  - S-band → handheld UEs, omnidirectional antenna (0 dBi).  
  - Ka-band → VSAT (60 cm aperture, 39.7 dBi gain).  
- **Coverage:** Max 100 spotbeams/satellite, each 42 km diameter.  
- **Max transmit power per sat:** 300 W.  

### 2.3 NR Protocol Setup
- Simulator implements NR full stack:  
  - RLC acknowledged mode.  
  - CQI-based scheduling.  
  - Adaptive MCS.  
  - HARQ retransmissions.  

---

## 3. Beam Hopping Methodology

### 3.1 Spotbeam Layout
- Beams are **Earth-fixed**.  
- UE served by its closest satellite (reduces interference).  
- GNSS coordinates used to map UEs → beams.  
- **Spectrum fully reused** across beams.

### 3.2 Beam Hopping Scheme (Algorithm 1)
- Time divided into slots.  
- Max number of illuminated beams per satellite: I_max.  
- Steps:  
  1. For each satellite, start with empty active set.  
  2. Iteratively pick the beam covering UE with **highest demand**.  
  3. Check **distance limit constraint** (D = 42 km).  
     - If beam too close to another active beam → skip it.  
  4. Repeat until limit reached or no beams left.  
  5. Distribute power equally across active beams.  

- **Key idea:** Avoid illuminating neighboring beams (high interference), while still focusing on high-demand beams.  

---

## 4. Results

### 4.1 Full-Buffer Model
- **SINR:**  
  - Increases with more beams (up to a point).  
  - Without distance limit → SINR drops as beams increase (interference + lower per-beam power).  
  - With distance limit → SINR stable beyond I_max = 40.  

- **Throughput:**  
  - Distance limit scheme always higher throughput than round robin or no-limit.  
  - At I_max = 100, distance-limit still best, despite other schemes using all beams.  
  - Proves **“more beams ≠ better if interference dominates.”**  

### 4.2 FTP Model (Realistic Traffic)
- **Packet lifetime (delay):**  
  - Shortest with distance-limit scheme.  
  - Longer for round robin.  
- **User satisfaction (throughput demand met):**  
  - Ka-band: 69.5% (distance-limit) vs 59.9% (no limit) vs 37.3% (round robin).  
  - S-band: 70.1% (distance-limit) vs 66.3% vs 44.9%.  

- **Insight:** Distance limit constraint **balances fairness + throughput**, giving best QoS.  

---

## 5. Key Takeaways
- Beam hopping **essential for LEO NR NTN** (power saving + traffic demand adaptation).  
- Distance-limit BH scheme:  
  - Reduces interference.  
  - Maintains SINR even with many beams.  
  - Achieves **~10–30% higher satisfaction** than baselines.  
- Results show NR-based NTN can achieve **sub-50ms latency and stable throughput** with smart scheduling.  


---

## Additional Notes / Insights
- The “distance limit” constraint helps a lot with interference control, especially as number of active beams increases.  
- There’s a trade-off: increasing Imax gives more capacity (more beams), but reduces power per beam → lower SINR → might degrade throughput per user. The proposed scheme with distance limit mitigates this trade-off.  
- Also, performance in **Ka-band** is significantly better in terms of throughput (higher bandwidth, higher antenna gains) compared to S-band, but Ka-band suffers more from path-loss / gain constraints.  
- Under realistic traffic (FTP traffic model), the “UE satisfaction” metric shows how often demanded throughput is met; the proposed scheme with distance limit gives a much higher satisfaction ratio vs baselines.  

---
# Glossary of Terms 

This file explains the hard technical terms in the paper. Each term has:
- **Formal Definition** (as used in communications engineering).  
- **Simpler Meaning** (explained in everyday terms).  

---

## 3GPP NTN
- **Definition:** 3rd Generation Partnership Project (3GPP) standards for Non-Terrestrial Networks (NTN), i.e., cellular systems (like 5G New Radio) adapted to work via satellites.  
- **Simple:** The rules and standards that make sure 5G works even if the signal comes from satellites instead of towers.

## 5G NR (New Radio)
- **Definition:** The global standard radio interface for 5G mobile networks.  
- **Simple:** The "language" phones and networks use to talk in 5G.

## LEO Satellite (Low Earth Orbit)
- **Definition:** Satellites orbiting at 200–2000 km altitude; in this paper, ~600 km. They move fast relative to Earth’s surface.  
- **Simple:** Small satellites flying close to Earth (compared to TV satellites), circling quickly above us.

## GEO & MEO
- **Definition:** GEO = Geostationary Earth Orbit (~36,000 km, fixed over one spot). MEO = Medium Earth Orbit (~2000–20,000 km).  
- **Simple:** GEO are far away and look “fixed in the sky”; MEO are in-between; LEO are the closest.

## Beam Hopping (BH)
- **Definition:** A satellite dynamically turns beams on/off in time slots to serve different areas with limited power/resources.  
- **Simple:** Like shining a flashlight on one village at a time, moving it quickly, instead of lighting the whole region at once.
  
## Spotbeam
- **Definition:** A focused radio beam projected by a satellite antenna to cover a small region on Earth (e.g., 42 km diameter).  
- **Simple:** Think of it like a spotlight from the satellite, lighting up one patch of ground.

## Phased Array Antenna
- **Definition:** An antenna made of many small elements, electronically steered to form and direct beams without moving parts.  
- **Simple:** Like a school of fish moving in sync—together they form a pattern and can “look” in different directions instantly.

## SINR (Signal-to-Interference-plus-Noise Ratio)
- **Definition:** Measure of signal quality = useful signal power divided by interference + noise. Higher is better.  
- **Simple:** How clear the radio signal is compared to background noise and other signals. Like hearing a friend’s voice in a noisy crowd.

## Throughput
- **Definition:** The rate of successful data delivery over a channel, usually in Mbps or Gbps.  
- **Simple:** How much internet speed (data) you actually get.

## Packet Lifetime
- **Definition:** Time taken from when a data packet is created until it is successfully received.  
- **Simple:** How long a message takes to travel from sender to receiver.

## User Satisfaction (QoS metric)
- **Definition:** Ratio of the throughput a user gets vs. what they requested.  
- **Simple:** Did the user get the speed they wanted, or only part of it?

## Full-Buffer Traffic Model
- **Definition:** Assumes users always have data to send (infinite backlog). Used for stress testing capacity.  
- **Simple:** Pretend everyone is downloading endlessly, like a stress test of maximum demand.

## FTP Model 3 (3GPP traffic model)
- **Definition:** A traffic model where packets arrive randomly (Poisson process), with realistic file sizes.  
- **Simple:** Users sometimes download files, sometimes not, in a random pattern—closer to real-world use.


## Walker Constellation
- **Definition:** A way to arrange satellites in multiple orbital planes evenly spaced around Earth.  
- **Simple:** Imagine evenly spacing fishing boats around a lake in circular paths so they cover all areas.


## Regenerative Payload
- **Definition:** Satellite processes baseband signals onboard instead of just relaying them.  
- **Simple:** A “smart” satellite that doesn’t just mirror the signal but also cleans and manages it before sending it back.


## CQI (Channel Quality Indicator)
- **Definition:** A number reported by user devices to tell the network how good the radio channel is.  
- **Simple:** Like telling your friend “I can hear you clearly” or “You’re breaking up”.


## HARQ (Hybrid Automatic Repeat reQuest)
- **Definition:** Error-control method combining retransmissions with error correction to improve reliability.  
- **Simple:** Like asking “What?” if you didn’t hear clearly, and your friend repeats or corrects the message.


## Adaptive Modulation and Coding (AMC)
- **Definition:** Adjusting data transmission speed and error protection depending on channel quality.  
- **Simple:** If the weather is clear, talk fast; if it’s noisy, slow down and repeat carefully.

## Path Loss
- **Definition:** Reduction of signal power as it travels through space and atmosphere.  
- **Simple:** The farther the sound, the weaker it gets—same with radio signals.

## Shadowed-Rician Fading (mentioned in related works)
- **Definition:** A radio propagation model that includes line-of-sight signal plus multiple scattered signals, with random fading due to obstacles.  
- **Simple:** Like sunlight through clouds: strong direct rays plus scattered light, sometimes dimmed.

## Inter-beam Interference
- **Definition:** When signals from two satellite beams overlap and interfere.  
- **Simple:** Like two fishing boats using sonar in the same spot—the signals clash.

## Round Robin Scheduling
- **Definition:** Assigning resources cyclically, one after another, regardless of demand.  
- **Simple:** Taking turns equally, like giving each student in class one chance to speak, even if some don’t need it.

## Distance Limit Constraint (in BH scheme)
- **Definition:** Rule that prevents two adjacent beams from being active at the same time to reduce interference.  
- **Simple:** Like making sure two loudspeakers aren’t blasting right next to each other.

---
