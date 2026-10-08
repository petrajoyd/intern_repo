# Open Fronthaul Conformance Test, WG11 Mapping Checklist

Reviewer: Joy. Author of the work under review: Rizky Wildansani (San), branch
`2026-TEEP-Sani`. Date: 2026-10-08.

Purpose: before any fuzzing, confirm that each O-RAN WG11 security control on the
Open Fronthaul (OFH) is actually enforced on the lab link. This is the technical
note for this week. It lists the test cases, how to run them, the tools, and the
expected outcome, and it ties every case to a real OFH threat.

OFH = Open Fronthaul, the O-DU to O-RU link. O-DU = O-RAN Distributed Unit.
O-RU = O-RAN Radio Unit. C/U/S/M = Control, User, Synchronisation and Management
planes.

> [!IMPORTANT]
> Verification status. The document identities, versions, the clause-11
> structure, the per-plane control families and the threat scheme below were
> checked against O-RAN and ETSI published material. The exact sub-clause numbers
> and `TC_*` names marked `confirm` still have to be read off the spec PDFs. That
> read is this week's grounding task. Nothing in a `confirm` cell is to be quoted
> as final until it is checked against the PDF.

## 1. Test basis (the documents to cite)

Every reference below is public. The O-RAN Alliance publishes these, and ETSI
republishes them as Publicly Available Specifications (PAS).

| Short name | Document | Version checked | Use |
|---|---|---|---|
| STS | O-RAN Security Test Specifications, ETSI TS 104 105 | V7.0.0, 2025-06 (O-RAN R003 v07.00; O-RAN-native is higher, up to v13) | The `TC_*` security test cases. OFH tests are in clause 11.1 |
| SRCS | O-RAN Security Requirements and Controls Specifications, ETSI TS 104 104 | V9.1.0, 2025-06 (O-RAN SecReqSpecs R003 v09.01) | The `REQ-SEC-OF*` requirements and `SEC-CTL-*` controls |
| SecProtSpec | O-RAN Security Protocols Specifications, ETSI TS 104 107 | V9.0.0, 2025-05 | How each control is realised: MACsec, 802.1X, SSH, TLS |
| Threat Model | O-RAN Security Threat Modeling and Remediation Analysis, ETSI TR 104 106 | V3.0.0, 2025-06 (O-RAN-native O-R004 v05.00) | The threat IDs each test defends against |
| WG4 CUS | O-RAN WG4 Control, User and Synchronization Plane Specification | confirm | Protocol detail for eCPRI, PTP, SyncE |
| WG4 MP | O-RAN WG4 Management Plane Specification | confirm | Protocol detail for NETCONF or YANG over the M-plane |
| TS 33.117 | 3GPP Security Assurance, general SBA test cases | 4.4.4 | The robustness and fuzz procedure where STS gives no case |

References:
- STS (ETSI TS 104 105 V7.0.0): https://www.etsi.org/deliver/etsi_TS/104100_104199/104105/07.00.00_60/ts_104105v070000p.pdf
- SRCS (ETSI TS 104 104 V9.1.0): https://www.etsi.org/deliver/etsi_TS/104100_104199/104104/09.01.00_60/ts_104104v090100p.pdf
- SecProtSpec (ETSI TS 104 107 V9.0.0): https://www.etsi.org/deliver/etsi_TS/104100_104199/104107/09.00.00_60/ts_104107v090000p.pdf
- Threat Model (ETSI TR 104 106 V3.0.0): https://www.etsi.org/deliver/etsi_tr/104100_104199/104106/03.00.00_60/tr_104106v030000p.pdf
- O-RAN threat model, native copy: https://specifications.o-ran.org/download?id=846
- O-RAN Alliance security update 2025: https://www.o-ran.org/blog/o-ran-alliance-security-update-2025

> [!NOTE]
> Two kinds of "conformance" exist and must not be mixed. WG4 publishes the
> functional Conformance Test Specification (does the M-plane or CUS-plane work).
> WG11 STS publishes the security test cases (is the control enforced). This
> week's note is about the WG11 security side: does each control turn on and
> reject what it should.

## 2. Scope of STS clause 11.1 (verified)

Clause 11 of STS covers security tests of the O-RAN interfaces. Clause 11.1 is the
Open Fronthaul. Its stated scope:

- Open Fronthaul CUS-plane between O-RU and O-DU.
- Open Fronthaul M-plane between O-RU and O-DU, and between O-RU and SMO.
- One device under test (DUT) or system under test at a time.

SMO = Service Management and Orchestration.

## 3. Mapping checklist

Legend. `[ ]` to do, `[x]` done. "Conformance check" is the question the test
answers: is the control on and does it reject what it should. "Threat" names the
OFH threat the control defends against, from the Threat Model.

### 3.1 C-Plane (eCPRI control messages, O-DU to O-RU)

| [ ] | Requirement (SRCS) | Control and mechanism | Conformance check | Threat defended | Spec ref | Tool |
|---|---|---|---|---|---|---|
| [ ] | REQ-SEC-OFCP-1 (verified wording: the C-plane shall support authentication and authorization of O-DUs that exchange C-plane messages with O-RUs) | IEEE 802.1X-2020 port-based access control | Is 802.1X enforced on the C-plane port. Does the port drop a supplicant that is not authorised (EAPOL only, frames blocked) | C-plane message injection posing as the O-DU, leading to cell DoS (Threat Model, fronthaul; confirm ID, e.g. T-FRHAUL-nn) | SRCS OFCP; STS 11.1 (confirm case); SecProtSpec 802.1X clause (confirm) | wpa_supplicant state, switch port log, tcpdump |
| [ ] | REQ-SEC-OFCP-2..4 (confirm wording) | MACsec optional for integrity and confidentiality | Is MACsec available and, if required, enforced on the C-plane link | C-plane tampering, eavesdrop | SRCS OFCP (confirm); SecProtSpec MACsec clause (confirm) | MACsec status (`ip macsec show`), tcpdump |

### 3.2 U-Plane (eCPRI user data, O-DU to O-RU)

| [ ] | Requirement (SRCS) | Control and mechanism | Conformance check | Threat defended | Spec ref | Tool |
|---|---|---|---|---|---|---|
| [ ] | REQ-SEC-OFUP-1..4 (confirm wording) | PDCP for confidentiality, integrity and replay prevention; MACsec optional on OFH | Is the U-plane protected as the control states, or is it explicitly out of protection scope on this link | U-plane injection, eavesdrop, replay | SRCS OFUP (confirm); SecProtSpec (confirm) | tcpdump, MACsec status |

PDCP = Packet Data Convergence Protocol.

### 3.3 S-Plane (timing: PTP per IEEE 1588-2019, SyncE)

| [ ] | Requirement (SRCS) | Control and mechanism | Conformance check | Threat defended | Spec ref | Tool |
|---|---|---|---|---|---|---|
| [ ] | REQ-SEC-OFSP-1..8 and SEC-CTL-OFSP-1..4 (confirm wording) | MACsec optional; input validation on the S-plane; timing-node authorisation | Does the O-RU reject a rogue PTP master. Does it hold clock on spoofed Announce. Is input validation present | DoS by flooding the master clock with timing packets; rogue-master takeover (Threat Model; confirm ID) | SRCS OFSP; STS 11.1 (confirm case); WG4 CUS | PTP tooling, phc2sys offset log, tcpdump |

PTP = Precision Time Protocol.

### 3.4 M-Plane (NETCONF or YANG, O-RU to O-DU and O-RU to SMO), test only if reachable

| [ ] | Requirement (SRCS) | Control and mechanism | Conformance check | Threat defended | Spec ref | Tool |
|---|---|---|---|---|---|---|
| [ ] | REQ-SEC-OFHM-1..4 and REQ-SEC-OFHPLS-1..3 (confirm wording) | NETCONF over SSH or TLS; mutual authentication (mTLS or SSH host and client keys), mandatory in the newer spec version | Does the M-plane require mutual auth. Does it reject a client with no valid certificate. Is only TLS 1.2 or higher, or SSH, accepted | M-plane man-in-the-middle, passive wiretap, DoS (STRIDE classes in the Threat Model; confirm ID) | SRCS OFHM; STS 11.1 (confirm case); SecProtSpec TLS and SSH clauses; WG4 MP | ssh, a NETCONF client, openssl s_client, testssl.sh |

NETCONF = Network Configuration Protocol. mTLS = mutual Transport Layer Security.

## 4. How this maps to the week's deliverable

| Plan item | Where it is in this checklist |
|---|---|
| 1. Test cases | the Requirement column plus the STS case in the Spec-ref column, one row per control |
| 2. How we test | the Conformance check column, each expanded into numbered steps before the run |
| 3. Tools | the Tool column (San confirms each tool and its version) |
| 4. Expected outcome | per row: control enforced yes or no, with the capture or log that proves it |

## 5. To confirm this week (the grounding)

- [ ] Open STS (TS 104 105) clause 11.1 and read the exact OFH `TC_*` names and sub-clauses. Fill every `confirm` cell.
- [ ] Open SRCS (TS 104 104) and confirm the exact `REQ-SEC-OF*` and `SEC-CTL-OF*` IDs and wording per plane.
- [ ] Open the Threat Model (TR 104 106) and replace Sani's `T-CPLANE-*` IDs with the real fronthaul threat IDs (`T-FRHAUL-nn`, `T-O-RAN-nn`). Link one threat ID per row.
- [ ] Mark each row testable or blocked against the lab preconditions below.

## 6. Overall plan (week by week)

The order is deliberate. Confirm the controls are on before capturing baseline
traffic, and capture a clean baseline before fuzzing. No point fuzzing an
interface whose controls were never enforced.

| Week | Focus | Deliverable |
|---|---|---|
| This week (W2) | Scoping and this technical note | This checklist, with every `confirm` cell filled from the PDFs, tools listed, expected outcome per case |
| Next week (W3) | Run the OFH security conformance test | Per control: enforced yes or no, with the capture or log that proves it |
| W4, only if controls are enforced | Grounding, capture baseline | Clean C/U/S seed corpus (PCAP) from the live link, used as the mutation template |
| W5 to W6 | Fuzz the OFH planes | Mutation campaigns (eCPRI, PTP), iterations per the STS normative case, each with a valid-input baseline oracle, anomaly triage and a liveness probe |
| W6 | A1 and M-plane robustness if time allows | A1 robustness case; M-plane robustness if the O-RU is reachable |
| W7 | Coverage matrix and the STS clause 7.4 documentation duty | Executed-versus-catalogue matrix; tool, version, settings, outputs, and the inputs that caused unexpected behaviour |
| W8 | Paper, IEEE report, handover | Draft and submission; report PDF; slides and demo video |

The fuzz phase in W5 to W6 must carry the full loop, which the current harness is
missing: send a valid message and confirm the DUT answers per spec (the oracle),
then mutate, then on a failure classify it (a correct reject is a pass, a crash is
a fail), minimise the triggering packet, re-run to confirm, and map it back to the
clause. Check the DUT is still alive during the run, or a crash halfway logs false
passes for the rest.
