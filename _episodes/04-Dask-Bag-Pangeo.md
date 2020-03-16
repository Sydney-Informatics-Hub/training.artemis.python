---
title: "Introducing dask Bag and more dask examples"
teaching: 20
exercises: 20
questions:
- "How do I deal with large irregular data and show me some real world examples of Dask"
objectives:
- "Deal with semi-structured and unstructured data in memory efficient and parallel manner"
keypoints:
- "Dask Bag uses map filter and group by operations on python objects or semi/unstrucutred data"
- "dask.multiprocessing is under the hood"
---

# What is Dask Bag
Dask Bag implements operations like map, filter, groupby and aggregations on collections of Python objects. It does this in parallel and in small memory using Python iterators.

Dask Bags are often used to do simple preprocessing on log files, JSON records, or other user defined Python objects

Execution on bags provide two ***benefits***:
1. ***Parallel:*** data is split up, allowing multiple cores or machines to execute in parallel
2. ***Iterating:*** data processes lazily, allowing smooth execution of ***larger-than-memory data***, even on a single machine within a single partition

By default, dask.bag uses dask.multiprocessing for computation. As a benefit, Dask bypasses the GIL and uses multiple cores on pure Python objects. As a drawback, Dask Bag doesn’t perform well on computations that include a great deal of inter-worker communication.

Because the multiprocessing scheduler requires moving functions between multiple processes, we encourage that Dask Bag users also install the cloudpickle library to enable the transfer of more complex functions

What are the ***drawbacks*** ?

 - Bag operations tend to be slower than array/DataFrame computations in the same way that standard Python containers tend to be slower than NumPy arrays and Pandas DataFrames
 
 - Bags are immutable and so you can not change individual elements
 
 - By default, bag relies on the multiprocessing scheduler, which has known limitations - the main ones being:
  	a, The multiprocessing scheduler must serialize data between workers and the central process, which can be expensive
	b, The multiprocessing scheduler must serialize functions between workers, which can fail. The Dask site recommends using 		cloudpickle to enable the transfer of more complex functions.

### To learn Bag - Lets get our hands dirty with some examples

Lets get back to ipython, ensuring that the dask conda environment is still activate.

~~~
source /project/Training/kmarTrain/miniconda3/bin/activate
conda activate dask
qsub -I -P Training -l select=1:ncpus=4:mem=4GB -l walltime=00:40:00 (depending on class participation 24 cpu limit)
cd /project/Training/myname
ipython
~~~

We will investigate data located on the web that logs all juypter notebook instances run on the net. Two files are 
1. Log files of every entry specific to a certain day
2. An index of daily log files

Before we start, some python packages are needed. Types these commands directly into ipython
~~~
import dask.bag as db
import json
import os
import re
import time
~~~
{: .bash}

Investigate underlying data by reading text file that houses daily data into a bag. First few rows are displayed
~~~
db.read_text('https://archive.analytics.mybinder.org/events-2018-11-03.jsonl').take(3)
~~~
{: .bash}

The output should give you a task for the underlying data in the daily log files. Essential this is a text file in json format.
~~~
('{"timestamp": "2018-11-03T00:00:00+00:00", "schema": "binderhub.jupyter.org/launch", "version": 1, "provider": "GitHub", "spe                c": "Qiskit/qiskit-tutorial/master", "status": "success"}\n',
 '{"timestamp": "2018-11-03T00:00:00+00:00", "schema": "binderhub.jupyter.org/launch", "version": 1, "provider": "GitHub", "spe                c": "ipython/ipython-in-depth/master", "status": "success"}\n',
 '{"timestamp": "2018-11-03T00:00:00+00:00", "schema": "binderhub.jupyter.org/launch", "version": 1, "provider": "GitHub", "spe                c": "QISKit/qiskit-tutorial/master", "status": "success"}\n')

~~~
{: .output}


Index file loaded as a bag. No data transfer or computation kicked off - just organising mapping the file to a json structure
~~~
index = db.read_text('https://archive.analytics.mybinder.org/index.jsonl').map(json.loads)
index
index.take(2)
~~~
{: .bash}

These files aren't big at all - a sign is the number of partitians of 1. Dask would automatically split the data up for large data. 
~~~
In [5]: index = db.read_text('https://archive.analytics.mybinder.org/index.jsonl').map(json.loads)
In [6]: index
Out[6]: dask.bag<loads, npartitions=1>
In [7]: index.take(2)
Out[7]:
({'name': 'events-2018-11-03.jsonl', 'date': '2018-11-03', 'count': '7057'},
 {'name': 'events-2018-11-04.jsonl', 'date': '2018-11-04', 'count': '7489'})
~~~
{: .output}

We can perform some operations on these two files. Please note the Dask Bag API (in the provided links section) for the signatures of available functions and their requirements.

Read the index file as a dask bag and perform a mapping of the data to the function json.loads(). This function loads strings into a json object - given it adheres to the json structure.

~~~
index = db.read_text('https://archive.analytics.mybinder.org/index.jsonl').map(json.loads)
print(index)
~~~
{: .bash}

### PANGEO GEOSCIENCE EXAMPLE
Pangeo is first and foremost a community promoting open, reproducible, and scalable science.
In practice its not realy a python package, but a collection of packages, supported datasets, tutorials and documentation used to promote scalable science. Its motivation was driven by data becoming increasingly large, the fragmentation of software making reproducability difficult, and a growing technology gap between industry and traditional science.

As such the Pangeo community supports using dask on HPC. We will run through an example of using our new found knowledge of dask on large dataset computation and visualisation.


### Some links:

on dask bag fundamentals
https://docs.dask.org/en/latest/bag.html

Bag API's:
https://docs.dask.org/en/latest/bag-api.html

Dask bag limitations:
https://docs.dask.org/en/latest/shared.html

Pangeo info:
https://pangeo.io/#what-is-pangeo


<br>

___
**Notes**   
<sup id="f1">1[↩](#a1)</sup>In this construction, only either _j_ or _m_ are required, so ```:j``` will retrieve the _j_-th element and beyond, whilst ```::m``` will retrieve the first _m_ elements (up to element _m-1_, as the indexing starts with _0_).

___
<br>
