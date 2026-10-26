# Sani: run the Open Fronthaul tests for real, what to do next

For Rizky Wildansani (Sani). Written by Joy, 2026-10-22.

Your bench work is done and the method is proven (12/12 NOW cases, real O-RAN-FH parser, IKEv2, A1).

**The goal now is one thing: run your tests on the real build, with your own hands.** Not more
coverage, not a perfect product verdict. I want you to bring up the real DU and RU yourself and run
the tests on real gear, so you get the hands-on experience. M-plane is out of scope, do not spend
time on it.

The only catch is that the real gear is shared, so the real run happens in a **booked, isolated
window**, never on the live shared fronthaul. That is the whole plan: prepare on your own bench, then
do the real run safely.

## The two rules

1. **Never point a DoS or fuzz tool at the live shared fronthaul.** The OcNOS switch, the fronthaul
   VLANs and the PTP domain (24) are shared by joule, kepler and the DGX Spark, and the setup is
   fragile (PTP breakage has knocked joule over before). The real run happens only in an isolated
   window where nothing else is attached.
2. **Document everything as you go.** Every step, command, capture, result and blocker, in your daily
   log with the date and the evidence link. If it is not written down, it did not happen. The
   hands-on write-up is the point of this exercise.

## Effect and risk on the shared build (why rule 1 exists)

This is what each test would do **if you ran it on the live shared fronthaul**. The last two rows are
why the real run must be in an isolated window.

| Test | What it does on a real link | Risk on the shared build |
|---|---|---|
| §11.1.2 802.1X | plug in, try to auth on a fronthaul port | Low. Observational, unless you edit the switch 802.1X config |
| Read-only `tcpdump` | sniff and parse captured frames | Low. No injection |
| §11.1.4 **U-plane DoS / fuzz** (`cufh-attacker`, `tcpreplay`) | floods mutated eCPRI at a set Mbps onto the fronthaul VLAN | **High.** Saturates the link, can knock RU-A and joule's cell offline; malformed eCPRI can crash the Pegatron RU firmware (shared, needs a power-cycle); leakage to VLAN 103 hits kepler and the DGX Spark |
| §11.1.5 **S-plane PTP attacks** (rogue GM, drop, delay, flood) | injects onto the PTP domain | **Severe.** The grandmaster and domain 24 are shared, so a rogue GM, delay or flood desyncs timing for joule and kepler and anything slaved to OcNOS, not just your DUT. Recovery is manual and has broken joule before |

An isolated window removes all of this: your own DU + RU-A on their own VLAN and PTP domain, with
kepler and the DGX Spark off, so nothing else can be hit.

## How long this should take

| Step | What | Estimate |
|---|---|---|
| 0 | Housekeeping | about 0.5 day |
| 1 | Prepare on your own bench so the real run is fast | 1 to 2 days |
| 2 | The real run on joule + RU-A (the goal) | about 0.5 day prep for Ravi, then a half-to-one-day window |

The real run depends on when Ravi can give you a window; everything else you do yourself.

---

## Step 0: housekeeping (do first, all yourself)

- [ ] Fix the DUT name in your docs: the RU is **Pegatron** (RU-A on joule), **not Kigen**. Correct
  it everywhere (COVERAGE.md, scope docs, daily log).
- [ ] Read Rico's tools now in our repo: `vendor-tools/` (in the internship repo)
  (the `cu-plane-dos-attacker` and `S-Plane-DoS-attacker` kits, with a README index). The seed
  captures are in
  `vendor-tools/cu-plane-dos-attacker/Traffic/`.

## Step 1: prepare on your own bench so the real run is fast

Your bench is a **self-contained software setup** (network namespaces + veth on one host): isolated
enough to rehearse on safely, but software only, which is why it cannot give the real-radio result.
Use it to get everything ready so your lab window is short and you are not improvising on live gear.

- [ ] Have your scripts, seeds and run order ready to go: `tcpreplay` and the mutation steps, the
  `Traffic/` seeds, the 802.1X run, the S-plane steps.
- [ ] Finish the C-plane dissector work on the bench (run `fuzz-oran` against the `cp_*.pcap` seeds)
  so that part is already done before the window.
- [ ] Write the exact command list you will run in the lab, in order, so you only have to follow it.

## Step 2: the real run on joule + RU-A (this is the goal)

This is the hands-on part. You bring up the real DU and RU yourself and run the tests on real gear.

- [ ] Ask Ravi for **one isolated window on joule + RU-A** (RU-A is joule's own radio, VLAN 5), with
  kepler and the DGX Spark off, and PTP isolated for the window. Agree the PTP restart procedure
  first (recovery is manual).
- [ ] Bring up the real fronthaul yourself, hands-on: the E810 NIC and VF, the VLANs, PTP, the DU to
  RU connection. Confirm the link is up and PTP is locked (`tcpdump` the C/U/S traffic, watch it).
- [ ] Snapshot the switch and DU config first (rollback point).
- [ ] Run your tests on the real gear, in this order, by blast radius:
  - [ ] §11.1.2 802.1X against the real port.
  - [ ] §11.1.4 U-plane, rate ramped `0.01 → 1 → 10 → 100` Mbps, stop at the first effect, one
    mutation class at a time, baseline and liveness between runs.
  - [ ] §11.1.5 S-plane PTP attacks **last**, only on the isolated PTP domain.
- [ ] Write up the hands-on run: what you did, what you saw, how it compared to the bench.

---

## What the result will show (and why that is fine)

The real-radio run will probably **confirm** what you already found on the bench (your U-plane
dissector result and your real-`ptp4l` S-plane findings). That is expected and fine. The point of
this step is the **hands-on experience of running it on real gear**, not a new verdict. So do the
run, learn the real fronthaul, and document it. The only thing that can stop you is not getting a
window, and that is a scheduling question for Ravi, not a technical one.

---

## Quick reference (lab facts)

| Thing | Value |
|---|---|
| joule fronthaul | E810 `enp202s0f0` → switch `xe12`, VLAN 5 → RU-A (Pegatron) on `xe10`; DU MAC `00:11:22:33:44:55` |
| Switch | OcNOS: fronthaul VLANs **and** the PTP grandmaster (GPS, G.8275.1, clock class 6) |
| PTP | each worker runs `ptp-fh` (`ptp4l` slaveOnly, domain 24) + `phc-fh` (`phc2sys`); shared; fragile |
| Shared with | kepler and the DGX Spark use the same switch and PTP domain 24 |

Sources: Joy's lab threat model (private lab repo), Joy's lab topology map (private lab repo) (§8, S4 to S11),
Rico's tools under `vendor-tools/` in the internship repo.
