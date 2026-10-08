# STS Clause 11, Security Tests of O-RAN Interfaces

The interface security tests in the O-RAN WG11 Security Test Specifications (STS,
ETSI TS 104 105). Transcribed from the document's clause 11 contents. This is the
table prof asked for, then broken into sections, with Sani's current work marked.

Verified: clauses 11.1 to 11.6 and their sub-clauses and page numbers are taken
from the STS clause 11 table of contents. 11.7 R1 continues past the page shown and
is listed from an earlier reference.

## 1. Clause 11 table

| Clause | Title | Page | Section | Scope |
|---|---|---|---|---|
| 11 | Security tests of O-RAN interfaces | 100 | | |
| 11.1 | Open FH | 100 | Open Fronthaul | **NOW** |
| 11.1.1 | Overview | 100 | Open Fronthaul | NOW |
| 11.1.2 | Open Fronthaul Point-to-Point LAN Segment | 100 | Open Fronthaul | **NOW** (carries C-plane) |
| 11.1.3 | M-Plane | 108 | Open Fronthaul | later (if O-RU reachable) |
| 11.1.4 | U-Plane | 121 | Open Fronthaul | **NOW** |
| 11.1.5 | S-Plane | 123 | Open Fronthaul | **NOW** |
| 11.2 | Y1 | 131 | Y1 | not in scope |
| 11.2.1 | Y1 Authenticity | 131 | Y1 | not in scope |
| 11.2.2 | Y1 confidentiality, integrity, and replay protection | 132 | Y1 | not in scope |
| 11.2.3 | Y1 Authorization | 134 | Y1 | not in scope |
| 11.3 | O1 | 135 | O1 | not in scope |
| 11.3.1 | O1 Authenticity | 135 | O1 | not in scope |
| 11.3.2 | O1 confidentiality, integrity and replay protection | 136 | O1 | not in scope |
| 11.3.3 | O1 Interface Network Configuration Access Control Model (NACM) Validation | 136 | O1 | not in scope |
| 11.3.4 | O1 Interface Secure File Transfer | 138 | O1 | not in scope |
| 11.4 | O2 | 139 | O2 | not in scope |
| 11.4.1 | O2 Authenticity | 139 | O2 | not in scope |
| 11.4.2 | O2 confidentiality, integrity and replay protection | 140 | O2 | not in scope |
| 11.4.3 | O2 Authorization | 141 | O2 | not in scope |
| 11.5 | E2 | 143 | E2 | later: follow-on |
| 11.5.1 | E2 confidentiality, integrity and replay protection | 143 | E2 | later |
| 11.5.2 | E2 Authenticity | 143 | E2 | later |
| 11.6 | A1 | 146 | A1 | later |
| 11.6.1 | A1 Authenticity | 146 | A1 | later |
| 11.6.2 | A1 confidentiality, integrity and replay protection | 147 | A1 | later |
| 11.7 | R1 | (past page shown) | R1 | not in scope |

## 2. Sections

Clause 11 is one section per interface. Each interface is tested along the same
security properties: Authenticity, Confidentiality / integrity / replay, and
Authorization. Those property sub-clauses are the conformance (positive) tests.
The malformed-input and robustness cases (fuzz) sit inside the same plane or
property sub-clauses.

- **11.1 Open FH**  the O-RU to O-DU link. Split into the Point-to-Point LAN Segment (11.1.2, where C-plane and the shared link protections live, 802.1X and MACsec), then M-Plane (11.1.3), U-Plane (11.1.4), S-Plane (11.1.5).
- **11.2 Y1**  the Y1 interface (RAN analytics exposure). Authenticity, CIA and replay, Authorization.
- **11.3 O1**  management interface. Adds NACM validation and secure file transfer on top of Authenticity and CIA.
- **11.4 O2**  O-Cloud interface. Authenticity, CIA and replay, Authorization.
- **11.5 E2**  Near-RT RIC interface. CIA and replay, Authenticity.
- **11.6 A1**  Non-RT RIC policy interface. Authenticity, CIA and replay.
- **11.7 R1**  SMO / rApp interface. (Past the page shown.)

## 3. Where Sani's work sits

Under **11.1 Open FH**. In the real structure that means:

| Sub-clause | Plane | Now or later |
|---|---|---|
| 11.1.2 Point-to-Point LAN Segment | C-plane and shared link (802.1X, MACsec) | **NOW** |
| 11.1.4 U-Plane | eCPRI user data | **NOW** |
| 11.1.5 S-Plane | PTP timing | **NOW** |
| 11.1.3 M-Plane | NETCONF / YANG | later, if the O-RU is reachable |

Correction to our earlier framing: there is no standalone "C-Plane" test section in
the STS. C-plane security is tested inside 11.1.2 (the Point-to-Point LAN Segment).
So the scope reads LAN Segment (with C-plane), U-Plane, S-Plane now, M-Plane later,
not "C/U/S/M" as separate planes.

Order per sub-clause: run the positive security cases first (is the control
enforced, that is conformance), then the malformed / robustness cases (fuzz), per
`REQ-SEC-TRAN-1` with its 250k-iteration rule.

Everything outside 11.1 (Y1, O1, O2, E2, A1, R1) is later or out of scope. Full
landscape: [`WG11_Test_Scope_Tree.md`](WG11_Test_Scope_Tree.md). Per-control
detail: [`OFH_Conformance_WG11_Checklist.md`](OFH_Conformance_WG11_Checklist.md).
