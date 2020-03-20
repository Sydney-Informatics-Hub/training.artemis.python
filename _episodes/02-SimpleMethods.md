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

String matching report example.

# Multi-threading/processing 

python -m cProfile MYSCRIPT.py myinput1 myinpit2 myinputetc > out.txt

# MPI: Message Passing Interface
MPI is a standardized and portable message-passing system designed to function on a wide variety of parallel computers.
The standard defines the syntax and semantics of a core of library routines useful to a wide range of users writing portable message-passing programs in C, C++, and Fortran. There are several well-tested and efficient implementations of MPI, many of which are open-source or in the public domain.

http://openmpi.org

http://mpich.org

**MPI for Python**
mpi4py provides bindings of the MPI standard for the Python programming language, allowing any Python program to exploit multiple processors.


