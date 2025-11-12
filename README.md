# 📚 2025-Fall-MS-Joy **(Internship Documentation)**

## 📑 Table of Contents
<!-- (GitHub auto-generates ToC based on headers. You may use a Markdown TOC generator plugin for static ToC if needed.) -->

---

## 🏁 Short-Term Goals

### 1. Study DVB-S2X and DVB-RCS2 [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/study_notes)
- [x] 3.1 DVB-S2X & DVB-RCS2 Standard
- [x] 3.2 Study Paper Related to DVBs
- [ ] 3.3 Evaluate the Performance of the Algorithm Using Xeoverse & ns-3 NTN

### 2. Make a 50 Slide PPT About [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft):
- [x] 4.1 System Architecture of the DVB-E2E System 
- [x] 4.2 DVBs Protocol Stack
- [x] 4.3 L1 Frame Structure
- [x] 4.4 DVBs Logical and Physical Channels
- [x] 4.5 Store All Visual Code in GitHub repository

### 3. Learn about NS-3 Satellite Module [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md):
- [x] 5.1 Foundation Check: Understand directory structure and helper-channel interaction
- [x] 5.2 Device & PHY-MAC Exploration: `SatNetDevice`, `SatPhy`, `SatMac`
- [x] 5.3 Tracing and Logging System: `satellite-log.cc`, `satellite-packet-trace.cc`, `PacketTrace.log`
- [x] 5.4 Channel & Propagation Mastery: `satellite-channel.cc`, `satellite-propagation-delay-model.cc`, `satellite-fading-models`
- [x] 5.5 Scenario & Example Customization: `examples/sat-tutorial-example.cc`, `sat-iot-example.cc`
- [x] 5.6 Integrating Everything: End-to-End system understanding
- [x] 5.7 Experiment & Report: Create my own minimal simulation

### 4. Learn about NTN NR Basics Fundamentals
- [ ] 4.1 3GPP NTN NR
  - [x] 4.1.1 System Architecture Diagram
  - [x] 4.1.2 Protocol Stacks
  - [ ] 4.1.3 Message Sequence Charts (MSCs)
  - [ ] 4.1.4 NTN O&M Parameters 
- [ ] 4.2 DVB-S2X & DVB-RCS2
  - [x] 4.2.1 System Architecture Diagram
  - [x] 4.2.2 Protocol Stacks
  - [ ] 4.2.3 Message Sequence Charts (MSCs)
  - [ ] 4.2.4 O&M Parameters
- [ ] 4.3 LEO RRM
  - [x] 4.3.1 Resource Management Algorithms
  - [x] 4.3.2 QoS Aware Routing Algoritms
  - [ ] 4.3.3 Beam Hopping Algoritm
---

## 📆 Daily Logs

### 2025.07.11

**🎯 Short-term Goal:**  
1. Short-Term Goal: Study DVB-S2X and DVB-RCS2 [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/study_notes)
2. Skim official DVB-S2X and DVB-RCS2 specifications (ETSI)
3. Identify core architecture components: UT, FT, Satellite
4. Sketch draft system diagram

**📝 Daily Logs:**  
- `9:00–12:00`:
  - Read basic S2X/RCS2 descriptions [Jump To: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md)
  - Jotted down key differences (DL vs UL) [Jump To Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md#dvb-s2x-direction-of-traffic-and-interfaces-downlink)
- `13:00–17:00`:
  - Found ETSI TS 102 606 (S2X) and TS 101 545-2 (RCS2) [Jump To: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md)
  - Started rough block diagram in notebook [Jump To: Study Notes](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/study_notes)

---

### 2025.07.14

**🎯 Short-term Goal:**  
1. Milestone 3: Study DVB-S2X and DVB-RCS2 [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/study_notes)
2. Study DVB-S2X and DVB-RCS2
3. Resource Management Algorithms in DVB

**📝 Daily Logs:**  
- `9:00–12:00`:
  - Draft system architecture diagrams for [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md):
    - DVB-S2X (Downlink): UT ← Satellite ← FT
    - DVB-RCS2 (Uplink): UT → Satellite → FT
  - Label all major components: UT (User Terminal), FT (Feeder Terminal/Gateway), Satellite, GWs, NMS [Jump To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md#dvb-s2x-direction-of-traffic-and-interfaces-downlink)
  - Highlight direction of traffic and interfaces [Jump To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md#direction-of-traffic)
- `13:00–15:30`:
  - Create tables/notes for both standards:
    - DVB-S2X: BBFRAME, FECFRAME, PLFRAME, SuperFrame [Jump To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB-S2X_SPECS.md)
    - DVB-RCS2: MF-TDMA frame, time slots, channels (ACM/NCC control) [Jump To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB-RSC2_SPECS.md)
  - Identify data/control/management channels (uplink vs downlink) [Jump To: Notes]((https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#dvb-s2x-downlink-architecture-forward-link))
- `15:30–17:00`:
  - Skim 1–2 key papers [Go To: Journal Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/journals/SJ_DT%3AADTFN-GSSDAV.md)
  - Note down:
    - System architecture and signaling
    - Traffic model (e.g., CBR/VBR), bandwidth allocation
    - QoS mechanism (diffserv, MPLS)
  - Jot any terms or unfamiliar algorithm

---

### 2025.07.15

**🎯 Short-term Goal:**  
1. Milestone 3: Study DVB-S2X and DVB-RCS2 [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/study_notes)
2. Deepen understanding of resource management algorithms in DVB systems
3. Extract traffic model, bandwidth allocation, and QoS control strategies
4. Begin exploring beam hopping and MPLS-based satellite routing logic
5. Lay the foundation for upcoming simulation work in Xeoverse and ns-3 NTN


**📝 Daily Logs:**  
- `9:00–10:00`:
  - Revisited DVB-S2X and DVB-RCS2 architecture diagrams [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md)
  - Cleaned up annotations on system components (FT, UT, Satellite, NMS/NCC) [Jump To: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#major-components-in-dvb-s2x--dvb-rcs2-systems)
  - Added missing interface directions and RF control flow [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md)
- `10:00–11:30`:
  - Skimmed two resource management papers [Journal 1](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/journals/SJ_IDAFMDVB_RCSS.md) [Journal 2](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/journals/SJ_AOTUODRAMFIT.md)
  - Focused on identifying traffic models:
    - CRA (constant rate)
    - RBDC (rate-based)
    - VBDC (volume-based)
    - RA (random access)
  - Started comparative matrix (traffic type × allocation method)
- `11:30–12:00`:
  - Extracted QoS approaches [Journal 1](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/journals/SJ_IDAFMDVB_RCSS.md) [Journal 2](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/journals/SJ_AOTUODRAMFIT.md):
    - DSCP-based classification (DiffServ)
    - Use of priority queues
    - TCP optimization via PEP (e.g., SCPS-TP, TCP splitting)
- `13:00–14:30`:
  - Studied beam hopping algorithm concepts [Go To: Beam Hopping Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/beam_hopping-basics.md)
    - Scheduling techniques for 4, 8, 16 beams
    - Metrics: fairness, throughput, switching delay
  - Noted factors like dwell time, user demand mapping, frequency re-use
- `14:30–15:30`:
  - Skimmed routing models with MPLS-based pathing [Go To: MPLS-BP Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/MPLS_LSP_Service-Class-Differentiation.md)
    - LSP setup logic
    - Differentiation by service class (EF, AF, BE)
    - Noted how satellite systems implement pre-defined routes
- `15:30–16:30`:
  - Launched and explored ns-3 NTN test scenario [sns3](https://github.com/sns3/sns3-satellite) [ns3/LEO](https://hackmd.io/@ycchiu514/BJPVmfInR)
    - Identified input parameters (topology, delay, MODCOD config)
    - Observed basic link behavior (UT ↔ GW)

---

### 2025.07.16

**🎯 Short-term Goal:**  
1. Milestone 3: Study DVB-S2X and DVB-RCS2 [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/study_notes)
2. Lay the foundation for upcoming simulation work in  ns-3


**📝 Daily Logs:**  
- `9:00–12:00`:
  - Finalized the ns-3 environment setup [Setup](https://github.com/sns3/sns3-satellite/blob/master/README.md).
  - Resolved the CMake dependency issue by installing the package via apt-get [Setup: CMake](https://github.com/sns3/sns3-satellite/blob/master/README.md#cmake).
  - Successfully ran ./ns3 configure and ./ns3 build to confirm a healthy, compiled project.
- `13:00–15:00`:
  - Performed a "tour" of the ns-3 directory structure to understand the layout (src, examples, scratch).
  - Analyzed the basic examples/tutorial/first.cc script to identify the five core ns-3 components (Node, Channel, NetDevice, Application, Helper).
- `15:30–17:00`:
  - Launched and explored the ntn-simple.cc test scenario with a better understanding of the ns-3 framework.
  - Identified key NTN-specific input parameters and helpers within the code (NtnEciMobilityHelper, DvbS2Helper).
  - Observed basic link behavior (UT ↔ GW) by successfully running the script and confirming packet transmission via console logs.

---

### 2025.07.21

**🎯 Short-term Goal:**  
1. **Bonus Milestone** – Pre-task Day: Admin Clearance & Initial Setup
2. Complete yudisium administration and prepare for NTUST admission
3. Pick up passport for Taiwan travel
4. Begin pre-reading and structure planning for Prof. Ray’s DVB presentation
5. Outline major sections of the 50-slide deliverable
6. Set up working environment for document drafting

**📝 Daily Logs:**  
- `9:00–11:00`:
  - Finalize yudisium administration at Universitas Brawijaya
  - Print required forms, fill out graduation documents
  - Check graduation status & confirm transcript availability
- `11:00–12:00`:
  - Go to immigration office to grab passport
  - Review NTUST visa-related steps (physical vs e-copy requirements) while waiting
- `13:00–16:00`:
  - Re-read Prof. Ray’s task description
  - Draft 50-slide presentation outline [Go To: Slide Draft](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/initial_draft.md):
    - System architecture
    - Protocol stack
    - L1 frame structure
    - Logical & physical channels
  - Set up folder structure: /slides_draft, /specs, /figures, /logs [GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
  - Collect DVB-S2X and RCS2 references into /specs folder
  - Write a short summary of what you’ll focus on first (e.g., system overview) [Go To: Slide Draft](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/initial_draft.md#slides-timeline):

---

### 2025.07.29

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Wrap up reading and note-taking on DVB-S2X and DVB-RCS2 specs
3. Extract system architecture diagrams and L1 frame references
4. Draft slides for “System Architecture” and “Protocol Stack” sections
5. Organize slide folder, index, and backups

**📝 Daily Logs:**  
- `9:00–10:30`:
  - Revisit DVB-S2X overview [System Architecture Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md) [Key Structural Elements](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVBs_Key-Elements.md)
  - Extract system architecture (uplink/downlink + forward/return links) [System Architecture Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md)
  - Screenshot or vector-recreate key block diagrams
- `10:30–12:00`:
  - Read DVB-RCS2: focus on return link structure and MAC layers
- `12:00–13:00`:
  - Lunch break + backup work folder to GitHub/Trello repo
- `13:00–14:30`:
  - Begin drafting slides 1–5 [Draft Slides](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/initial_draft.md#sldie-1-10):
    - Title page
    - Introduction
    - DVB system E2E architecture [Jump Into: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#study-notes-dvb-end-to-end-system-architecture)
    - DVB-S2X functional blocks [Jump Into: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#system-analysis-following-the-data-path)
    - General system layout
- `14:30–16:00`:
  - Draft slides 6–10 [Draft Slides](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/initial_draft.md#sldie-1-10):
    - System Components: DVB-RCS2 [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#24-return-link-control-system-benchmark-dvb-rcs2-components)
    - DVB E2E Forward Link Architecture [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#25-physical-forward-link-benchmark-forward-link-architecture)
    - DVB E2E Return Link Architecture [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#26-physical-return-link-benchmark-return-link-architecture)
    - Combined DVB System Architecture [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#27-the-combined-e2e-architecture-benchmark-combined-dvb-system)
    - Key Takeaways [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#3-key-takeaways--dvb-e2e-recap)
    - Plan Next Action [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#4-next-steps-deeper-dive)

---

### 2025.07.31

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Complete key reading sections from DVB-S2X and DVB-RCS2
3. Extract diagrams: System Architecture, Protocol Stack, and notes on L1 Frame Structure
4. Start drafting slides 1–10 (System Architecture + Protocol Stack)

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Read DVB-S2X section on system overview and physical layer design [System Architecture Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVB_System-Architecture.md)
  - Identify and screenshot E2E system architecture diagrams (uplink/downlink) [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#2-system-analysis-following-the-data-path)
- `19:00–20:00`:
  - Skim DVB-RCS2 spec for return link and MAC overview [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#24-return-link-control-system-benchmark-dvb-rcs2-components)
  - Note the logical and physical channel mappings in RCS2 [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#26-physical-return-link-benchmark-return-link-architecture)
  - Begin mapping how NR may align or differ in logical/physical structure
- `20:00–21:00`:
  - Begin drafting slides 1–5 [Draft Slides](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/initial_draft.md#sldie-1-10):
    - Title page
    - Introduction
    - DVB system E2E architecture [Jump Into: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#study-notes-dvb-end-to-end-system-architecture)
    - DVB-S2X functional blocks [Jump Into: Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#system-analysis-following-the-data-path)
    - General system layout
- `21:00–22:00`:
  - Draft slides 6–10 [Draft Slides](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/initial_draft.md#sldie-1-10):
    - System Components: DVB-RCS2 [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#24-return-link-control-system-benchmark-dvb-rcs2-components)
    - DVB E2E Forward Link Architecture [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#25-physical-forward-link-benchmark-forward-link-architecture)
    - DVB E2E Return Link Architecture [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#26-physical-return-link-benchmark-return-link-architecture)
    - Combined DVB System Architecture [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#27-the-combined-e2e-architecture-benchmark-combined-dvb-system)
    - Key Takeaways [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#3-key-takeaways--dvb-e2e-recap)
    - Plan Next Action [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ms1_dvb_e2e.md#4-next-steps-deeper-dive)
 
---

### 2025.08.01

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Deepen explanation of DVB-RCS2 architecture
3. Visualize Forward and Return link logic
4. Introduce initial protocol stack overview
5. Ensure visual consistency with previous slides

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Refresher: Read key RCS2 diagrams + extract components [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/study_notes/DVBs_Key-Elements.md#dvb-rcs2-key-structural-elements-etsi-en-302-307-2)
  - Sketch structure for slides 11–13 [Slide Draft](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/initial_draft.md#slide-10-20)
- `19:00–20:00`:
  - Draft Slide 11–13 (focus on RCS2 architecture and functional flow) [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#24-dvb-rcs2-system-overview)
  - Align visual style with earlier DVB-S2X slides
- `20:00–21:00`:
  - Work on Slides 14–15 (Forward/Return link comparison)
  - Build protocol stack draft layout [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#27-dvb-rcs2-return-link-protocol-stack)
- `21:00–22:00`:
  - Finish Slides 16–17
  - Quick review of today’s draft [Recap](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#3-key-takeaways--recap)
  - Push progress to GitHub (if time permits) [Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#study-notes-protocol-stack-for-dvb-s2x-and-dvb-rcs2)
 
---

### 2025.08.04

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Insert DVB-S2X protocol stack explanation as new Slide 11 (before RCS2 begins)
3. Generate visual for DVB-S2X layered stack (IP → GSE → BBFRAME → PLFRAME)
4. Adjust slide numbering: shift existing RCS2 slides to start from Slide 14
5. Review coherence between forward (S2X) and return (RCS2) flow
6. Start organizing L1 framing section prep (for Aug 5–6)

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Draft content + bullet points for new Slide 11: “DVB-S2X Forward Link Protocol Stack” [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#22-dvb-s2x-forward-link-protocol-stack)
  - Begin visual design (layered block layout)
- `19:00–20:00`:
  - Create visual diagram (Python or manual import)
  - Insert into the PPT and label layers clearly
  - Review transitions from Slide 10 → 11 → 14
- `20:00–21:00`:
  - Refactor RCS2 slides (Slide 14 onward): ensure flow is maintained
- `21:00–22:00`:
  - Optional: Polish text for Slide 12 (GSE + BBFRAME) [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#23-forward-link-framing-context)
  - Review entire Section 2 (Slides 11–20) for logical completeness
 
---

### 2025.08.05

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Transition from Protocol Stack to Layer 1 focus
3. Begin deep-diving into BBFRAME → PLFRAME composition
4. Highlight synchronization elements (PLS, pilots)
5. Establish slide layout logic for L1 deep-dive

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Final review of Protocol Stack section (Slides 11–13)
  - Create bridge/transition bullet points into L1 section
  - Outline slide flow [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#23-forward-link-framing-context):
    - BBFRAME internals
    - PLS Header
    - PLFRAME
    - Pilot symbols
- `19:00–20:00`:
  - Build Slide 21: L1 Framing Overview [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#23-forward-link-framing-context)
    - Visual: high-level framing pipeline (BBFRAME → FEC → PLFRAME)
    - Bullet summary of framing layers
    - Caption: “DVB-S2X physical layer builds robust frames through coding, framing, and modulation”
- `20:00–21:00`:
  - Build Slide 22: Baseband Frame (BBFRAME) Details
    - Visual: BBHEADER + payload layout
    - Bullet: BBHEADER fields (stream type, sync, etc.)
    - Caption: “BBFRAME is the basic data unit encapsulated before FEC and modulation”
- `21:00–22:00`:
  - Build Slide 23: PLS Header + PLFRAME Composition
    - Explain structure of PLS Header
    - Show: PLFRAME = PLS Header + FEC Payload + Pilots
    - Include MODCOD pointer (link to ACM)
 
---

### 2025.08.06

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Finalize return link framing slides (Slides 25–27)
3. Begin transition from return framing to scheduling/MF-TDMA view
4. Cover both framing mechanics and how it fits timing/modulation


**📝 Daily Logs:**  
- `18:00–19:00`:
  - Finalize Slide 25 (RCST Burst Structure)
  - Draft Slide 26: MF-TDMA time-frequency grid structure [MF-TDMA Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#28-physical-layer-dvb-rcs2-return-link-transmission)
    - Explain how bursts are scheduled
    - Visual idea: 2D slot grid (Freq × Time)
- `19:00–20:00`:
  - Build Slide 27: How RCSTs are assigned slots [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/ProtStack_DVB.md#29-qos-and-scheduling-in-dvb-rcs2-return-link)
    - Cover types: contention-based vs scheduled bursts
    - Include a diagram showing dynamic allocation
    - Optional: DRA process or NCC’s role in scheduling”
- `20:00–21:00`:
  - Draft Slide 28–29: Return link framing recap & slot timing
    - Timing sync, guard time explanation
    - Pointers to L1 physical modulation
- `21:00–22:00`:
  - Begin Slide 30–31 (Framing flow wrap-up)
    - Visual: End-to-end flow for both forward and return
    - Text: Bridge into next section (Logical/Physical Channels)
 
---

### 2025.08.07

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Introduce the concept of logical vs physical channels
3. Build foundational slides explaining what each represents in the DVB context
4. Relate back to previously discussed framing structures
5. Prepare 3 key slides (36–38) and outline slide structure for the remaining 39–45

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Draft Slide 36: Intro to Logical vs Physical Channels [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/lnp_channel_dvb.md#21-slide-37--intro-to-logical--physical-channels)
  - Clarify why this section matters post-framing
  - Visual idea: Side-by-side comparison table or layered flow
- `19:00–20:00`:
  - Draft Slide 37: What Are Logical Channels? [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/lnp_channel_dvb.md#22-slide-38--what-are-logical-channels)
  - Cover: GSE Streams, TS Packets, traffic flows
  - Mention how logical channels ride inside physical framing
  - Optional: Add diagram showing GSE flow split across BBFRAMEs
- `20:00–21:00`:
  - Draft Slide 38: What Are Physical Channels? [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/lnp_channel_dvb.md#23-slide-39--what-are-physical-channels)
  - Cover: Modulation, frequency, time slots, bursts
  - Show how physical resources carry logical channels
  - Visual: Grid showing frequency/time/slot allocation
- `21:00–22:00`:
  - Outline Slides 39–45 (titles, key topics per slide) [Slide Draft](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/initial_draft.md#slide-36-45)
    - Draft Notes for: Slides 40-45 [Go To: Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/lnp_channel_dvb.md#24-slide-40--logical-channels-in-dvb-s2x)
 
---

### 2025.08.11

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Explain NTN NR in ETSI-aligned terms (no oversimplification)
3. Identify NTN NR attributes relevant to DVB-S2X and DVB-RCS2 L1
4. Show protocol stack mapping NR over DVB L1 (forward & return)
5. Highlight required adaptations (timers, HARQ, scheduling, Doppler)
6. Present a concise, defendable final answer to the research question

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Draft Slide 46: What is NTN NR?
    - Explain 3GPP NTN NR basics: non-terrestrial adaptation, satellite delay/Doppler
    - Visual idea: NTN NR system diagram with space segment, ground segment, UE
- `19:00–20:00`:
  - Draft Slide 47: NTN NR Attributes & DVB L1 Implications
    - Table: NTN NR attribute (e.g., long RTT, high Doppler) → DVB L1 impact
    - Show forward vs return path relevance
- `20:00–21:00`:
  - Draft Slide 48: Protocol Stack Mapping — NR over DVB L1
    - Visual: Stack diagram with NR PDCP/SDAP over GSE/IP → BBFRAME → PLFRAME/MF-TDMA
    - Cover both S2X forward link & RCS2 return link mapping
- `21:00–21:30`:
  - Draft Slide 49: Required Adaptations for NTN NR over DVB L1
    - Before vs After table (Standard NR vs NR over DVB L1)
    - Include timer tuning, HARQ changes, scheduling adaptation, Doppler pre-compensation
- `21:30–22:00`:
  - Draft Slide 50: Final Answer & Conclusion
    - 3–4 bullet points directly answering the research question
    - Final visual: End-to-end integration flow (Gateway → DVB PHY → Satellite → UE)
    - Review slides 1–50 for consistency before pushing to GitHub
 
---

### 2025.08.12

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Collect all study notes, diagrams, and references from daily work sessions.
3. Upload each day’s notes to the GitHub repository in chronological order.
4. Add links to each hourly task in the Trello cards for traceability.
5. Ensure diagrams (Graphviz PNGs) are included in relevant GitHub folders.
6. Cross-check with PPT to confirm every slide has supporting notes.

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Organize folders in GitHub repo by section (System Architecture, Protocol Stack, L1 Frame Structure, Logical & Physical Channels, Final Integration). [GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
- `19:00–20:00`:
  - Upload notes for July 29-31 with diagrams and references.
  - Add links to each Trello task for these dates.
- `20:00–21:00`:
  - Upload notes for Aug 1–5 with diagrams and references.
  - Update Trello task cards with GitHub links.
- `21:00–22:00`:
  - Upload notes for Aug 6–11 with diagrams and references
  - Perform final link check across Trello & GitHub.
 
---

### 2025.08.13

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Merge and simplify slides per Prof’s feedback (combine figures, add missing definitions, remove unnecessary content).
3. Contact Kitty to collaborate on slide fixes and split workload.
4. Collect all study notes, diagrams, and references from daily work sessions.
5. Add links to each hourly task in Trello for traceability.
6. Ensure diagrams (Graphviz PNGs) are included in relevant GitHub folders.
7. Cross-check with PPT to confirm every slide has supporting notes.

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Fix slides based on Prof’s feedback (merge BBFRAME + GSE diagram pages, expand abbreviations, clean content).
  - Create revised versions of affected slides and push to shared folder.
- `19:00–20:00`:
  - Contact Kitty about collaboration, share updated PPT, and split slide-fixing tasks.
  - Organize folders in GitHub repo by section (System Architecture, Protocol Stack, L1 Frame Structure, Logical & Physical Channels, Final Integration).
- `20:00–21:00`:
  - Upload notes for Aug 1–5 with diagrams and references.
  - Add GitHub links to each corresponding Trello card.
- `21:00–22:00`:
  - Upload notes for Aug 6–11 with diagrams and references.
  - Perform final link check across Trello & GitHub.
 
---

### 2025.08.14

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Continue uploading study notes from remaining backlog to GitHub.
3. Push all relevant codes (Graphviz diagrams, scripts, data processing) to GitHub.
4. Revise slides according to Prof’s recommendations:
  - Merge overlapping slides where possible.
  - Replace excessive abbreviations with full names or explanatory notes.
  - Remove or explain non-essential ideas.
6. Draft a working strategy with Kitty for slide preparation and division of work.
7. Cross-check that every updated slide has supporting notes/code in GitHub.

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Organize GitHub repo structure (Notes, Diagrams, Codes).
  - Start revising first set of slides based on Prof’s feedback (merge figures, clarify abbreviations).
- `19:00–20:00`:
  - Upload remaining study notes from Aug 6–11 with diagrams and references.
  - Link uploads to their Trello cards.
- `20:00–21:00`:
  - Upload code files (Graphviz, Python scripts, processing tools).
  - Continue slide revisions: ensure essential ideas are retained and explained clearly.
- `21:00–22:00`:
  - Outline collaboration strategy with Kitty (division of slides, shared GitHub folder for drafts).
  - Final link check: ensure Trello tasks have GitHub references for both notes and code.

---
### 2025.08.21

**🎯 Short-term Goal:**  
1. Milestone 4: Make a 50 Slide PPT [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft)
2. Create branch 2025-Fall-MS-Joy in the lab repo.
3. Set up 3 required files/folders:
4. Upload study notes from backlog (Aug 6–11).
5. Upload relevant code (Graphviz diagrams, scripts, data processing) with README explanations.
6. Attach GitHub links (headers) back into Trello checklist items.

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Clone BMW Lab repo.
  - Create branch 2025-Fall-MS-Joy. [Branch](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson)
  - Set up folder structure (README.md, milestones.md, ./docs/).
- `19:00–20:00`:
  - Move existing study notes into ./docs/.
  - Add diagrams + references.
  - Commit & push changes.
- `20:00–21:00`:
  - Upload all codes (Graphviz, Python, processing tools). [Section 1](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/section1_code.py) [Section 2](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/section2_code.py) [Section 3](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/section3_code.py) [Section 4](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/slide_draft/section4_code.py)
  - Write short README explanations for each code file.
- `21:00–22:00`:
  - Link Trello checklist items to GitHub headers.
  - Final check: repo structure matches Kak Ian’s requirements.

---  

### 2025.09.29

**🎯 Short-term Goal:**  
1. Milestone: Collect and document scheduling and routing algorithms for 3GPP NTN and DVB-S2X/DVB-RCS2
2. Set up 3 required files/folders inside repo:
    - ./docs/paper_list.md → list of IEEE & equivalent papers with GitHub headers.
    - ./docs/notes/ → plain-English summaries of each paper (scheduling vs routing).
    - ./algorithms/ → placeholder scripts or diagrams (Graphviz / pseudo-code of schedulers).
3. Upload paper metadata (title, authors, year, link to IEEE/DOI).
4. Add short explanation for each algorithm (1–3 sentences in README style).
5. Attach GitHub headers back into Trello checklist items.

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Set up folder structure (README.md, milestones.md, ./docs/, ./algorithms/). [Folder Structure](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing)

- `19:00–20:00`:
  - Add paper list in paper_list.md (IEEE + surveys + standards). [Paper List](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper_list.md)
  - Write plain-English summary for each paper in ./docs/notes/. [Notes Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing)
  - Commit & push changes.

- `20:00–21:00`:
  - Upload paper Notes [Paper 1](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-a_notes.md), Note down [Problems](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-a_notes.md#problem) , [Solution](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-a_notes.md#high-level-concept-of-the-solution), [Results verificator](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-a_notes.md#how-they-verify-their-results), [KPIs](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-a_notes.md#kpis--actual-numbers).
  - Add references (IEEE citation format). [Paper List](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper_list.md)
  - Commit & push changes.

- `21:00–22:00`:
  - Link Trello checklist items to specific GitHub headers
  - Final check repo matches Kak Ian’s SOP (naming + doc structure).

---  

### 2025.10.01

**🎯 Short-term Goal:**  
1. Milestone 4: Study the algorithm or simulator of Joanna & 侯亮妤’s NTN work, and continue study notes on beam-hopping scheduling papers.
2. Add new paper notes (Zhang 2023 WCNC, Deng 2024 JSAC) into ./docs/notes/.
3. Update ./docs/paper_list.md with IEEE citation + links.
4. Start NTN simulator replication: clone Joanna & 侯亮妤’s repos and run example scripts.
5. Upload replication logs and errors (if any) into repo for tracking.
6. Copy GitHub permalinks of paper_list.md + study notes into Trello checklist items.

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Pull yesterday’s commits, check file consistency. 
- `19:00–20:00`:
  - Add study notes for Deng 2024 JSAC (beam hopping) [Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-b_notes.md#study-notes--satellites-beam-hopping-scheduling-for-interference-avoidance). Note down [Problems](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-b_notes.md#problem) , [Solution](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-b_notes.md#high-level-solution), [Results verificator](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-b_notes.md#verification-method), [KPIs](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-b_notes.md#kpis-considered).
  - Add study notes for  Zheng China Communications [Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-c_notes.md). Note down [Problems](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-c_notes.md#problems) , [Solution](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-c_notes.md#high-level-concept-of-the-solution), [Results verificator](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-c_notes.md#how-the-authors-verify-their-results), [KPIs](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-c_notes.md#kpis-considered).
  - Put paper's summary:
    - Chapter Summary [Paper 1](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-a_notes.md#1-introduction) [Paper 2](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-b_notes.md#1-introduction) [Paper 3](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-c_notes.md#1-introduction)
    - Glossaries: Filled with unknown terminology [Paper 1](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-a_notes.md#glossary-of-terms) [Paper 2](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-b_notes.md#glossary-of-terms) [Paper 3](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-c_notes.md#glossary-of-terms)
  - Commit & push changes.
- `20:00–21:00`:
  - Read Kitty’s note on the simulator [Kitty's Note](https://github.com/liang924/SNS3)
  - Learn the fundamentals and basics
  - Create notes and structural folder for the simulator [Notes Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes)
- `21:00–22:00`:
  - Copy GitHub permalinks from GitHub README into Trello .
  - Final check: repo structure matches SOP (naming + headers + README.md).
  - Confirm all links are correct

---  

### 2025.10.03

**🎯 Short-term Goal:**  
1. Milestone 4: Study the algorithm or simulator of Joanna & 侯亮妤’s NTN work, and continue study notes on beam-hopping scheduling papers.
2. Add 1 new IEEE paper to paper_list.md with study notes (Problem, Solution, Verification, KPIs).
3. Update repo with new .md notes and permalinks.
4. Read Kitty’s ns-3 installation note.
5. Build ns-3 on local machine and run at least one LTE/NR example.
6. Document replication status (success/fail + logs).

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Search and select new IEEE paper (last 5 years, LEO NTN, scheduling/routing).
  - Write 4-point study note draft in .md. [Problem ](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-d_notes.md#1-problem) [Solution](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-d_notes.md#2-high-level-concept-of-the-solution) [Verification](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-d_notes.md#3-verification-method) [KPIs](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper-d_notes.md#4-kpis-considered)
  - Update paper_list.md. [Paper List](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/paper_list.md#4-dsroq-dynamic-scheduling-and-routing-for-qoe-management-in-leo-satellite-networks)
- `19:00–20:00`:
  - Read Kitty’s note about ns-3 installation steps. [Kitty's Note](https://github.com/liang924/SNS3/blob/main/Installation.md)
  - Clone ns-3 repo.
  - Configure and start build (./ns3 configure --enable-examples --enable-tests). [Replication Note](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/replication_log.md#task-build-ns-3-with-examples--tests)
- `20:00–21:00`:
  - Run simple LTE/NR example (lte-epc-simple, cttc-nr-demo). [LTE Simple Test](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/replication_log.md#task-run-a-simple-ltenrsatellite-example)
  - If build fails → capture errors, document in replication log.
  - If succeeds → confirm output log and screenshots. [LTE Simple Test](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/replication_log.md#task-run-a-simple-ltenrsatellite-example)
- `21:00–22:00`:
  - Check for NTN module (satellite/3GPP NTN extension). [Satellite Module Error](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/replication_log.md#command-run-2) [Sat Module Fix](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/troubleshooting.md#4-full-recovery-procedure)
  - Update Trello with GitHub permalinks for today’s paper + notes.
  - Post replication status (PASS/FAIL/PARTIAL). [Replication Success](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/replication_log.md)

---  


### 2025.10.07

**🎯 Short-term Goal:**  
1. Learn about NS-3 Satellite Module [Full Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md)
2. Foundation Check: Understand directory structure and helper-channel interaction
3. Identify the core files inside model/, helper/, and examples/. 
4. Trace the workflow of one example (sat-tutorial-example.cc).

**📝 Daily Logs:**  
- `18:00–19:00`:
  - Explore File Structure
    - Open contrib/satellite/ → focus on model/, helper/, examples/, and data/. Identify major .cc / .h files and what they do. [Jump To Note](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#directory-structure-and-helper-channel-interaction)  [model](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#model-directory), [helper](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#helper-directory), [examples](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#examples-directory), [data](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#data-directory)
- `19:00–20:00`:
  - Understand Simulation Initialization [Sims Init Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#example-simulation-workflow)
    - Study sat-tutorial-example.cc → note how the simulation starts (e.g., [scenario setup](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#phase-1-boilerplate-and-configuration), [node creation](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#phase-2-creating-the-network-nodes), [channel config](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#phase-3-setting-up-the-satellite-helper-and-channel), [Device, Signing IP, and Running](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#phase-4-installing-network-devices-and-the-ip-stack)).
- `20:00–21:00`:
  - Study Helper and Model Connections
    - Explore satellite-helper.cc and satellite-channel-model.cc to see how helpers connect PHY/MAC/Network layers. [Jump To Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#satellite-module-satellite-helpercc-satellite-channel-modelcc)
- `21:00–22:00`:
  - Summarize
    - Compile learned  things into [NOTES_NS-3](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md). Summarize: What does this module simulate? What can it output? 

---  

### 2025.10.09

**🎯 Short-term Goal:**  
1. Learn about NS-3 Satellite Module [Full Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md)
2. Device & PHY-MAC Exploration
3. Explore how data packets move from the application layer down to PHY.
4. Understand the role of SatNetDevice, SatPhy, and SatMac.
5. Review how trace logs are generated and connected to simulation events.
6. Expand the module documentation with “Packet Transmission Path” and “Trace System” sections.

**📝 Daily Logs:**  
- `18:00–19:00`: Analyze SatNetDevice [SatNetDevice Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#analyze-satnetdevice)
  - Study satellite-net-device.cc and satellite-net-device.h.
  - Understand how network devices are installed and connected through helpers. [Installation](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#installation-process-via-helper), [Connection](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#connection-flow-between-layers)
  - Note down data member roles (e.g., PHY pointer, MAC pointer, Channel pointer). [Key Member](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#key-data-members)
- `19:00–20:00`: Examine PHY and MAC Layers [PHY and MAC Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#phy-and-mac-layer-analysis)
  - Open satellite-phy.cc and satellite-mac.cc.
  - Observe how Transmit() and Receive() are handled. [PHY](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#phy-layer--transmit-and-receive) [MAC](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#mac-layer--transmit-and-receive)
  - Understand how signals are propagated to the channel and logged. [Logging & Trace](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#logging-and-trace-output)
- `20:00–21:00`: Trace Logging and Stats [Trace & Stats Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#tracing-and-statistics)
  - Explore how logs like PacketTrace.log and SimInfo.log are generated. [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#how-logs-are-generated)
  - Identify functions responsible for writing trace data. [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#functions-responsible-for-writing-trace-data)
  - Locate trace connection code in satellite-helper.cc or satellite-log.cc. [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#locating-the-trace-connection-code)
- `21:00–22:00`:
  - Update NOTES_NS-3 with new sections:
    - [SatNetDevice](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#analyze-satnetdevice)
    - [PHY-MAC Layer Analysis](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#phy-and-mac-layer-analysis)
    - [Trace Logging And Statistics](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#tracing-and-statistics)

---  

### 2025.10.27

**🎯 Short-term Goal:**  
1. Learn about NS-3 Satellite Module [Full Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md)

**📝 Daily Logs:**  
- `18:00–19:00`: Study Satellite Channel Implementation [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#ns-3-satellite-channel)
  - Open and review:
    - `contrib/satellite/model/satellite-channel.cc` [Link](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#1-contribsatellitemodelsatellite-channelcc)
    - `contrib/satellite/model/satellite-propagation-delay-model.cc`[Link](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#1-contribsatellitemodelsatellite-propagation-delay-modelcc)
  - Identify how the channel receives a transmitted signal and delivers it to the destination PHY.  [delay notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#how-it-calculates-delay)
- `19:00–20:00`: Analyze [Propagation Delay & Fading Models](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#ns-3-satellite-propagation-delay--fading-models)
  - Focus on how the delay is computed from satellite geometry. [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#1-propagation-delay-computed-from-satellite-geometry)
  - Explore files like:
    -  `satellite-fading-model.cc` [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#2-fading-models-and-their-pluggable-architecture)
    -  `satellite-propagation-delay-model.cc` [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#3-exploring-different-fading-models)
  - Note how different fading models (Rayleigh, Loo, Markov) are plugged in. [Satellite Fading Models](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#common-satellite-fading-models)
- `21:00–22:00`: Trace PHY-to-Channel Signal Flow [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#tracing-the-ns-3-phy-to-channel-signal-flow)
  - Follow what happens when `SatPhy::Transmit()` is called. [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#1-the-call-to-satphytransmit)
  - Observe how the packet is sent into the channel and received by another PHY. [Packet sent](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#2-the-packets-journey-through-the-channel)
  - Open PacketTrace.log (e.g., example-cbr/simple) and confirm events like SND and RCV match channel activities.  [packettrace.log](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#confirming-events-with-packettracelog) [analyzing the log](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#analyzing-the-log)

### 2025.10.29

**🎯 Short-term Goal:**  
1. Learn about NS-3 Satellite Module [Full Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md)

**📝 Daily Logs:**  
- `18:00–19:00`: : **Explore Example Scenarios:** [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#ns-3-satellite-example-scenarios-a-walkthrough)
  - Open examples/sat-tutorial-example.cc and sat-iot-example.cc.
  - Identify the major blocks:
    - [Node creation (CreateNodes)](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#1-node-creation-createnodes)
    - Channel configuration (SatHelper::ConfigurePropagationChannel) [Channel & Device Configs](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#2-channel--device-configuration)
    - Application setup (CBR, OnOff, or IoT-style traffic). [App Setup](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#3-application-setup-generating-traffic)
    - Note how parameters (e.g., data rate, delay, frequency) are set or loaded. [Setting And Loading Params](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#4-setting-and-loading-parameters)
- `19:00–20:00`: **Run and Observe Outputs** [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#running-simulations-and-observing-outputs)
  - Run both examples using ./ns3 run "...". [trying examples](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#1-trying-examples)
  - Locate corresponding log files in /data/sims/example-*. [locating dir](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#2-locating-the-example-dir-files) [locate logs](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#3-locate-log--trace-outputs)
  - Compare PacketTrace.log results — look for event sequences (SND, RCV, etc.). [compare both logs](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#4-compare-both-log-file)
- `20:00–21:00`: **Modify Parameters** [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#trying-examples-and-modify-parameters)
  - Change one key setting (e.g., number of UTs, traffic rate, or delay model). [change variables](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#3-modify-the-parameter)
  - Rerun simulations to observe performance impact. [rerun](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#4-rerun-the-simulation)
  - Record before-and-after differences in packet timing or throughput. [compare](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#5-compare-logs) [analyzing differences](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#6-analyze-the-differences)
- `21:00–22:00`: **Summarize Findings** [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#scenario--example-customization)

### 2025.11.08

**🎯 Short-term Goal:**  
1. Learn about NS-3 Satellite Module [Full Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md)
2. Learn NTN 3GPP NTN NR Background Knowledge to teach undergraduates. [Full Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/NTN/NTN_Back-Knowledge_NOTES.md)

**📝 Daily Logs:**  
- `09:00–15:00`: Publishing Prof Bambang’s Book
- `18:00–19:00`: [Trace the Packet Journey (Theory Mapping)](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#the-packet-flow-path-end-to-end)
  - Sketch the flow path [flow sketch](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#this-is-a-visual-graph-for-explaining-how-the-e2e-satellite-simulation)
  - Identify where logs (PacketTrace.log, SimInfo.log) are triggered in the flow. [notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#where-logs-are-triggered-in-the-flow)
- `19:00–20:00`: [Run and Observe Full Simulation](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#run-and-observe-full-simulation)
  - Execute a full example like: sat-tutorial-example [execute sims ex](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#1-execute-simulation)
  - Watch console logs and open resulting files in data/sims/example-tutorial/. [open files](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#2-open-resulting-files)
  - Locate matching “SND” → “RCV” pairs to confirm packet completion. And breakdown results [find SND](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#step-1-find-a-snd-event) [find RCV](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#step-2-find-a-rcv-event) [follow single packet](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#step-3-try-to-follow-a-single-packet) [a packet breakdown](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#lets-break-it-down)
- `20:00–21:00`: [Code-Level Tracing](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#code-level-tracing)
  - Open [satellite-net-device.cc](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#1-satellite-net-devicecc-the-front-door), [satellite-phy.cc](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#3-satellite-phycc-the-radio), and [satellite-mac.cc.](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#2-satellite-maccc-the-traffic-manager)
  - Search for m_packetTrace, Send(), and Receive() calls.
  - Note exactly where packets get logged, queued, and dequeued.
- `21:00–22:00`: Summarize Findings [full today's notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#integrating-everything-end-to-end-system-understanding)
- `22:00–23:00`: Task from Prof Ray
  - Share ns-3 (satellite) notes as NTN’s simulator [Putting notes on reference](https://hackmd.io/FE9LWUw1SoGNnEsC201K-w?both#NTN-Simulator)
  -  Learn “NTN 3GPP NTN NR” Fundamentals [Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/NTN/NTN_Back-Knowledge_NOTES.md)

### 2025.11.10

**🎯 Short-term Goal:**  
1. Learn NTN 3GPP NTN NR Background Knowledge to teach undergraduates. [Full Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/NTN/NTN_Back-Knowledge_NOTES.md)
2. Learn about NS-3 Satellite Module [Full Study Notes](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?view#3GPP-NTN)

**📝 Daily Logs:**  
- `11:00–12:30`: Study 3GPP NTN (Non-Terrestrial Network) [study notes](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#3GPP-NTN)
  - Focuses on:
    -  [System architecture](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#3GPP-NTN-Architecture-Overview)
    -  [Transparent vs Regenerative mode](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#3GPP-NTN-Architecture-Overview)
    -  [NR protocol stack](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#NR-Protocol-Stack-Overview)
    -  [Integration with terrestrial 5G core](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Integration-Between-Satellite-and-Terrestrial-5G-Core)
    -  [Delay compensation](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Delay-Compensation)
    -  [Logical & physical channels](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Logical-and-Physical-Channel), [logical](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Logical-Transport-And-Physical-Layer-Flow), [transport](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Transport-Channels), [physical](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Physical-Channels)
- `12:30–14:00`: Study DVB-S2X / DVB-RCS2 [study notes](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#DVB-S2X-amp-DVB-RCS2)
  - Focuses on:
    - [System architecture](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#System-Architecture-amp-Topologies) ([tranparent architecture](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Transparent-Architecture), [regenerative architecture](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Regenerative-Architecture))
    - [Uplink (RCS2) and downlink (S2X) segmentation](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Uplink-amp-Downlink-Segmentation-Encapsulation)
    - [Protocol stack](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Protocol-Stack)
    - [Frame structures](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Frame-Structures)
    - [MAC & scheduling](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#MAC-amp-Scheduling)
    - [Role of QoS](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#Role-of-QoS-Quality-of-Service)
- `14:00–15:00`: Write Summary at GitHub and HackMD [GitHub](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/NTN/NTN_Back-Knowledge_NOTES.md) [HackMD](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?both#3GPP-NTN)
- `15:00–16:00`: Setup the Simulation Script [script](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#experiment--report-create-my-own-minimal-simulation)
  - Start from a simple example (like sat-cbr-example.cc).
  - Copy it into your own folder, e.g. my-simulations/minimal-sat.cc.
  - Reduce nodes to: 1 GW, 1 SAT, 1 UT.
  - Use SatHelper to configure nodes, PHY, MAC, and channel.
- `17:00–18:00`: Run & Observe Output [run and observe](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#observe-output)
  - [Locate log files.](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#locate-log-files)
  - [Use grep, head, and tail to see send/receive activity.](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#use-grep-head-and-tail-to-see-sendreceive-activity)
  - [Confirm data is flowing end-to-end (GW → SAT → UT)](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#confirm-data-is-flowing-end-to-end-gw--sat--ut)
- `19:00–20:00`: [Analyze & Visualize Results](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#analyze--visualize-results)
  - [Summarize what happen in logs](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/scheduling_routing/simulation/simulation_notes/NOTES_NS-3_SATELLITE.md#summarize-what-happen-in-logs)
    - When the packet was sent
    - Where it was received
    - Delay and jitter behavior
    - Draw a simple diagram of the data path

### 2025.11.11

**🎯 Short-term Goal:**  
1. Learn NTN 3GPP NTN NR Background Knowledge to teach undergraduates. [Full Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/NTN/NTN_Back-Knowledge_NOTES.md)
2. [DONE] Learn about NS-3 Satellite Module [Full Study Notes](https://hackmd.io/D7pXmmKXQf-7enZbBZy8cw?view#3GPP-NTN)

**📝 Daily Logs:**  
- `10:00–11:30`: **Understands QoS Concepts & Models** [QoS Notes](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#QoS-Quality-Of-Service)
- [QoS parameters](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#QoS-Parameters-The-Metrics)
- [QoS layers](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#QoS-Layers-Where-It-Happened):
  - [Application layer (service class, e.g. video, voice, data)](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#Application-Layer)
  - [Network layer (IP-level DiffServ, IntServ)](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#Network-QoS-The-Policy-Layer)
  - MAC/PHY layer (resource allocation, priority queues) [notes](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#Link-QoS-The-Action-Layer)
- [QoS control models](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#QoS-Control-Models-The-Strategies):
  - Best-effort, Integrated Services (IntServ), Differentiated Services (DiffServ), SLA-based.
- `11:30–13:00`: Study [Scheduling & Routing](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#Scheduling-And-Routing-Mechanisms)
- Scheduling types:
  - [Fixed vs. dynamic](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#Fixed-vs-Dynamic)
  - [Priority Queue](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#1-Priority-Queue-PQ), [Weighted Fair Queuing (WFQ)](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#3-Weighted-Fair-Queuing-WFQ), [Round Robin](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#2-Round-Robin-RR)
  - [Admission control & resource reservation](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#4-Admission-Control-amp-Resource-Reservation)
- [Routing types](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#Routing-Types):
  - Static, adaptive, load-aware, QoS-aware routing
  - [Trade-offs](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#The-Trade-offs)
- `13:30–15:00`: [3GPP NTN QoS Architecture Implementation](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#3GPP-QoS-Framework)
- QoS in 3GPP:
  - [5QI values](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#5QI-5G-QoS-Identifier)
  - [GBR vs. Non-GBR flows](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#GBR-vs-Non-GBR-Flows)
  - [End-to-end QoS through 5G Core](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#End-to-End-QoS)
- [Scheduling](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#3GPP-Scheduling):
  - [gNB scheduler, HARQ, RLC/PDCP buffering](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#gNB-Scheduler)
  - [NTN adaptations](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#NTN-Adaptations-The-Big-Change)
- [Routing](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#3GPP-Routing):
  - UPF routes based on policy, with NTN extensions
  - Onboard processing (regenerative gNB)
- `15:00–16:00`: [DVB-S2X / RCS2 QoS Architecture](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#DVB-QoS-Framework)
- QoS in DVB:
  - SLA and service profiles
  - QoS enforcement via NMC/NCC
- [Scheduling](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#DVBs-Scheduling):
  - DAMA, RBDC, VBDC mechanisms
  - Centralized vs. distributed scheduling
- [Routing](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#DVBs-Routing):
  - Hub-centric routing (transparent)
  - OBP-based onboard routing (regenerative)
- [QoS control tools](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#QoS-Control-Tools):
  - ACM/MODCOD for adaptive link QoS
- `16:00–17:00`: Summarize, and set milestones x goals for tommorow.
  - [Summarize Comparison](https://hackmd.io/Ml8Y-_JkQ06ZWIwsyclaJA#Comparison-3GPP-NTN-Vs-DVBs-QoS)

### 2025.11.12

**🎯 Short-term Goal:**  
1. Learn NTN 3GPP NTN NR Background Knowledge to teach undergraduates. [Full Study Notes](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/NTN/NTN_Back-Knowledge_NOTES.md)

**📝 Daily Logs:**  
- `10:00–11:30`: Making and putting references to [references.bib](https://github.com/petrajoyd/intern_repo/blob/2026-Spring-MS-Petrajoy_Davidson/references/references.bib)
- `11:30–13:00`: Routing and Scheduling Interaction under Delay & Mobility
- Routing with dynamic topology:
  - 3GPP NTN: delay-compensated routing (path stability metric)
  - DVB-RCS2: transparent vs regenerative routing (centralized vs distributed)
- Scheduling under long RTT:
  - Predictive scheduling using delay budget
  - Slot allocation ahead of feedback
  - QoS challenge: maintaining stability when handover occurs
- `13:30–15:00`: Advanced Scheduling Algorithms & Tradeoffs
  - Delay-Aware Proportional Fair (DA-PF)
  - QoS-driven Reinforcement Learning Scheduling
  - Hierarchical Scheduling (Gateway + RCST)
- `15:00–17:00`: Journals Paper Review
  - Journal 1 = [NB-IoT via LEO satellites: An efficient resource….](https://arxiv.org/pdf/2107.01067)
  - Focus on  finding
    - Motivation x Problem Statement
    - System/Network Model
    - Scheduling & Routing Mechanisms
    - QoS Metrics Used
    - Performance Gains
    - Limitations
  - Journal 2 = [Overall Design of Satellite Networks for Internet Services with QoS Support](https://www.mdpi.com/2079-9292/8/6/683)
  - Focus on  finding
    - Motivation x Problem Statement
    - System/Network Model
    - Scheduling & Routing Mechanisms
    - QoS Metrics Used
    - Performance Gains
    - Limitations
- `16:00–17:00`: Summarize, and set milestones x goals for tommorow.














































