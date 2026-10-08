# RTEC C7.3 Fuzz Capability (interface picture)

RTEC questionnaire C7.3 asks: which interfaces can you fuzz (F1, E1, E2, O1, Xn,
Open Fronthaul M-plane), with what tooling, plus vulnerability scanning,
penetration testing, slice isolation and resiliency testing, and whether you run
an isolated enclave with a responsible disclosure process.

This is the full interface picture for C7.3, and it marks the slice we execute
first: the Open Fronthaul.

## 1. Interface fuzz matrix

Standard: O-RAN = WG11 scope, 3GPP = 3GPP TS 33.501 scope (outside WG11).
Our harness = Scapy mutation layer + seed corpus + replay (eCPRI, PTP, IKEv2
built; others extendable).

| Interface | Protocol | Standard | Fuzzable with our harness | Tooling | Priority |
|---|---|---|---|---|---|
| Open Fronthaul C-plane | eCPRI | O-RAN | Yes, built | `ecpri_packet.py` + replay | **NOW (priority)** |
| Open Fronthaul U-plane | eCPRI | O-RAN | Yes, built | `ecpri_packet.py` + replay | **NOW (priority)** |
| Open Fronthaul S-plane | PTP (IEEE 1588) | O-RAN | Yes, built | `ptp_packet.py` + replay | **NOW (priority)** |
| Open Fronthaul M-plane | NETCONF / YANG | O-RAN | Yes, planned | NETCONF client + mutation | Later (if O-RU reachable) |
| E2 | E2AP over SCTP | O-RAN | Extendable | Scapy SCTP + E2AP grammar (to build) | Follow-on (STS 7.4 gap) |
| O1 | NETCONF / YANG | O-RAN | Extendable | same as M-plane | Follow-on |
| F1 | F1AP over SCTP | 3GPP | Extendable | Scapy SCTP + F1AP grammar (to build) | Deferred (3GPP TS 33.501) |
| E1 | E1AP over SCTP | 3GPP | Extendable | Scapy SCTP (to build) | Deferred (3GPP) |
| Xn | XnAP over SCTP | 3GPP | Extendable | Scapy SCTP (to build) | Deferred (3GPP) |

Note: C7.3 names the Open Fronthaul M-plane specifically. We lead with C/U/S
because the normative mutation test cases in STS are there (eCPRI and PTP), and
include M-plane when the O-RU is reachable.

## 2. Why Open Fronthaul first

- Three of the normative mutation cases in STS are Open Fronthaul (C-plane eCPRI, S-plane PTP).
- The lab already generates valid Open Fronthaul traffic, so the harness only adds the mutation layer.
- Finishing OFH gives a concrete, evidenced answer to C7.3 for the Open Fronthaul row, then E2, O1 and the 3GPP interfaces follow the same harness pattern.

## 3. The other C7.3 parts (for the lab to confirm)

These are lab-capability statements, not something the harness proves. To be
answered with Prof. Ray and the lab engineer, not assumed here:

- Vulnerability scanning: tooling in use (for example trivy, grype, pip-audit).
- Penetration testing capability: who, scope, past campaigns.
- Slice isolation and resiliency testing: whether a method exists.
- Isolated enclave and responsible disclosure: whether the lab runs one, and the documented process.

## 4. Status

Draft. The fuzz matrix (section 1) reflects what the harness can do now. Clause
backing for each case is pending the STS PDF, tracked in
[`WG11_Test_Scope_Tree.md`](WG11_Test_Scope_Tree.md) and
[`OFH_Conformance_WG11_Checklist.md`](OFH_Conformance_WG11_Checklist.md).
