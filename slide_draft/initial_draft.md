# Slide Subheadings
1. Intro to DVB (3 slides)
2. DVB-S2X Overview (10 slides)
3. DVB-RCS2 Overview (10 slides)
4. Protocol Stack Analysis (8 slides)
5. Frame Structure Breakdown (10 slides)
6. Logical vs Physical Channels (5 slides)
7. NTN NR Introduction (2–3 slides)
8. NTN on DVB L1 – Concept Proposal (2–3 slides)

### slides timeline
| Phase                          | Date Range         | Focus                                                                 |
| ------------------------------ | ------------------ | --------------------------------------------------------------------- |
| **⚠️ Catch-Up + Drafting**     | **Jul 29 – Aug 2** | Wrap up spec study **and** begin drafting slides in parallel          |
| **Slide Refinement**           | Aug 3 – Aug 6      | Clean formatting, fill gaps, refine figures, speaker notes            |
| **Final Integration + Answer** | Aug 7 – Aug 10     | Answer "NTN NR on top of DVB L1", polish storyline, submit final deck |

### slides draft
| Section                        | Target Slides | Status            |
| ------------------------------ | ------------- | ----------------- |
| 1. System Architecture         | 1–10          |🗓️  july 21-25   |
| 2. Protocol Stack              | 11–20         | 🗓️  july 25-30      |
| 3. L1 Frame Structure          | 21–35         | 🗓️ July 30–Aug 1 |
| 4. Logical & Physical Channels | 36–45         | 🗓️ Aug 2–4       |
| 5. NTN NR Integration Concept  | 46–50         | 🗓️ Aug 5–6       |
| Polish & Final Check           | —             | 🗓️ Aug 7–10      |

### sldie 1-10
| Slide | Title                                 | Content                                                                                      |
| ----- | ------------------------------------- | -------------------------------------------------------------------------------------------- |
| 1     | **Title Slide**                       | Your name, project title, NTUST / BMW Lab, date                                              |
| 2     | **Project Overview**                  | Goal of the project, final question: "How can we implement NTN NR on top of DVB L1?"         |
| 3     | **What is DVB-S2X?**                  | Short explanation of DVB-S2X (forward link, high efficiency broadcast, mainly GW → terminal) |
| 4     | **What is DVB-RCS2?**                 | Short explanation of DVB-RCS2 (return link, interactive, terminal → GW, MAC-centric)         |
| 5     | **System Components: DVB-S2X**        | Key blocks: gateway, modulators, BBFRAMEs, PLFRAMEs, satellite, Rx chain                     |
| 6     | **System Components: DVB-RCS2**       | Key blocks: user terminal (RCST), scheduler, return carrier configuration, upstream MAC      |
| 7     | **DVB E2E Forward Link Architecture** | Diagram of Gateway → Satellite → User Terminal path (S2X)                                    |
| 8     | **DVB E2E Return Link Architecture**  | Diagram of User Terminal → Satellite → NCC/Gateway (RCS2)                                    |
| 9     | **Combined DVB System Architecture**  | Merge both diagrams into a complete E2E view (forward + return)                              |
| 10    | **Key Takeaways**                     | Summary: roles of S2X and RCS2 in full system; early thought on how NR might layer on top    |

### Slide 10-20
| Slide    | Topic                                                       |
| -------- | ----------------------------------------------------------- |
| **8–10** | System Architecture (Overall E2E)                          |
| **11**   | DVB-S2X Forward Link Protocol Stack (Big Picture) |
| **12**   | Step 1 & 2: GSE + BBFRAME                                   |
| **13**   | Step 3: FEC, PLFRAME, ACM                                   |
| **14**   | RCS2 System Overview                                        |
| 15       | NCC & Scheduling                                            |
| 16       | RCST Behavior                                               |
| 17       | RCS2 Return Link Protocol Stack                             |
| 18–20    | Remaining RCS2 layers, SI-SAP, DVB vs NR comparison         |

### L1 FRAME OF: DVB-S2X
| Slide | Topic                                  | Description                                                                               |
| ----- | -------------------------------------- | ----------------------------------------------------------------------------------------- |
| 21    | **Intro to L1 Framing**                | Overview of L1 framing in DVB (S2X + RCS2) — show both forward & return paths at a glance |
| 22    | **BBFRAME (Baseband Frame)**           | Structure of BBFRAME: BBHEADER + payload (GSE packets)                                    |
| 23    | **FEC Layer**                          | LDPC + BCH encoding overview (step between BBFRAME & PLFRAME)                             |
| 24    | **PLS Header & MODCOD**                | Contents of Physical Layer Signaling (PLS) Header — tells how to decode                   |
| 25    | **PLFRAME Structure**                  | Composition of PLFRAME: PLS Header + FEC Payload + Pilot symbols                          |
| 26    | **Pilot Symbols & Sync**               | Purpose and placement of pilots — helps with channel estimation and symbol sync           |
| 27    | **MODCOD Mapping**                     | Show table or logic of MODCOD selection (linked to ACM, already introduced)               |
| 28    | **S2X: Frame Transmission Example**    | End-to-end framing visual (IP → PLFRAME → satellite)                                      |
| 29    | **Superframe Concept (Optional)**      | Advanced topic: Superframes used for synchronization (optional depth)                     |
| 30    | **Transition: Forward to Return Link** | Clean break slide, explain that now we'll look at RCS2 (return link framing)              |

### L1 FRAME OF: DVB-RCS2
| Slide | Topic                           | Description                                                              |
| ----- | ------------------------------- | ------------------------------------------------------------------------ |
| 31    | **MF-TDMA Burst Framing**       | Explain the structure of time slots and frequency channels (return link) |
| 32    | **RCST Burst Format**           | RCST transmit format: Preamble, Header, Payload, Guard time              |
| 33    | **Slot Mapping Example**        | MF-TDMA grid visualization (slots over time/freq per RCST)               |
| 34    | **QoS + Scheduling Recap**      | How QoS affects scheduling and framing return bursts                     |
| 35    | **Compare S2X vs RCS2 Framing** | Wrap-up comparison: forward vs return framing features                   |

### slide 36-45
| Slide  | Title                                 | Description                                                                                     |
| ------ | ------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **36** | Intro to Logical & Physical Channels  | High-level overview of both channel types and why they matter in DVB systems                    |
| **37** | What Are Logical Channels?            | Define logical channels (data flow abstraction), examples (e.g., MAC PDUs, GSE streams)         |
| **38** | What Are Physical Channels?           | Define physical channels (actual transmission medium: frequencies, symbols, time)               |
| **39** | Logical Channels in DVB-S2X           | Explain stream types: TS Stream, GSE Stream, Generic Continuous / Packetized Stream             |
| **40** | Physical Channels in DVB-S2X          | How PLFRAMEs are modulated and transmitted on the physical layer (MODCOD, bursts)               |
| **41** | Logical Channels in DVB-RCS2          | Show RCST's MAC layer structures (FLS, MF-TDMA structure)                                       |
| **42** | Physical Channels in DVB-RCS2         | Bursts, slots, time/freq allocations, and mapping of logical → physical                         |
| **43** | Channel Mapping: Gateway to RCST      | Flow: IP packet → GSE → BBFRAME → PLFRAME → Superframe (map logical to physical)                |
| **44** | Channel Mapping: RCST to Gateway      | Flow: RCST data → MAC PDU → burst → slot in MF-TDMA                                             |
| **45** | Recap: Channel Abstraction vs Reality | Comparison table or layered stack diagram summarizing logic-to-physical flow in both directions |
