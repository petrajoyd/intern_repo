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
├─ 11  Interface security tests
│   ├─ 11.1 Open Fronthaul ................... [NOW: Sani]
│   │   ├─ CUS-plane  (O-RU to O-DU)
│   │   │   ├─ C-plane   eCPRI  ....... conformance now, fuzz later
│   │   │   ├─ U-plane   eCPRI  ....... conformance now, fuzz later
│   │   │   └─ S-plane   PTP    ....... conformance now, fuzz later
│   │   └─ M-plane     NETCONF / YANG ... [later: if O-RU reachable]
│   ├─ E2   (Near-RT RIC) ................... [later: follow-on]  (clause 11.x, confirm)
│   ├─ A1   (Non-RT RIC) .................... [later]             (clause 11.x, confirm)
│   ├─ O1 ................................... [not in scope]      (clause 11.x, confirm)
│   ├─ O2 ................................... [not in scope]      (clause 11.x, confirm)
│   └─ 11.7 R1 ............................. [not in scope]
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

- Verified from the STS table of contents: the clause-6 protocol list (6.2 SSH, 6.3 TLS, 6.4 DTLS, 6.5 IPsec, 6.6 OAuth 2.0), clause 7 Common Network Security Tests, clause 11 interface tests with 11.1 Open Fronthaul and 11.7 R1, clause 18 O-Cloud tests, and the test-area set (protocol and API validation, common network security, system and software evaluations, Open Fronthaul, xApp security, O-Cloud, end-to-end).
- Pending the PDF: the exact sub-clause numbers for E2, A1, O1, O2, the end-to-end A1 robustness case, and every individual `TC_*`.

## Normative basis

- Requirement: SRCS (TS 104 104) `REQ-SEC-TRAN-1`, its note names random mutation and eCPRI.
- Scope: STS (TS 104 105) clause 7.4 Network Protocol Fuzzing, names the protocols, gives no fuzz test case.
- Procedure: 3GPP TS 33.117 clause 4.4.4, used where STS has no case.
