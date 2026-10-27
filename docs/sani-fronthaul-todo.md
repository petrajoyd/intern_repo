# Sani: run the Open Fronthaul tests for real, what to do next

For Rizky Wildansani (Sani). Written by Joy, 2026-10-22, updated 2026-10-27 (JURA rig).

Your bench work is done and the method is proven (12/12 NOW cases, real O-RAN-FH parser, IKEv2, A1).

**The goal now is one thing: run your tests on a real build, with your own hands.** Not more
coverage, not a perfect product verdict. I want you to bring up a real DU and RU yourself and run the
tests on real gear, so you get the hands-on experience. M-plane is out of scope, do not spend time on
it.

**Good news: you do not need the shared O-Cloud or a booked window.** Prof Ray pointed to the
Metanoia JURA O-RU, and the lab already has a documented OAI gNB to JURA rig that is self-contained:
its own server (Dell R750), its own fronthaul VLAN (3), its own PTP. It is not part of the StarlingX
O-Cloud (joule, kepler, OcNOS), so the severe tests run on it without touching the shared build. You
bring it up yourself from the lab's integration guides and run your tests on top.

## The two rules

1. **Use the standalone rig only.** The OAI + JURA rig is separate from the shared O-Cloud. Before any
   severe test, confirm nothing shared is on the segment. Never bridge it to the shared fronthaul, and
   never point a DoS or fuzz tool at the shared O-Cloud (the OcNOS switch and the PTP domain 24 are
   shared by joule, kepler and the DGX Spark, and PTP breakage has knocked joule over before).
2. **Document everything as you go.** Every step, command, capture, result and blocker, in your daily
   log with the date and the evidence link. If it is not written down, it did not happen. The
   hands-on write-up is the point of this exercise.

## Why OAI now, OCUDU later

The lab has a documented, validated OAI gNB to JURA build. There is no OCUDU to JURA integration on
record yet. So do the proven OAI path first: the DU side just works, and the only real variable is the
fronthaul and the O-RU, which is what you are testing. Your C/U/S findings transfer to any DU, so once
the method is proven and documented here, repeating it with OCUDU (the DU on Ravi's platform) is a
clean follow-up. That OCUDU re-run is what lands the method on the platform, so it is the next step,
not this one.

## Blast radius (why rule 1 exists)

On the standalone rig the effect of each test is contained to the rig. The last column is what the
same test would do on the shared O-Cloud fronthaul, which is why you never run it there.

| Test | Effect on the standalone rig | Same test on the shared O-Cloud fronthaul |
|---|---|---|
| §11.1.2 802.1X | plug in, try to auth on a port | observational |
| Read-only `tcpdump` | sniff and parse frames | low, no injection |
| §11.1.4 **U-plane DoS / fuzz** | floods mutated eCPRI; can knock this rig's RU or gNB offline, recover by restart | **High.** Saturates the shared link, can crash a shared Pegatron RU, can leak to VLAN 103 (kepler, DGX Spark) |
| §11.1.5 **S-plane PTP attacks** | injects onto this rig's PTP only; recover by restart | **Severe.** Desyncs timing for joule and kepler, not just the DUT. Recovery is manual and has broken joule before |

## How long this should take

| Step | What | Estimate |
|---|---|---|
| 0 | Housekeeping | about 0.5 day |
| 1 | Prepare on your own bench so the real run is fast | 1 to 2 days |
| 2 | Bring up the OAI + JURA rig and run the tests (the goal) | 1 to 2 days, depends on getting access to the rig |

---

## Step 0: housekeeping (do first, all yourself)

- [ ] Fix the DUT name in your docs: the real-run RU is the **Metanoia JURA** O-RU, driven by an
  **OAI gNB**. Not Kigen. Correct it everywhere (COVERAGE.md, scope docs, daily log).
- [ ] Read Rico's tools in the internship repo: `vendor-tools/` (the `cu-plane-dos-attacker` and
  `S-Plane-DoS-attacker` kits, with a README index). The seed captures are in
  `vendor-tools/cu-plane-dos-attacker/Traffic/`.
- [ ] Read the OAI + JURA integration guides so you know the bring-up before you touch the rig
  (links in the run-guide, Additional Resources).

## Step 1: prepare on your own bench so the real run is fast

Your bench is a **self-contained software setup** (network namespaces + veth on one host): isolated
enough to rehearse on safely, but software only, which is why it cannot give the real-radio result.
Use it to get everything ready so your time on the rig is short and you are not improvising.

- [ ] Have your scripts, seeds and run order ready to go: `tcpreplay` and the mutation steps, the
  `Traffic/` seeds, the 802.1X run, the S-plane steps.
- [ ] Finish the C-plane dissector work on the bench (run `fuzz-oran` against the `cp_*.pcap` seeds)
  so that part is already done before the rig.
- [ ] Write the exact command list you will run on the rig, in order, so you only have to follow it.

## Step 2: bring up the OAI + JURA rig and run the tests (this is the goal)

This is the hands-on part. You bring up the real DU and RU yourself and run the tests on real gear.

- [ ] Get access to the OAI gNB server (R750), the JURA O-RU and a 5G core. This rig is separate from
  the O-Cloud, so it is a lighter ask than a shared window: you need the rig, not a shutdown of joule
  and kepler.
- [ ] Bring up the rig yourself, hands-on, from the lab integration guides: the core, the JURA M-plane
  config and PTP sync, the OAI gNB, then confirm the fronthaul is up and PTP is locked (`tcpdump` the
  C/U/S traffic, watch it).
- [ ] Snapshot the config first (rollback point).
- [ ] Confirm nothing shared is on the segment before the severe tests.
- [ ] Run your tests on the real gear, in this order, by blast radius:
  - [ ] §11.1.2 802.1X against the real port.
  - [ ] §11.1.4 U-plane, rate ramped `0.01 -> 1 -> 10 -> 100` Mbps, stop at the first effect, one
    mutation class at a time, baseline and liveness between runs.
  - [ ] §11.1.5 S-plane PTP attacks **last**, on this rig's PTP only.
- [ ] Write up the hands-on run: what you did, what you saw, how it compared to the bench.

Use the SOP skeleton in [`sani-fronthaul-run-guide.md`](sani-fronthaul-run-guide.md) to record it.

---

## What the result will show (and why that is fine)

The real-radio run will probably **confirm** what you already found on the bench (your U-plane result
and your real-`ptp4l` S-plane findings). That is expected and fine. The point of this step is the
**hands-on experience of running it on real gear**, not a new verdict. So do the run, learn the real
fronthaul, and document it.

---

## Quick reference (OAI + JURA rig facts)

| Thing | Value |
|---|---|
| gNB | OAI monolithic BBU, FHI 7.2, on a Dell R750; LinuxPTP on the server (G.8275.1) |
| gNB fronthaul NIC | `ens7f0`, SR-IOV VFs, DU MAC `00:11:22:33:44:66`, VLAN 3, MTU 9000, DPDK |
| JURA O-RU | Metanoia, RU MAC `00:E0:0C:00:AE:06`, VLAN 3; band n78, 100 MHz, 273 PRB, SCS 30 kHz, 2T2R; M-plane NETCONF port 830 (out of scope) |
| PTP | own fronthaul PTP on the R750 to JURA, separate from the O-Cloud OcNOS grandmaster (domain 24) |
| Core | Open5GS / free5GC / OAI CN over N2/N3 |

Sources: the OAI + JURA integration guides and the JURA M-plane guide (see
`sani-fronthaul-run-guide.md`), Joy's lab threat model (private lab repo) §7, Joy's lab topology map
(private lab repo), Rico's tools under `vendor-tools/` in the internship repo.
