# FAQ for week 10

## About Report Submission

You need to write a **single page** proposal (see instruction 0: Write a report) and organize experimental results 
into a single report PDF file.


## How to get `androbench.sql`

You can download `androbench.sql` file (single SQL file) in our Github.

https://github.com/meeeejin/SWE3033-F2021/blob/main/week-10/androbench.sql

## What is default jounral mode ?

The default journal mode is `del` (i.e., delete RBJ mode).


## SQLite build error

- Problem : `tclsh: not found`
    ```
    /bin/sh: 1: tclsh: not found
    Makefile:1101: recipe for target 'shell.c' failed
    make: *** [shell.c] Error 127
    make: *** Waiting for unfinished jobs....
    ```

- Soultion
    ```
    sudo apt-get install tcl-dev
    make clean
    make -j
    ```
