# Week #3

## Overview

In this week, you will learn how to measure the hit/miss ratio in MySQL while running the TPC-C benchmark. Follow the guide below. If you have any questions, don't hesitate to get in touch with me via email (Mijin An / meeeeejin@gmail.com)

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

### 3. Monitor the hit/miss ratio of MySQL

1. Log in to the MySQL server:

```bash
$ ./bin/mysql -uroot -pyourPassword
```

2. Monitor the buffer pool hit ratio using `show engine innodb status`:

```bash
mysql> show engine innodb status
...
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 274857984
Dictionary memory allocated 152634
Buffer pool size   16384
Free buffers       68
Database pages     15636
Old database pages 5751
Modified db pages  8056
Pending reads      6
Pending writes: LRU 121, flush list 0, single page 0
Pages made young 108465, not young 1341192
0.00 youngs/s, 0.00 non-youngs/s
Pages read 418624, created 5399, written 193750
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
Buffer pool hit rate 969 / 1000, young-making rate 8 / 1000 not 107 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 15636, unzip_LRU len: 0
I/O sum[215312]:cur[1985], unzip sum[0]:cur[0]
...
```

`Buffer pool hit rate` indicates the hit ratio. You can calculate the miss ratio by `1 - Buffer pool hit rate`. For example, `Buffer pool hit rate 969 / 1000` corresponds to 96.9% hit rate.

Or you can use `show status like '%innodb_buffer_pool%'` to monitor the hit/miss ratio:

```bash
mysql> show status like '%innodb_buffer_pool%';
+---------------------------------------+--------------------------------------------------+
| Variable_name                         | Value                                            |
+---------------------------------------+--------------------------------------------------+
| Innodb_buffer_pool_dump_status        | Dumping of buffer pool not started               |
| Innodb_buffer_pool_load_status        | Buffer pool(s) load completed at 210903 17:40:25 |
| Innodb_buffer_pool_resize_status      |                                                  |
| Innodb_buffer_pool_pages_data         | 15631                                            |
| Innodb_buffer_pool_bytes_data         | 256098304                                        |
| Innodb_buffer_pool_pages_dirty        | 7975                                             |
| Innodb_buffer_pool_bytes_dirty        | 130662400                                        |
| Innodb_buffer_pool_pages_flushed      | 414867                                           |
| Innodb_buffer_pool_pages_free         | 97                                               |
| Innodb_buffer_pool_pages_misc         | 656                                              |
| Innodb_buffer_pool_pages_total        | 16384                                            |
| Innodb_buffer_pool_read_ahead_rnd     | 0                                                |
| Innodb_buffer_pool_read_ahead         | 0                                                |
| Innodb_buffer_pool_read_ahead_evicted | 0                                                |
| Innodb_buffer_pool_read_requests      | 29154292                                         |
| Innodb_buffer_pool_reads              | 916923                                           |
| Innodb_buffer_pool_wait_free          | 46805                                            |
| Innodb_buffer_pool_write_requests     | 6627242                                          |
+---------------------------------------+--------------------------------------------------+
```

In this case, the hit ratio is `1 - (Innodb_buffer_pool_reads / Innodb_buffer_pool_read_requests)`.

3. Observe how the hit/miss ratio changes over time and analyze the results yourself. You should submit a report for this results.

4. Close the MySQL session:

```bash
mysql> quit
```

5. Shut down the MySQL server:

```bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
```

## Report Submission

1. Run the TPC-C benchmark on MySQL
2. Observe how the hit rate of MySQL's buffer pool changes over time
3. Analyze the results

Organize the results into a single report and submit it. Follow the [submission guide](report-submission-guide.md) for your report.

## Reference
- [Build and install the source code (5.7)](https://github.com/meeeejin/til/blob/master/mysql/build-and-install-the-source-code-5.7.md)
- [Percona-Lab/tpcc-mysql](https://github.com/Percona-Lab/tpcc-mysql)
- [tpcc-mysql: Simple usage steps and how to build graphs with gnuplot](https://www.percona.com/blog/2013/07/01/tpcc-mysql-simple-usage-steps-and-how-to-build-graphs-with-gnuplot/)
- [SHOW ENGINE Statement](https://dev.mysql.com/doc/refman/5.7/en/show-engine.html)