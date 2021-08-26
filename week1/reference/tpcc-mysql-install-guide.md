# tpcc-mysql: Quick Start Guide

**:warning: This guide is based on Ubuntu 18.04.**

## How to install MySQL 5.7

Building MySQL 5.7 from the source code enables you to customize build parameters, compiler optimizations, and installation location.

### Prerequisites

- libreadline

```bash
$ sudo apt-get install libreadline6 libreadline6-dev
```

If the package `libreadline6` is not available, download the newer version:

```bash
$ sudo apt-get install libreadline7
```

- libaio

```bash
$ sudo apt-get install libaio1 libaio-dev
```

- etc.

```bash
$ sudo apt-get install build-essential cmake libncurses5 libncurses5-dev bison
```

### Build and install

1. Download the latest source code of [MySQL 5.7 Community Server](https://dev.mysql.com/downloads/mysql/5.7.html#downloads) using `wget`:

```bash
$ wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.33.tar.gz
```

2. Extract the `mysql-5.7.33.tar.gz` file:

```bash
$ tar -xvzf mysql-5.7.33.tar.gz
$ cd mysql-5.7.33
```

3. In MySQL 5.7, the Boost library is required to build MySQL. Therefore, download it first:

```bash
$ cmake -DDOWNLOAD_BOOST=ON -DWITH_BOOST=/path/to/download/boost -DCMAKE_INSTALL_PREFIX=/path/to/dir
```

If you already have the Boost library, update the path to the boost directory:

```bash
$ cmake -DWITH_BOOST=/path/to/boost -DCMAKE_INSTALL_PREFIX=/path/to/dir
```

4. Then, build and install the source code:
(8: # of cores in your machine)

```bash
$ make -j8 install
```

5. Use `mysqld --initialize` to initialize the data directory, including the *mysql* database containing the initial MySQL grant tables that determine how users are permitted to connect to the server:
- `--datadir` : the path to the MySQL data directory
- `--basedir` : the path to the MySQL installation directory

```bash
$ ./bin/mysqld --initialize --user=mysql --datadir=/path/to/datadir --basedir=/path/to/basedir
```

6. Reset the root password:

```bash
$ ./bin/mysqld_safe --skip-grant-tables --datadir=/path/to/datadir

$ ./bin/mysql -uroot

root:(none)> use mysql;

root:mysql> update user set authentication_string=password('yourPassword') where user='root';
root:mysql> flush privileges;
root:mysql> quit;

$ ./bin/mysql -uroot -p

root:mysql> set password = password('yourPassword');
root:mysql> quit;
```

7. Open `.bashrc` and add MySQL installation path to your path:

```bash
$ vi ~/.bashrc

export PATH=/path/to/basedir/bin:$PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/basedir/lib/

$ source ~/.bashrc
```

8. Create a configuration file (`my.cnf`) for the MySQL server. For example, create a `my.cnf` in your local directory and copy the below texts to your `my.cnf`. You should modify the path `/path/to/datadir` to your local path. And if you separate the log device from the data device, uncomment `innodb_log_group_home_dir=/path/to/logdir/` and correct the path of the log directory:

```bash
$ vi my.cnf

#
# The MySQL database server configuration file
#
[client]
user    = root
port    = 3306
socket  = /tmp/mysql.sock

[mysql]
prompt  = \u:\d>\_

[mysqld_safe]
socket  = /tmp/mysql.sock

[mysqld]
# Basic settings
default-storage-engine = innodb
pid-file        = /path/to/datadir/mysql.pid
socket          = /tmp/mysql.sock
port            = 3306
datadir         = /path/to/datadir/
log-error       = /path/to/datadir/mysql_error.log

#
# Innodb settings
#
# Page size
innodb_page_size=16KB

# Buffer pool settings
innodb_buffer_pool_size=1G
innodb_buffer_pool_instances=8

# Transaction log settings
innodb_log_file_size=100M
innodb_log_files_in_group=2
innodb_log_buffer_size=32M

# Log group path (iblog0, iblog1)
# If you separate the log device, uncomment and correct the path
#innodb_log_group_home_dir=/path/to/logdir/

# Flush settings (SSD-optimized)
# 0: every 1 seconds, 1: fsync on commits, 2: writes on commits
innodb_flush_log_at_trx_commit=0
innodb_flush_neighbors=0
innodb_flush_method=O_DIRECT
```

For details on each variable, refer to [MySQL 5.7 Server System Variable](https://dev.mysql.com/doc/refman/5.7/en/server-system-variable-reference.html)

9. Shut down the MySQL server and restart it with `my.cnf`:

```bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
$ ./bin/mysqld_safe --defaults-file=/path/to/my.cnf
```

10. You can shut down the server using the below command:

```bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
```

## How to install tpcc-mysql

### Prerequisite

- libmysqlclient-dev

```bash
$ sudo apt-get install libmysqlclient-dev
```

### Installation

1. Clone *tpcc-mysql* from the [Percona GitHub repository](https://github.com/Percona-Lab/tpcc-mysql):

```bash
$ git clone https://github.com/Percona-Lab/tpcc-mysql.git
```

2. Go to the tpcc-mysql directory and build binaries:

```bash
$ cd tpcc-mysql/src
$ make
```

3. Before running the benchmark, you should create a database for the TPC-C test. Go to the MySQL base directory and run the following commands:

```bash
$ ./bin/mysql -u root -p -e "CREATE DATABASE tpcc;"
$ ./bin/mysql -u root -p tpcc < /path/to/tpcc-mysql/create_table.sql
$ ./bin/mysql -u root -p tpcc < /path/to/tpcc-mysql/add_fkey_idx.sql
```

4. Then go back to the tpcc-mysql directory and load data:

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

5. After loading, run the TPC-C test:

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

### Output

```bash
***************************************
*** ###easy### TPC-C Load Generator ***
***************************************
option h with value '127.0.0.1'
option S (socket) with value '/tmp/mysql.sock'
option d with value 'tpcc'
option u with value 'root'
option p with value 'yourPassword'
option w with value '20'
option c with value '8'
option r with value '10'
option l with value '1200'
<Parameters>
     [server]: 127.0.0.1
     [port]: 3306
     [DBname]: tpcc
       [user]: root
       [pass]: 1234
  [warehouse]: 20
 [connection]: 8
     [rampup]: 10 (sec.)
    [measure]: 1200 (sec.)

RAMP-UP TIME.(10 sec.)

MEASURING START.

10, trx: 11204, 95%: 5.681, 99%: 6.864, max_rt: 17.443, 11206|15.114, 1121|1.739, 1120|17.696, 1120|20.223
20, trx: 11173, 95%: 5.588, 99%: 6.494, max_rt: 13.516, 11169|6.150, 1117|0.965, 1117|17.845, 1118|18.300
30, trx: 11335, 95%: 5.475, 99%: 6.095, max_rt: 10.298, 11338|8.255, 1133|0.937, 1134|16.660, 1133|17.742
...
1200, trx: 11080, 95%: 5.632, 99%: 6.517, max_rt: 10.721, 11082|8.663, 1109|2.241, 1108|18.519, 1107|18.890

STOPPING THREADS........

<Raw Results>
  [0] sc:1125874 lt:223356  rt:0  fl:0 avg_rt: 3.9 (5)
  [1] sc:1347068 lt:2142  rt:0  fl:0 avg_rt: 1.0 (5)
  [2] sc:134924 lt:0  rt:0  fl:0 avg_rt: 0.5 (5)
  [3] sc:134923 lt:0  rt:0  fl:0 avg_rt: 10.5 (80)
  [4] sc:134890 lt:32  rt:0  fl:0 avg_rt: 12.5 (20)
 in 1200 sec.

<Raw Results2(sum ver.)>
  [0] sc:1125875  lt:223356  rt:0  fl:0
  [1] sc:1347090  lt:2142  rt:0  fl:0
  [2] sc:134924  lt:0  rt:0  fl:0
  [3] sc:134923  lt:0  rt:0  fl:0
  [4] sc:134890  lt:32  rt:0  fl:0

<Constraint Check> (all must be [OK])
 [transaction percentage]
        Payment: 43.48% (>=43.0%) [OK]
   Order-Status: 4.35% (>= 4.0%) [OK]
       Delivery: 4.35% (>= 4.0%) [OK]
    Stock-Level: 4.35% (>= 4.0%) [OK]
 [response time (at least 90% passed)]
      New-Order: 83.45%  [NG] *
        Payment: 99.84%  [OK]
   Order-Status: 100.00%  [OK]
       Delivery: 100.00%  [OK]
    Stock-Level: 99.98%  [OK]

<TpmC>
                 67461.500 TpmC
```

Note the `TpmC` value. The metric for evaluating TPC-C performance is the number of transactions completed per minute (TpmC).

#### Optional

With the defined interval (`-i` option), the tool will produce the following output:

```bash
10, trx: 11204, 95%: 5.681, 99%: 6.864, max_rt: 17.443, 11206|15.114, 1121|1.739, 1120|17.696, 1120|20.223
20, trx: 11173, 95%: 5.588, 99%: 6.494, max_rt: 13.516, 11169|6.150, 1117|0.965, 1117|17.845, 1118|18.300
30, trx: 11335, 95%: 5.475, 99%: 6.095, max_rt: 10.298, 11338|8.255, 1133|0.937, 1134|16.660, 1133|17.742
40, trx: 11191, 95%: 5.578, 99%: 6.436, max_rt: 10.070, 11192|7.076, 1119|0.809, 1119|16.984, 1119|19.247
50, trx: 11210, 95%: 5.605, 99%: 6.498, max_rt: 10.288, 11204|6.858, 1121|0.958, 1121|17.104, 1120|19.208
```

Where:

- `10` - the seconds from the start of the benchmark
- `95%: 5.681` - The 95% Response time of New Order transactions per given interval. In this case it is 5.681 sec
- `99%: 6.864` - The 99% Response time of New Order transactions per given interval. In this case it is 6.864 sec
- `max_rt: 17.443` - The Max Response time of New Order transactions per given interval. In this case it is 17.443 sec
- `11206|15.114, 1121|1.739, 1120|17.696, 1120|20.223` - throughput and max response time for the other kind of transactions and can be ignored

## Reference
- [Build and install the source code (5.7)](https://github.com/meeeejin/til/blob/master/mysql/build-and-install-the-source-code-5.7.md)
- [Percona-Lab/tpcc-mysql](https://github.com/Percona-Lab/tpcc-mysql)
- [tpcc-mysql: Simple usage steps and how to build graphs with gnuplot](https://www.percona.com/blog/2013/07/01/tpcc-mysql-simple-usage-steps-and-how-to-build-graphs-with-gnuplot/)
- [MySQL 5.7 Server System Variable Reference](https://dev.mysql.com/doc/refman/5.7/en/server-system-variable-reference.html)
