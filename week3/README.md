# Week #2

## Overview

This week, you will learn to monitor the system performance while running the TPC-C benchmark on MySQL. You will also learn what those performance metrics mean.

Follow the guide below. If you have any questions, don't hesitate to contact me via email (Mijin An / meeeeejin@gmail.com)

> NOTE: This lab is based on the Linux environment. If you don't have a Linux machine, use [VirturalBox](https://www.virtualbox.org/). (Recommend Ubuntu 18.04)

## Instructions

### 1. Start a MySQL server

1. Before starting a MySQL server, update the buffer pool size to 10% of your TPC-C database size. For example, if you load 20 warehouses (e.g., about 2G database size), change the value of `innodb_buffer_pool_size` in *my.cnf* to 200M:

```bash
$ vi /path/to/my.cnf
...
innodb_buffer_pool_size=200M
...
```

2. Start a MySQL server:

```bash
$ ./bin/mysqld_safe --defaults-file=/path/to/my.cnf
```

### 2. Run the TPC-C benchmark

Run the benchmark by modifying the experimental parameters to match your system specifications. For example:

```bash
$ ./tpcc_start -h 127.0.0.1 -S /tmp/mysql.sock -d tpcc -u root -p "yourPassword" -w 20 -c 8 -r 10 -l 1200 | tee tpcc-result.txt
```

It means:

- Host: 127.0.0.1
- MySQL Socket: /tmp/mysql.sock
- DB: tpcc
- User: root
- Password: yourPassword
- Warehouse: 20
- Connection: 8
- Rampup time: 10 (sec)
- Measure: 1200 (sec)

### 3. Monitor the MySQL and overall system

1. *While running the benchmark*, collect performance metrics (e.g., I/O status and transaction throughput) and record them in a separate file for future analysis. Refer to the [performance monitoring guide](reference/performance-monitoring-guide.md) to gather performance metrics while running the TPC-C benchmark on MySQL 5.7

2. After the benchmark ends, take a screenshot of the TPC-C output and add it to your report

3. Shut down the MySQL server:

```bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
```

## Report Submission

1. Run the TPC-C benchmark on MySQL
2. Observe how the performance metrics and TpmC change over time
3. Present the experimental results
4. Analyze the results

Organize the results into a single report and submit it. Follow the [submission guide](../report-submission-guide.md) for your report.

## Reference
- [Build and install the source code (5.7)](https://github.com/meeeejin/til/blob/master/mysql/build-and-install-the-source-code-5.7.md)
- [Percona-Lab/tpcc-mysql](https://github.com/Percona-Lab/tpcc-mysql)
- [tpcc-mysql: Simple usage steps and how to build graphs with gnuplot](https://www.percona.com/blog/2013/07/01/tpcc-mysql-simple-usage-steps-and-how-to-build-graphs-with-gnuplot/)
