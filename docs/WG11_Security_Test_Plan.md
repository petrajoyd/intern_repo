# O-RAN WG11 Security Testing, Plan and Scope

One document for the WG11 security testing work. Full test landscape, the Open
Fronthaul slice we execute, every test case with tested or not, how it answers RTEC
C7.3, and the week-by-week plan.

Author of the work: Rizky Wildansani (San), branch `2026-TEEP-Sani`.
Reviewer: Joy. Date: 2026-10-08.
Source document read in full: O-RAN.WG11.TS.STS-R005-v14.00 (25 test clauses, 189 test cases).

## Table of contents

1. [Overview and scope](#1-overview-and-scope)
2. [Big picture: the full STS landscape](#2-big-picture-the-full-sts-landscape)
3. [Clause 11, interface tests](#3-clause-11-interface-tests)
4. [Our scope: Open Fronthaul test-case coverage](#4-our-scope-open-fronthaul-test-case-coverage)
5. [Conformance vs fuzz](#5-conformance-vs-fuzz)
6. [Open Fronthaul control mapping (C/U/S/M)](#6-open-fronthaul-control-mapping-cusm)
7. [RTEC C7.3 fuzz capability](#7-rtec-c73-fuzz-capability)
8. [Week-by-week plan](#8-week-by-week-plan)
9. [Normative basis and findings](#9-normative-basis-and-findings)
10. [References](#10-references)

---

## 1. Overview and scope

Build a WG11 security test harness and run it against BMW Lab's Open Fronthaul (OFH)
testbed. The executed priority is the OFH planes: conformance first (is the control
enforced), then fuzz. A1 and E2 are follow-on.

OFH is the Open Fronthaul, the O-DU to O-RU link. O-DU is the O-RAN Distributed Unit,
O-RU the O-RAN Radio Unit. C, U, S and M are the Control, User, Synchronisation and
Management planes.

Legend used throughout. Type: C = conformance (positive, is the control enforced),
F = fuzz / robustness / DoS. Scope: NOW = run now, later = if the O-RU or interface
is reachable, no = out of scope.

---

## 2. Big picture: the full STS landscape

All 25 test clauses in STS R005 v14.00, with the one slice in scope marked.

```
O-RAN WG11 Security Test Specifications (STS, R005 v14.00), 189 test cases
│
├─ 6  Security Protocol and API Validation .......... [not in scope]
│     SSH, TLS, DTLS, IPsec/IKE (incl IKE fuzzing),
│     OAuth2, X.509, eCPRI input validation + timeout,
│     SCTP, REST API, MACsec, SFTP/FTPES
├─ 7  Common Network Security Tests ................. [not in scope]
│     enumeration, password, robustness/DDoS, overload,
│     input-validation error handling, config enforcement, logging
├─ 8  System security evaluation .................... [not in scope]
├─ 9  Software security evaluation (SBOM, signatures) [not in scope]
├─ 10 ML security validation ........................ [not in scope]
│
├─ 11 Security tests of O-RAN interfaces
│   ├─ 11.1 Open FH .......................... [NOW]  (12 cases now, 13 later)
│   │   ├─ 11.1.2 Point-to-Point LAN Segment .. [NOW] 6 conformance (C-plane, 802.1X)
│   │   ├─ 11.1.3 M-Plane ................... [later: if O-RU reachable] 13 conformance
│   │   ├─ 11.1.4 U-Plane ................... [NOW] 2 fuzz (eCPRI malformed, payload)
│   │   └─ 11.1.5 S-Plane ................... [NOW] 4 robustness (rogue PTP, delay, DoS)
│   ├─ 11.2 Y1 ...... [not in scope]   ├─ 11.6 A1 ...... [later]
│   ├─ 11.3 O1 ...... [not in scope]   ├─ 11.7 R1 ...... [not in scope]
│   ├─ 11.4 O2 ...... [not in scope]   └─ 11.8 D2 ...... [not in scope]
│   └─ 11.5 E2 ...... [later: follow-on]
│
├─ 12 O-RU   ├─ 15 Non-RT RIC  ├─ 18 O-Cloud   ├─ 21 O-CU-CP   ├─ 24 End-to-End [later]
├─ 13 Near-RT RIC  ├─ 16 rApps  ├─ 19 VNF/CNF  ├─ 22 O-CU-UP   └─ 25 Shared O-RU
├─ 14 xApps  ├─ 17 SMO   ├─ 20 Common App Lifecycle Mgmt  ├─ 23 O-DU
│     (clauses 12 to 23, 25: all [not in scope])
│
└─ Deferred outside WG11 ............................ [out]
    └─ F1, E1, Xn   -> 3GPP TS 33.501
```

Where the fuzz lives (not only in clause 11): clause 6 (IKE fuzzing, eCPRI input
validation and timeout), clause 7 (robustness/DDoS, overload, input-validation error
handling), clause 11.1.4 and 11.1.5 (U-plane and S-plane), clause 24 (end-to-end).
Our scope covers the 11.1 Open FH fuzz plus the matching clause 6 eCPRI cases.

---

## 3. Clause 11, interface tests

Clause 11 is one section per interface, each tested along the same properties:
Authenticity, Confidentiality / integrity / replay, Authorization. Verified from the
STS contents.

| Clause | Interface | Scope |
|---|---|---|
| 11.1 | Open FH (LAN Segment, M, U, S plane) | **NOW** |
| 11.2 | Y1 (RAN analytics exposure) | not in scope |
| 11.3 | O1 (management) | not in scope |
| 11.4 | O2 (O-Cloud) | not in scope |
| 11.5 | E2 (Near-RT RIC) | later |
| 11.6 | A1 (Non-RT RIC) | later |
| 11.7 | R1 (SMO / rApp) | not in scope |
| 11.8 | D2 | not in scope |

---

## 4. Our scope: Open Fronthaul test-case coverage

Every clause 11.1 test case by its real `TC_*` ID, read from the document.

### 11.1.2 Point-to-Point LAN Segment (802.1X, carries the C-plane), p100

| TC ID | Type | Scope | Tested |
|---|---|---|---|
| TC_Authenticator_Validation | C | NOW | |
| TC_Supplicant_Validation | C | NOW | |
| TC_Port_Access_Enforcement_Validation | C | NOW | |
| TC_ORU_Authentication_Lifecycle_Validation | C | NOW | |
| TC_OFHPLS_Unauthorized_Port_Behavior | C (negative) | NOW | |
| TC_OFHPLS_Authorization_Policy_Enforcement | C | NOW | |

### 11.1.3 M-Plane (SSH / TLS / NACM / file transfer), p108, later

| TC ID | Type | Scope | Tested |
|---|---|---|---|
| TC_OFH_MPLANE_SSH-PASSWORD-BASED_AUTHENTICATION | C | later | |
| TC_OFH_MPLANE_SSH-CERTIFICATE-BASED_AUTHENTICATION_O-RU | C | later | |
| TC_OFH_MPLANE_SSH-CERTIFICATE-BASED_AUTHENTICATION_O-DU | C | later | |
| TC_OFH_MPLANE_SSH-KEY-BASED_AUTHENTICATION_O-RU | C | later | |
| TC_OFH_MPLANE_SSH-KEY-BASED_AUTHENTICATION_O-DU | C | later | |
| TC_OFH_MPLANE_SSH_CONFIDENTIALITY_INTEGRITY_REPLAY | C | later | |
| TC_OFH_MPLANE_TLS_AUTHENTICATION | C | later | |
| TC_OFH_MPLANE_TLS_CONFIDENTIALITY_INTEGRITY_REPLAY | C | later | |
| TC_OFH_NACM_VALIDATION | C | later | |
| TC_OFH_MPLANE_SFTP_O-RU | C | later | |
| TC_OFH_MPLANE_FTPES_O-RU | C | later | |
| TC_OFH_MPLANE_SFTP_CLIENT | C | later | |
| TC_OFH_MPLANE_FTPES_CLIENT | C | later | |

### 11.1.4 U-Plane (eCPRI unexpected input), p121

| TC ID | Type | Scope | Tested |
|---|---|---|---|
| TC_OFH_U-PLANE_MALFORMED_PACKET | F | NOW | |
| TC_OFH_U-PLANE_UNEXPECTED_PAYLOAD_SIZE | F | NOW | |

Requirement reference: REQ-SEC-TRAN-1, SRCS clause 5.3.4.1 and 5.2.5.2.1.

### 11.1.5 S-Plane (PTP timing), p123

| TC ID | Type | Scope | Tested |
|---|---|---|---|
| DoS timeTransmitter LLS C1 C2 C3 (no formal TC_ ID) | F (DoS) | NOW | |
| DoS timeTransmitter LLS C4 (no formal TC_ ID) | F (DoS) | no (local PRTC) | |
| TC_ROGUE_PTP_INSTANCE | F | NOW | |
| TC_SELECTIVE_INTERCEPTION_REMOVAL_PTP_TIMING_PACKETS | F | NOW | |
| TC_DELAY_ATTACK_PTP_TIMING_PACKETS | F | NOW | |

Requirement reference (S-plane spoofing): SEC-CTL-OFSP-3, SRCS clause 5.2.5.3.3.3.1.

### Rest of section 11 (not our scope now)

| Clause | TC IDs | Scope |
|---|---|---|
| 11.2 Y1 | TC_Y1_AUTHENTICATION, TC_Y1_CONFIDENTIALITY_INTEGRITY_REPLAY, TC_Y1_AUTHORIZATION | no |
| 11.3 O1 | TC_O1_AUTHENTICATION, TC_O1_CONFIDENTIALITY_INTEGRITY_REPLAY, TC_O1_NACM_VALIDATION, TC_OFH_O1_FTPES, TC_OFH_O1_SFTP | no |
| 11.4 O2 | TC_O2_AUTHENTICATION, TC_O2_CONFIDENTIALITY_INTEGRITY_REPLAY, TC_O2_AUTHORIZATION | no |
| 11.5 E2 | TC_E2_CONFIDENTIALITY_INTEGRITY_REPLAY, TC_E2_AUTHENTICATION_CERT, TC_E2_AUTHENTICATION_PSK, TC_E2_Interface_data_validation_by_NearRTRIC | later |
| 11.6 A1 | TC_A1_Authentication, TC_A1_CONFIDENTIALITY_INTEGRITY_REPLAY, TC_A1_Authorization | later |
| 11.7 R1 | TC_R1_AUTHENTICATION, TC_R1_CONFIDENTIALITY_INTEGRITY_REPLAY, TC_R1_AUTHORIZATION | no |
| 11.8 D2 | TC_D2_CONFIDENTIALITY_INTEGRITY_REPLAY_IPSEC | no |

Summary: 12 cases in scope now (6 LAN/C conformance, 2 U-plane fuzz, 4 S-plane
robustness), 13 M-plane cases later, the rest out of scope or follow-on.

---

## 5. Conformance vs fuzz

Read each clause as two halves:
- Positive case = is the control enforced on valid input. This is conformance (week 1).
- Negative / malformed / robustness case = does it hold under attack. This is fuzz (later).

Per plane, run the conformance (C) cases first, then the fuzz (F) cases, per
REQ-SEC-TRAN-1 with its 250,000-iteration rule. Functional conformance of the
interface (does the radio work per protocol) is a different document, the WG4
Conformance Test Specification, and is not our job.

Note: there is no C-plane malformed-packet case in STS. C-plane security is the
802.1X enforcement set in 11.1.2 (conformance). The eCPRI malformed and payload fuzz
is specified on the U-plane (11.1.4), with matching cases in clause 6
(TC_eCPRI_INPUT_VALIDATION, TC_eCPRI_TIMEOUT_HANDLING).

---

## 6. Open Fronthaul control mapping (C/U/S/M)

What "is the control enforced" means per plane, and the tool.

| Plane | Control and mechanism | Conformance check | Threat defended | Tool |
|---|---|---|---|---|
| C-plane (in 11.1.2 LAN Segment) | 802.1X port-based access control | Is 802.1X enforced; does the port drop an unauthorised supplicant | C-plane injection posing as O-DU, cell DoS | wpa_supplicant, switch port log, tcpdump |
| U-plane | eCPRI, input validation, MACsec optional | Does the O-DU reject a malformed or oversize eCPRI packet | U-plane malformed input, DoS | mutation layer + replay, tcpdump |
| S-plane | PTP (IEEE 1588), timing-node authorisation | Does the O-RU reject a rogue PTP master; hold clock on spoofed Announce | timing-flood DoS, rogue master, delay attack | PTP tooling, phc2sys offset, mutation layer |
| M-plane (later) | NETCONF over SSH/TLS, mutual auth, NACM | Does M-plane require mutual auth; reject no-cert client; TLS 1.2+ or SSH only | MITM, wiretap, DoS | ssh, NETCONF client, openssl, testssl.sh |

---

## 7. RTEC C7.3 fuzz capability

C7.3 asks which interfaces we can fuzz (F1, E1, E2, O1, Xn, Open Fronthaul M-plane)
and with what tooling. Harness = Scapy mutation layer + seed corpus + replay
(eCPRI, PTP, IKEv2 built; others extendable).

| Interface | Protocol | Standard | Fuzzable | Tooling | Priority |
|---|---|---|---|---|---|
| OFH C-plane | eCPRI | O-RAN | built | ecpri_packet.py + replay | **NOW** |
| OFH U-plane | eCPRI | O-RAN | built | ecpri_packet.py + replay | **NOW** |
| OFH S-plane | PTP | O-RAN | built | ptp_packet.py + replay | **NOW** |
| OFH M-plane | NETCONF/YANG | O-RAN | planned | NETCONF client + mutation | later |
| E2 | E2AP/SCTP | O-RAN | extendable | Scapy SCTP + E2AP (to build) | follow-on |
| O1 | NETCONF/YANG | O-RAN | extendable | as M-plane | follow-on |
| F1, E1, Xn | SCTP app protocols | 3GPP | extendable | Scapy SCTP (to build) | deferred (TS 33.501) |

The non-technical parts of C7.3 (vulnerability scanning, pen-test, slice isolation,
isolated enclave, responsible disclosure) are lab-capability statements, to be
answered with Prof. Ray and the lab engineer, not assumed here.

---

## 8. Week-by-week plan

| Week | Focus | Deliverable |
|---|---|---|
| This week (W2) | Scoping and this plan | This document, every TC mapped, tools listed, expected outcome per case |
| Next week (W3) | Run the OFH conformance test | Per control: enforced yes or no, with the capture or log that proves it |
| W4, if enforced | Grounding, capture baseline | Clean C/U/S seed corpus (PCAP) from the live link |
| W5 to W6 | Fuzz the OFH planes | U-plane and S-plane campaigns, iterations per the STS case, with a valid-input oracle, anomaly triage and a liveness probe |
| W6 | A1 and M-plane if time allows | A1 robustness; M-plane if O-RU reachable |
| W7 | Coverage matrix and STS 7.4 duty | Executed-vs-catalogue matrix; tool, version, settings, outputs, unexpected inputs |
| W8 | Paper, IEEE report, handover | Draft and submission; report PDF; slides and demo video |

The fuzz phase must carry the full loop: send a valid message and confirm the DUT
answers per spec (the oracle), mutate, on a failure classify it (correct reject is a
pass, crash is a fail), minimise the triggering packet, re-run to confirm, map back
to the clause. Check the DUT is alive during the run, or a crash halfway logs false
passes.

---

## 9. Normative basis and findings

Normative basis:
- Requirement: SRCS `REQ-SEC-TRAN-1` (clause 5.3.4.1), named in the U-plane fuzz cases.
- Scope: STS clause 7 and 11 robustness and fuzz cases.
- Procedure: 3GPP TS 33.117 clause 4.4.4 where STS gives no executable case.

Findings from reading the document:
1. No C-plane malformed-packet test in STS. C-plane security is 802.1X enforcement (conformance); the eCPRI fuzz is the U-plane.
2. The U-plane fuzz cases cite REQ-SEC-TRAN-1, matching our anchor.
3. Fuzz is spread across clauses 6, 7, 11 and 24, not only clause 11.
4. S-plane DoS timeTransmitter cases are written as procedures with no formal `TC_` ID; LLS-C4 is a local PRTC with no external path, so out of scope.

---

## 10. References

- STS: O-RAN.WG11.TS.STS-R005-v14.00 (read in full for this plan).
- SRCS: O-RAN Security Requirements and Controls Specifications (ETSI TS 104 104). https://www.etsi.org/deliver/etsi_TS/104100_104199/104104/09.01.00_60/ts_104104v090100p.pdf
- STS public version: ETSI TS 104 105. https://www.etsi.org/deliver/etsi_TS/104100_104199/104105/07.00.00_60/ts_104105v070000p.pdf
- SecProtSpec: ETSI TS 104 107. https://www.etsi.org/deliver/etsi_TS/104100_104199/104107/09.00.00_60/ts_104107v090000p.pdf
- Threat Model: ETSI TR 104 106. https://www.etsi.org/deliver/etsi_tr/104100_104199/104106/03.00.00_60/tr_104106v030000p.pdf
- 3GPP TS 33.117 (robustness and fuzz procedure).
