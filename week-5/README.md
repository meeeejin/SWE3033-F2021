# Week #5

## Overview

This week you will measure the space utilization of TPC-C tables on MySQL. By observing how the space utilization of each table changes before and after the benchmark, you can see the impact of MySQL's B+Tree on space usage.

Follow the guide below. If you have any questions, don't hesitate to contact me via email (Mijin An / meeeeejin@gmail.com)

> NOTE: This lab is based on the Linux environment. If you don't have a Linux machine, use [VirturalBox](https://www.virtualbox.org/). (Recommend Ubuntu 18.04)

## Instructions

### 1. Install `innodb_ruby`

```bash
$ sudo apt-get install gem
$ sudo apt-get install rubygems ruby-dev
```

```bash
$ sudo gem install rake
$ sudo gem install innodb_ruby -v 0.9.16
```

### 2. Clone the lab repository

```bash
$ git clone https://github.com/meeeejin/SWE3033-F2021
$ cd week-5/bin
```

There are two binary files: `cal-free-space-from-ibd` and `cal-free-space-percentile-from-parsed-file`. 

1. `cal-free-space-from-ibd`: calculate the free space of each page from the *.ibd* file
2. `cal-free-space-percentile-from-parsed-file`: calculate the space percentile from the parsed file.

Use the appropriate file according to the database page size you use. For example, if you use 4K page size, use `cal-free-space-from-ibd-4k` and `cal-free-space-percentile-from-parsed-file-4k`.

### 3. Check the space utilization before the TPC-C benchmark

```bash
$ ./cal-free-space-from-ibd-16k tpcc/item.ibd result.txt
$ ./cal-free-space-percentile-from-parsed-file-16k space-summary.txt result.txt
```

- `cal-free-space-from-ibd [target ibd file] [output file]`
  - `target ibd file`: *.ibd* file is the data file extension of MySQL (e.g., tpcc/history.ibd = History table of TPC-C database)
  - After running the binary file, you can get the overall space utilization from the `output file`:
  ```bash
  # Parsing Result
  Total number of pages = 5117
  Total free space = 11653729
  Average free space per page = 2277 (55.60%)

  - Leaf Pages
  Total number of pages = 5106
  Total free space = 11645697
  Average free space per page = 2280 (55.68%)
  Min free space = 259
  Max free space = 4096

  - Non-leaf Pages
  Total number of pages = 11
  Total free space = 8032
  Average free space per page = 730 (17.83%)
  Min free space = 8
  Max free space = 3822
  ```
  - You can also get another output file `space-summary.txt` which shows the free space of each page in `target ibd file`:
  ```bash
  page        index   level   data    free    records
  3           46      2       935     3015    55
  4           51      2       882     3066    42
  ...
  ```
- `cal-free-space-percentile-from-parsed-file [parsed file] [output file]`
  - After running the binary file, you can get the percentile information from the `output file`:
  ```bash
  Percentage      Count
  0 - 10          5934    (34.1%)
  10 - 20         447     (2.6%)
  20 - 30         427     (2.5%)
  30 - 40         497     (2.9%)
  40 - 50         1280    (7.4%)
  50 - 60         705     (4.1%)
  60 - 70         4842    (27.8%)
  70 - 80         1264    (7.3%)
  80 - 90         1459    (8.4%)
  90 - 100        544     (3%)
  Total           17399   (100%)
  ```

### 4. Start a MySQL server

```bash
$ ./bin/mysqld_safe --defaults-file=/path/to/my.cnf
```

### 5. Run the benchmark

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

### 6. After the benchmark, check the space utilization of the TPC-C tables

```bash
$ ./cal-free-space-from-ibd-16k tpcc/item.ibd result.txt
$ ./cal-free-space-percentile-from-parsed-file-16k space-summary.txt result.txt
```

Check the space utilization of the pages of each table. And compare the percentile distribution at the end of the benchmark to the start of the benchmark. ([example](https://gist.github.com/meeeejin/78de52ef0bc60833d0bf70102b21a367))

### 7. Shut down the MySQL server
```bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
```

## Report Submission

1. Before the benchmark, check the space utilization (of TPC-C 9 tables)
2. Run the TPC-C benchmark on MySQL
3. After the benchmark, check the space utilization (of TPC-C 9 tables)
4. Compare the space utilization at the end of the benchmark to the start of the benchmark
5. Present the experimental results and analyze them

Organize the results into a single report and submit it. Follow the [submission guide](../report-submission-guide.md) for your report.

## Reference
- [Build and install the source code (5.7)](https://github.com/meeeejin/til/blob/master/mysql/build-and-install-the-source-code-5.7.md)
- [Percona-Lab/tpcc-mysql](https://github.com/Percona-Lab/tpcc-mysql)
- [tpcc-mysql: Simple usage steps and how to build graphs with gnuplot](https://www.percona.com/blog/2013/07/01/tpcc-mysql-simple-usage-steps-and-how-to-build-graphs-with-gnuplot/)
