# Replication Log

## Date: 2025-10-03
- ### **Task:** Build ns-3 with examples & tests
- **Command Run:**  
  ```bash
  ./ns3 configure --enable-examples --enable-tests
  ./ns3 build
- **Status:** PASS / PARTIAL FAIL (Satellite contrib module did not compile).
- **Output:** <img width="1109" height="418" alt="image" src="https://github.com/user-attachments/assets/a9f6f562-14c1-4d40-91b3-fabe55f63c1e" />
**build succeeded** (core ns-3 works), only contrib satellite is broken (but fixable).  

- ### **Command Run:**
  ```bash
  ./test.py --no-build`
- **Status:** PASS
- **Output:** <img width="1084" height="510" alt="image" src="https://github.com/user-attachments/assets/363f80d2-a2df-454d-8b62-2d7f2e7122ad" />
The ns-3.43 installation on Ubuntu was successfully completed. The full validation suite was executed with 435 out of 435 tests passed (0 skipped, 0 failed, 0 crashed, 0 valgrind errors).

## Date: 2025-10-07
- ### **Task:** run a simple LTE/NR/Satellite example
- ### **Command Run:**  
  ```bash
  ./ns3 run "lena-simple-epc --simTime=5s --numNodePairs=4"
  ls
  head -n 20 DlRlcStats.txt
- **Status:** PASS
- **Output:** <img width="1103" height="552" alt="image" src="https://github.com/user-attachments/assets/40a96e29-6c8f-4bf5-b6ce-072515263232" />
The program runs succesfully and outputting data.

- ### **Command Run:**
  ```bash
  ./ns3 run sat-mobility-example
- **Status:** FAIL
<img width="1095" height="454" alt="image" src="https://github.com/user-attachments/assets/93d8c6ea-4360-4268-a5dd-b853bd1d89b0" />
  ```bash  
  error: ‘map’ in namespace ‘std’ does not name a template type
  note: ‘std::map’ is defined in header ‘<map>’; did you forget to ‘#include <map>’?

- **FIX:**
  ```bash
  ./ns3 run sat-mobility-example
  nano contrib/satellite/model/satellite-wave-form-conf.h
  #include <map>
  ./ns3 build
  
- **Status:** PASS <img width="1094" height="459" alt="image" src="https://github.com/user-attachments/assets/591d58d7-0547-4b3d-95c0-60a1ceb4f78b" />
building contrib/satellite completed. Now we test it with
  ```bash
  ./ns3 show | grep satellite
  propagation               satellite                 sixlowpan
  ./ns3 run contrib/satellite/examples/sat-onoff-example
  find . -type f -mtime -1 | grep -E "txt|csv|log" | head -n 20
  ./ns3 run contrib/satellite/examples/sat-tutorial-example
  ls -lh *.txt
<img width="960" height="819" alt="image" src="https://github.com/user-attachments/assets/79a90120-94a6-42f2-80e6-860432648538" />
<img width="1300" height="424" alt="image" src="https://github.com/user-attachments/assets/bdece198-eaa3-4f6a-8428-b411857ee0bd" />
The full troubleshooting notes available on 
