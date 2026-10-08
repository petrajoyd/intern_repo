# STS Open Fronthaul Test Case Coverage (11.1)

Every test case under STS clause 11.1 (Open FH), with which ones we test and which
we do not. Source: STS clause 11.1 body, pages 100 to 131 (ETSI TS 104 105).

> Status: template. The clause headings and pages are verified. The individual
> `TC_*` rows are pending the pages 100 to 131, which this session cannot open.
> They will be transcribed verbatim, not invented. Columns are ready to fill.

Legend for Type: C = conformance (positive, is the control enforced), F = fuzz /
robustness (malformed or unexpected input). In scope: yes = we run it now,
later = if reachable, no = out of scope. Tested: filled after the run.

## 11.1.2 Point-to-Point LAN Segment (carries C-plane, 802.1X, MACsec), p100

| TC ID | What it checks | Type | In scope | Tested | Evidence |
|---|---|---|---|---|---|
| _pending p100-108_ |  |  | yes |  |  |

## 11.1.3 M-Plane (NETCONF / YANG), p108

| TC ID | What it checks | Type | In scope | Tested | Evidence |
|---|---|---|---|---|---|
| _pending p108-121_ |  |  | later |  |  |

## 11.1.4 U-Plane (eCPRI), p121

| TC ID | What it checks | Type | In scope | Tested | Evidence |
|---|---|---|---|---|---|
| _pending p121-123_ |  |  | yes |  |  |

## 11.1.5 S-Plane (PTP), p123

| TC ID | What it checks | Type | In scope | Tested | Evidence |
|---|---|---|---|---|---|
| _pending p123-131_ |  |  | yes |  |  |

## How to read it once filled

- Each row is one STS test case, by its real ID.
- Type C rows are the conformance test (week 1). Type F rows are the fuzz (later).
- "In scope / Tested" is the coverage answer: what we cover and what we do not, which is the STS clause 7.4 documentation duty.

Clause map and sections: [`STS_Clause11_Interface_Tests.md`](STS_Clause11_Interface_Tests.md).
