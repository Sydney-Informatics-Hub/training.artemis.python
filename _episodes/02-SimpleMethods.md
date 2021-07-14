---
title: "2. Simple methods"
teaching: 10
exercises: 15
questions:
- "Which method for acceleration should I choose?"
objectives:
- "Learn simples methods to profile your code"
- "See how numpy and pandas use Vectorising to improve perfomance for some data"
- "Use MPI to communicate between workers"
keypoints:
- "Understand there are different ways to accelerate"
- "The best method depends on your algorithms, code and data"
---
This episode shows you a few of the basic tools that we can use in Python to make our code go faster. There is no perfect method for optimising code. Efficiency gains depend on what your end goal is, what libraries are available, what method or approach you want to take when writing algorithms, what your data is like, what hardware you have. Hopefully these notes will allow you to think about your problems from different perspectives to give you the best opportunity to make your development and execution as efficient as possible.


# Profiling your code

Before you get stuck into making things fast, it is important to find out what is exactly slow in your code. Is it a particular function running slow? Or are you calling a really fast function a million times? You can save yourself a lot development time by profiling your code to give you an idea for where efficiencies can be found. Let's profile a simple Python script and then think about how we could make it faster.

Put this code in a script (or download from [here](https://sydney-informatics-hub.github.io/training.artemis.python/files/faster.py)):
~~~
#A test function to see how you can profile code for speedups

import time

def waithere():
	print("waiting for 1 second")
	time.sleep(1)

def add2(a=0,b=0):
	print("adding", a, "and", b)
	return(a+b)

def main():
	print("Hello, try timing some parts of this code!")
	waithere()	
	add2(4,7)
	add2(3,1)


if __name__=='__main__':
	main()
~~~
{: .python}

There are several ways to debug and profile Python, a very elegant and built in one is [cProfile](https://docs.python.org/3/library/profile.html)
It analyses your code as it executes. Run it with ```python -m cProfile faster.py```  and see the output of the script and the profiling:

~~~
Hello, try timing some parts of this code!
waiting for 1 second
adding 4 and 7
adding 3 and 1

12 function calls in 1.008 seconds

Ordered by: standard name

ncalls  tottime  percall  cumtime  percall filename:lineno(function)
1    0.000    0.000    1.008    1.008 faster.py:12(main)
1    0.000    0.000    1.008    1.008 faster.py:2(<module>)
1    0.000    0.000    1.002    1.002 faster.py:4(waithere)
2    0.000    0.000    0.005    0.003 faster.py:8(add2)
1    0.000    0.000    1.008    1.008 {built-in method builtins.exec}
4    0.007    0.002    0.007    0.002 {built-in method builtins.print}
1    1.001    1.001    1.001    1.001 {built-in method time.sleep}
1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
~~~
{: .output}

You can now interrogate your code and see where you should devote your time to improving it.

Special note on style: Developing python software in a ***modular*** manner assists with debugging and time profiling.


# Loops and vectorising code with numpy and pandas

Your problem might be solved by using the fast way certain packages handle certain datatypes. 

Generally speaking, pandas and numpy libraries should be libraries you frequently use. They offer advantages in high performance computing including:
1. Efficient datastructures that under the hood are implemented in fast C code rather than python.
2. Promoting explicit use of datatype declarations - making memory management of data and functions working on this data, faster.
3. Elegant Syntax promoting consise behaviour. 
4. Data structures come with common built in functions that are designed to be used in a vectorised way.

Lets explore this last point on vectorisation with an example. Take this nested for loop [example](https://sydney-informatics-hub.github.io/training.artemis.python/files/vector.py):

~~~
#import packages
import numpy as np
import pandas as pd
import time 

#Create some data to work with
AllPubs = pd.DataFrame(np.random.randint(0,100,size=(100, 4)), columns=['user','publication','id','other'])
Users = pd.DataFrame(np.random.randint(0,100,size=(50, 1)), columns=['id'])

#This could perhaps be the id of a person and the list of publications they have made. 
#You want to match up their publications with some other list, 
#perhaps the publications they made by using Artemis.

#Create an emtpy dataframe to fill with the resulting matches
totalSlow=pd.DataFrame(columns=AllPubs.columns)
totalFast=pd.DataFrame(columns=AllPubs.columns)

~~~
{: .python}

Now compare the nested for-loop method:
~~~
tic=time.time()
for index,pub in AllPubs.iterrows():
  for index2,user in Users.iterrows():
    if user['id']==pub['id']:
      totalSlow=totalSlow.append(pub,ignore_index=True)
      
totalSlow=totalSlow.drop_duplicates()
toc=time.time()
print("Runtime:",toc-tic, "seconds")

~~~
{: .python}

Or the vectorised method:
~~~
tic=time.time()
totalFast=AllPubs[AllPubs['id'].isin(Users.id.tolist())]
totalFast=totalFast.drop_duplicates()
toc=time.time()
print("Runtime:",toc-tic, "seconds")

~~~
{: .python}


Which one is faster? Note the use of some really basic timing functions, these can help you understand the speed of your code.


# A note on environments suited for python

Using conda environments help with porting code developed locally to high performance computing. It ensures you are running code using the same packages and versions in different places. The Setup page describes how to create an environment using packages specified in a yml file. Some other basic commands are:

~~~
conda activate <environment_name>
conda deactivate
conda env list 
conda env remove --name <environment_name>
~~~
{: .python}

Moreover, there is an increasing trend, particularly when investigating data and exploring things interactively, to use jupyter notebook (this has been installed in the conda environment you are using). We'll go through the basics of the two, and for the remander of the training where code has been run locally, we will illustrate using jupyter notebook.



Let's now get stuck into some more specific use-cases and tools to use.

