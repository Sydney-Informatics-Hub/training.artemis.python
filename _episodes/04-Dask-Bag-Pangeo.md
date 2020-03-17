---
title: "Introducing dask Bag and more dask examples"
teaching: 20
exercises: 20
questions:
- "How do I deal with large irregular data and show me some real world examples of Dask"
objectives:
- "Deal with semi-structured and unstructured data in memory efficient and parallel manner"
- "Show me examples of using Dask on Large Datasets"
keypoints:
- "Dask Bag uses map filter and group by operations on python objects or semi/unstrucutred data"
- "dask.multiprocessing is under the hood"
- "Xarray for holding scientific data"
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

The example we will submit is an altered version of Pangeos meteorology use case found here:
https://pangeo.io/use_cases/meteorology/newmann_ensemble_meteorology.html

In the files, open the python script we will run called pangeo.py.
~~~
cd /project/Training/myname/files
nano pangeo.py
~~~

Rather than using a dask dataframe, data is loaded from multiple netcdf files in the data folder relative to where the script resides. 
Xarray is an opensource python package that uses dask in its inner workings. Its design to make working with multi-dimensional data easier by introducing labels in the form of dimensions, coordinates and attributes on top of raw NumPy-like arrays, which allows for a more intuitive, more concise, and less error-prone developer experience.

Its particulary suited for working with netcdf files and its tightly integrated with dask parallel computing



Steps to do:
1. Alter the instance of the Client() object by redirecting the path in the local_directory argument to your /project/Training/myname folder. The Client object sets up a local cluster that uses all availble resources. In this case it is created on one compute node. The optional local_directory specifies a storage area that allows dask to copy temporary data on if RAM is insufficient to work on large datasets - i.e. its a path where dask can spill over some data to still perform data calculations. 

2. Submit the ***pangeo.pbs*** file to the scheduler.

3. Notice the xarray open_mfdaset function loads multiple files that match a naming pattern. Chunking size is specific to the axis ***time***, one chunck for each year. 

~~~
qsub pangeo.pbs
~~~
{: .bash}

Png files should be created based on calculation in the code that measure the variance in temperatures (max minus min observations). As seen before, these calculations are triggered by a ***.compute()*** call. Two images are created, with one demostrating how we can persist the xarray dataset in memory for quick retrievals via the ***.persist()*** call. 

Lets see the image. If you have X11 forwarding enabled you can view it directly from artemis. Alternatively, use scp to copy it locally and view.
~~~
module load imagemagick
display variance_temp.png &
~~~
{: .bash}

<figure>
  <img src="{{ page.root }}/fig/USA_Temp.png" style="margin:10px;width:600px"/>
  <figcaption> Dask Temperature visualisation </figcaption>
</figure><br>



### Some links:

on dask bag fundamentals
https://docs.dask.org/en/latest/bag.html

Bag API's:
https://docs.dask.org/en/latest/bag-api.html

Dask bag limitations:
https://docs.dask.org/en/latest/shared.html

Pangeo info:
https://pangeo.io/#what-is-pangeo

Xarray:
http://xarray.pydata.org/en/stable/


<br>

___
**Notes**   
<sup id="f1">1[↩](#a1)</sup>In this construction, only either _j_ or _m_ are required, so ```:j``` will retrieve the _j_-th element and beyond, whilst ```::m``` will retrieve the first _m_ elements (up to element _m-1_, as the indexing starts with _0_).

___
<br>
