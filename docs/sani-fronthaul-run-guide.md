<h1 align="center">Open Fronthaul security test run (OAI gNB to Metanoia JURA O-RU)</h1>

> Skeleton for Sani, in the bmw SOP implementation-guide format
> ([example](https://github.com/petrajoyd/intern_repo/blob/main/docs/EXAMPLE_SOP_Guide_Format.md)).
> Fill the command blocks and outputs as you do the real hands-on run. The point is that anyone can
> reproduce it: you bring up the documented OAI gNB to JURA O-RU rig yourself and run the WG11 §11.1
> tests on top. OCUDU as the DU is the later follow-up (see Scope). Keep secret values out of Git,
> list pointers only. No em dashes (bmw writing rule).

## Table of Contents

- [Scope and DUT](#scope-and-dut)
- [Why this rig](#why-this-rig)
- [Rules of engagement and safety](#rules-of-engagement-and-safety)
- [Prerequisites](#prerequisites)
- [Project Root](#project-root)
- [Project Description](#project-description)
- [System Architecture](#system-architecture)
- [Access Method](#access-method)
- [Bring-up steps](#bring-up-steps)
- [Post-bring-up verification](#post-bring-up-verification)
- [Test execution and results](#test-execution-and-results)
- [Findings](#findings)
- [Known Issues](#known-issues)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

## Scope and DUT

**In scope (WG11 STS §11.1, against the real OAI gNB and JURA O-RU):**
- §11.1.2 LAN segment / 802.1X port access
- §11.1.4 U-plane input validation and robustness
- §11.1.5 S-plane (PTP) robustness

**Out of scope:** §11.1.3 M-plane (dropped per Joy, do not spend time on it).

**DUT (device under test):** Metanoia JURA O-RU, driven by an OAI gNB (monolithic BBU, FHI 7.2
driver) on a Dell R750. Anchors: WG11 STS R005 v13.00 §11.1; method in Joy's lab threat model
(private lab repo) §7; bench results in
[`COVERAGE.md`](https://github.com/bmw-ntust-internship/internship/blob/2026-TEEP-Sani/lab/results/COVERAGE.md).

**Later follow-up (not this run):** repeat the same tests with OCUDU as the DU. OCUDU is the DU on
Ravi's platform, so the OCUDU run is what lands the method on the platform. There is no OCUDU to JURA
integration on record yet, so do the proven OAI path first, document it, then OCUDU re-runs from the
same steps.

## Why this rig

The OAI gNB to JURA rig is a **self-contained fronthaul segment**: its own server (Dell R750), its
own fronthaul VLAN (3), and its own PTP (LinuxPTP on the R750, G.8275.1). It is **not part of the
StarlingX O-Cloud** (joule, kepler, OcNOS). So the severe tests (U-plane DoS, S-plane PTP) run here
without touching the shared build. It is also the documented, repeatedly validated path: the lab has
OAI gNB + JURA integration guides (see Additional Resources). You bring it up yourself from those
guides.

## Rules of engagement and safety

**Two rules.**

1. **Use the standalone rig only.** Before any severe test, confirm nothing shared (the O-Cloud,
   OcNOS, joule or kepler) is on this segment. Never bridge this rig to the shared fronthaul, and
   never point a DoS or fuzz tool at the shared build.
2. **Document everything as you go.** Every step, command, capture, result and blocker, with the date
   and the evidence link. If it is not written down, it did not happen.

**Safety.** The JURA is a live transmitting radio (band n78). Keep an antenna attached (the JURA
burns out without one) and keep it in the lab RF setup.

Blast radius of each test. On this standalone rig the effect is contained to the rig. The right-hand
column is what the same test would do on the shared O-Cloud fronthaul, which is why you never run it
there.

| Test | Effect on the standalone rig | Same test on the shared O-Cloud fronthaul |
| --- | --- | --- |
| §11.1.2 802.1X | plug in, try to auth on a port | observational |
| Read-only `tcpdump` | sniff and parse frames | low, no injection |
| §11.1.4 U-plane DoS / fuzz | floods mutated eCPRI; can knock this rig's RU or gNB offline, recover by restart | High. Saturates the shared link, can crash a shared Pegatron RU, can leak to other VLANs |
| §11.1.5 S-plane PTP attacks | injects onto this rig's PTP only; desyncs this rig, recover by restart | Severe. Desyncs timing for joule and kepler, not just the DUT. Recovery is manual |

Before you start:
- [ ] Confirmed this is the standalone OAI + JURA rig, nothing shared attached.
- [ ] Config snapshot taken (rollback point).
- [ ] Run order: 802.1X, then U-plane (rate ramped, stop at first effect), then S-plane last.

## Prerequisites

| Input | What it is | Example / pointer |
| --- | --- | --- |
| Access | SSH to the OAI gNB server (R750), console to the JURA, a 5G core | `ssh <user>@<r750-ip>` |
| Tooling | attacker + capture on the test machine | `tcpreplay`, Rico's tools in `vendor-tools/` |
| Seeds | real C/U-plane captures | `vendor-tools/cu-plane-dos-attacker/Traffic/` |
| Credentials | pointers only, never values | `<secret store path>` |

## Project Root

| Item | Value |
| --- | --- |
| SSH entry | `ssh <user>@<r750-ip>` |
| JURA console | `<serial console, e.g. COM port>` |
| Test machine | `<test-machine, NIC on the fronthaul segment>` |
| Working dir | `<path>` , every step runs here unless its `# run from:` says otherwise |

## Project Description

**Project Name:** Open Fronthaul security test run on the OAI gNB + JURA rig.

**Description:** Bring up the real OAI gNB to JURA O-RU fronthaul, then run the WG11 §11.1 LAN,
U-plane and S-plane tests against it, with evidence, so the procedure is reproducible.

**Target Users:** the next intern who will run security tests on the lab's fronthaul, and the OCUDU
re-run on Ravi's platform.

## System Architecture

Fill in real IPs. This rig is self-contained: nothing from the O-Cloud is attached.

```mermaid
flowchart LR
  core["5G core<br/>Open5GS / free5GC"]
  subgraph rig["Standalone rig (not the O-Cloud)"]
    direction LR
    du["OAI gNB<br/>Dell R750, FHI 7.2<br/>ens7f0 VF, VLAN 3"]
    ru["JURA O-RU<br/>Metanoia, n78 100 MHz"]
    du -->|"eCPRI C/U + PTP (G.8275.1), VLAN 3"| ru
  end
  test["Test machine<br/>attacker + capture"]
  core -->|"N2/N3"| du
  ru -->|"RF"| ue["UE"]
  test -->|"mutated frames, fronthaul segment"| du
```

## Access Method

```shell
# run from: your test machine
ssh <user>@<r750-ip>

# <paste the real banner / prompt>
```

## Bring-up steps

Bring the rig up from the lab's OAI gNB + JURA integration guides (see Additional Resources), then
record it here. One command per block, with its output commented in the same block, a `# run from:`
line, and a "What this proves" sentence. Replace every `<...>` with the real value and real output.

### Step 0: check the starting state

Confirm the fronthaul NIC and the SR-IOV VFs on the gNB server before you touch anything.

```shell
# run from: <r750>
ip -brief link

# <paste output: ens7f0 up, VFs present on VLAN 3>
```

What this proves: the fronthaul NIC and VFs are up.

### Step 1: bring up the core

Start the 5G core and confirm the AMF is up.

```shell
# run from: <core host>
<command to start the core>

# <paste output: AMF running>
```

What this proves: the core is up for NG Setup.

### Step 2: configure the JURA and confirm PTP sync

Push the M-plane config to the JURA (per the lab M-plane guide), then confirm it is PTP synchronised.

```shell
# run from: <r750 or SMO host>
/root/bin/mpcli.py -m get -p splane/ptp-time

# <paste output: JURA synchronized: YES>
```

What this proves: the JURA is locked to the fronthaul PTP.

### Step 3: bring up the OAI gNB

Start the OAI gNB and confirm the fronthaul interface initialises and NG Setup succeeds.

```shell
# run from: <r750>
<command to start the OAI gNB>

# <paste output: fronthaul init, NG Setup response, cell active>
```

What this proves: the gNB is up, attached to the core, and driving the fronthaul.

### Step 4: confirm the fronthaul link both directions

Confirm C/U-plane frames flow between the gNB and the JURA.

```shell
# run from: <test machine or r750>
sudo tcpdump -i <iface> -c 20 vlan 3

# <paste output: eCPRI C/U-plane frames to and from the JURA>
```

What this proves: the gNB and the JURA are exchanging fronthaul traffic.

## Post-bring-up verification

One check that proves the link is live before you start testing.

```shell
# run from: <r750>
<command that shows the cell up and the JURA connected>

# <paste output>
```

Pass when the gNB shows the cell active, the JURA connected and PTP locked.

## Test execution and results

Run in order: 802.1X, then U-plane (rate ramped, stop at first effect), then S-plane last. Record one
command block per test (same format as above), then fill this table. Verdict is PASS, FAIL (a
finding), or UNKNOWN (insufficient observation). Link the evidence (capture, log) per row.

| STS | Test | Expected | Observed | Verdict | Evidence |
| --- | --- | --- | --- | --- | --- |
| §11.1.2 | 802.1X port access | only an authorized cert passes, others blocked | `<fill>` | `<fill>` | `<link>` |
| §11.1.4 | U-plane malformed / unexpected size | DU or RU rejects, stays alive | `<fill>` | `<fill>` | `<link>` |
| §11.1.5 | S-plane PTP (rogue GM, drop, delay, flood) | per STS (defence audit) | `<fill>` | `<fill>` | `<link>` |

## Findings

Record each real finding: what you injected, what the DUT did, and whether it matches the bench
result. State clearly that this is a real-radio run on the OAI + JURA rig.

- `<finding 1>`
- `<finding 2>`

## Known Issues

| # | Behaviour | Handling |
| --- | --- | --- |
| `<n>` | `<what you hit>` | `<how you worked around it>` |

## Troubleshooting

### Problem 1: `<the symptom as the message says it>`

**Symptom.** `<what you saw>`

**Cause.** `<plain words>`

**Fix.**

```shell
# run from: <where>
<fix command>

# <paste output>
```

## Additional Resources

- SOP format spec: [`EXAMPLE_SOP_Guide_Format.md`](https://github.com/petrajoyd/intern_repo/blob/main/docs/EXAMPLE_SOP_Guide_Format.md).
- OAI gNB + JURA integration guides: `bmw-ece-ntust/summer-integration-oai`, folder `OAIgNB(FH7.2) with JURA RU/` (R750 and R740 builds; lab repo, ask Ravi for access).
- JURA M-plane bring-up: `bmw-ece-ntust/ravi_ric`, `installation/O-RU with O-Cloud OAI gNBFHI72.md` (lab repo).
- JURA RU notes (firmware, M-plane): `bmw-ece-ntust/o-ran-docs`, folder `RU/JURA/` (lab repo).
- M-plane config scripts (Pegatron and Metanoia-Jura): `bmw-ece-ntust/SMO-Mplane` (lab repo).
- What to do next: [`sani-fronthaul-todo.md`](sani-fronthaul-todo.md).
- Method and threat model: Joy's lab threat model (private lab repo) §7.
- Lab topology facts: Joy's lab topology map (private lab repo).
- Tools and seeds: Rico's tools under `vendor-tools/` in the internship repo.
- Bench results: [`COVERAGE.md`](https://github.com/bmw-ntust-internship/internship/blob/2026-TEEP-Sani/lab/results/COVERAGE.md).
