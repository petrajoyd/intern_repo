# WG11 Security Test Scope (full picture)

The full O-RAN WG11 security test landscape, by test area, and which part Sani is
working on now. Source: O-RAN Security Test Specifications (STS, ETSI TS 104 105,
V7.0.0). The top-level areas and the clause-6 protocol list are taken from the
document's own table of contents. Items marked `confirm` are exact sub-clause or
case numbers still to be checked against the PDF.

> This tree is the structure, the real document has about 178 individual test
> cases, so each leaf below stands for several `TC_*`, not one. For the interface
> fuzz capability that answers RTEC questionnaire C7.3, see
> [`RTEC_C7_3_Fuzz_Capability.md`](RTEC_C7_3_Fuzz_Capability.md).

Legend: `[NOW]` current work · `[later]` planned if time permits ·
`[not in scope]` WG11 covers it, Sani is not doing it · `[out]` deferred outside WG11.

```
O-RAN WG11 Security Test Specification (STS, ETSI TS 104 105)
│
├─ 6  Security Protocol and API Validation .......... [not in scope]
│   ├─ 6.2 SSH server and client
│   ├─ 6.3 TLS and mTLS
│   ├─ 6.4 DTLS
│   ├─ 6.5 IPsec / IKEv2
│   ├─ 6.6 OAuth 2.0
│   └─ REST API auth, authorization, input validation, logging
│
├─ 7  Common Network Security Tests ................. [partly: the fuzz basis]
│   └─ 7.4 Network Protocol Fuzzing .......... [the gap we fill]
│         names eCPRI, PTP, TLS, HTTP and more; no executable case supplied
│
├─ System and software security evaluations ......... [not in scope]
│       X.509 / certificate lifecycle (CMPv2), hardening   (clause confirm)
│
├─ 11  Security tests of O-RAN interfaces
│   ├─ 11.1 Open FH ......................... [NOW: Sani]
│   │   ├─ 11.1.2 Point-to-Point LAN Segment . [NOW] (carries C-plane: 802.1X, MACsec)
│   │   ├─ 11.1.3 M-Plane ................... [later: if O-RU reachable]
│   │   ├─ 11.1.4 U-Plane ................... [NOW]
│   │   └─ 11.1.5 S-Plane ................... [NOW]
│   ├─ 11.2 Y1 .............................. [not in scope]
│   ├─ 11.3 O1 .............................. [not in scope]
│   ├─ 11.4 O2 .............................. [not in scope]
│   ├─ 11.5 E2  (Near-RT RIC) ............... [later: follow-on]
│   ├─ 11.6 A1  (Non-RT RIC) ................ [later]
│   └─ 11.7 R1 ............................. [not in scope] (past the page shown)
│
├─ xApp security .................................... [not in scope]
│
├─ 18  O-Cloud security tests ....................... [not in scope]
│   ├─ 18.3.8  exploitation of O-Cloud component vulnerabilities
│   └─ 18.3.11 privilege-escalation prevention
│
├─ 24  End-to-end security .......................... [later]
│   └─ Near-RT RIC A1 robustness (fuzz)   (clause 24.2.3, confirm)
│
└─ Deferred outside WG11 ............................ [out]
    ├─ F1, E1, Xn    -> 3GPP TS 33.501
    └─ E2AP fuzzing  -> STS 7.4 names it, no case supplied -> future work
```

## Where Sani is now

One branch is live: Open Fronthaul C/U/S/M (clause 11.1), conformance first, then
fuzz. The order is conformance (is the control enforced), then grounding (capture a
clean baseline), then fuzz (250k mutated iterations per normative case). Detail and
the per-plane mapping: [`OFH_Conformance_WG11_Checklist.md`](OFH_Conformance_WG11_Checklist.md).

## What is verified vs pending

- Verified from the STS contents: the clause-6 protocol list (6.2 SSH, 6.3 TLS, 6.4 DTLS, 6.5 IPsec, 6.6 OAuth 2.0), clause 7 Common Network Security Tests, clause 18 O-Cloud tests, and clause 11 interface tests in full (11.1 Open FH with LAN Segment, M, U, S plane; 11.2 Y1; 11.3 O1; 11.4 O2; 11.5 E2; 11.6 A1; 11.7 R1). Clause 11 breakdown: [`STS_Clause11_Interface_Tests.md`](STS_Clause11_Interface_Tests.md).
- Pending the PDF: the end-to-end (clause 24) A1 robustness case, and every individual `TC_*`.

## Normative basis

- Requirement: SRCS (TS 104 104) `REQ-SEC-TRAN-1`, its note names random mutation and eCPRI.
- Scope: STS (TS 104 105) clause 7.4 Network Protocol Fuzzing, names the protocols, gives no fuzz test case.
- Procedure: 3GPP TS 33.117 clause 4.4.4, used where STS has no case.
