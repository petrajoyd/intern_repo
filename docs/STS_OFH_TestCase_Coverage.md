# STS Section 11 Test Case Coverage

Every security test case under STS clause 11 (Security tests of O-RAN interfaces),
with which ones we test and which we do not. Read directly from the document:
O-RAN.WG11.TS.STS-R005-v14.00. Each row is a real `TC_*` taken from the "Test Name"
field in the clause body.

Legend. Type: C = conformance (positive, is the control enforced), F = fuzz /
robustness / DoS (malformed or unexpected input or flooding). Scope: NOW = we run
it now, later = if the O-RU or interface is reachable, no = out of scope. Tested:
filled after each run (none run yet).

## Part A. 11.1 Open FH (our scope)

### 11.1.2 Point-to-Point LAN Segment (802.1X, carries the C-plane), p100

| TC ID | Type | Scope | Tested |
|---|---|---|---|
| TC_Authenticator_Validation | C | NOW | |
| TC_Supplicant_Validation | C | NOW | |
| TC_Port_Access_Enforcement_Validation | C | NOW | |
| TC_ORU_Authentication_Lifecycle_Validation | C | NOW | |
| TC_OFHPLS_Unauthorized_Port_Behavior | C (negative) | NOW | |
| TC_OFHPLS_Authorization_Policy_Enforcement | C | NOW | |

### 11.1.3 M-Plane (SSH / TLS / NACM / file transfer), p108

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

Requirement reference (from the body): REQ-SEC-TRAN-1, SRCS clause 5.3.4.1 and
5.2.5.2.1. This is the fuzz anchor.

### 11.1.5 S-Plane (PTP timing), p123

| TC ID | Type | Scope | Tested |
|---|---|---|---|
| DoS timeTransmitter LLS C1 C2 C3 (no formal TC_ ID in body) | F (DoS) | NOW | |
| DoS timeTransmitter LLS C4 (no formal TC_ ID in body) | F (DoS) | no (local PRTC, no external path) | |
| TC_ROGUE_PTP_INSTANCE | F | NOW | |
| TC_SELECTIVE_INTERCEPTION_REMOVAL_PTP_TIMING_PACKETS | F | NOW | |
| TC_DELAY_ATTACK_PTP_TIMING_PACKETS | F | NOW | |

S-plane spoofing requirement (from the body): SEC-CTL-OFSP-3, SRCS clause 5.2.5.3.3.3.1.

## Part B. Rest of section 11 (not our scope now)

| Clause | TC ID | Type | Scope |
|---|---|---|---|
| 11.2 Y1 | TC_Y1_AUTHENTICATION | C | no |
| 11.2 Y1 | TC_Y1_CONFIDENTIALITY_INTEGRITY_REPLAY | C | no |
| 11.2 Y1 | TC_Y1_AUTHORIZATION | C | no |
| 11.3 O1 | TC_O1_AUTHENTICATION | C | no |
| 11.3 O1 | TC_O1_CONFIDENTIALITY_INTEGRITY_REPLAY | C | no |
| 11.3 O1 | TC_O1_NACM_VALIDATION | C | no |
| 11.3 O1 | TC_OFH_O1_FTPES | C | no |
| 11.3 O1 | TC_OFH_O1_SFTP | C | no |
| 11.4 O2 | TC_O2_AUTHENTICATION | C | no |
| 11.4 O2 | TC_O2_CONFIDENTIALITY_INTEGRITY_REPLAY | C | no |
| 11.4 O2 | TC_O2_AUTHORIZATION | C | no |
| 11.5 E2 | TC_E2_CONFIDENTIALITY_INTEGRITY_REPLAY | C | later |
| 11.5 E2 | TC_E2_AUTHENTICATION_CERT | C | later |
| 11.5 E2 | TC_E2_AUTHENTICATION_PSK | C | later |
| 11.5 E2 | TC_E2_Interface_data_validation_by_NearRTRIC | F | later |
| 11.6 A1 | TC_A1_Authentication | C | later |
| 11.6 A1 | TC_A1_CONFIDENTIALITY_INTEGRITY_REPLAY | C | later |
| 11.6 A1 | TC_A1_Authorization | C | later |
| 11.7 R1 | TC_R1_AUTHENTICATION | C | no |
| 11.7 R1 | TC_R1_CONFIDENTIALITY_INTEGRITY_REPLAY | C | no |
| 11.7 R1 | TC_R1_AUTHORIZATION | C | no |
| 11.8 D2 | TC_D2_CONFIDENTIALITY_INTEGRITY_REPLAY_IPSEC | C | no |

## Summary

- Our scope now (11.1 Open FH): 6 LAN Segment conformance cases, 2 U-plane fuzz cases, 4 S-plane robustness cases = **12 cases NOW**.
- Later (if O-RU reachable): 13 M-plane cases. E2 and A1 are follow-on.
- Out of scope: Y1, O1, O2, R1, D2, and S-plane LLS-C4.

## Notes

- There is no C-plane malformed-packet case in STS. C-plane security is the 802.1X enforcement set in 11.1.2 (conformance). The eCPRI malformed and payload fuzz is specified on the U-plane (11.1.4).
- Related eCPRI cases live in clause 6 (common tests): TC_eCPRI_INPUT_VALIDATION and TC_eCPRI_TIMEOUT_HANDLING. Worth running alongside the U-plane fuzz.
- Per plane: run the conformance (C) cases first (is the control enforced), then the fuzz (F) cases, per REQ-SEC-TRAN-1 with its 250k-iteration rule.

Source document: O-RAN.WG11.TS.STS-R005-v14.00. Clause map and sections:
[`STS_Clause11_Interface_Tests.md`](STS_Clause11_Interface_Tests.md).
