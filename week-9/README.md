# Week 9

## Overview

This week you will observe how WAF changes by varying the SST file size.

Follow the guide below. If you have any questions, don't hesitate to contact me via email (Mijin An / meeeeejin@gmail.com)

> NOTE: This lab is based on the Linux environment. If you don't have a Linux machine, use [VirturalBox](https://www.virtualbox.org/). (Recommend Ubuntu 18.04)

## Instructions

### 1. Run DB_Bench

```bash
./db_bench --benchmarks="readrandomwriterandom" \
        -db="/home/mijin/backup/rocksdb-data" \
        -use_direct_io_for_flush_and_compaction=true \
        -use_direct_reads=true \
        -target_file_size_base=67108864 \
        -duration=7200 \
        -statistics \
        -stats_dump_period_sec=30 \
        -stats_interval_seconds=10 2>&1 | tee result.txt
```

- `--benchmarks="readrandomwriterandom"`: 1 writer, N threads doing random reads
- `-db="/home/mijin/backup/rocksdb-data"`: The path of RocksDB data directory
- `-use_direct_io_for_flush_and_compaction=true`: Use O_DIRECT for background flush and compaction I/O
- `-use_direct_reads=true`: Use O_DIRECT for reading data
- `-target_file_size_base`: Target file size at level-1
    - **You should change this value according to the capacity of your system**
- `-duration=7200`: Time in seconds for the random-ops tests to run
- `-statistics`: Database statistics
- `-stats_dump_period_sec=30`: Gap between printing stats to log in seconds
- `-stats_interval_seconds=10`: Report stats every N second

> When using the leveled compaction, if the level does not increase, you need to increase the benchmark execution time or adjust the memtable size or the multiplier value according to the free capacity of your system.

### 2. Record and analysis the RocksDB stats

In the RocksDB log file, you can get the below stat:

```bash
$ vim /path/to/rocksdb-data
...
** Compaction Stats [default] **
Level    Files   Size     Score Read(GB)  Rn(GB) Rnp1(GB) Write(GB) Wnew(GB) Moved(GB) W-Amp Rd(MB/s) Wr(MB/s) Comp(sec) CompMergeCPU(sec) Comp(cnt) Avg(sec) KeyIn KeyDrop Rblob(GB) Wblob(GB)
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  L0      2/0    1.82 MB   0.5      0.0     0.0      0.0       1.2      1.2       0.0   1.0      0.0     87.3     14.64             13.69      1406    0.010       0      0       0.0       0.0
  L1      1/0   15.99 MB   1.0      4.1     1.2      2.8       3.9      1.1       0.0   3.1     72.7     69.5     57.15             53.19       351    0.163     63M  2808K       0.0       0.0
  L2      1/0   61.43 MB   0.4      4.3     1.0      3.3       3.3      0.0       0.0   3.2     87.2     67.1     50.55             46.96        57    0.887     70M    15M       0.0       0.0
 Sum      4/0   79.24 MB   0.0      8.4     2.3      6.1       8.4      2.4       0.0   6.8     70.0     70.6    122.35            113.85      1814    0.067    134M    18M       0.0       0.0
 Int      0/0    0.00 KB   0.0      0.0     0.0      0.0       0.0      0.0       0.0   4.5     61.1     74.9      0.27              0.26         6    0.046    256K    11K       0.0       0.0
 ...
```

Observe how WAF (`W-Amp`) differs from level to level. Also, compare these results with other results with different file sizes.

### 3. Change the SST file size and repeat steps 1-2

Compare **at least three** different SST file sizes. Resize the SST file considering your system's capacity.

## Report Submission

1. Run DB_Bench by varying the SST file size
    - At least three different SST file sizes
2. Observe how WAF changes
    - Observe the correlation between SST file size and WAF
3. Present the experimental results
4. Analyze the results

Organize the results into a single report and submit it. Follow the [submission guide](../report-submission-guide.md) for your report.

## Reference
- [RocksDB GitHub](https://github.com/facebook/rocksdb) 
- [RocksDB Installation](https://github.com/facebook/rocksdb/blob/main/INSTALL.md)
- [RocksDB Benchmarking tools](https://github.com/facebook/rocksdb/wiki/Benchmarking-tools)
- [Mijin An, How to use db_bench](https://github.com/meeeejin/til/blob/master/rocksdb/how-to-use-db_bench.md)
- [Rocksdb Universal Compaction](https://github.com/facebook/rocksdb/wiki/Universal-Compaction)