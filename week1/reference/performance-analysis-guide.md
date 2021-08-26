# Performance Analysis

## Check your system's specs

### OS version

```bash
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.5 LTS
Release:        18.04
Codename:       bionic
```

### Kernel version

```bash
$ uname -a
Linux u18 5.4.0-66-generic #74~18.04.2-Ubuntu SMP Fri Feb 5 11:17:31 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```

### SATA device info

```bash
$ sudo smartctl -a /dev/sda1
smartctl 6.6 2016-05-31 r4324 [x86_64-linux-5.4.0-66-generic] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Family:     Samsung based SSDs
Device Model:     Samsung SSD 850 PRO Series
...
```

### NVMe device info

```bash
$ sudo nvme list
Node             SN                   Model                                    Namespace Usage                      Format           FW Rev
---------------- -------------------- ---------------------------------------- --------- -------------------------- ---------------- --------
/dev/nvme0n1     S4EUN1234567890      Samsung SSD 970 EVO Plus 250GB           1          60.02  GB / 250.06  GB    512   B +  0 B   2B2QEXM7
/dev/nvme1n1     PHMB74751234567890   INTEL SSDPED1D480GA                      1         480.10  GB / 480.10  GB    512   B +  0 B   E2010325
```

### CPU info

```bash
$ cat /proc/cpuinfo
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 85                                                                                                                                          model name      : Intel(R) Xeon(R) Silver 4216 CPU @ 2.10GHz
...
```

- CPU cores: `grep 'cpu cores' /proc/cpuinfo | uniq`
- CPU threads: `echo "CPU threads: $(grep -c processor /proc/cpuinfo)"`

### Memory info

```bash
$ cat /proc/meminfo
MemTotal:       32513496 kB
MemFree:        19273628 kB
MemAvailable:   28705904 kB
Buffers:          152336 kB
...
```

## TPC-C benchmarking

```bash
***************************************                                                         
*** ###easy### TPC-C Load Generator ***
***************************************
option h with value 'localhost'
option S (socket) with value '/tmp/mysql.sock'
option d with value 'tpcc2000'
option u with value 'root'
option p with value 'vldb7988'
option w with value '2000'
option c with value '64'
option r with value '1200'
option l with value '3600'
<Parameters>
     [server]: localhost
     [port]: 3306
     [DBname]: tpcc2000
       [user]: root
       [pass]: vldb7988
  [warehouse]: 2000
 [connection]: 64
     [rampup]: 1200 (sec.)
    [measure]: 3600 (sec.)

RAMP-UP TIME.(1200 sec.)

MEASURING START.

  10, trx: 493, 95%: 764.362, 99%: 1011.230, max_rt: 3165.036, 460|7427.418, 48|1684.188, 51|3922.317, 51|
5881.677
  20, trx: 456, 95%: 838.937, 99%: 1053.247, max_rt: 1200.276, 465|3532.571, 46|324.480, 45|1275.379, 49|5
178.482
  30, trx: 503, 95%: 828.704, 99%: 1013.655, max_rt: 2262.165, 491|4421.898, 50|410.298, 50|1592.177, 44|4
334.301
  40, trx: 511, 95%: 905.753, 99%: 1057.354, max_rt: 1341.675, 529|2798.316, 51|387.980, 51|3083.647, 50|5
330.978
...
STOPPING THREADS................................................................

<Raw Results>
  [0] sc:4 lt:218054  rt:0  fl:0 avg_rt: 576.5 (5)
  [1] sc:0 lt:217563  rt:0  fl:0 avg_rt: 387.7 (5)
  [2] sc:3587 lt:18220  rt:0  fl:0 avg_rt: 131.3 (5)
  [3] sc:0 lt:21800  rt:0  fl:0 avg_rt: 1576.9 (80)
  [4] sc:2610 lt:19202  rt:0  fl:0 avg_rt: 2724.4 (20)
 in 3600 sec.

<Raw Results2(sum ver.)>
  [0] sc:4  lt:218068  rt:0  fl:0 
  [1] sc:0  lt:218068  rt:0  fl:0 
  [2] sc:3587  lt:18220  rt:0  fl:0 
  [3] sc:0  lt:21802  rt:0  fl:0 
  [4] sc:2610  lt:19202  rt:0  fl:0 

<Constraint Check> (all must be [OK])
 [transaction percentage]
        Payment: 43.42% (>=43.0%) [OK]
   Order-Status: 4.35% (>= 4.0%) [OK]
       Delivery: 4.35% (>= 4.0%) [OK]
    Stock-Level: 4.35% (>= 4.0%) [OK]
 [response time (at least 90% passed)]
      New-Order: 0.00%  [NG] *
        Payment: 0.00%  [NG] *
   Order-Status: 16.45%  [NG] *
       Delivery: 0.00%  [NG] *
    Stock-Level: 11.97%  [NG] *

<TpmC>
                 3634.300 TpmC
```

- Note the `TpmC` value
- The metric for evaluating TPC-C performance is the number of transactions completed per minute (TpmC)
- In the example above, the `TpmC = 3634`. And it means that 3634 *New-Order* transactions were processed per minute

## Monitoring I/O status using `iostat`

```bash
$ iostat -mx 1
Linux 4.15.0-55-generic (mijin-desktop)         2019년 09월 16일        _x86_64_        (32 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          10.41    0.00    5.03   21.70    0.00   62.85

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
loop0             0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
nvme0n1           0.00   746.00 24513.00  574.00    95.75   205.30    24.58    13.75    0.60    0.55    2.43   0.04 100.00
sda               0.00     2.00  598.00    5.00     2.34     2.37    15.99     0.04    0.06    0.05    1.60   0.05   3.20
sdb               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdc               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           9.25    0.00    4.43   21.94    0.00   64.38

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
loop0             0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
nvme0n1           0.00   732.00 22845.00  547.00    89.24   192.13    24.63    13.89    0.64    0.59    2.37   0.04  99.20
sda               0.00     2.00  570.00    5.00     2.23     2.43    16.57     0.02    0.04    0.03    1.60   0.02   1.20
sdb               0.00     4.00    0.00    3.00     0.00     0.18   122.67     0.04   13.33    0.00   13.33  13.33   4.00
sdc               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

...
```

- Note each value of `iostat` of your data device
  - Especially, `r/s`, `w/s`, `rMB/s`, `wMB/s`, `%util`, `avg-cpu`
  - [More details](https://man7.org/linux/man-pages/man1/iostat.1.html)

## Monitoring MySQL's Buffer Hit Rate

```bash
$ ./bin/mysql -uroot -pyourPassword
Welcome to the MySQL monitor.  Commands end with ; or \g.Your MySQL connection id is 8Server version: 8.0.15 Source distribution
Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show engine innodb status;
...
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 2353004544
Dictionary memory allocated 373557
Buffer pool size   524288
Free buffers       0
Database pages     524287
Old database pages 193695
Modified db pages  0
Pending reads      1
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 36985
0.00 youngs/s, 2465.50 non-youngs/s
Pages read 576950, created 160, written 177
38431.37 reads/s, 0.13 creates/s, 11.13 writes/s
Buffer pool hit rate 986 / 1000, young-making rate 0 / 1000 not 63 / 1000
Pages read ahead 0.00/s, evicted without access 3444.10/s, Random read ahead 0.00/s
LRU len: 524287, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
...
```

- Note the `Buffer pool hit rate` metric
  - It means *the buffer pool page hit rate for pages read from the buffer pool vs. from disk storage*
  - In the example above, `buffer pool hit rate = 0.986`
