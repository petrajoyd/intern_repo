# Study Notes – Joint Beam Scheduling and Power Optimization for Beam Hopping LEO Satellite Systems

**Paper:**  
"Joint Beam Scheduling and Power Optimization for Beam Hopping LEO Satellite Systems"  
Authors: Shuang Zheng, Xing Zhang*, Peng Wang, Wenbo Wang  
Source: *China Communications*, Dec 2023 (IEEE-linked journal)  
DOI: 10.23919/JCC.ea.YYYY-MMMM.NNNN  

---

## Problems
- **Context:**  
  - LEO satellite systems face **highly dynamic traffic demands** due to user mobility and varying geographic distributions. 
  - Existing beam hopping (BH) methods often optimize **scheduling only**, ignoring **power allocation** across beams.  
  - This leads to **low resource utilization**, **unfair distribution**, and **reduced throughput** in dynamic LEO environments.  

- **Problem Statement:**  
  - How to jointly design **beam scheduling** and **power allocation** for LEO beam hopping satellites to maximize throughput and fairness.  

---

## High-Level Concept of the Solution
- **Proposed Algorithm:** **Joint Beam Scheduling and Power Optimization for Beam Hopping (JBSPO-BH)**.  

- **Main Ideas:**  
  1. **Decoupling Approach:**  
     - Separate the joint optimization problem into two subproblems:  
       - **Beam Scheduling** → modeled as a **potential game**, solved by finding a **Nash Equilibrium (NE)** strategy.  
       - **Power Allocation** → solved using the **Penalty Function Interior Point Method** (mathematical optimization).  
  2. **Beam Scheduling as Game Theory:**  
     - Users/cells treated as players in a potential game.  
     - Nash Equilibrium ensures stable, conflict-free beam schedules.  
  3. **Power Optimization:**  
     - Allocates power proportionally across beams while minimizing interference and ensuring fairness.  

---

## How the Authors Verify Their Results
- **Verification Tools:**  
  - **Custom-built simulator** 
  - Simulation conducted on dynamic LEO beam hopping networks.  
  - Compared against:  
    - Greedy-based BH.  
    - Greedy-based BH with power optimization.  
    - Round-robin BH.  
    - Max-SINR BH.  
    - Satellite resource allocation baseline algorithm.  

- **Simulation Environment:**  
  - LEO satellite beam hopping with spatio-temporal traffic variations.  
  - Multiple sink nodes with geographic distribution differences.  

---

## KPIs Considered 
- **Throughput Improvement:**  
  - JBSPO-BH achieves gains compared to other schemes:  
    - +44.99% vs Greedy-BH.  
    - +20.79% vs Greedy-BH + Power.  
    - +156.06% vs Round-Robin BH.  
    - +15.39% vs Max-SINR BH.  
    - +8.17% vs Satellite Resource Allocation Algorithm.  

- **Fairness:**  
  - JBSPO-BH ensures **balanced service** across cells by distributing power fairly.  
  - Maintains stable performance in both high and low demand regions.  

- **Convergence & Complexity:**  
  - Algorithm has **low time complexity** and **fast convergence**, suitable for real-time LEO scheduling.  

| **KPI**                  | **Scenario**      | **Algorithm**            | **Value / Result**                           | **Comparison**                         |
|---------------------------|------------------|--------------------------|-----------------------------------------------|-----------------------------------------|
| **System Throughput**     | LEO BH dynamic   | **JBSPO-BH**             | Best performance                              | Baseline reference                       |
|                           |                  | Greedy BH                | –44.99% lower than JBSPO-BH                   | Weak                                     |
|                           |                  | Greedy BH + Power        | –20.79% lower than JBSPO-BH                   | Better than Greedy, worse than JBSPO-BH  |
|                           |                  | Round-Robin BH           | –156.06% lower than JBSPO-BH                  | Worst                                    |
|                           |                  | Max-SINR BH              | –15.39% lower than JBSPO-BH                   | Competitive but less fair                |
|                           |                  | Resource Allocation Algo | –8.17% lower than JBSPO-BH                    | Strong baseline, still beaten            |
| **Fairness (min throughput per cell)** | LEO multi-cell | **JBSPO-BH** | Ensures non-zero service to all cells         | Best fairness                            |
|                           |                  | Greedy/Max-SINR          | Some cells underserved (min < 50%)            | Weak fairness                            |
|                           |                  | Round-Robin              | Poor balance, ignores demand                  | Worst fairness                           |
| **Convergence Speed**     | LEO BH network   | **JBSPO-BH**             | Converges in tens of iterations (fast)        | Practical for real-time                  |
|                           |                  | Others                   | Either faster but unstable, or inefficient    |                                           |
| **Computational Complexity** | LEO system   | **JBSPO-BH**             | Polynomial time, scalable                     | Suitable for real deployments            |
| **Robustness**            | Varying traffic / fading | **JBSPO-BH**   | Maintains throughput & fairness advantages    | Outperforms all baselines                |

---

## Core Summary

1. **Problem:** How to jointly optimize **beam scheduling and power allocation** in dynamic LEO beam hopping systems.  
2. **Solution:** JBSPO-BH → decouples into **potential game (beam scheduling)** + **interior point method (power allocation)**.  
3. **Verification:** Custom simulator, comparing against Greedy, Round-Robin, Max-SINR, and baseline methods.  
4. **KPIs:** Throughput improvement (+8% to +156% depending on baseline), fairness across cells, convergence speed, and computational efficiency.  

---

## Key Takeaways
- JBSPO-BH integrates **game theory + optimization** for beam hopping.  
- Significantly outperforms classical heuristics in **throughput** and **fairness**.  
- Verified with simulations, showing up to **156% throughput improvement** over round-robin BH.  
- Promising candidate for real-world LEO satellite NTN scheduling.

---

## 1. Introduction

- **Background:**  
  - The growing demand for broadband internet via **LEO satellite constellations** introduces challenges in resource allocation.  
  - **Beam Hopping (BH)** is a promising technique: instead of illuminating all beams simultaneously (which is power-intensive and hardware-limited), satellites switch beams across time slots to cover users in different cells.  
  - However, existing BH studies mainly consider **beam scheduling only**, neglecting the role of **power allocation across beams**, which is critical for managing interference and maximizing throughput.

- **Problem:**  
  - How to jointly optimize **beam scheduling** (which beams are active in each slot) and **power allocation** (how much power each active beam gets).  
  - The optimization must account for **time-varying traffic demands** and **geographical differences** in user distribution.  
  - Existing methods (greedy, round-robin, Max-SINR) either lack fairness, suffer from high interference, or don’t scale well.

- **Contributions of the Paper:**  
  1. Proposes a **Joint Beam Scheduling and Power Optimization algorithm for Beam Hopping (JBSPO-BH)**.  
  2. Models **beam scheduling** as a **potential game**, ensuring stable equilibrium.  
  3. Solves **power allocation** using the **Penalty Function Interior Point Method**, ensuring feasible solutions under constraints.  
  4. Provides simulation comparisons against multiple baselines (greedy, round-robin, Max-SINR, resource allocation algorithms).  
  5. Demonstrates significant throughput and fairness improvements, with low complexity suitable for real-time use.

---

## 2. Methodology

### 2.1 Joint Optimization Formulation
- **Objective:** Maximize total system throughput while maintaining fairness and respecting power/interference constraints.  
- **Constraints:**  
  - Limited number of beams active per time slot.  
  - Total transmit power budget across beams.  
  - Interference avoidance and SINR thresholds for each user.

- **Decomposition:**  
  - The joint problem is NP-hard → decoupled into two subproblems:
    1. **Beam Scheduling Subproblem:**  
       - Modeled as a **potential game**.  
       - Each “player” (beam or user cell) chooses strategies (beam activation) to maximize its payoff (achievable rate).  
       - Solution: find **Nash Equilibrium (NE)**, ensuring a stable schedule without conflicts.  
    2. **Power Allocation Subproblem:**  
       - Given the active beam schedule, optimize power distribution.  
       - Solved via **Penalty Function Interior Point Method**, which handles inequality constraints smoothly.  
       - Objective: allocate power proportionally to maximize SINR and fairness.

### 2.2 Algorithm: JBSPO-BH
- **Workflow:**  
  1. Initialize scheduling and power distribution.  
  2. Apply potential game iterations to converge beam schedule → NE.  
  3. Optimize power allocation using interior point method.  
  4. Repeat until convergence.  
- **Complexity:** Polynomial time, much lower than exhaustive search.  
- **Stability:** Game-theoretic design guarantees convergence.  

### 2.3 Baselines for Comparison
- **Greedy BH:** Activate beams with highest instantaneous SINR.  
- **Greedy BH + Power:** Same as above, but adds power allocation.  
- **Round-Robin BH:** Beams activated in fixed cyclic order.  
- **Max-SINR BH:** Always choose beams with maximum SINR without fairness consideration.  
- **Satellite Resource Allocation Algorithm (baseline):** Classical heuristic.  

---

## 3. Results

### 3.1 Throughput Performance
- **JBSPO-BH vs Other Methods:**  
  - +44.99% throughput vs Greedy BH.  
  - +20.79% vs Greedy BH + Power.  
  - +156.06% vs Round-Robin BH.  
  - +15.39% vs Max-SINR BH.  
  - +8.17% vs Satellite Resource Allocation Algorithm.  

- **Interpretation:**  
  - Round-Robin performs worst because it ignores traffic dynamics.  
  - Greedy improves, but without fairness mechanisms some cells are underserved.  
  - JBSPO-BH balances throughput and fairness effectively.

### 3.2 Fairness and Satisfaction
- **JBSPO-BH** ensures all cells achieve **non-zero service levels**, preventing starvation.  
- Minimum throughput per cell is significantly higher than greedy/random baselines.  
- Traffic satisfaction (served vs requested) much more balanced across geographic regions.

### 3.3 Convergence and Complexity
- Algorithm converges within **tens of iterations**, practical for real-time scheduling.  
- Lower computational cost compared to exhaustive search-based optimizations.  

### 3.4 Robustness
- JBSPO-BH maintains performance advantage under:  
  - Varying traffic intensities.  
  - Different geographic distributions.  
  - Power budget constraints.  

---

# 🔑 Summary

- **Problem:** Beam hopping in LEO needs joint **scheduling + power allocation** to handle dynamic traffic.  
- **Solution:** JBSPO-BH = **potential game (beam scheduling)** + **interior point method (power allocation)**.  
- **Verification:** Custom-built simulator, comparing against greedy, round-robin, Max-SINR, and baseline algorithms.  
- **KPIs:** Throughput improvement (+8%–156%), fairness (minimum throughput per cell), convergence speed, and robustness.

---

# Glossary of Terms 

This glossary explains new technical terms from Paper 3, avoiding overlap with the previous glossaries.

---

## Joint Optimization
- **Definition:** A mathematical approach where two or more interdependent problems (beam scheduling + power allocation) are solved together.  
- **Simple:** Like planning both your fishing route and how much fuel to use at the same time, instead of deciding separately.

## Potential Game
- **Definition:** A class of game theory models where the incentives of all players can be represented by a single global “potential function.” Players improving their own payoff also improve the system’s objective.  
- **Simple:** Imagine every student trying to improve their grade; if the average grade also improves, the game is “potential.”

## Nash Equilibrium (NE)
- **Definition:** A stable state in a game where no player can improve their payoff by changing their strategy alone.  
- **Simple:** Like two fishermen agreeing on fishing spots—if either moves alone, both lose.

## Interior Point Method
- **Definition:** A mathematical optimization technique for solving constrained nonlinear problems by exploring the interior of feasible regions rather than boundaries.  
- **Simple:** Instead of walking around the fence of a field, you cut straight across the middle to reach the best spot.

## Penalty Function
- **Definition:** A method in optimization where violations of constraints (like exceeding power budget) are penalized in the objective function.  
- **Simple:** Like paying a fine if you overfish—so you avoid breaking the rule.

## JBSPO-BH
- **Definition:** Acronym for **Joint Beam Scheduling and Power Optimization for Beam Hopping**. The paper’s proposed algorithm combining game theory (scheduling) and optimization (power allocation).  
- **Simple:** A smarter way to decide “which beams to turn on” and “how much power each gets” at the same time.

## Resource Allocation Algorithm
- **Definition:** General class of algorithms that distribute limited resources (time, power, bandwidth) among users or beams.  
- **Simple:** Like a teacher dividing snacks among kids fairly.

## Max-SINR Scheduling
- **Definition:** A baseline method that always activates beams/users with the maximum signal quality (SINR).  
- **Simple:** Always serving the students with the loudest voices, ignoring the quieter ones.

## Round-Robin Scheduling
- **Definition:** Scheduling beams/users in a fixed cyclic order regardless of demand or quality.  
- **Simple:** Taking turns one by one, even if some don’t need it.

## Greedy Algorithm (in Beam Scheduling)
- **Definition:** Chooses beams that maximize short-term gains (like highest SINR or largest demand) without considering long-term fairness.  
- **Simple:** Always grabbing the biggest fish first, without thinking about tomorrow.

## Convergence (Algorithm)
- **Definition:** The point where iterative optimization stops changing significantly, meaning a stable solution is reached.  
- **Simple:** Like a boat finally settling into calm water after rocking.

## Time Complexity
- **Definition:** A measure of how the computation time of an algorithm grows with problem size (number of beams, users).  
- **Simple:** How long your fishing trip takes if you add more boats—does it grow slowly or explode?

## NP-Hard
- **Definition:** A class of problems that are computationally intractable to solve exactly within polynomial time.  
- **Simple:** Like trying to test every possible way of fishing in the ocean—it would take forever.

## Throughput Gain (%)
- **Definition:** Improvement in data rate achieved by one algorithm compared to a baseline.  
- **Simple:** Like saying “this fishing method gets 50% more fish than the old one.”

## Traffic Demand Dynamics
- **Definition:** Variations in user data needs across time and geography in LEO networks.  
- **Simple:** Some villages want lots of fish today, some want less, and it changes daily.

---
