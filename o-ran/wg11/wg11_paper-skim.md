# Towards a Multi-Layer Defence Framework for Securing Near-Real-Time Operations in Open RAN

Author = Alimohammadi, H., Mayhoub, S., Chatzimiltis, *et al.* </br>
Year = 2025


## Abstract
Securing near-real-time (near-RT) control in Open RAN is becoming more important as attacks increasingly target the system while it is already running. However, most existing security mechanisms focus on onboarding, authentication, or confguration, and do little to protect the near-RT control loop during live operation. In the paper, they focus specifically on runtime security of the near-RT RIC (RAN Intelligent Controller). They group operational-time threats into 3 practical categories; message level, data level, and control logic level and design a multi layer defence framework that places lightweight detection and mitigation mechanisms at each of these points. Their impplementation includes a 
1. signature-based E2 message inspection module for detecting malicious signalling, 
2. a telemetry poisoning detector based on temporal anomaly detection using and LSTM model, 
3. and a runtime xApp attestation mechanism based on execution-time hash verification.

The framework is implemented on an Open RAN testbed using FlexRIC and a commercial RAN emulator.
</br>
Experimental results show that the proposed safeguards achieve effective detection while staying within near-RT timing constraints, introducing less than 80 ms of overhead even in scenarios with 500 UEs. Overall, this work shows that layered, policy-driven untime protection for the near-RT RIC is both feasible and practical, and provides a foundation for extending Open RAN security beyond static, pre-runtime defences

## System Model & Architecture

### A. System Model
They consider an Open RAN architecture where the RAN is disaggregated into O-RUs, O-DUs, and O-CUs, following the 3GPP NR Functional split (7.2). The near-real-time RIC operates at sub-second timescales (10 ms & 1 s) and interacts with the RAN through the standardised E2 interface. Control and optimisation logic is implemented in xApps, which run on top of the near-RT RIC platform and consume telematry data from E2 nodes to make control decisions. </br>
Within this system, the near-RT RIC sits directly in the operational control loop: it receives E2 signalling and telemetry (e.g., KPM reports), processes this information through xApps, and issues control actions that can affect scheduling, mobility, and resource allocation. Because of this position, any manipulation of inputs or control logic at runtime can immediately impact network behaviour.
In the paper, they focus on runtime threats that originate outside the near-RT RIC platform itself. They assume that the internal RIC platform is trusted and secured using mechanisms defined by O-RAN Alliance specifications (e.g., secure onboarding, authentication, and platform hardening). Threats that require compromising the RIC operating system or infrastructure are therefore out of scope. Instead, we consider adversaries that influence the near-RT control loop through external interfaces or components, such as compromised E2 nodes, man-in-the-middle (MitM) attacks on the E2 interface, or tampered xApps.

### B. Architectural Design Principles
To address threats mentioned in the paper, they design a multi-layer runtime defence architecture that aligns protection mechanisms with their corresponding threat surfaces. The key ideas of design principles are:
1. Layered Protection: Each threat category is handled by a dedicated detection mechanism placed as close as possible to the point where the threat manifests.
2. Runtime operation: All safeguards operate during live execution and do not rely solely on pre-deployment validation
3. Low overhead: Detection and mitigation must stay within near-RT timing constraints
4. Modularity and extensibility: Components can be deployed independently and extended to support future threat types.

### C. Proposed Architecture

<p align="center">
    <img src="../../NTN/notes-png/proposed-archi-paper1.png" alt="Architecture of the proposed runtime security framework " />
    <br />
    <em>Figure 1. Architecture of the proposed runtime security framework </em>
</p>

Figure 1 illustrates how the proposed defence components integrate into the near-RT RIC architecture. The framework consists of three detection modules, each paired with a policy-driven mitigation unit deployed within the near-RT RIC platform:
- A message-level E2 inspection module placed at the E2 ingress, intercepting incoming messages before they reach the RIC message handler.
- A data-level telemetry poisoning detector, implemented as a standalone xApp that analyses KPM reports and provides verified telemetry to other xApps.
- A control logic-level runtime xApp attestation mechanism, where the near-RT RIC periodically verifies the integrity of executing xApps.

All mitigation actions such as dropping messages, isolating telemetry, or revoking xApp privileges are handled centrally by the near-RT RIC platform based on operator-defined policies. This separation between detection and mitigation allows flexible responses without modifying detection logic.

### E2 Message Inspection 
This part focuses on protecting the E2 interface. All E2 messages entering the near-RT RIC are checked before they are processed further. The idea is that some attacks use messages that are still protocol-compliant, so basic ASN.1 validation is not enough.

<p align="center">
    <img src="../../NTN/notes-png/E2_INSPECTION_ARCHI.png" alt="E2 message inspection and associated mitigation" />
    <br />
    <em>E2 message inspection and associated mitigation</em>
</p>

Incoming E2 messages are inspected at the byte level and compared against a set of known malicious signatures. These signatures are patterns that have been observed to cause harmful behaviour once the message is decoded or consumed by xApps. If a match is found, the message is marked as malicious and does not enter the normal RIC processing pipeline. </br>
The detection logic is kept simple and fast, since E2 messages are relatively small. Mitigation is handled separately and can include dropping the message or blocking the source E2 node. This inspection step adds very little delay and does not affect near-real-time operation.

### KPM Poisoning Detection (And Mitigation)
This component deals with attacks that target telemetry instead of signalling. Even if E2 messages are valid, the values inside KPM reports can be manipulated to mislead xApps.
<p align="center">
    <img src="../../NTN/notes-png/KPM_DETECTION_ARCHITECTURE.png" alt="KPM poisoning detection and mitigation." />
    <br />
    <em>KPM poisoning detection and mitigation.</em>
</p>
The poisoning detector is implemented as a separate xApp that subscribes to KPM reports. It observes how KPM values evolve over time and learns normal temporal patterns. Instead of checking individual values, it looks for abnormal behaviour across sequences of measurements. </br>
An LSTM model is used to capture these temporal dependencies. When incoming telemetry deviates significantly from learned patterns, it is flagged as suspicious. Poisoned data can then be discarded or isolated so that other xApps do not act on it.</br>
This detection runs within the near-RT time window and remains under the allowed latency budget, even when the number of UEs increases.

### xApp Attestation and Integrity Violation Mitigation
This part focuses on the integrity of xApps themselves. Since xApps can be developed by third parties and run continuously, there is a risk that their code is modified at runtime.

<p align="center">
    <img src="../../NTN/notes-png/KPM_PD.png" alt="xApp attestation and integrity violation mitigation." />
    <br />
    <em>xApp attestation and integrity violation mitigation.</em>
</p>

The near-RT RIC periodically challenges running xApps to prove that their code has not been altered. Each challenge includes a random seed, which the xApp uses to compute a hash over its runtime memory. The RIC compares this hash with a reference value generated from a trusted version of the xApp. </br>
If the values do not match, the xApp is considered compromised. The RIC can then revoke its permissions or stop it entirely. This process runs outside the critical control path, so it does not interfere with real-time decision making.

### Summary of Runtime Behaviour
All three components operate at the same time and protect different parts of the near-RT control loop. E2 message inspection filters malicious signalling, KPM poisoning detection ensures telemetry can be trusted, and runtime attestation checks that xApps themselves remain intact.
Together, these mechanisms provide continuous protection while the system is running, without relying only on pre-deployment security checks and without violating near-real-time constraints.

## Security Assumptions
In the paper, they make several assumptions to clearly define the threat model and the scope of the proposed runtime defences. </br>
1. **The near-RT RIC Platform itself is assumed to be trusted**. This includes the underlying operating system, container runtime, and core RIC services.
2. **They assume that external RAN components are not fully trusted.** This includes E2 nodes such as O-DUs and O-CUs. These components may be compromised, misconfigured, or behave maliciously, either intentionally or due to software faults. As a result, signalling and telemetry arriving at the near-RT RIC cannot be blindly trusted.
3. **E2 interface is treated as a potential attack surface.** Even though secure transport may be used, we assume that attackers may still inject or manipulate E2 messages, for example through compromised nodes or man-in-the-middle scenarios. Therefore, E2 messages are considered untrusted until validated at runtime.
4. **Telemetry data (KPM reports) is assumed to be untrusted by default.** The paper assumes that attackers can poison performance measurements in ways that remain protocol-compliant but still mislead control logic. This is why telemetry must be checked during operation rather than trusted implicitly.
5. **xApps are assumed to be semi-trusted.** They are trusted at deployment time, but not assumed to remain trustworthy throughout execution. Since xApps may be third-party and long-running, the paper assumes they can be modified or tampered with at runtime, which motivates the need for continuous attestation.
6. **The attacker is assumed to operate during system runtime, not just before deployment.** The attacker’s goal is to influence the near-RT control loop by manipulating messages, data, or control logic, rather than by taking down the infrastructure entirely.

## Threat Model / Attack Scenario
Based on how the near-RT contorl loop operates, they model runtime threats along 3 main surfaces:

### 1. Message-Level Threats
These threats target the signalling path between E2 nodes and the near-RT RIC. An attacker may inject syntactically valid but semantically malicious E2 messages that exploit weaknesses in message handling or downstream logic. Because these messages are protocol-compliant, basic validation is often insufficient to detect them.

### 2. Data-Level Threats
Data-Level threats aim to manipulate telemetry used by xApps, such as Key Performance Measurements (KPMs). By poisoning or falsifying these inputs, an attacker can mislead ML-based or rule-based control logic, causing suboptimal or harmful decisions even though the signalling itself appears legitimate.

### Control Logic-Level Threats
These threat catogories are complementary rather than overlapping. Each targets a different stage of the near-RT execution pipeline, which means that no single protection mechanisms can address all of them effectively.

## Methodology

<p align="center">
    <img src="../../NTN/notes-png/E2E_FLOW.png" alt="E2E runtime workflow" />
    <br />
    <em>E2E Runtime Workflow</em>
</p>


## Validation & Evaluation
How they prove it works

## Key Concepts 
New ideas or mechanisms introduced

## Terminologies To Study
Protocols, acronyms, interfaces i don’t fully know yet

## Standard References
Protocols, acronyms, interfaces you don’t fully know yet

## Open Questions / Gaps
- What they didn’t test
- What feels unrealistic
- What could be done in a real lab

















# Paper 2 Title

## Abstract
(What problem, what angle, why it matters)

## System Model & Architecture
(What part of O-RAN / WG11 they touch: RIC, E2, xApps, data flow)

## Security Assumptions
(What they assume is trusted / untrusted)

## Threat Model / Attack Scenario
Threat Model / Attack Scenario

## Methodology
How they did the research

## Validation & Evaluation
How they prove it works

## Key Concepts 
New ideas or mechanisms introduced

## Terminologies To Study
Protocols, acronyms, interfaces i don’t fully know yet

## Standard References
Protocols, acronyms, interfaces you don’t fully know yet

## Open Questions / Gaps
- What they didn’t test
- What feels unrealistic
- What could be done in a real lab



































# Paper 3 Title

## Abstract
(What problem, what angle, why it matters)

## System Model & Architecture
(What part of O-RAN / WG11 they touch: RIC, E2, xApps, data flow)

## Security Assumptions
(What they assume is trusted / untrusted)

## Threat Model / Attack Scenario
Threat Model / Attack Scenario

## Methodology
How they did the research

## Validation & Evaluation
How they prove it works

## Key Concepts 
New ideas or mechanisms introduced

## Terminologies To Study
Protocols, acronyms, interfaces i don’t fully know yet

## Standard References
Protocols, acronyms, interfaces you don’t fully know yet

## Open Questions / Gaps
- What they didn’t test
- What feels unrealistic
- What could be done in a real lab