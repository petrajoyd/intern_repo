# Paper List: Scheduling and Routing Algorithms for 3GPP NTN & DVB-S2X/RCS2

## IEEE Transactions & Conference Papers

### 1. System-Level Evaluation of Beam Hopping in NR-Based LEO Satellite Communication System
- **Authors:** J. Zhang, D. Qin, C. Kong, F. Zhao, R. Li, J. Wang, Y. Wang  
- **Source:** IEEE WCNC, 2023  
- **Why:** Canonical IEEE paper on **beam hopping scheduling** in NR-based LEO NTN systems. Directly matches 3GPP NTN scheduling under satellite mobility.  
- **Link:** [Free arXiv PDF](https://arxiv.org/abs/2205.10721)

### 2. Satellites Beam Hopping Scheduling for Interference-Aware SAT-Terrestrial Systems
- **Authors:** H. Deng, et al.  
- **Source:** IEEE Journal on Selected Areas in Communications (JSAC), 2024  
- **Why:** Explores **beam hopping scheduling** under interference constraints, citing DVB payload models; aligns with DVB-S2X flexible payload and scheduling concepts.  
- **Link:** [Free arXiv PDF](https://ieeexplore.ieee.org/document/10713888)

### 3. Joint Beam Scheduling and Power Optimization for Beam Hopping LEO Satellite Systems
- **Authors:** S. Zheng, et al.  
- **Source:** IEEE WCNC/Globecom Workshop, 2023  
- **Why:** Proposes **joint scheduling + power optimization** for LEO beam hopping. Relevant to DVB-S2X scheduling (forward link) under beam-hopping architectures.  
- **Link:** [Free arXiv PDF](https://arxiv.org/abs/2312.01292)


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
