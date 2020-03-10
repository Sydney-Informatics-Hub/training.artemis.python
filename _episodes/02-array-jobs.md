---
title: "Traditional python approaches to multi cpu and nodes"
teaching: 25
exercises: 5
questions:
- "How to utilise multiple cpus and nodes on Artemis"
objectives:
- "Discover python multiprocessing and mpi execution"
keypoints:
- "load multiprocessing library to execute a function in a parallel manner"

---

## Python Multiprocessing

While the PBS resource request #PBS -l ncpus signals to the scheduler how many cpus you want your job to run, you may need more flexible ways to manage resources from within python code. This is traditionally done with the **multiprocessing** library. 

With multiprocessing, Python creates new processes. A process here can be thought of as almost a completely different program, though technically they’re usually defined as a collection of resources where the resources include memory, file handles and things like that. One way to think about it is that each process runs in its own Python interpreter, and multiprocessing farms out parts of your program to run on each process.

## Some terminology
The multiprocessing library was designed to break down the Global Interpreter Lock (GIL) that limits one thread to control python interpreter. In Python, the things that are occurring simultaneously are called by different names (thread, task, process). While they all fall under the definition of concurrency (multiple things happening anaologous to different trains of thought) - only multiprocessing actually runs these trains of thought at literally the same time. We'll only cover multiprocessing that assist in cpu bound operations - but keep in mind others exist and could be useful for IO bound operations (like reading files from the internet - files etc).

Some basic concepts in the multiprocessing library are:
1. the ```Pool(processes)``` object creates a pool of processes. processes is the number of worker processes to use (i.e python interpreters). If processes is None then the number returned by os.cpu_count() is used.
2. The ```map(function,list)``` attribute of this object uses the pool to map a defined function to a list/iterator object


To implement multiprocessing in its basic form, you can implement the below in an interactive session.

~~~
qsub -I -P Training -l ncpus=4:mem=6GB -l walltime=00:15:00
~~~

Create your basic pbs script that loads python and runs the below code

~~~

from multiprocessing import Pool

def addit(x):
        return x + 1

def main():
        print(addit(4))

        with Pool(2) as p:
                print(p.map(addit,[1,2,3]))

main()

~~~
{: .bash}

The output should be
~~~
[1, 4, 9]
~~~
{: .output}


## Useful links

https://realpython.com/python-concurrency/





<br>

___
**Notes**   
<sup id="f1">1[↩](#a1)</sup>As you should recall from the [Introduction to Artemis HPC]({{ site.sih_pages }}/training.artemis.introhpc/) course, the **scheduler** is the software that runs the cluster, allocating jobs to physical compute resources. Artemis HPC provides us with a separate 'mini-cluster' for _Training_, which has a separate PBS scheduler instance and dedicated resources.

___
<br>
