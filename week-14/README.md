# Week 14

## Overview

This week you will learn how to evaluate performance between two journal mode (RBJ and WAL)
on SQLite database engine using TPC-C benchmark (`pytpcc`).

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

## `pytpcc` benchmark

### How to { build | use } 
- refer to [week13](/week-13/README.md)

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
python tpcc.py --warehouse=10 --config=./sqlite.config --no-load --duration=1800 --journal=wal sqlite

```

- For each runs, prepare the same database file (use `backup.db` database file)
- To minimize the impact of the performance interference, flsuh all caches in the system using `vm.drop_caches=3` command.
- Run TPC-C benchmark for 1800 sec


### 3. Change the journal mode and repeat step 2

- Change the `journal` mode of the SQLite database engine 
- Compare two different journal: `delete` and `wal` modes
  - Configure three different values for `--journal` 

## Report Submission

1. Run TPC-C benchmark for two journal modes
  - Change journal mode to **delete** and **wal**

2. Observe how TPS (txn/s) changes 
  - Record and analyze the TPS for each transaction (DELIVERY, NEW_ORDER, ORDER_STATUS, PAYMENT, STOCK_LEVEL)

3. Present experimental results

4. Analyze the results
  - hint. The root cause of the performance gap between `delete` mode and `wal` is xxx.

Organize the results and your answer into a single report and submit it. 
Follow the [submission guide](../report-submission-guide.md) for your report.


### References
- [Python TPC-C](https://github.com/apavlo/py-tpcc)
- [Journal mode](https://www.sqlite.org/pragma.html#pragma_journal_mode)