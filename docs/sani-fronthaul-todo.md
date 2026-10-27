# Sani: run the Open Fronthaul tests for real, what to do next

For Rizky Wildansani (Sani). Written by Joy, 2026-10-22, updated 2026-10-27 (JURA rig, shared FHG).

Your bench work is done and the method is proven (12/12 NOW cases, real O-RAN-FH parser, IKEv2, A1).

**The goal now is one thing: run your tests on a real build, with your own hands.** Not more
coverage, not a perfect product verdict. I want you to bring up a real DU and RU yourself and run the
tests on real gear, so you get the hands-on experience. M-plane is out of scope, do not spend time on
it.

**What the JURA RU gives you: compute independence.** Prof Ray pointed to the Metanoia JURA O-RU. The
lab has a documented OAI gNB to JURA rig where the gNB runs on a standalone server (a Dell R750), so
you can bring up the gNB and drive a real O-RU **without needing the joule worker node or OCUDU**. You
bring it up yourself. That is the point of using the JURA.

**What it does not give you: an isolated fronthaul.** The gNB and the JURA both plug into the lab's
**shared fronthaul switch** (a Ufispace OcNOS switch at `192.168.8.25`), whose PTP grandmaster
(G.8275.1, domain 24, GPS) also serves joule, kepler, the DGX Spark and the Pegatron RUs. So the
switch and the timing are shared with the O-Cloud. The severe tests still need a **booked isolated
window**, same as on joule. 802.1X and read-only capture are fine any time.

## The two rules

1. **The severe tests run only in a booked isolated window, never on the live shared fronthaul.** The
   U-plane DoS and the S-plane PTP attacks hit the shared switch and the shared PTP grandmaster
   (domain 24), so they desync timing for joule and kepler, not just your DUT, and PTP breakage has
   knocked joule over before. 802.1X and read-only capture (the FHG has a mirror port) are fine any
   time.
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

Because the fronthaul switch and the PTP grandmaster are shared, the last two tests hit joule and
kepler too, not just your DUT. That is why they need an isolated window.

| Test | What it does | Risk on the shared build |
|---|---|---|
| §11.1.2 802.1X | plug into an FHG port, try to auth | Low. Observational |
| Read-only `tcpdump` / FHG mirror port | capture frames | Low. No injection |
| §11.1.4 **U-plane DoS / fuzz** | floods mutated eCPRI on the fronthaul VLAN | **High.** Loads the shared switch, can knock the JURA and the gNB cell offline, can leak to other VLANs and devices on the switch |
| §11.1.5 **S-plane PTP attacks** | injects onto the shared PTP (domain 24) | **Severe.** Desyncs timing for joule and kepler, not just the DUT. Recovery is manual and has broken joule before |

## How long this should take

| Step | What | Estimate |
|---|---|---|
| 0 | Housekeeping | about 0.5 day |
| 1 | Prepare on your own bench so the real run is fast | 1 to 2 days |
| 2 | Bring up the OAI + JURA rig and run the tests | 1 to 2 days self-service, plus a booked window for the severe tests |

---

## Step 0: housekeeping (do first, all yourself)

- [ ] Fix the DUT name in your docs: the real-run RU is the **Metanoia JURA** O-RU, driven by an
  **OAI gNB**. Not Kigen. Correct it everywhere (COVERAGE.md, scope docs, daily log).
- [ ] Read Rico's tools in the internship repo: `vendor-tools/`
  (the `cu-plane-dos-attacker` and `S-Plane-DoS-attacker` kits, with a README index). The seed
  captures are in
  `vendor-tools/cu-plane-dos-attacker/Traffic/`.
- [ ] Read the OAI + JURA integration guides and the FHG config so you know the bring-up before you
  touch the rig (links in the run-guide, Additional Resources).

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

- [ ] Get access to the gNB server (R750), the JURA O-RU, a 5G core and a port on the shared FHG.
  Because the switch and PTP are shared, agree the plan with Ravi first (which port and VLAN, and a
  booked window for the severe tests).
- [ ] Bring up the rig yourself, hands-on, from the lab integration guides and the FHG config: the
  core, the JURA M-plane config and PTP sync (to the shared domain-24 grandmaster), the OAI gNB, then
  confirm the fronthaul is up and PTP is locked (`tcpdump` the C/U/S traffic from the FHG mirror port).
- [ ] 802.1X (§11.1.2) and read-only capture: run any time.
- [ ] Snapshot the config first (rollback point).
- [ ] Severe tests (§11.1.4 U-plane, §11.1.5 S-plane) **only in the booked window**, with joule,
  kepler and the DGX Spark idle (or the JURA ports on a separate PTP domain, if Ravi sets that up).
  Agree the PTP restart procedure first.
  - [ ] §11.1.4 U-plane, rate ramped `0.01 -> 1 -> 10 -> 100` Mbps, stop at the first effect, one
    mutation class at a time, baseline and liveness between runs.
  - [ ] §11.1.5 S-plane PTP attacks **last**.
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
| gNB | OAI monolithic BBU, FHI 7.2, on a standalone Dell R750 (no joule, no OCUDU needed); server runs `ptp4l` as a PTP slave |
| gNB fronthaul NIC | `ens7f0`, SR-IOV VFs, DU MAC `00:11:22:33:44:66`, MTU 9000, DPDK |
| JURA O-RU | Metanoia, RU MAC `00:E0:0C:00:AE:06`; band n78, 100 MHz, 273 PRB, SCS 30 kHz, 2T2R; M-plane NETCONF port 830 (out of scope) |
| Shared FHG | Ufispace OcNOS switch at `192.168.8.25`; PTP grandmaster G.8275.1 domain 24 (GPS); also serves joule, kepler, DGX Spark, Pegatron RUs; has a mirror port for capture |
| Core | Open5GS / free5GC / OAI CN over N2/N3 |

Sources: the OAI + JURA integration guides, the FHG config and the JURA M-plane guide (links in
[`sani-fronthaul-run-guide.md`](sani-fronthaul-run-guide.md)), Joy's lab threat model (private lab repo)
§7, Joy's lab topology map (private lab repo) (§1 and §11),
Rico's tools under `vendor-tools/` in the internship repo.
