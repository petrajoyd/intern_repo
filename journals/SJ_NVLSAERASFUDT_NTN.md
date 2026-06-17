# Skim Paper/Journal: Part 4.
Journal name = NB-IoT via LEO satellites: An efficient resource allocation strategy for uplink data transmission.</br>
Authors = Oltjon Kodheli, Nicola Maturo, Symeon Chatzinotas, Stefano Andrenacci, Frank Zimmer.</br>
BibTex = [@kodheli]
```
@article{kodheli,
author = {Kodheli, Oltjon and Maturo, Nicola and Andrenacci, Stefano and Zimmer, Frank},
year = {2021},
month = {09},
pages = {1-1},
title = {NB-IoT via LEO Satellites: An Efficient Resource Allocation Strategy for Uplink Data Transmission},
volume = {PP},
journal = {IEEE Internet of Things Journal},
doi = {10.1109/JIOT.2021.3109456}
}
```

# About The Paper (Briefly)
This paper proposes a novel uplink resource allocation strategy for Narrowband Internet of Things (NB-IoT) systems operating over Low-Earth Orbit (LEO) satellites. It addresses the inapplicability of conventional terrestrial algorithms, which fail due to three key LEO-specific impairments: 
1) the highly dynamic system, where rapid satellite movement causes channel parameters to become outdated;
2) the high propagation delay (4-16x terrestrial), which demands minimal message exchange; and
3) the significant differential Doppler shift between users, which violates NB-IoT standard limits if not managed.

The paper's strategy tackles these issues by first separating the access and data phases, then modeling the data phase scheduling as a 0-1 Two-dimensional Knapsack Problem (2D-KP). To manage the differential Doppler, users are grouped by location, and to reduce complexity for the dynamic system, the problem is simplified to a 0-1 Multiple Knapsack Problem (MKP) by enforcing single-tone transmission. The paper introduces a weighted profit function that prioritizes users based on their buffer status, channel conditions (MCS), and remaining satellite coverage time. To solve this MKP, a low-complexity "Approximate" algorithm is proposed, which simulations show achieves near-optimal throughput (101 kbit/s vs. 103 kbit/s for the exact solution) with a fraction of the computational time. Finally, system-level simulations of 600,000 UEs demonstrate the framework's effectiveness and quantify the performance trade-offs between maximizing throughput and ensuring fairness for users in different coverage levels.

# Motivation x Problem Statement
## Motivation: 
The core motivation is to use LEO satellites to provide Narrowband Internet of Things (NB-IoT) connectivity to on-ground User Equipments (UEs). Terrestrial-only networks can't guarantee global coverage, and satellites are "inevitable" for areas with no terrestrial infrastructure. This work is timely, as 3GPP (the 5G standards body) is actively working on integrating non-terrestrial networks (NTN) into the 5G standard (Releases 16 and 17).</br>
## Problem Statement: 
The paper states that conventional resource allocation algorithms designed for terrestrial NB-IoT systems "cannot be applied over LEO satellite-based NB-IoT systems". This is because the LEO environment introduces three unique, show-stopping problems:
- A Highly Dynamic System: LEO satellites move very fast. This means channel parameters for a user change quickly. If the base station delays scheduling a user, the resource allocation will be "based on outdated parameters". In terrestrial systems, the channel "varies very slowly in time".
- Differential Doppler Shift: This is a major impairment. While the main Doppler shift can be corrected, the difference in Doppler shift between users in different locations is a problem. If users that are too far apart are scheduled in the same radio frame, the differential Doppler would "violate the differential Doppler limit supported by the NB-IoT standard".
- Increased Propagation Delay: The Round Trip Delay (RTD) over a LEO satellite is 4-16 times higher than in a terrestrial system. This high delay "impos[es] the need for message exchange minimization" between the UEs and the base station.

The paper's goal is to propose a "novel uplink resource allocation strategy" that jointly solves all these LEO-specific challenges while also considering traditional metrics like channel conditions and user data demands.

# System / Network Model
The paper bases its strategy on a specific system model and a set of key assumptions (visualized in Figure below).

<img width="732" height="444" alt="image" src="https://github.com/user-attachments/assets/8b69563f-490b-49c4-9412-5a9b3183abcc" />

### System Architecture:
#### Satellite: 
A LEO satellite with a "transparent payload". This means the satellite is just a simple "bent-pipe" relay and doesn't do any smart processing on board.
#### Ground Segment: 
The satellite connects on-ground UEs to an on-ground Gateway (GW), which is then connected to the Serving Base Station (BS). The BS handles all the resource allocation logic.
#### Cell Movement: 
The system uses an "Earth-moving cell". The satellite's beam "moves" across the Earth as the satellite orbits, and UEs continuously enter and leave the coverage area.

#### Key Assumptions: 
The authors make three critical assumptions to make the problem solvable:
1. Fixed user location on Earth: The IoT UEs are assumed to be stationary, with no mobility. This allows the BS to know their geographical coordinates.
2. Perfect... compensation of the common Doppler shift at the GW: The model assumes the Gateway (GW) perfectly estimates and removes the main Doppler shift experienced by all users. This is crucial because it isolates the differential Doppler shift as the only remaining Doppler problem to be solved.
3. Separation of access and data phases: The paper proposes separating the "Access Phase" (where UEs first contact the network) from the "Data Phase" (where they send their data). This is a key design choice. It allows the BS to control the differential Doppler only during the data phase via a resource allocation strategy. It also helps the "highly dynamic system" by allowing the BS to make "proactive 'short-term' resource allocation decisions" based on the most recent channel estimates from the UEs.

# Scheduling & Routing Mechanisms
The paper's core contribution is its scheduling mechanism. (Note: It focuses entirely on resource allocation/scheduling, not on routing). The mechanism models the problem as a Knapsack Problem.

## The Initial Problem
0-1 2D Knapsack Problem (2D-KP) The task of scheduling UEs into a radio frame is modeled as a 0-1 Two-dimensional Knapsack Problem.
- The Knapsack: This is the available resource grid for the data phase. It has a width W (time, in subframes) and a height B (frequency, in subcarriers)
- The Items: These are the UEs that need to be scheduled. Each $UE_i$ is a 2D rectangle with a time duration ($w_i$) and a frequency bandwidth
- The Goal: Pack the "items" (UEs) into the "knapsack" (resource grid) to maximize the total "profit" ($P_i$) of the scheduled
- The Doppler Constraint: A critical constraint (Eq. 13) is added to handle the Doppler problem. UEs are first divided into groups based on their physical location. The constraint only allows users from the same group to be scheduled together in the same time subframe, which guarantees the differential Doppler limit is not violated.

## The Simpliication:
0-1 Multiple Knapsack Problem (MKP) The 2D-KP is "strongly NP hard" and too slow for a dynamic LEO system. The authors simplify it with a "crucial" step:
- Forcing Single-Tone Transmission: They "relax the problem" by fixing the transmission mode to single-tone only. This means every UE can only use 1 subcarrier ($b_i = 1$).
- The New Problem: This "converts the problem" from a 2D one into a much simpler 0-1 Multiple Knapsack Problem (MKP).
  - The Knapsacks (plural): The single 2D grid becomes $B=12$ separate 1D knapsacks (one for each of the 12 subcarriers in the NB-IoT channel).
  - The Items: UEs are now 1D items, defined only by their "weight," which is their time-length $w_k$.The Goal: Pack the 1D UEs into the 12 1D knapsacks to maximize total profit.
 
### The Scheduling Algorithms
The paper then compares three algorithms for solving this final MKP:
- Exact Solution (B&B): A complex "Bound and bound (B&B)" algorithm that finds the perfect optimal solution but is slow.
G- reedy Algorithm: A standard, fast algorithm that sorts UEs by their profit-per-weight ($P_k / w_k$) and fills the knapsacks.
- Approximate Solution (Proposed): The authors' novel, low-complexity algorithm44. It first uses a Greedy approach on one merged knapsack, then re-sorts the selected UEs by their weight (largest first) and places them into the individual knapsacks with the most remaining capacity.

# QoS Metrics Used
The paper does not use traditional QoS metrics like delay or jitter. Instead, it defines a flexible profit function as its core scheduling metric and then evaluates the system's performance using three Key Performance Indicators (KPIs).

## 1. The Profit Function (The "Scheduler's Brain"):
The BS assigns a "profit" $P_k$ to each user based on a weighted formula (Eq. 35). The weights ($\alpha, \beta, \gamma$) can be tuned to prioritize different goals:
- Priority $\alpha$ (Buffer): Gives priority to users with a high buffer status ($D_k$).
- Priority $\beta$ (Channel): Gives priority to users with good channel conditions (higher $MCS_k$).
- Priority $\gamma$ (Coverage): Gives priority to users with low satellite coverage time ($t_k^{sat}$), i.e., users who are about to lose connection.

## 2. The Key Performance Indicators (KPIs):
These are used to evaluate the performance of the scheduling policy:
- KPI 1: Data Phase Throughput (R): The total bits transmitted divided by the total time. This measures pure system efficiency.
- KPI 2: User Fairness (J): Uses the Jain's Fairness Index (Eq. 37) to measure if all individual users are getting a fair share of service based on their data demand.
- KPI 3: Group Fairness ($J_{CL}$): Also uses the Jain's Index (Eq. 38), but measures fairness between different coverage levels (CL 1: good, CL 3: bad). This ensures users at the edge of the satellite beam aren't being completely ignored.

# Performance Gains (Fig. 8, 9, 15)
The paper analyzes the "gains" as a series of trade-offs, showing what the system achieves when each KPI is maximized. </br>
## Algorithm Gains :
The proposed Approximate algorithm is shown to be the clear winner for practical use:
### Profit/Throughput: 
It achieves a maximum throughput of 101 kbit/s, which is "very close" to the perfect optimal solution (103 kbit/s) and much better than the Greedy algorithm (90 kbit/s).
### Complexity: 
It has a "near-optimal performance and very low complexity". Its running time is extremely fast, similar to the Greedy algorithm, while the B&B algorithm's time increases exponentially (Fig. 9).
### Fairness: 
For the two fairness KPIs, all three algorithms performed identically.
## System Performance Trade-offs (Fig. 11-14):
This shows the impact of tuning the profit function weights:
### Maximizing KPI 1 (Throughput): 
By setting $\beta=0.7$ (prioritizing good channels), the system achieved its highest throughput of 103 kbit/s. The cost was terrible fairness: it served a "narrower beam" of users and ignored those at the edge of the coverage area.
### Maximizing KPI 2 (User Fairness): 
This (setting $\gamma=0.6$) provided a middle ground, achieving 77 kbit/s.
### Maximizing KPI 3 (Group Fairness): 
By setting $\alpha=0.8$ (prioritizing buffer status), the system achieved the "best distribution of services among users in different coverage levels". The cost was the lowest throughput, at only 58 kbit/s61.
### Maximizing KPI 2 (User Fairness): 
This (setting $\gamma=0.6$) provided a middle ground, achieving 77 kbit/s.

<img width="272" height="635" alt="image" src="https://github.com/user-attachments/assets/fdc54ec8-ffcd-49ce-b019-b1b56163a981" /> <img width="272" height="635" alt="image" src="https://github.com/user-attachments/assets/9f586d1e-38ff-4c70-9bc8-c9f8fee09d13" /> <img width="272" height="635" alt="image" src="https://github.com/user-attachments/assets/4083a759-6fc1-485d-aa93-0658577f6dc3" />

# How The Author Verifies The Ideas
The authors verify their ideas in two main stages using MATLAB simulations.
## 1. Algorithm Comparison (Verification of Section V):
### What: 
They first verify that their "Approximate" scheduling algorithm is actually good. They do this by comparing its Sum Profit (Fig. 8) and Running Time (Fig. 9) against the "Exact (B&B)" and "Greedy" algorithms.
### How: 
They run 1000 iterations for different problem sizes (150 to 500 users) 65for a system with 12 knapsacks (subcarriers) and a capacity of 160 subframes. This test proves their algorithm is fast and near-optimal.
## 2. System-Level Simulation (Verification of Section VI):
### What: 
They then verify the entire system's performance by running a large-scale simulation and tuning the profit function weights ($\alpha, \beta, \gamma$) to see the effect on the three KPIs.
### How: 
This is a much larger simulation of a LEO satellite at 1000 km altitude 69moving over a 4000 km geographical area70. The simulation includes:
- 600,000 UEs uniformly distributed.
- A 3GPP-standard IoT traffic model (Pareto distribution, 20-200 Byte packets).
- 20 distinct user groups to manage the differential Doppler, calculated for a 2 GHz carrier frequency.
- The simulation is run for 480 satellite passages to get long-term average results.
- Result: This simulation generated the 3D plots (Fig. 11-13) and color maps (Fig. 14) that prove the performance trade-offs.

# Limitations
The authors explicitly state the paper's limitations in the conclusion, which they label as "future work".
- Access Phase is Ignored: The biggest limitation. The paper's solution for differential Doppler only works for the Data Phase. The authors admit the Doppler problem "will also impact the access phase" and that they have not solved this.
- Static Traffic Assumption: The paper assumes a "fully-loaded system" where there are always enough UEs to schedule. It doesn't account for the satellite flying over areas with different user densities (e.g., a city vs. an ocean). They plan to create a future model that can "dynamically change the access phase duration" to adapt to this.
- Single Satellite Focus: The entire analysis is for a single satellite. It does not address the "future work" of a complete "satellite constellation design targeting a global NB-IoT coverage".
