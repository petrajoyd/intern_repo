# 📚 2025-Fall-MS-Joy **(Internship Documentation)**

## 📑 Table of Contents
<!-- (GitHub auto-generates ToC based on headers. You may use a Markdown TOC generator plugin for static ToC if needed.) -->

---

## 🏁 Milestones

### 1. Networking Basics
- [ ] 1.1 CCNA Fundamentals
- [x] 1.2 IP Addressing and Subnetting
- [ ] 1.3 Routing and Switching Basics

### 2. O-RAN Architecture Overview
- [ ] 2.1 O-RAN Architecture Study Notes
- [ ] 2.2 Protocol Stack Overview

### 3. Study DVB-S2X and DVB-RCS2 [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/study_notes)
- [x] 3.1 DVB-S2X & DVB-RCS2 Standard
- [x] 3.2 Study Paper Related to DVBs
- [ ] 3.3 Evaluate the Performance of the Algorithm Using Xeoverse & ns-3 NTN

### 4. Make a 50 Slide PPT About [Go To GitHub Folder](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/slide_draft):
- [x] 4.1 System Architecture of the DVB-E2E System 
- [x] 4.2 DVBs Protocol Stack
- [x] 4.3 L1 Frame Structure
- [x] 4.4 DVBs Logical and Physical Channels
- [x] 4.5 Store All Visual Code in GitHub repository

---

## 📆 Daily Logs

### 2025.07.11

**🎯 Short-term Goal:**  
1. Milestone 3: Study DVB-S2X and DVB-RCS2 [Go To: Study Notes](https://github.com/petrajoyd/intern_repo/tree/2026-Spring-MS-Petrajoy_Davidson/study_notes)
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
  - Write 4-point study note draft in .md.
  - Update paper_list.md.
- `19:00–20:00`:
  - Read Kitty’s note about ns-3 installation steps.
  - Clone ns-3 repo.
  - Configure and start build (./ns3 configure --enable-examples --enable-tests).
- `20:00–21:00`:
  - Run simple LTE/NR example (lte-epc-simple, cttc-nr-demo).
  - If build fails → capture errors, document in replication log.
  - If succeeds → confirm output log and screenshots.
- `21:00–22:00`:
  - Check for NTN module (satellite/3GPP NTN extension).
  - Update Trello with GitHub permalinks for today’s paper + notes.
  - Post replication status (PASS/FAIL/PARTIAL).

---  
