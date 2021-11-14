# Week 12

## Overview

This week you will learn about the transaction management in SQLite database engine.
Specifically, the following excercises are carried out 
- A. Performacne evaluation: system transaction vs. user transaction
- B. Understanding deadlock in SQLite
- C. Performacne evaluation: jounral mode

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

## A. Performacne evaluation: system transaction vs. user transaction

### 1. Download the database and test query files

- You can find the `database.db` and `test_query` folder in week-12 directory

```bash

# Clone the SWE3033-F2021 github repository
git clone https://github.com/meeeejin/SWE3033-F2021.git
cd week-12

# If you already cloned the reposity then 
cd SWE3033-F2021
git pull

```

### 2. Evaluate the performance using `time` command

```bash
# 1. Copy the database file
cp database.db test.db

# 2. Evaluate the performance for each query files
time sqlite3 test.db < ./test_query/system_trx.sql

real    0m10.947s
user    0m0.546s
sys     0m0.362s

# 3. Present the exeprimental results using `real` time 

```

### 3. Repeat Step 2 for following files in `test_query` directory

- system_trx.sql
- user_trx.sql


## B. Understanding deadlock in SQLite

### 1. Generate sample table  

```bash

vldb@NVDIMM:~/SWE3033/TEST$ sqlite3 deadlock.db
SQLite version 3.36.0 2021-06-18 18:36:39
Enter ".help" for usage hints.
sqlite> CREATE TABLE test (num int);
sqlite> INSERT INTO test VALUES(1);
sqlite> INSERT INTO test VALUES(2);
sqlite> .schema
CREATE TABLE test (num int);
sqlite> .exit

```

### 2. Open two terminals 

- Open two terminals (terminal A and B) and open the same database (`deadlock.db`) concurrently.
- Type the SQL command in following orders: 
> Note. "[termnia A]" means execute the SQL statement in the terminal A

```bash
step1. [terminal A] BEGIN; 
step2. [terminal B] BEGIN; 
step3. [terminal B] INSERT INTO TEST VALUES (3);
step4. [terminal A] SELECT * FROM TEST;
step5. [terminal A] SINSERT INTO TEST VALUES (4);
step6. See what happened in terminal A
step7. [terminal B] COMMIT;
step8. See what happened in terminal A 
```

### 3. Resolve the deadlock case 

- Write your own answer to resolve this deadlock issue and show the execution reuslts 

> Hint. Use ROLLBACK command


## C. Performance evaluation: jounral mode

### 1. Evaluate the performance using `time` command

```bash
# 1. Copy the database file
cp database.db test.db

# 2. Evaluate the performance for each query files
time sqlite3 test.db < ./test_query/sync10_truncate.sql

real    0m10.947s
user    0m0.546s
sys     0m0.362s

# 3. Present the exeprimental results using `real` time 
```

### 2. Repeat Step 1 for following files in `test_query` directory

- sync1_delete.sql
- sync10_delete.sql
- sync20_delete.sql
- sync1_persist.sql
- sync10_persist.sql
- sync20_persist.sql
- sync1_truncate.sql
- sync10_truncate.sql
- sync20_truncate.sql

> syncN_[journal_mode].sql : N means the number of SQL statements in transaction

## Report Submission

1. Do excercise A, B, and C following the instructions

2. Present experiment results (e.g., table or graph format) and write your own answer for following questions

  - For excercise A: The major reason of performance gap between `user transaction` and `system transaction`
  - For excercise B: Explain your solution to resovle the deadlock issuea and attach the screenshot of run results 
  - For excercise C: (1) Compare the performance between journal mode and number of SQL statements in each transactions
  (2) Analyze the performance using characteristics of each journal mode (Please refer to [SQLite jounal mode](https://www.sqlite.org/pragma.html#pragma_journal_mode)).

Organize the results and your answer into a single report and submit it. 
Follow the [submission guide](../report-submission-guide.md) for your report.


### References
- [SQLite jounal mode](https://www.sqlite.org/pragma.html#pragma_journal_mode)
