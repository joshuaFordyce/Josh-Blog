---
title: 'ParallelQueryDecoder Part2 '
description: 'In this article, we explored the concept of Partitioning in Data Warehouses'
pubDate: 'April 28, 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/pexels-flodahm-699459.jpg'
tags: ['ML']
---

In part two of the our Parallel QUery Decoder series, we'll dig deep into the mechanics and purpose of the GIL.


What is the GIL:

- The Global Intepreter Lock is a mutex (mutual exclusion lock) that protects acceess to Python objects, preventing multiple native threads
from executing Python Bytecodes simultaneously in the CPython interpreter

- THe GIL is not a feature of the Python language itself but its an implementation detail of the most popular interpreter, CPython.

- Why does it Exist
  - The reason the GIL exists is to simplify Python's memory management and ensure thread safety
    1. CPython manages memory using a simple, efficient technique called reference counting. Every object
       carries a counter of how many variables are curretnly referencing it. When the counter drops to zero,
       the object's memory is immediately freed

    2. If two different threads tried to simultaneously increment or decrement an object's reference counter, the operation would be non-atomic
       leading to a race condition. The counter could become incorrect, resulting in memory leaks (counter never hits zero) or crashes

    3. The GIL essentially acts like a heavy handed security guard. By ensuring only one thread can execute
       Python bytecode at a time, it guarantees that all refererence count manipulations are thread-safe without requiring locks on every single object.
How the GIL throttles Performance

- The Gil effectively enforces that even on a machine with 64 CPU cores, a single CPython process can
  only execute Python code on one core at a time.

  1. CPU-BOUND Trap
     - When a  task involves heavy calculation, the thread continuously executes Python bytecode. The GIL is only perodically released after a certain amount of work( a "time slice", around 5ms)
       - While the decompression work is running, the thread holds the GIL. If the task takes 50 ms, the main application thread and any other Python threads are completely paused for that full 50ms, waiting for the GIL to be returned.
       THis is why multi-threading for CPU-bound work is useless in CPython

  2. The I/O Escape Hatch
     - The one saving grace is for I/O-bound work(waiting for a netowrk, diskread or database)
       - When a thread issues an I/O ops, it knows it will be blocked by the operating system for a long time
       At this moment, the thread voluntarily releases the GIL
       - This allows another Python thread to immediately acquire the GIL and execute code while the first thread waits.

Necessity of ProcessPoolExecutor:

- Given the GIL constraint, the ProcessPool Executor is the only reliable way to achieve the parallel performance the Trino Decoder needs



