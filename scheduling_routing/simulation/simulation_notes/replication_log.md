# Replication Log

## Date: 2025-10-03
- **Task:** Build ns-3 with examples & tests
- **Command Run:**  
  ```bash
  ./ns3 configure --enable-examples --enable-tests
  ./ns3 build
- **Status:** PASS / PARTIAL FAIL (Satellite contrib module did not compile).
- **Output:** <img width="1109" height="418" alt="image" src="https://github.com/user-attachments/assets/a9f6f562-14c1-4d40-91b3-fabe55f63c1e" />
**build succeeded** (core ns-3 works), only contrib satellite is broken (but fixable).  

- **Command Run:**
  ```bash
  ./test.py --no-build`
- **Status:** PASS
- **Output:** <img width="1090" height="532" alt="image" src="https://github.com/user-attachments/assets/e5cee3c6-dd47-4942-afeb-363696c03f8f" />
currently stuck at **[37/435]** in the build/test suite.
