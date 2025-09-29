# Paper List: Scheduling and Routing Algorithms for 3GPP NTN & DVB-S2X/RCS2

## IEEE Transactions & Conference Papers

### 1. Multicast Multigroup Precoding and User Scheduling for Frame-Based Satellite Communications
- **Authors:** D. Christopoulos, S. Chatzinotas, B. Ottersten  
- **Source:** *IEEE Transactions on Wireless Communications*, 2015  
- **Why:** Canonical IEEE Transactions paper on frame-based satellite scheduling + multicast group scheduling and precoding; highly relevant to DVB-S2X forward link frame scheduling problems.  
- **Link:** [arXiv preprint](https://arxiv.org/abs/1406.7699))

### 2. Terminal-Aware Multi-Connectivity Scheduler for Uplink Multi-Layer Non-Terrestrial Networks
- **Authors:** M. Dazhi, et al.  
- **Source:** *IEEE Globecom Workshops*, 2022  
- **Why:** Explicit scheduler design for multi-orbit NTN and multi-connectivity — directly applicable to 3GPP NTN solutions (multi-connectivity and terminal-aware scheduling).  
- **Link:** [University of Luxembourg ORBilu](https://orbilu.uni.lu/handle/10993/52241)  

### 3. Enhanced Hybrid ARQ Scheduling for Non-Terrestrial IoT Networks
- **Authors:** G. Prasad, et al.  
- **Source:** *IEEE Open Journal of the Communications Society*, 2023  
- **Why:** Tackles HARQ / long RTT problems in NTNs and proposes transport-block scheduling methods that improve throughput for CIoT over NTN — important for uplink reliability and scheduler design.  
- **Link:** [arXiv preprint](https://arxiv.org/abs/2305.07114)


## Standards (Authoritative Sources)

### 4. 3GPP NTN Technical Reports
- **TR 38.811:** Study on New Radio (NR) to Support Non-Terrestrial Networks  
- **TR 22.822:** Study on Support for NR to Support Non-Terrestrial Networks  
- **TR 38.821:** Solutions for NR to Support Non-Terrestrial Networks  
- **Why:** Standards context — lists the procedures (multi-connectivity, mobility, HARQ timing, resource assignment) that any scheduler for 3GPP NTN must respect.  
- **Link:** [3GPP Portal](https://www.3gpp.org/technologies/ntn-overview)  

### 5. DVB Bluebooks / Standards
- **DVB-S2X:** Digital Video Broadcasting - Second Generation Extensions  
- **DVB-RCS2:** Digital Video Broadcasting - Return Channel via Satellite (2nd Gen)  
- **Why:** Authoritative source for how scheduling / resource allocation is defined (DAMA, dynamic connectivity, superframe/frame formats).  
- **Link:** [DVB Standards]((https://dvb.org/wp-content/uploads/2019/12/a155-2_dvb-rcs2_lower_layers_satellite_v1_3_1_update.pdf))  

---

## Surveys & Tutorials

### 6. Comprehensive Surveys on NTN & Satellite Resource Management
- **Example:** “A Survey on Non-Terrestrial Networks (NTN): Architecture, Resource Management, and Challenges.” (*IEEE Communications Surveys & Tutorials*)  
- **Why:** Provides overview of algorithm categories (AI/DRL schedulers, multi-connectivity, precoding + scheduling). Great for context and trade-off discussions.  
- **Link:** [IEEE Xplore Search](https://ieeexplore.ieee.org/Xplore/home.jsp)  
