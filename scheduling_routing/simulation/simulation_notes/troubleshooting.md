# ns-3.43 Satellite Module Troubleshooting Documentation

## 1. Initial Setup

* **Environment:** Ubuntu (WSL)
* **ns-3 Version:** 3.43
* **Goal:** Build and test `satellite`, `traffic`, and `magister-stats` contrib modules.

---

## 2. Initial Problem

Running the default satellite example:

```bash
./ns3 run sat-mobility-example
```

Resulted in the following fatal error:

```
msg="SatEnvVariables::LocateDirectory - Directory not found within 10 levels: contrib/satellite/data/sims"
NS_FATAL, terminating
```

### Cause

The directory `contrib/satellite/data` was missing. The satellite module depends on a separate dataset (`sns3-data`) that includes scenario definitions, trace files, and XML configurations.

---

## 3. Attempts and Failures

* Added missing header manually: `<map>` to fix compilation errors in `satellite-wave-form-conf.h`.
* Rebuilt the project multiple times with `./ns3 build`.
* Attempted to clone `signetlabdei/ns3-satellite.git`, but it was unavailable:

  ```bash
  Repository not found.
  ```
* Running examples like `sat-cbr-example` or `sat-onoff-example` produced no output files.
* Tried using `git submodule update` earlier, but no submodule was registered at that time.

---

## 4. Full Recovery Procedure

### ✅ Step 1: Remove Broken Satellite Module

```bash
rm -rf contrib/satellite
```

### ✅ Step 2: Clone Working Version from SNS3

```bash
git clone https://github.com/sns3/sns3-satellite.git contrib/satellite
```

### ✅ Step 3: Initialize Data Submodule

```bash
cd contrib/satellite
git submodule update --init --recursive
```

**Output:**

```
Submodule 'data' (https://github.com/sns3/sns3-data.git) registered
Submodule path 'data': checked out '4200675f...'
```

**Size Verification:**

```bash
du -sh contrib/satellite/data
# Output: 3.6G
```

### ✅ Step 4: Rebuild ns-3

```bash
cd ~/ns-3.43
./ns3 build
```

### ✅ Step 5: Verify Module Availability

```bash
./ns3 show | grep satellite
```

**Expected Output:**

```
propagation  satellite  sixlowpan
```

### ✅ Step 6: Run Working Example

```bash
./ns3 run contrib/satellite/examples/sat-tutorial-example
```

### ✅ Step 7: Check Output Files

```bash
ls -lh *.txt
```

**Generated Files:**

```
DlMacStats.txt
DlRlcStats.txt
UlSinrStats.txt
DlRsrpSinrStats.txt
UlPdcpStats.txt
UlRlcStats.txt
...
```

---

## 5. Verification

Listing new simulation results confirmed successful output:

```bash
ls -lh *.txt
```

✅ Valid RLC, PDCP, PHY, and SINR stats confirmed.

---

## 6. Key Lessons Learned

* The **sns3-satellite** repository depends on **sns3-data** submodule (~3.6 GB).
* Missing `data/sims` directory causes fatal termination at runtime.
* **signetlabdei/ns3-satellite** is deprecated — use **sns3/sns3-satellite** instead.
* Always rebuild ns-3 after modifying any contrib module.
* To verify modules and tests:

  ```bash
  ./test.py
  ./ns3 show
  ./ns3 run <example>
  ```

---

## 7. Summary

This troubleshooting process successfully restored ns-3.43 with a working satellite module. All dependent data files were properly linked, examples executed without fatal errors, and output statistics were generated as expected.

