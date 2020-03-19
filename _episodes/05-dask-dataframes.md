---
title: "Intro to Dask and Dask Dataframes"
teaching: 25
exercises: 5
questions:
- Use a modern python library and elegant syntax for performance benefits
objectives:
- "Intro to Dask concepts and High level datastructures"
- "Use dask dataframes"
- "Use dask delayed functions"
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


<figure>
  <img src="{{ page.root }}/fig/dask_pic1.png" style="margin:10px;width:600px"/>
  <figcaption> Dask High Level Schema <a href="https://docs.dask.org/en/latest/">https://docs.dask.org/en/latest/</a></figcaption>
</figure><br>

Dask provides high level collections - these are ***Dask Dataframes, bags, and arrays***.
On a low level, dask dynamic task schedulers to scale up or down processes, and presents parallel computations by implementing task graphs. It provides an alternative to scaling out tasks instead of threading (IO Bound) and multiprocessing (cpu bound).



# Dask Dataframes

A Dask DataFrame is a large parallel DataFrame composed of many smaller Pandas DataFrames, split along the index. These Pandas DataFrames may live on disk for larger-than-memory computing on a single machine, or on many different machines in a cluster. One Dask DataFrame operation triggers many operations on the constituent Pandas DataFrames.

<figure>
  <img src="{{ page.root }}/fig/dask_pic2.png" style="margin:6px;width:400px"/>
  <figcaption> Dask High Level Schema <a href="https://docs.dask.org/en/latest/dataframe.html/">https://docs.dask.org/en/latest/dataframe.html/</a></figcaption>
</figure><br>

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


Lets inspect the data in its types, and also take the first 5 rows. 

By default, dataframe operations are ***lazy*** meaning no computation takes place until specified. The ***.compute()*** triggers such a computation - and we will see later on that it converts a dask dataframe into a pandas dataframe. ***head(rows)*** also triggers a computation - but is really helpful in exploring the underlying data.
~~~
data.dtypes
data.head(5)
~~~
{: .bash}

You should see the below output
~~~
In [6]: data.head(5)
Out[6]:
   age     occupation          telephone  ...       street-address          city  income
0   54  Acupuncturist     (528) 747-6949  ...  1242 Gough Crescent  Laguna Beach  116640
1   38   Shelf Filler       111.247.5833  ...       10 Brook Court     Paragould   57760
2   29    Tax Manager       035-458-1895  ...  278 Homestead Trace    Scottsdale   33640
3   19      Publisher  +1-(018)-082-3905  ...     310 Ada Sideline    East Ridge   14440
4   25      Stationer     1-004-960-0770  ...        711 Card Mall     Grayslake   2500
~~~
{: .output}

Lets perform some familiar operations for those who use pandas.

filter operation - filter people who are older than 60 and assign to another dask array called data2

~~~
data2 = data[data.age > 60]
~~~
{: .bash}

group by operation - calculate the average incomes by occupation. Notice the compute() trigger that performs the operations.

~~~
data.groupby('occupation').income.mean().compute()
~~~
{: .python}

A memory efficient style is to create pipelines of operations and trigger a final compute at the end. 
~~~
datapipe = data[data.age < 20]
datapipe = datapipe.groupby('income').mean()
datapipe.head(5)
~~~

~~~
        age
income
10240   16.0
11560   17.0
12960   18.0
14440   19.0

~~~
{: .output}

Chaining syntax can also be used to do the same thing, but keep readability in your code in mind.
~~~
pandasdata = (data[data.age < 20].groupby('income').mean()).compute()
Chaining syntax can also be used to do the same thing, but keep readability in your code in mind.
~~~
{: .bash}

sort operation - get the occupations with the largest people working in them
~~~
data.occupation.value_counts().nlargest(5).compute()
~~~
{: .bash}

write the output of a filter result to csv
~~~
data[data.city == 'Madison Heights'].compute().to_csv('Madison.csv')
~~~
{: .bash}

### Custom made operations.... dask.delayed

But what if you need to run your own function, or a function outside of the pandas subset that dask dataframes make available? Dask delayed is your friend. It uses ***python decorator syntax*** to convert a function into a lazy executable. The functions can then be applied to build data pipeline operations in a similar manner to what we've just encountered.

Lets explore a larger example of using dask dataframes and dask delayed functions.

In the /files directory, use your preferred editor to view the complex_system.py file. This script uses dask delayed functions that are applied to a sequence of data using pythonic ***list comprehension syntax*** . The code simualtes financial defaults in a very theoretical way, and outputs the summation of these predicted defaults. 

Please get out of ipython and the interactive session if you are using it, and submit this file in the traditional PBS script manner.


~~~
qsub complex_system.pbs

~~~

~~~
Delayed('add-c62bfd969d75abe76f3d8dcf2a9ef99c')
407.5

~~~
{: .output}


### Exercise 1 - Medium to Difficult:
The above script is a great example of dask delayed functions that are applied to lists, made in an elegant pythonic syntax. Lets try using these delayed default functions on our data of income and occupations. 

Make your own lazy function using the decorator syntax, and perform the computation you have described on a column of the data previously used in the makedata.data() helper file. For bonus points perform an aggregation on this column.

## Exercise 2 - Easy:
Given what you know of dask delayed function, please alter the file called "computepi_pawsey.py", which calculated estimates of pi without using extra parallel libraries, and alter the code with a dask delayed wrapper to make it lazy and fast 
 
### Helpful links

Dask Dataframe intro
https://docs.dask.org/en/latest/dataframe.html

API list for Dask Dataframes
https://docs.dask.org/en/latest/dataframe-api.html


___
**Notes**   
<sup id="f1">1[↩](#a1)</sup>As you should recall from the [Introduction to Artemis HPC]({{ site.sih_pages }}/training.artemis.introhpc/) course, the **scheduler** is the software that runs the cluster, allocating jobs to physical compute resources. Artemis HPC provides us with a separate 'mini-cluster' for _Training_, which has a separate PBS scheduler instance and dedicated resources.

<sup id="f2">2[↩](#a2)</sup>In fact, _Awk_ is really a shell, a programming language and a command all in one!

___
<br>