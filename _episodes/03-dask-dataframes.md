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
Lets activate a conda environment with all the python packages pre installed for ease of use with these tutorials.
We will interactively learn dask dataframe fundamentals, so running on the compute nodes interactively is advisable for this section.

~~~
qsub -I -P Training -l select=1:ncpus=4:mem=6GB -l walltime=00:30:00
source /project/Training/kmarTrain/miniconda3/bin/activate
~~~
{: .bash}

You should see an extra conda environment named dask which we will activate
~~~
conda list env
~~~
{:. bash}

~~~
                         /home/kmar7637/miniconda3
base                  *  /project/Training/kmarTrain/miniconda3
dask                     /project/Training/kmarTrain/miniconda3/envs/dask
~~~
{: .output}

Activate the dask conda environment. Notice the environment on the left of the terminal change to (dask). We will interactively play with dask dataframes in ipython to get a feel for dataframe manipulations and outputs it causes.
~~~
conda activate dask
cd /project/Training/myname/files
ipython
~~~
{: .bash}

We will generate some data using one of the python files makedata.py by importing it in ipython. 
~~~
import makedata
data = makedata.data()
data
~~~
{:. bash}

The data is preloaded into a dask dataframe. Notice the output to data shows the dataframe metadata. 
The concept of splitting the dask dataframe into pandas sub dataframes can be seen by the ***nopartitians=10*** output. This is the number of partitians the dataframe is split into and in this case was automatically calibrated, but can be specified. There is a trade of between splitting data too much that improves memory management, and the number of extra tasks it generates. For instance, if you have a 1000 GB of data and are using 10 MB chunks, then you have 100,000 partitions. Every operation on such a collection will generate at least 100,000 tasks. But more on this later. For now lets become familiar with some basic Dataframe operations.

___
**Notes**   
<sup id="f1">1[↩](#a1)</sup>As you should recall from the [Introduction to Artemis HPC]({{ site.sih_pages }}/training.artemis.introhpc/) course, the **scheduler** is the software that runs the cluster, allocating jobs to physical compute resources. Artemis HPC provides us with a separate 'mini-cluster' for _Training_, which has a separate PBS scheduler instance and dedicated resources.

<sup id="f2">2[↩](#a2)</sup>In fact, _Awk_ is really a shell, a programming language and a command all in one!

___
<br>
