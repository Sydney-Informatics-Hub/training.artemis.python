---
title: "Accelerating Python"
teaching: 15
exercises: 0
questions:
- "Who are the Sydney Informatics Hub?"
objectives:
- "Understand the Accelerated Python ecosystem."
keypoints:
- "Recall how to connect to Artemis HPC"
- "Recall the difference between _batch_ and _interactive_ jobs"
- "Recall the special **Data Transfer Queue**"
---
This episode introduces the [Sydney Informatics Hub](https://informatics.sydney.edu.au/), and returns us to Artemis HPC.


# The Sydney Informatics Hub

The Sydney Informatics Hub (SIH) is a _[Core Research Facility](https://sydney.edu.au/research/facilities.html)_ of the University of Sydney. Core Research Facilities centralise essential research equipment and services that would otherwise be too expensive or impractical for individual Faculties to purchase and maintain. The classic example might be the room-size electron-microscopes, built into specialised rooms in the Sydney Microscopy & Microanalysis unit.

<figure>
  <img src="{{ page.root }}/fig/01_crf.png" style="margin:10px;width:600px"/>
  <figcaption> USyd Core Research Facilities <a href="https://sydney.edu.au/research/facilities.html">https://sydney.edu.au/research/facilities.html</a></figcaption>
</figure><br>

**Artemis HPC** itself is a multi-million dollar set of equipment, a 'supercomputer', and is the main piece of equipment supported by SIH. However, we also provide a wide range of research services to aid investigators, such as:

* [Training and workshops](https://sydney.edu.au/research/facilities/sydney-informatics-hub/workshops-and-training.html)
* [Project consulting and assisstance](https://sydney.edu.au/research/facilities/sydney-informatics-hub/project-support.html) with Statistics, Data Science, Research Engineering, Bioinformatics, Modeling/Simulation/Visualisation.
* [Research data management](https://sydney.edu.au/research/facilities/sydney-informatics-hub/digital-research-infrastructure.html) consulting and platform support.

We also aim to cultivate a **data community** at USyd, organising monthly [Hacky Hours](https://sydney.edu.au/research/facilities/sydney-informatics-hub/workshops-and-training/hacky-hour.html), outside training events (eg NVIDIA, Pawsey Center), and data/coding-related events. Look out for everthing happening on our [calander](https://sydney.edu.au/research/facilities/sydney-informatics-hub/workshops-and-training.html) or contact us to get some digital collaboration going.

<br>
# Acceleration, Paralleisation, Vectorising, Threading, make-Python-go-fast 

We will cover a few of the ways that you can potentially speed up Python. As we will learn there are multitudes of methods to make Python code more efficient, and also different implentations of libraries, tools, techniques that can all be utilised depending on how your code and/or data is organised. This is a rich and evolving ecosystem and there is no one perfect way to implement efficiencies.

Some key words that might come up:

* Vectorisation
* MPI message parsing interface
* CPU, core, node, thread, process, worker, job, task
* Parallelisation
* Python decorators and functional programming.


<br>
# Course pre-requisites
You should have some experience with Python. You should be able to connect to a remote computer (i.e. Artemis) via ssh and submit a job to a scheduler.


<br>
# What does *parallel* mean?
Seperate workers or processes exchanging information and data. You could classify different parallel workflows into different categories, which can make you think about how to write your code and what approaches to take.

### Embarrassingly parallel:
Requires no communication between processors

* running same algorithm for a range of input parameters
* rendering video frames in computer animation
* proof-of-work systems used in cryptocurrency

### Coarse/Fine-grained parallel:
Requires occasional or frequent communication between processors

* finite difference time-stepping on parallel grid
* domain decomposition modeling for finite element method

Let's get started with some examples....


