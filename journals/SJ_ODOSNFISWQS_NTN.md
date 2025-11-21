# Skim Paper/Journal: Part 5
Journal Name = Overall Design of Satellite Networks for Internet Services with QoS Support.</br>
Authors =  Kyu-Hwan Lee, Kyoung Youl Park. </br>
BibTex = [electronics8060683]
```
@Article{electronics8060683,
AUTHOR = {Lee, Kyu-Hwan and Park, Kyoung Youl},
TITLE = {Overall Design of Satellite Networks for Internet Services with QoS Support},
JOURNAL = {Electronics},
VOLUME = {8},
YEAR = {2019},
NUMBER = {6},
ARTICLE-NUMBER = {683},
URL = {https://www.mdpi.com/2079-9292/8/6/683},
ISSN = {2079-9292},
ABSTRACT = {The satellite network is useful in various applications because of its coverage, broadcast capability, costs independent of the distance, and easy deployment. Recently, thanks to the advanced technologies in the satellite communication, the high throughput satellite system with mesh connections has been applied to the Internet backbone. In this paper, we propose a practical overall design of the satellite network to provide Internet services with quality of service (QoS) support via the satellite network. In the proposed design, we consider two service types such as delay-tolerance and delay-sensitive services allowing the long propagation delay of the satellite link. Since it is crucial to evaluate the user satisfaction in the application layer for various environments to provide the QoS support, we also define the performance metrics for the user satisfaction and derive the major factors to be considered in the QoS policy. The results of the performance evaluation show that there are factors such as the traffic load and burstiness in the QoS policy for the delay-tolerance service with volume-based dynamic capacity in the satellite network. For the delay-sensitive service with rate-based dynamic capacity, it is additionally indicated that it is important for the estimation of effective data transmission rate to guarantee the QoS. Furthermore, it is shown that the small data size has an effect on the slight reduction of the QoS performance in the satellite network.},
DOI = {10.3390/electronics8060683}
}
```

# About The Paper (Briefly)
This paper proposes a practical, overall design for a high-throughput satellite network intended to provide Internet services with Quality of Service (QoS) support, specifically addressing the challenges of a GEO satellite's long propagation delay. The system model is based on the DVB-RCS standard, featuring a GEO satellite with on-board processing to enable mesh connections, and a MF-TDMA (Multi-Frequency Time Division Multiple Access) link layer. The core of its QoS architecture is a DAMA (Demand Assigned Multiple Access) scheme that separates traffic into two types: delay-tolerant services (like file transfers) managed by Volume-Based Dynamic Capacity (VBDC), and delay-sensitive services (like video streaming) managed by Rate-Based Dynamic Capacity (RBDC). The paper argues that to create an effective QoS policy, it is crucial to evaluate "user satisfaction" at the application layer, not just network-level metrics. To accomplish this, it defines and measures performance using "Satisfaction Ratio" (the percentage of packets arriving within a delay constraint) and "Goodput" (the total data volume arriving within that constraint). Through event-driven simulations, the paper evaluates this entire design and successfully identifies the most critical factors for a QoS policy: for delay-tolerant services, traffic load and burstiness are the key factors; for delay-sensitive services, the "estimation of effective data transmission rate" is an additional critical factor, as simple averaging fails for bursty video. Finally, it also shows that a high number of users with small data packets (like IoT) can cause a "slight reduction" in QoS due to inefficient time-slot packing.

# Motivation x Problem Statement
- Motivation: Future networks need massively more capacity for growing user traffic and new applications, all while guaranteeing QoS. Satellite networks are a "cost-effective solution" to provide this internet access, especially in "rural or other difficult-to-serve areas". Modern High Throughput Satellite (HTS) systems, even in geostationary orbit (GEO), can now act as an "Internet backbone".
- Problem Statement: Providing reliable QoS over satellites is difficult due to their "inherent characteristics such as a long propagation delay". While many individual technologies for satellite QoS exist (like routing or admission control), the authors state that, to their knowledge, a "whole network-level performance evaluation and design" has not been properly explored. It's crucial to evaluate "user satisfaction in the application layer" while accounting for "practical network performance considering control and protocol overheads".

# System/Network Model
The paper proposes a detailed system model based on the DVB-RCS standard, visualized in Figures 1.
<img width="676" height="370" alt="image" src="https://github.com/user-attachments/assets/a5132620-0ff6-44b7-b63d-e5ea1f887df7" />
- Architecture: It uses a GEO satellite with on-board processing. This allows for direct mesh connections, meaning User Terminals (UTs) can communicate directly with each other through the satellite without going to a hub first.
- Components (Fig. 1):
  - User Terminals (UTs): The end-user devices.
  - Hub Terminal: A central ground station that manages the network. It contains the Network Control Center (NCC) for resource allocation and the Network Management Center (NMC) for policies and admission control.
  - Control Channels: The hub uses a Forward Control Channel (FCC) to send commands, and UTs use a Return Control Channel (RCC) and Log-On Channel (LOC) to send requests and status updates.

# Scheduling & Routing Mechanisms
The design specifies mechanisms for both the link and network layers.

### Scheduling (Link Layer):
- The system uses MF-TDMA (Multi-Frequency Time Division Multiple Access).
- The core scheduling scheme is DAMA (Demand Assigned Multiple Access), where the NCC grants resources to UTs based on their requests.
- To support QoS, it uses three DAMA types:
  - VBDC (Volume-Based Dynamic Capacity): For delay-tolerant services. A UT requests a volume of data. This method has a long resource allocation delay (3-4 super-frames).
  - RBDC (Rate-Based Dynamic Capacity): For delay-sensitive services (like video). A UT requests a specific data rate, which helps "eliminate the resource allocation delay".
  - CRA (Continuous Resource Assignment): A third mode (likely for constant, high-priority traffic).

### Routing (Network Layer - Fig. 2):
<img width="693" height="624" alt="image" src="https://github.com/user-attachments/assets/55e07ac7-8fca-499d-be39-91cd66a5046d" />
- The network uses a hierarchical routing structure to reduce overhead.
- The entire network is divided into multiple Autonomous Systems (ASs), which might represent different service providers or beams.
- OSPF (Open Shortest Path First) is used for routing traffic inside a single AS.
- BGP (Border Gateway Protocol) is used for routing traffic between different ASs.
- This means traffic between users in the same AS is a single satellite hop (UT -> Sat -> UT). Traffic between users in different ASs becomes multi-hop (e.g., UT -> Sat -> Hub -> Sat -> UT).

# QoS Metrics Used
The paper evaluates performance by defining two custom metrics to measure "user satisfaction" at the application layer, in addition to standard ones.
1. Throughput: The total data rate (Mbps) in the network.
2. 95th percentile end-to-end delay: A standard measure of delay experienced by the vast majority of packets.
3. Satisfaction Ratio (Custom): The percentage of data packets that successfully arrive at the destination within the specified delay constraint.
4. Goodput (Custom): The total volume of data that successfully arrives at the destination within the specified delay constraint.

# Performance Gains
The paper doesn't claim "gains" over another system, but rather identifies the key factors that a QoS policy must consider to be effective.
- For Delay-Tolerant Services (VBDC): The performance (Satisfaction Ratio and Goodput) is heavily affected by traffic load and burstiness. Even if the load is low, high burstiness (spiky traffic) causes instantaneous congestion, which increases delay and severely degrades the Satisfaction Ratio.
- For Delay-Sensitive Services (RBDC): In addition to load and burstiness, the most important factor is the "estimation of effective data transmission rate". The simulation showed that using a simple average data rate for RBDC is an "imperfect estimation" for variable-bit-rate (VBR) services like video, leading to packet loss and reduced QoS.
- Impact of Packet Size: When the number of users increases (and average data size per user decreases), there is a "slight reduction of the QoS performance". This is because small data packets don't efficiently fill the fixed MF-TDMA time slots, wasting capacity.

# How The Author Verifies The Ideas
The authors verified their design and findings entirely through simulation.
- Tool: They built an event-driven simulator using Riverbed modeler (Version 18.6).
- Model: They implemented their entire "overall design" in the simulator, including the DVB-RCS link layer (VBDC, RBDC), the network layer (OSPF, BGP), and the QoS architecture .
- Methodology: They ran three distinct scenarios to test different parts of the design:
  - Scenario 1: Focused on delay-tolerant (VBDC) service, varying load and burstiness.
  - Scenario 2: Focused on delay-sensitive (RBDC) service, varying load and burstiness.
  - Scenario 3: Focused on VBDC service, varying the number of UTs to see the effect of smaller data packet sizes.

# Limitation
The paper's conclusion highlights the inherent challenges (limitations) of this environment.
- Inherent GEO Problems: The paper states that in satellite communication, it "has not been easy to guarantee the QoS" because of the "long propagation delay, the resource allocation time, [and] the large super-frame size".
- Imperfect Estimation: A key weakness identified is the "imperfect estimation of the required resource" , especially for the RBDC mode used for delay-sensitive services. The paper points this out as a major problem to be solved.
- Other Factors Not Modeled: The paper mentions other important technologies like Performance Enhancing Proxies (PEP) and On-The-Move (OTM) terminals but lists them as "Other Considerations" that were not part of the core performance evaluation.
