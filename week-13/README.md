# Week 13

## Overview

This week you will learn how to run TPC-C benchmark (`pytpcc`) on SQLite database engine. 

Follow the guide below. If you have any questions, Please feel free to contact me via email (Jonghyeok Park / akindo19@skku.edu)

> NOTE: This lab is based on the Linux environment. If you don't have a Linux machine, use [VirturalBox](https://www.virtualbox.org/). (Recommend Ubuntu 18.04)

## Prerequisite
 
### [mandatory] Install SQLite Library
- Skip this process if you already installed SQLite library

```bash
# go to the SQLite build directory 

cd {PATH}/sqlite-src-3360000/build
make -j
sudo make install -j 

# you can use SQLite database in any directroy
cd ~
sqltie --version

```

### Install python

```bash
sudo apt-get install python
```

## Build `pytpcc` benchmark

### 1. Setup `pytpcc` benchmark

```bash

# 1. Clone SWE3033-F2021 github repository
git clone https://github.com/meeeejin/SWE3033-F2021.git
cd week-13

# 2. Unzip pytpcc
unzip pytpcc

# 2. Prepare the SQLite configuration file
cd pytpcc
python tpcc.py --print-config sqlite &> sqlite.config

# 3. Open sqlite.config file and modify the database path (./tpcc.db)
vim sqltie.config
```

```
# SqliteDriver Configuration File
# Created 2021-11-21 02:46:44.486741
[sqlite]

# The path to the SQLite database
database             = ./tpcc.db
```

### 2. Load TPC-C database 

```bash
python tpcc.py --warehouse=10 --config=./sqlite.config --no-execute sqlite
```
- `warehouse`: The number of warehouse 
- `config` : Configuration file path
- `no-execute` : Loading only version (no execute)



### 3. Run TPC-C benchmark

```bash
python tpcc.py --warehouse=10 --config=./sqlite.config --no-load --duration=300 --buffer=1024 sqlite
```

- `warehouse`: The number of warehouse 
- `config` : Configuration file path
- `no-load` : Running only version (no loading phase)
- `duration` : Total execution time (N seconds)
- `buffer` : Page cache size (N pages in the buffer)


```bash
vldb@NVDIMM:~/SWE3033/py-tpcc/pytpcc$ python tpcc.py --warehouse=10 --config=./sqlite.config --no-load --duration=300 sqlite

11-21-2021 03:17:44 [<module>:234] INFO : Initializing TPC-C benchmark using SqliteDriver
11-21-2021 03:17:44 [execute:056] INFO : Executing benchmark for 300 seconds
==================================================================
Execution Results after 300 seconds
------------------------------------------------------------------
                  Executed        Time (µs)       Rate
  DELIVERY        86              21259987.8311   4.05 txn/s
  NEW_ORDER       1043            161138629.436   6.47 txn/s
  ORDER_STATUS    76              18683.1951141   4067.83 txn/s
  PAYMENT         982             117047047.377   8.39 txn/s
  STOCK_LEVEL     95              210156.917572   452.04 txn/s
------------------------------------------------------------------
  TOTAL           2282            299674504.757   7.61 txn/s
```

## Instructions

### 1. Loading database (warehouse 10)

```bash

# loading
python tpcc.py --warehouse=10 --config=./sqlite.config --no-execute sqlite

# change database file name 
cp tpcc.db backup.db

```

- Prepare the database using loading command 
- Change database file name as `backup.db`


### 2. Run TPC-C Benchmark 

```bash

# prepare the database file
cp backup.db tpcc.db

# flush all cache
sudo sysctl vm.drop_caches=3

# run
python tpcc.py --warehouse=10 --config=./sqlite.config --no-load --duration=1800 --buffer=100 sqlite

```

- For each runs, prepare the same database file (use `backup.db` database file)
- To minimize the impact of the performance interference, flsuh all caches in the system using `vm.drop_caches=3` command.
- Run TPC-C benchmark for 1800 sec


### 3. Change page cache size and repeat step 2

- Resize the `cache_size` of the SQLite database engine 
- Compare **three** different page_cache sizes (50, 100, 150, 200)
  - Configure three different values for `--buffer` 

## Report Submission

1. Run TPC-C benchmark by varying the cache size 
  - Change cache size to **50** , **100**, **150**, **200** 

2. Observe how TPS (txn/s) changes 
  - Record and analyze the TPS for each transaction (DELIVERY, NEW_ORDER, ORDER_STATUS, PAYMENT, STOCK_LEVEL)

3. Present experimental results

4. Analyze the results

Organize the results and your answer into a single report and submit it. 
Follow the [submission guide](../report-submission-guide.md) for your report.


### References
- [Python TPC-C](https://github.com/apavlo/py-tpcc)
- [Page cache size](https://www.sqlite.org/pragma.html#pragma_cache_size)