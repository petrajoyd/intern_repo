# Study Notes – Satellites Beam Hopping Scheduling for Interference Avoidance

**Paper:**  
"Satellites Beam Hopping Scheduling for Interference Avoidance"  
Authors: Huimin Deng, Kai Ying, Daquan Feng, Lin Gui, Yuanzhi He, Xiang-Gen Xia  
Source: IEEE Journal on Selected Areas in Communications (JSAC), Vol. 42, No. 12, Dec 2024  
DOI: 10.1109/JSAC.2024.3459083  

---

## Problem
- LEO mega-constellations increase spectrum scarcity and interference.  
- When LEO and GEO satellites share spectrum with terrestrial networks, **co-frequency interference** occurs.  
- Conventional fixed multi-beam satellites cannot adapt resources to time-varying heterogeneous demands.  
- Key challenge: how to design **beam hopping (BH) scheduling** to maximize throughput and user satisfaction **while avoiding interference** across LEO/GEO/terrestrial systems.

---

## High-Level Solution
- Authors formulate two optimization problems:  
  - **TDBH (Throughput-Driven Beam Hopping):**  
    - For GEO satellites.  
    - Maximizes system throughput with adjustable user demand (traffic can be scaled with adaptive rate factors).  
    - Uses problem relaxation to reduce complexity.  
  - **SDBH (Satisfaction-Driven Beam Hopping):**  
    - For LEO satellites.  
    - Maximizes user traffic satisfaction rate with fixed user demands.  
    - Uses **Genetic Algorithm (GA)** to find near-optimal scheduling.  

- Both schemes rely on **interference avoidance patterns** provided by ground controllers to ensure no harmful interference to GEO/terrestrial users.

---

## Verification Method
- Simulations conducted using:  
  - **TDBH (GEO scenario):** Implemented in **MATLAB**, 19 cells, max 4 beams, 33 time slots.  
  - **SDBH (LEO scenario):** Implemented in **Python 3.8**, 19 cells, max 4 beams, Ka-band 500 MHz per beam, Shadowed-Rician fading channels.  

- No ns-3 was used. Evaluations combined mathematical optimization (relaxation) and GA.

---

## KPIs Considered
- **System throughput** (for GEO TDBH).  
- **Traffic satisfaction rate** (per-cell and average across network for LEO SDBH).  
- **Fairness** (minimum satisfaction rate across cells).  
- **Impact of channel fading conditions** (light/average/heavy shadow fading).  
- **Robustness under different traffic load intensities** and distributions.  

| **KPI**                                  | **Scenario**                                             | **Algorithm**       | **Value / Result**                                                                                          | **Comparison**                                 |
| ---------------------------------------- | -------------------------------------------------------- | ------------------- | ----------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **System Throughput**                    | GEO (Ka-band, 19 cells, 4 beams active, 33 slots/frame)  | **TDBH**            | Significantly higher throughput, especially at high demand (exact value not given, but >20% gain vs greedy) | Outperforms Greedy + Random                    |
|                                          |                                                          | **Greedy**          | Lower than TDBH, ~15–20% less at high load                                                                  | Worse                                          |
|                                          |                                                          | **Random**          | Worst performance (large underutilization)                                                                  | Worst                                          |
| **User Satisfaction Rate**               | LEO (Ka-band, 19 cells, 500 MHz, shadowed-Rician fading) | **SDBH**            | >90% (average fading)                                                                                       | Outperforms greedy (~70–75%) and random (~60%) |
|                                          |                                                          | **Greedy**          | ~70–75% (average fading)                                                                                    | 15–25% lower than SDBH                         |
|                                          |                                                          | **Random**          | ~60%                                                                                                        | Worst                                          |
| **Fairness (Minimum Satisfaction Rate)** | LEO                                                      | **SDBH**            | Ensures fairness across all cells (minimum satisfaction ~70–80%)                                            | Better                                         |
|                                          |                                                          | **Greedy/Random**   | Some cells underserved (minimum satisfaction <50%)                                                          | Worse                                          |
| **Impact of Fading**                     | LEO – Heavy fading                                       | **SDBH**            | Maintains advantage, but satisfaction drops to ~65–70%                                                      | 20–30% better than greedy                      |
|                                          |                                                          | **Greedy**          | Drops below 50%                                                                                             | Much worse                                     |
|                                          |                                                          | **Random**          | ~40%                                                                                                        | Worst                                          |
| **Scalability**                          | Large-scale LEO                                          | **SDBH (GA-based)** | Achieves near-optimal results with reasonable computation time                                              | Practical                                      |

---

## 1. Introduction
- **Background:**  
  - Future LEO mega-constellations (e.g., Starlink) will coexist with GEO satellites and terrestrial systems.  
  - Spectrum sharing creates **co-frequency interference**.  
  - Traditional **fixed multibeam satellites** cannot adapt to rapidly changing and heterogeneous user demands.  

- **Beam Hopping (BH):**  
  - Dynamically switching active beams in different time slots.  
  - Supported in DVB-S2X.  
  - Allows fewer onboard resources to serve more cells flexibly.  
  - Can reduce interference if properly scheduled.  

- **Motivation:**  
  - GEO satellites emphasize **throughput maximization**.  
  - LEO satellites emphasize **user satisfaction** and fairness (shorter visibility time, higher dynamics).  
  - Both scenarios require **interference-avoidance beam hopping scheduling**.  

- **Contribution:**  
  - Propose **TDBH (Throughput-Driven BH)** for GEO.  
  - Propose **SDBH (Satisfaction-Driven BH)** for LEO.  
  - Provide optimization models + simulation evaluations in realistic fading channels.  

---

## 2. Methodology

### 2.1 TDBH (Throughput-Driven BH, GEO)
- **Objective:** Maximize total throughput while satisfying interference constraints.  
- **Traffic Demand:** Adjustable via adaptive rate factors.  
- **Mathematical Model:**  
  - Formulated as a mixed-integer optimization problem.  
  - Relaxation applied for tractability.  
- **Simulation Setup:**  
  - 19-cell GEO system.  
  - Max 4 beams illuminated simultaneously.  
  - 33 time slots per frame.  
  - Considered flexible demand scaling.  

### 2.2 SDBH (Satisfaction-Driven BH, LEO)
- **Objective:** Maximize **satisfaction rate**, defined as ratio of served traffic to requested traffic.  
- **Traffic Demand:** Fixed (non-adjustable).  
- **Solution:**  
  - Optimization problem solved via **Genetic Algorithm (GA)**.  
  - GA chosen for good balance of solution quality vs complexity in large LEO systems.  
- **Simulation Setup:**  
  - 19-cell LEO system.  
  - Ka-band, 500 MHz bandwidth per beam.  
  - Shadowed-Rician fading (light, average, heavy).  
  - Max 4 beams active per slot.  

### 2.3 Interference Avoidance
- Both schemes rely on **pre-defined interference avoidance patterns** provided by ground control centers.  
- Ensures no harmful interference to GEO or terrestrial users.  

---

## 3. Results

### 3.1 GEO (TDBH Results)
- **Throughput vs Demand:**  
  - TDBH achieves significantly higher throughput compared to greedy and random BH scheduling.  
  - Gains increase with traffic demand intensity.  
- **Fairness:**  
  - TDBH maintains reasonable fairness via adaptive rate scaling.  
  - Prevents over-allocation to only high-demand cells.  

### 3.2 LEO (SDBH Results)
- **User Satisfaction Rate:**  
  - Under average shadow fading: SDBH achieves **>90% satisfaction rate**.  
  - Greedy BH achieves only ~70–75%.  
  - Random scheduling much lower (~60%).  
- **Fairness:**  
  - SDBH maximizes **minimum satisfaction rate** across cells.  
  - Ensures no cell is completely starved of resources.  
- **Impact of Fading:**  
  - Light fading: SDBH close to ideal performance (>95% satisfaction).  
  - Heavy fading: All schemes degrade, but SDBH maintains clear advantage (20–30% higher than greedy).  
- **Scalability:**  
  - GA-based SDBH scales well for large constellations, offering near-optimal performance with reasonable computation.  

---

## 4. Key Takeaways
- Beam hopping must be designed differently for GEO (throughput focus) vs LEO (satisfaction/fairness focus).  
- **TDBH**: Efficient throughput maximization, ensures scalable capacity.  
- **SDBH**: Fair and robust satisfaction rate optimization, critical in LEO dynamics.  
- Both algorithms rely on interference-avoidance patterns to protect coexisting networks.  
- Genetic Algorithms offer a practical tool for large-scale LEO BH scheduling.

---
# Glossary of Terms 

This glossary explains new technical terms appearing in this paper, different from those already defined in the first glossary.

---

## Co-frequency Interference
- **Definition:** Interference that occurs when two different systems (e.g., GEO satellite and terrestrial cell tower) transmit on the same frequency.  
- **Simple:** Like two people talking loudly on the same radio channel—you hear both, and they mess each other up.

## GEO vs LEO Scheduling Context
- **Definition:** GEO scheduling usually focuses on maximizing **throughput**, while LEO scheduling emphasizes **user satisfaction** due to shorter contact times and rapid handovers.  
- **Simple:** For GEO, it’s about “serve as much as possible”; for LEO, it’s “make sure everyone gets something fairly.”

## Traffic Demand (Adjustable vs Fixed)
- **Definition:** In scheduling, traffic demand may be **adjustable** (scaled according to available resources, e.g., GEO) or **fixed** (cannot be scaled, e.g., LEO).  
- **Simple:** Adjustable demand = students willing to accept shorter essays; fixed demand = students insisting on full-length essays.

## Genetic Algorithm (GA)
- **Definition:** An optimization method inspired by biological evolution (selection, crossover, mutation). Used to find near-optimal solutions in complex problems.  
- **Simple:** Like breeding the best traits of plants repeatedly until you get a very strong crop.

## Throughput-Driven Beam Hopping (TDBH)
- **Definition:** A GEO-focused scheduling algorithm that maximizes overall system throughput by adjusting traffic demands and activating beams efficiently.  
- **Simple:** Like a teacher focusing on boosting the total test scores of the whole class.

## Satisfaction-Driven Beam Hopping (SDBH)
- **Definition:** A LEO-focused scheduling algorithm that maximizes the **user satisfaction rate** (how much of their demand is served). Solved using GA.  
- **Simple:** Like a teacher making sure every student gets at least part of the homework done, even if the total class score isn’t the highest.

## Satisfaction Rate
- **Definition:** Ratio of the actual traffic delivered to a user over the traffic they requested.  
- **Simple:** If you asked for 10 fish and got 8, your satisfaction rate is 80%.

## Minimum Satisfaction Rate
- **Definition:** The lowest satisfaction rate among all users/cells. Used as a fairness metric.  
- **Simple:** “How badly is the worst-off student doing?” Ensures no one is left with zero.

## Shadowed-Rician Fading
- **Definition:** A wireless channel model that includes a strong line-of-sight signal plus weaker multipath signals, with random fading due to obstacles (e.g., mountains, clouds).  
- **Simple:** Like sunlight: mostly bright direct rays, but sometimes clouds block and scatter it, dimming it randomly.

## Ka-band
- **Definition:** A frequency band between 26.5–40 GHz used for high-throughput satellite communications.  
- **Simple:** A chunk of the radio spectrum where satellites can send very high-speed internet.

## Mixed-Integer Optimization
- **Definition:** Mathematical optimization where some decision variables must be integers (e.g., beam on/off = 0 or 1).  
- **Simple:** Like deciding which lights are ON or OFF in a house to save power, while also adjusting dimmers continuously.

## Relaxation (Mathematical)
- **Definition:** Technique to simplify an optimization problem by loosening integer constraints, making it solvable with standard methods.  
- **Simple:** Instead of “yes/no” answers, allow “maybe” values (like 0.5), solve it, then round later.

## Greedy Algorithm
- **Definition:** A simple algorithm that makes the locally optimal choice at each step, hoping to find a global optimum.  
- **Simple:** Like eating the tastiest-looking food first without planning the whole meal.

## Random Scheduling
- **Definition:** Allocating resources randomly without considering demand, fairness, or interference.  
- **Simple:** Like a teacher giving random students snacks without checking if they’re hungry.

## Scalability
- **Definition:** The ability of an algorithm/system to handle increasing size (more beams, more satellites, more users) without collapsing.  
- **Simple:** Like whether a small fishing boat method can still work if you suddenly have a whole fleet of 100 boats.

---
