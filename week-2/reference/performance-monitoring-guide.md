# Performance Monitoring

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

  10, trx: 493, 95%: 764.362, 99%: 1011.230, max_rt: 3165.036, 460|7427.418, 48|1684.188, 51|3922.317, 51|5881.677
  20, trx: 456, 95%: 838.937, 99%: 1053.247, max_rt: 1200.276, 465|3532.571, 46|324.480, 45|1275.379, 49|5178.482
  30, trx: 503, 95%: 828.704, 99%: 1013.655, max_rt: 2262.165, 491|4421.898, 50|410.298, 50|1592.177, 44|4334.301
  40, trx: 511, 95%: 905.753, 99%: 1057.354, max_rt: 1341.675, 529|2798.316, 51|387.980, 51|3083.647, 50|5330.978
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

- Note the `TpmC` value, a key performance metric of the TPC-C benchmark
  - It is the number of transactions completed per minute (TpmC)
  - In the example above, the `TpmC = 3634`. And it means that 3634 *New-Order* transactions were processed per minute
- `95%: 764.362` - The 95% Response time of New Order transactions per given interval
- `99%: 1011.230` - The 99% Response time of New Order transactions per given interval
- `max_rt: 3165.036` - The Max Response time of New Order transactions per given interval
- `460|7427.418, 48|1684.188, 51|3922.317, 51|5881.677` - throughput and max response time for the other kind of transactions and can be ignored

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

## Monitoring CPU status using `mpstat` or `htop`

### `mpstat`

```bash
$ mpstat -P ALL 1
Linux 5.4.0-80-generic (u18)    2021년 09월 04일        _x86_64_        (32 CPU)

16시 58분 06초  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
16시 58분 07초  all    4.17    0.00    1.10    1.63    0.00    0.44    0.00    0.00    0.00   92.66
16시 58분 07초    0    0.99    0.00    0.99    0.00    0.00    1.98    0.00    0.00    0.00   96.04
16시 58분 07초    1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
...
16시 58분 07초   30    1.03    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   98.97
16시 58분 07초   31    6.93    0.00    0.99    2.97    0.00    0.99    0.00    0.00    0.00   88.12
```

- Field description
  - CPU: Processor number
  - %usr: Show the percentage of CPU utilization that occurred while executing at the user level (application)
  - %sys: Show the percentage of CPU utilization that occurred while executing at the system level (kernel)
  - %iowait: Show the percentage of time that the CPU or CPUs were idle during which the system had an outstanding disk I/O request
  - %idle: Show the percentage of time that the CPU or CPUs were idle and the system did not have an outstanding disk I/O reque
- [More details](https://man7.org/linux/man-pages/man1/mpstat.1.html)

### `htop`

```bash
$ sudo apt-get install htop
$ htop
```

## Monitoring memory status using `vmstat`

```bash
$ $ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  3 1913088 19874980 1386900 9064876    0    0   224   223    1    0  2  0 97  1  0
 0  4 1913088 19873088 1387100 9064892    0    0 46624 53844 8703 55258  3  1 94  2  0
 3  3 1913088 19874632 1387200 9064876    0    0 52912 52444 9404 60254  3  2 93  2  0
 1  2 1913088 19876232 1387360 9064880    0    0 51280 51420 9337 61119  4  1 93  2  0
 1  3 1913088 19876308 1387528 9064844   28    0 57356 56148 10189 69357  4  2 91  2  0
 4  1 1913088 19874784 1387700 9064944    0    0 41920 45128 8703 53901  3  1 94  2  0
```

- Field description
  - Procs
    - r: The number of runnable processes (running or waiting for run time)
    - b: The number of processes blocked waiting for I/O to complete
  - Memory
    - swpd: the amount of swap memory used
    - free: the amount of idle memory
    - buff: the amount of memory used as buffers
  - Swap
    - si: Amount of memory swapped in from disk (/s)
    - so: Amount of memory swapped to disk (/s)
  - IO
    - bi: Blocks received from a block device (blocks/s)
    - bo: Blocks sent to a block device (blocks/s)
  - System
    - in: The number of interrupts per second, including the clock
    - cs: The number of context switches per second
  - CPU
    - us: Time spent running non-kernel code (user time, including nice time)
    - sy: Time spent running kernel code (system time)
    - id: Time spent idle
    - wa: Time spent waiting for IO
    - st: Time stolen from a virtual machine
- [More details](https://man7.org/linux/man-pages/man8/vmstat.8.html)