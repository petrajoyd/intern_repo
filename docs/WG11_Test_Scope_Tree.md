# WG11 Security Test Scope (full picture)

What O-RAN WG11 security testing covers, across every interface and test type, and
which part Sani is working on now. Source: O-RAN Security Test Specifications
(STS, ETSI TS 104 105). Clause numbers marked `confirm` still need the spec PDF.

> This is a draft map, not the full STS contents. The real STS has about 178
> test cases; this tree shows the structure and the current slice, not every
> case. Clause numbers are unverified until the PDF is read. For the interface
> fuzz capability that answers RTEC questionnaire C7.3, see
> [`RTEC_C7_3_Fuzz_Capability.md`](RTEC_C7_3_Fuzz_Capability.md).

Legend:
- `[NOW]` current work
- `[later]` planned if time permits
- `[not in scope]` WG11 covers it, Sani is not doing it
- `[out]` deferred outside WG11

```
O-RAN WG11 Security Test Specification (STS, ETSI TS 104 105)
│
├─ Common / transport security ...................... [not in scope]
│   ├─ TLS and mTLS                  (clause 6.3, confirm)
│   ├─ X.509 and certificate lifecycle, CMPv2  (clause 6.9, confirm)
│   ├─ OAuth 2.0                      (confirm)
│   └─ IPsec / IKEv2                  (clause 6.5.2 to 6.5.4, confirm)
│
├─ Open Fronthaul (clause 11.1) ..................... [NOW: Sani]
│   ├─ CUS-plane  (O-RU to O-DU)
│   │   ├─ C-plane   eCPRI  ......... conformance now, fuzz later
│   │   ├─ U-plane   eCPRI  ......... conformance now, fuzz later
│   │   └─ S-plane   PTP    ......... conformance now, fuzz later
│   └─ M-plane     NETCONF / YANG ... [later: if O-RU reachable]
│
├─ E2   (Near-RT RIC) ............................... [later: follow-on]
│   └─ IPsec, and A1-style robustness   (clause 11.5, confirm)
│
├─ A1   (Non-RT RIC) ................................ [later]
│   ├─ TLS / mTLS, OAuth 2.0          (clause 11.6, confirm)
│   └─ A1 robustness                  (clause 24.2.3, confirm)
│
├─ O1 / O2 / R1 .................................... [not in scope]
│
├─ Robustness and protocol fuzzing (clause 7.4) .... [the gap we fill]
│   names the protocols to fuzz (eCPRI, PTP, TLS, HTTP...)
│   but supplies no executable test case -> this is the contribution
│
└─ Deferred outside WG11 ........................... [out]
    ├─ F1, Xn      -> 3GPP TS 33.501
    └─ E2AP fuzzing -> future work
```

## Where Sani is now

One branch is live: Open Fronthaul C/U/S/M, conformance first, then fuzz. The
order is conformance (is the control enforced), then grounding (capture a clean
baseline), then fuzz (250k mutated iterations per normative case). Detail and the
per-plane mapping: [`OFH_Conformance_WG11_Checklist.md`](OFH_Conformance_WG11_Checklist.md).

## Normative basis

- Requirement: SRCS (TS 104 104) `REQ-SEC-TRAN-1`, its note names random mutation and eCPRI.
- Scope: STS (TS 104 105) clause 7.4, names the protocols, gives no fuzz test case.
- Procedure: 3GPP TS 33.117 clause 4.4.4, used where STS has no case.
