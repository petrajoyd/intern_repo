# WG11 Security Test Scope (full picture)

The full O-RAN WG11 security test landscape and which part Sani is working on now.
Read directly from O-RAN.WG11.TS.STS-R005-v14.00. The document has 25 test clauses
and 189 test cases in total. This tree shows every top-level clause and marks the
one slice in scope.

Legend: `[NOW]` current work · `[later]` planned if reachable ·
`[not in scope]` WG11 covers it, Sani is not doing it · `[out]` deferred outside WG11.

```
O-RAN WG11 Security Test Specifications (STS, R005 v14.00), 189 test cases
│
├─ 6  Security Protocol and API Validation .......... [not in scope]
│     SSH, TLS, DTLS, IPsec/IKE (incl IKE fuzzing),
│     OAuth2, X.509, eCPRI input validation + timeout,
│     SCTP, REST API, MACsec, SFTP/FTPES
│
├─ 7  Common Network Security Tests ................. [not in scope]
│     enumeration, password policy, robustness/DDoS,
│     overload handling, input-validation error handling,
│     config enforcement, logging
│
├─ 8  System security evaluation .................... [not in scope]
│     vulnerability scanning, data protection, system logging
│
├─ 9  Software security evaluation .................. [not in scope]
│     OSS component analysis, binary static analysis, SBOM, signatures
│
├─ 10 ML security validation ........................ [not in scope]
│
├─ 11 Security tests of O-RAN interfaces
│   ├─ 11.1 Open FH .......................... [NOW: Sani]  (12 cases now, 13 later)
│   │   ├─ 11.1.2 Point-to-Point LAN Segment .. [NOW] 6 conformance (C-plane, 802.1X)
│   │   ├─ 11.1.3 M-Plane ................... [later: if O-RU reachable] 13 conformance
│   │   ├─ 11.1.4 U-Plane ................... [NOW] 2 fuzz (eCPRI malformed, payload size)
│   │   └─ 11.1.5 S-Plane ................... [NOW] 4 robustness (rogue PTP, delay, DoS)
│   ├─ 11.2 Y1 .............................. [not in scope]
│   ├─ 11.3 O1 .............................. [not in scope]
│   ├─ 11.4 O2 .............................. [not in scope]
│   ├─ 11.5 E2 .............................. [later: follow-on]
│   ├─ 11.6 A1 .............................. [later]
│   ├─ 11.7 R1 .............................. [not in scope]
│   └─ 11.8 D2 .............................. [not in scope]
│
├─ 12 Security test of O-RU ......................... [not in scope]
├─ 13 Security test of Near-RT RIC ................. [not in scope]
├─ 14 Security test of xApps ....................... [not in scope]
├─ 15 Security test of Non-RT RIC .................. [not in scope]
├─ 16 Security test of rApps ....................... [not in scope]
├─ 17 Security test of SMO ......................... [not in scope]
├─ 18 Security test of O-Cloud ..................... [not in scope]
├─ 19 Security test of VNF/CNF ..................... [not in scope]
├─ 20 Common Application Lifecycle Management ...... [not in scope]
├─ 21 Security test of O-CU-CP ..................... [not in scope]
├─ 22 Security test of O-CU-UP ..................... [not in scope]
├─ 23 Security test of O-DU ........................ [not in scope]
├─ 24 End-to-End security test cases ............... [later]
│     includes the A1 / Near-RT RIC robustness cases
├─ 25 Security test of Shared O-RU ................. [not in scope]
│
└─ Deferred outside WG11 ............................ [out]
    └─ F1, E1, Xn   -> 3GPP TS 33.501
```

## Where the fuzz lives (not only in clause 11)

Robustness and fuzz cases are spread across the document, not in one place:
- clause 6: IKE fuzzing, eCPRI input validation and timeout handling
- clause 7: robustness/DDoS, overload handling, input-validation error handling
- clause 11.1.4 U-Plane: eCPRI malformed packet and unexpected payload size
- clause 11.1.5 S-Plane: rogue PTP, delay attack, DoS timeTransmitter
- clause 24: end-to-end robustness

Our scope covers the clause 11.1 Open FH fuzz (U-plane, S-plane) plus the matching
clause 6 eCPRI cases.

## Where Sani is now

One branch: clause 11.1 Open FH. Conformance first (the LAN Segment 802.1X cases,
which cover the C-plane), then fuzz (U-plane and S-plane). M-plane later if the O-RU
is reachable. The 12 cases in scope now, the 13 M-plane cases, and the rest of
section 11, are listed per test case in
[`STS_OFH_TestCase_Coverage.md`](STS_OFH_TestCase_Coverage.md). Clause 11 structure:
[`STS_Clause11_Interface_Tests.md`](STS_Clause11_Interface_Tests.md).

## Normative basis

- Requirement: SRCS `REQ-SEC-TRAN-1` (clause 5.3.4.1), named in the U-plane fuzz cases.
- Scope: STS clause 7 and 11 robustness and fuzz cases.
- Procedure: 3GPP TS 33.117 clause 4.4.4 where STS gives no executable case.

Source document: O-RAN.WG11.TS.STS-R005-v14.00.
