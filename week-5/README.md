# Week #5

## Overview

This week you will measure the space utilization of TPC-C tables on MySQL. By observing how the space utilization of each table changes before and after the benchmark, you can see the impact of MySQL's B+Tree on space usage.

Follow the guide below. If you have any questions, don't hesitate to contact me via email (Mijin An / meeeeejin@gmail.com)

> NOTE: This lab is based on the Linux environment. If you don't have a Linux machine, use [VirturalBox](https://www.virtualbox.org/). (Recommend Ubuntu 18.04)

## Instructions

### 1. Install `innodb_ruby`

```bash
$ sudo apt-get install gem
$ sudo gem install rake
$ sudo gem install innodb_ruby -v 0.9.16
```

```bash
$ ./cal-free-space-from-ibd-16k ~/data_backup2/data/tpcc500/item.ibd result.txt
$ ./cal-free-space-percentile-from-parsed-file-16k space-summary.txt result.txt
```

### 1. Initialize the MySQL data directory with the target page size

1. Remove all old data from MySQL data directory:

```bash
$ rm -rf /path/to/datadir/*
```

2. Use `mysqld --initialize` to initialize the data directory:
- `--datadir` : the path to the MySQL data directory
- `--basedir` : the path to the MySQL installation directory

If you want to change the page size to 4K (default: 16K), add the `innodb_page_size` parameter. For example:

```bash
$ ./bin/mysqld --initialize --innodb_page_size=4k --user=mysql --datadir=/path/to/datadir --basedir=/path/to/basedir
```

3. Reset the root password:

```bash
$ ./bin/mysqld_safe --skip-grant-tables --innodb_page_size=4k --datadir=/path/to/datadir

$ ./bin/mysql -uroot

root:(none)> use mysql;

root:mysql> update user set authentication_string=password('yourPassword') where user='root';
root:mysql> flush privileges;
root:mysql> quit;

$ ./bin/mysql -uroot -p

root:mysql> set password = password('yourPassword');
root:mysql> quit;
```

### 2. Start a MySQL server

1. Before starting a MySQL server, update the page size to the target page size. For example, if you want to use 4KB pages, change the value of `innodb_page_size` in *my.cnf* to 4KB:

```bash
$ vi /path/to/my.cnf
...
innodb_page_size=4KB
...
```

2. Start a MySQL server:

```bash
$ ./bin/mysqld_safe --defaults-file=/path/to/my.cnf
```

### 3. Load the TPC-C data and Run the benchmark

1. You need to create a database with the changed page size. Create a database for the TPC-C test. Go to the MySQL base directory and run the following commands:

```bash
$ ./bin/mysql -u root -p -e "CREATE DATABASE tpcc;"
$ ./bin/mysql -u root -p tpcc < /path/to/tpcc-mysql/create_table.sql
$ ./bin/mysql -u root -p tpcc < /path/to/tpcc-mysql/add_fkey_idx.sql
```

2. Then go back to the tpcc-mysql directory and load data:

> NOTE: Before executing the below commands, change the warehouse value (`-w 20`) to the proper value considering your system's free space. One warehouse occupies about 100MB.

```bash
$ cd tpcc-mysql
$ ./tpcc_load -h 127.0.0.1 -d tpcc -u root -p "yourPassword" -w 20
*************************************
*** TPCC-mysql Data Loader        ***
*************************************
option h with value '127.0.0.1'
option d with value 'tpcc'
option u with value 'root'
option p with value 'yourPassword'
option w with value '20'
<Parameters>
     [server]: 127.0.0.1
     [port]: 3306
     [DBname]: tpcc
       [user]: root
       [pass]: 1234
  [warehouse]: 20
TPCC Data Load Started...
Loading Item
.................................................. 5000
.................................................. 10000
.................................................. 15000
...
...DATA LOADING COMPLETED SUCCESSFULLY.
```

In this case, database size is about 2GB (= 20 warehouses).

3. After loading, run the benchmark by modifying the experimental parameters to match your system specifications. For example:

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

### 4. Monitor the buffer hit/miss ratio of MySQL

1. *While running the benchmark*, collect performance metrics (e.g., I/O status, transaction throughput, hit/miss ratio) and record them in a separate file for future analysis. Refer to the [performance monitoring guide](../week-2/reference/performance-monitoring-guide.md) and [hit ratio monitoring guide](../week-3/reference/hit-ratio-monitoring-guide.md)

2. After the benchmark ends, shut down the MySQL server:

```bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
```

### 5. Change the page size and repeat steps 1-4

## Report Submission

1. Load the TPC-C data with different page sizes and run the benchmark on MySQL
    - 4KB, 8KB, 16KB
    - Set the buffer size as you want (within 10% to 50% of database size)
2. Observe how the performance metrics (e.g., IOPS, hit ratio, etc.) and TpmC change over time
3. Present the experimental results
4. Analyze the results

Organize the results into a single report and submit it. Follow the [submission guide](../report-submission-guide.md) for your report.

## Reference
- [Build and install the source code (5.7)](https://github.com/meeeejin/til/blob/master/mysql/build-and-install-the-source-code-5.7.md)
- [Percona-Lab/tpcc-mysql](https://github.com/Percona-Lab/tpcc-mysql)
- [tpcc-mysql: Simple usage steps and how to build graphs with gnuplot](https://www.percona.com/blog/2013/07/01/tpcc-mysql-simple-usage-steps-and-how-to-build-graphs-with-gnuplot/)
