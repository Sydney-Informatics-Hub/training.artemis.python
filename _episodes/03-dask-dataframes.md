---
title: "Intro to Dask and Dask Dataframes"
teaching: 25
exercises: 5
questions:
- Scale code in a familiar way
objectives:
- "Intro to Dask concepts and High level datastructures"
- "Use dask dataframes"
keypoints:
- "Dask builds on numpy and pandas APIs but operates in a parallel manner"
- "Computations are by default lazy and must be triggered - this reduces unneccessary computation time"
---

# What is Dask?
Dask is a flexible library for parallel computing in Python.

Dask is composed of two parts:
Dynamic task scheduling optimized for computation. This is similar to Airflow, Luigi, Celery, or Make, but optimized for interactive computational workloads. “Big Data” collections like parallel arrays, dataframes, and lists that extend common interfaces like ***NumPy, Pandas, or Python iterators*** to larger-than-memory or distributed environments. These parallel collections run on top of dynamic task schedulers.

Dask emphasizes the following virtues:

Familiar: Provides parallelized NumPy array and Pandas DataFrame objects
Flexible: Provides a task scheduling interface for more custom workloads and integration with other projects.
Native: Enables distributed computing in pure Python with access to the PyData stack.
Fast: Operates with low overhead, low latency, and minimal serialization necessary for fast numerical algorithms
Scales up: Runs resiliently on clusters with 1000s of cores
Scales down: Trivial to set up and run on a laptop in a single process
Responsive: Designed with interactive computing in mind, it provides rapid feedback and diagnostics to aid humans

Dask provides high level collections - these are ***Dask Dataframes, bags, and arrays***.
On a low level, dask dynamic task schedulers to scale up or down processes, and presents parallel computations by implementing task graphs. It provides an alternative to scaling out tasks instead of threading (IO Bound) and multiprocessing (cpu bound).

For more background info:
https://docs.dask.org/en/latest/

# Dask Dataframes

A Dask DataFrame is a large parallel DataFrame composed of many smaller Pandas DataFrames, split along the index. These Pandas DataFrames may live on disk for larger-than-memory computing on a single machine, or on many different machines in a cluster. One Dask DataFrame operation triggers many operations on the constituent Pandas DataFrames.

Common Use Cases:
Dask DataFrame is used in situations where Pandas is commonly needed, usually when Pandas fails due to data size or computation speed:
- Manipulating large datasets, even when those datasets don’t fit in memory
- Accelerating long computations by using many cores
- Distributed computing on large datasets with standard Pandas operations like groupby, join, and time series computations

Dask Dataframes **may not be the best choice** if:
your data fits comfortable in RAM - Use pandas only!
If you need a proper database.
You need functions not implemented by dask dataframes - see Dask Delayed.

# Lets begin



___
**Notes**   
<sup id="f1">1[↩](#a1)</sup>As you should recall from the [Introduction to Artemis HPC]({{ site.sih_pages }}/training.artemis.introhpc/) course, the **scheduler** is the software that runs the cluster, allocating jobs to physical compute resources. Artemis HPC provides us with a separate 'mini-cluster' for _Training_, which has a separate PBS scheduler instance and dedicated resources.

<sup id="f2">2[↩](#a2)</sup>In fact, _Awk_ is really a shell, a programming language and a command all in one!

___
<br>
