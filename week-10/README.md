# Week 10

## Overview

This week you will write a report on **SQLite database engine** (see the deatil in Instruction 0)
and learn how to run the  Androbench benchmark on SQLite database engine.

Follow the guide below. If you have any questions, Please feel free to contact me via email (Jonghyeok Park / akindo19@skku.edu)

> NOTE: This lab is based on the Linux environment. If you don't have a Linux machine, use [VirturalBox](https://www.virtualbox.org/). (Recommend Ubuntu 18.04)

## Instructions

### Mount devices (Optional)

- If you have more than one storage device on your PC, read and try [this guide](../week-1/reference/mount-guide.md) to separate data device

### 0. Write Report

Write a **single page** report for SQLite database engine and the proposal must includes:

1. Compare with the SQLite and client-server RDBMS (e.g., MySQL, RocksDB, and Oracle)
  - Advantages of the SQLite 
  - Disdvantages of the SQLite (i.e., Situations where a client-server RDBMS may work better)

2. How SQLite supports cross-platfrom database
  - hint: VFS layer

3. Compare the SQLite and Filesystem


### 1. Build SQLite

```bash
# 1. Download SQLite database source code (3.36.0 version)
wget https://www.sqlite.org/2021/sqlite-src-3360000.zip

# 2. Unzip the source code file
unzip sqlite-src-3360000.zip

# 3. Create build directory
mkdir -p ./sqlite-src-3360000/build && cd ./sqlite-src-3360000/build

# 4. Configure 
../configure

# 5. Build using GCC compiler
make -j # of physical core
```

### 2. Record and analyze the SQLite trace file 
- Configure the SQLite environent setup
- Evaluate execution time and record `real` time

```bash
vldb@NVDIMM:~/SWE3033/sqlite-src-3360000/build$ time ./sqlite3 /home/vldb/ssd/androbench.db < androbench.sql &> /dev/null

real    0m10.947s
user    0m0.546s
sys     0m0.362s
```

### 3. Change the SQLite environment and repeat step 2

- Change the SQLite environment setup using `PRAGMA` command in `androbench.sql` file  
- Change the journal mode : `off`, `delete`, and `wal` mode 
```
/* SQLite environment setup 
* (jhpark): environment setup using PRAGMA command
*/
PRAGMA page_size;
PRAGMA page_size=4096;
PRAGMA foreign_keys;

/* Change journal mode here */
PRAGMA journal_mode=off;
/* Change cache size here */
PRAGMA cache_size=2000;
/* Change synchoronus mode */
PRAGMA synchronous=1;
/* Change locking mode */
PRAGMA locking_mode=NORMAL;

PRAGMA journal_mode;
PRAGMA synchronous;
/******************************/
```

- Change the page_cache size : 100, 500, 1000, 2000
```bash
PRAGMA cache_size=100;
```
- Change the page_size : 512, 1024, 2048, 4096, 8192, 16384
```bash
PRAGMA page_size=512;
```
- Chagne locking mode : `NORMAL`, `EXCLUSIVE`
```bash
PRAGMA page_size=512;
```
- Change the synchronous mode : 0, 1, 2 
```bash
PRAGMA synchronous=0;
```

## Report Submission

1. Run Androbench by changing the SQLite environment 
  - When you evaluate the performance against configuration factor, use **default value** for other configuration factors
    - `journal_mode` : del
    - `page_cache` : 2000
    - `page_size` : 4096
    - `locking_mode` : NORMAL
    - `synchronous` : 1

2. Observe how performance (execution time) changes 
  - Change `journal_mode`
  - Change `page_cache` size
  - Change `page_size` size
  - Change `locking_mode`
  - Chagne `synchronous` mode

3. Present the experimental results (talble or graph format)

4. Analyze the results
  - Explain each configuration factor.
  - Find the best value for each configuration factor with your own reason.

Organize the results and proposal into a single report and submit it. Follow the [submission guide](../report-submission-guide.md) for your report.


## Reference
- [SQLite Homepage](https://www.sqlite.org/index.html) 
- [SQLite Download](https://www.sqlite.org/download.html)
- [SQLite PRAGMA](https://www.sqlite.org/pragma.html)