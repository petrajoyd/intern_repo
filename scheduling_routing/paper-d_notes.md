# Study Notes – DSROQ: Dynamic Scheduling and Routing for QoE Management in LEO Satellite Networks

**Paper:**  
"DSROQ: Dynamic Scheduling and Routing for QoE Management in LEO Satellite Networks"  
Authors: D. Bhattacharjee, P. G. Madoery, A. Naik, H. Yanikomeroglu, G. K. Kurt, S. Martel, K. Ahmed  
Source: IEEE / arXiv preprint, Aug 2025  
[Link: arXiv PDF](https://arxiv.org/abs/2508.21047)

---

## 1. Problem
- LEO mega-constellations (e.g., Starlink) serve **heterogeneous traffic classes**:  
  - EF (Expedited Forwarding): latency-sensitive flows like VoIP, video calls.  
  - AF (Assured Forwarding): streaming flows needing stable throughput (live video).  
  - BE (Best Effort): elastic flows like file transfers.  
- Prior work optimizes **routing** or **queue scheduling** in isolation, not jointly.  
- The challenge:  
  - **Highly dynamic topology** (ISLs change every ~15 seconds).  
  - **Resource contention** between traffic classes.  
  - Need to maximize **end-user QoE** under real-world constraints.  

---

## 2. High-Level Concept of the Solution
- **DSROQ Algorithm** = joint scheduling + routing + bandwidth allocation.  
- Components:  
  - **Routing + Bandwidth Allocation:**  
    - NP-hard → solved using **Monte Carlo Tree Search (MCTS)** to explore feasible allocations.  
    - Accounts for flow QoS class weights.  
  - **Queue Scheduling:**  
    - **Lyapunov optimization framework** prioritizes latency-sensitive queues.  
    - Ensures system stability while meeting flow requirements.  
- Main novelty: integrates **QoE-aware routing** with **queue stability scheduling** into one framework.  

---

## 3. Verification Method
- **Simulator:** Custom system-level simulator (self-developed), not ns-3.  
- **Constellation:** Starlink Phase 1 Version 2-like, 4x4 grid, altitude 550 km, inclination 53°.  
- **Topology updates:** Every 15 s (new ISL links).  
- **Simulation runtime:** 60 seconds.  
- **Flows:** 10,000 total flows, mix of EF/AF/BE traffic classes.  
- **Traffic model:** Poisson arrivals, EF flows short packets, BE flows bulk file transfers.  

---

## 4. KPIs Considered 
### Latency (End-to-End)
- **EF flows (VoIP/video calls):**  
  - DSROQ avg. latency = **11 ms** (within 20 ms bound).  
  - Baseline shortest-path routing latency = **32 ms**.  
  - Improvement: **~65% lower latency**.  

### Throughput
- **AF flows (video streaming):**  
  - DSROQ achieves avg. **2.1 Mbps per flow**.  
  - Baseline = 1.6 Mbps.  
  - Improvement: **31% higher throughput**.  

- **BE flows (file transfers):**  
  - DSROQ achieves ~**400 Mbps total system throughput**.  
  - Baseline (routing only) = 290 Mbps.  
  - Improvement: **38% higher throughput**.  

### Packet Loss
- DSROQ packet loss < **0.5%** for EF flows.  
- Baseline scheduling = 2–3% loss.  

### QoE Score (Weighted Metric)
- Average QoE (normalized [0–1]):  
  - DSROQ = **0.82**.  
  - Baseline = 0.63.  
  - Improvement: **+19 percentage points**.  

### Fairness
- Jain’s fairness index (across QoE classes):  
  - DSROQ = **0.92**.  
  - Baseline = 0.78.  
  - DSROQ more evenly balances satisfaction across EF, AF, BE flows.  

| **KPI**              | **Baseline (Routing-only / Scheduling-only)** | **DSROQ (Proposed)** | **Improvement** |
|-----------------------|-----------------------------------------------|----------------------|-----------------|
| **End-to-End Latency (EF flows)** | 32 ms | 11 ms | ~65% lower |
| **Throughput (AF flows)** | 1.6 Mbps/flow | 2.1 Mbps/flow | +31% |
| **System Throughput (BE flows)** | 290 Mbps | 400 Mbps | +38% |
| **Packet Loss (EF flows)** | 2–3% | <0.5% | Reduced by ~80% |
| **QoE Score (0–1 normalized)** | 0.63 | 0.82 | +19 points |
| **Fairness Index (Jain’s)** | 0.78 | 0.92 | More balanced |

---

## Key Takeaways
- **Problem:** LEO NTN needs joint optimization → routing alone or scheduling alone isn’t enough.  
- **Solution:** DSROQ couples **MCTS-based routing/bandwidth allocation** with **Lyapunov-based scheduling**.  
- **Verification:** Self-developed system-level simulator, Starlink-like setup, 10k flows.  
- **KPIs:** Latency, throughput, packet loss, QoE, fairness.  
- **Results (numbers):**  
  - Latency cut by ~65%.  
  - Throughput up by 30–40%.  
  - QoE score improved from 0.63 → 0.82.  
  - Fairness index boosted from 0.78 → 0.92.  

---
# DSROQ: Detailed Notes

## Introduction
- **Context:**  
  - LEO constellations like Starlink introduce **dynamic topology** → satellites move fast, ISL (inter-satellite link) availability changes every few seconds.  
  - Challenge: Support **heterogeneous traffic classes** (real-time, streaming, elastic) while maximizing **user-perceived quality (QoE)**.  

- **Traffic types in NTN:**  
  - **EF (Expedited Forwarding):** real-time traffic (VoIP, video conferencing). Sensitive to **latency** and **packet loss**.  
  - **AF (Assured Forwarding):** video streaming/live streaming. Requires **stable throughput** and moderate latency.  
  - **BE (Best Effort):** file transfers. Can tolerate higher delay but needs **high throughput**.  

- **Problem Gap:**  
  - Prior works optimize either **routing** or **queue scheduling**.  
  - Routing-only → ignores queue stability, leads to buffer overflows or unfair latency.  
  - Scheduling-only → ignores path diversity and bandwidth allocation.  
  - Both approaches fail to jointly capture **QoE optimization** in dynamic LEO systems.  

- **Key Contribution:**  
  - Propose **DSROQ**, a **joint scheduling + routing + bandwidth allocation algorithm** for LEO constellations.  
  - Integrates **MCTS-based optimization** (for routing + bandwidth) and **Lyapunov optimization** (for queue scheduling).  
  - Targets **QoE maximization** by considering heterogeneous traffic weights.  

## Methodology

### System-Level Setup
- **Constellation model:** Starlink Phase 1 Version 2 (simplified).  
  - Altitude: **550 km**  
  - Inclination: **53°**  
  - Layout: **4×4 grid** of satellites.  
- **Simulation time:** 60 seconds.  
- **Topology updates:** every **15 s** (new ISL configurations).  
- **Flows:** **10,000 total flows** distributed across EF/AF/BE.  

### Traffic Classes
- **EF flows:**  
  - Small packets.  
  - Delay bound: **20 ms**.  
  - Loss-sensitive.  
- **AF flows:**  
  - Streaming packets.  
  - Throughput target: ~2 Mbps.  
  - Can tolerate moderate jitter.  
- **BE flows:**  
  - Large file transfers.  
  - Priority = lowest.  
  - Focus on maximizing aggregate throughput.  

### Algorithm Design
1. **Routing + Bandwidth Allocation**  
   - Problem: NP-hard joint optimization.  
   - Solution: **Monte Carlo Tree Search (MCTS)**.  
     - Explores candidate routes and bandwidth splits.  
     - Evaluates utility function based on QoE weights.  
   - Output: Feasible allocation of flows across ISLs.  

2. **Queue Scheduling**  
   - Problem: Prioritize packets within each flow.  
   - Solution: **Lyapunov optimization framework**.  
     - Tracks queue backlogs.  
     - Minimizes queue length drift subject to service constraints.  
     - Guarantees stability while respecting QoS weights.  

3. **QoE Metric (Utility Function)**  
   - Each flow gets a **QoS score** (normalized 0–1).  
   - Weighted combination:  
     - Latency-sensitive weight for EF.  
     - Throughput weight for AF.  
     - Elastic weight for BE.  
   - QoE = max of weighted average across flows.  

## Results

### Latency (EF flows)
- **Baseline (shortest path routing):** avg. **32 ms**.  
- **DSROQ:** avg. **11 ms**.  
- **Improvement:** ~65% lower latency.  
- 95th percentile latency for EF flows under DSROQ: **19 ms** (under 20 ms bound).  

### Throughput
- **AF flows:**  
  - Baseline: **1.6 Mbps per flow**.  
  - DSROQ: **2.1 Mbps per flow**.  
  - Improvement: +31%.  

- **BE flows:**  
  - Baseline: **290 Mbps aggregate**.  
  - DSROQ: **400 Mbps aggregate**.  
  - Improvement: +38%.  

### Packet Loss
- **EF flows:**  
  - Baseline: 2–3%.  
  - DSROQ: <0.5%.  
- **AF & BE flows:** negligible packet loss due to prioritization.  

### QoE Score
- Normalized [0–1] scale:  
  - Baseline: **0.63**.  
  - DSROQ: **0.82**.  
  - Gain: +19 percentage points.  

### Fairness
- **Jain’s Index:**  
  - Baseline: **0.78** (imbalanced, EF starved under load).  
  - DSROQ: **0.92** (more balanced across EF, AF, BE).  

### Sensitivity (Priority Weights)
- When EF flows given higher priority: latency dropped to **9 ms**, but BE throughput decreased by ~12%.  
- When BE flows prioritized: system throughput rose to **420 Mbps**, but EF latency exceeded 20 ms.  
- Shows DSROQ’s **adaptability** to operator-configured policies.  

## Key Insights
- DSROQ achieves **significant QoE gains** in dynamic LEO networks compared to routing-only or scheduling-only baselines.  
- **Latency reduction (~65%)** and **throughput gains (30–40%)** are the headline results.  
- Trade-offs between EF latency and BE throughput can be tuned via weights.  
- The framework is **scalable**, tested on 10,000 flows, and models real Starlink-like conditions.  
