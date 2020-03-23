---
title: "Simple methods"
teaching: 15
exercises: 0
questions:
- "Which method for accelration should I choose?"
objectives:
- "Learn about the speed differences in Loops, Iterators, and Generators"
- "See how numpy and pandas use Vectorising to improve perfomance for some data"
- "Multithreading"
- "Use MPI to communicate betwwen workers"
keypoints:
- "Understand there are different ways to accelerate"
- "The best method depends on your algorithms, code and data"
---
This episode shows you a few of the basic tools that we can use in Python to make our code go faster.

# Loops and Iterators
The simplest thing in python to make fast is perhaps a loop where each exuction of the loop is indepent of everything else, fo example.

groc_list=[banana,apple,orange]
for item in groceries1
groc_comp = [expression(i) for i in old_list if filter(i)]

# Vectorising code with numpy and pandas

Your problem might be solved by using the fast way certain packages handle certain datatypes. Often called vectorizing. Take this nested for loop example:

~~~
#import packages
import numpy as np
import pandas as pd
import time 

#Create some data to work with
AllPubs = pd.DataFrame(np.random.randint(0,100,size=(100, 4)), columns=list('ABCD'))
Users = pd.DataFrame(np.random.randint(0,100,size=(50, 1)), columns=list('A'))

#This could perhaps be the id of a person and the list of publications they have made. You want to match up their publications with some other list, perhaps the publications they made by using Artemis.

#Create an emtpy dataframe to fill with the resulting matches
totalSlow=pd.DataFrame(columns=AllPubs.columns)
totalFast=pd.DataFrame(columns=AllPubs.columns)

~~~
.{python}

Now compare the nested for-loop method:
~~~
tic=time.time()
for index,pub in AllPubs.iterrows():
  for index2,user in Users.iterrows():
    if user['A']==pub['A']:
      totalSlow=totalSlow.append(pub,ignore_index=True)
      
totalSlow=totalSlow.drop_duplicates()
toc=time.time()
print("Runtime:",toc-tic, "seconds")

~~~
.{python}

Or the vectorised method:
~~~
tic=time.time()
totalFast=AllPubs[AllPubs['A'].isin(Users.A.tolist())]
totalFast=totalSlow.drop_duplicates()
toc=time.time()
print("Runtime:",toc-tic, "seconds")

~~~
.{python}


Which one is faster? 


# Multi-threading/processing 

python -m cProfile MYSCRIPT.py myinput1 myinpit2 myinputetc > out.txt

# MPI: Message Passing Interface
MPI is a standardized and portable message-passing system designed to function on a wide variety of parallel computers.
The standard defines the syntax and semantics of a core of library routines useful to a wide range of users writing portable message-passing programs in C, C++, and Fortran. There are several well-tested and efficient implementations of MPI, many of which are open-source or in the public domain.

http://openmpi.org

http://mpich.org

**MPI for Python**
mpi4py provides bindings of the MPI standard for the Python programming language, allowing any Python program to exploit multiple processors.


