# Week 9

## Overview

This week you will observe how WAF changes by varying the SST file size.

Follow the guide below. If you have any questions, don't hesitate to contact me via email (Mijin An / meeeeejin@gmail.com)

> NOTE: This lab is based on the Linux environment. If you don't have a Linux machine, use [VirturalBox](https://www.virtualbox.org/). (Recommend Ubuntu 18.04)

## Instructions

### 1. Run DB_Bench

```bash
./db_bench --benchmarks="readrandomwriterandom" \
        -db="/home/mijin/swe3033/rocksdb-data" \
        -use_direct_io_for_flush_and_compaction=true \
        -use_direct_reads=true \
        -target_file_size_base=2097152 \
        -write_buffer_size=2097152 \
        -max_bytes_for_level_base=33554432 \
        -max_bytes_for_level_multiplier=5 \
        -duration=3600 \
        -statistics \
        -stats_dump_period_sec=30 \
        -stats_interval_seconds=10 2>&1 | tee result.txt
```

- `--benchmarks="readrandomwriterandom"`: 1 writer, N threads doing random reads
- `-db="/path/to/rocksdb-data"`: The path of RocksDB data directory
- `-use_direct_io_for_flush_and_compaction=true`: Use O_DIRECT for background flush and compaction I/O
- `-use_direct_reads=true`: Use O_DIRECT for reading data
- `-target_file_size_base`: SST file size
    - **You should change this value every experiment**
    - 💡 Change the SST file size to **2MB (2097152), 8MB (8388608), 16MB (16777216)**
- `-write_buffer_size`: Memtable size
    - **You should change this value every experiment equal to the SST file size**
    - 💡 Change the memtable size to **2MB (2097152), 8MB (8388608), 16MB (16777216)**
- `-max_bytes_for_level_base`: Max bytes for level-1
    - In this lab, we fix the max size to **32MB**
- `-max_bytes_for_level_multiplier`: A multiplier to compute max bytes for level-N (N >= 2)
    - In this lab, we fix the multipler to **5**
- `-duration`: Time in seconds for the tests to run
    - If the level does not increase, you need to increase the benchmark execution time
    - I recommend to run the benchmark until the level increases more than **2**
- `-statistics`: Database statistics
- `-stats_dump_period_sec`: Gap between printing stats to log in seconds
- `-stats_interval_seconds`: Report stats every N second

### 2. Record and analyze the RocksDB stats

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

- Resize the SST file
- Compare **three** different SST file sizes (2MB, 8MB, 16MB)
    - Configure three different values for `-target_file_size_base` and `-write_buffer_size`
    - To follow the default setting of RocksDB, set the memtable size equal to the sst file size each time

## Report Submission

1. Run DB_Bench by varying the SST file size
    - Change the SST file/memtable size to **2MB (2097152), 8MB (8388608), 16MB (16777216)**
2. Observe how WAF changes
    - Observe the correlation between SST file size and WAF
    - e.g., same level size, but the different file size and the number of files ➔ In terms of compaction, the impact of the increased number of SST files ➔ As a result, the effect on the WAF
3. Present the experimental results
4. Analyze the results

Organize the results into a single report and submit it. Follow the [submission guide](../report-submission-guide.md) for your report.

## Reference
- [RocksDB GitHub](https://github.com/facebook/rocksdb) 
- [RocksDB Installation](https://github.com/facebook/rocksdb/blob/main/INSTALL.md)
- [RocksDB Benchmarking tools](https://github.com/facebook/rocksdb/wiki/Benchmarking-tools)
- [Mijin An, How to use db_bench](https://github.com/meeeejin/til/blob/master/rocksdb/how-to-use-db_bench.md)
- [Rocksdb Universal Compaction](https://github.com/facebook/rocksdb/wiki/Universal-Compaction)