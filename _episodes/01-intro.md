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

* [Training and workshops].(https://sydney.edu.au/research/facilities/sydney-informatics-hub/workshops-and-training.html)
* [Project consulting and assisstance](https://sydney.edu.au/research/facilities/sydney-informatics-hub/project-support.html) with Statistics, Data Science, Research Engineering, Bioinformatics, Modeling/Simulation/Visualisation.
* [Research data management](https://sydney.edu.au/research/facilities/sydney-informatics-hub/digital-research-infrastructure.html) consulting and platform support.

We also aim to cultivate a **data community** at USyd, organising monthly [Hacky Hours](https://sydney.edu.au/research/facilities/sydney-informatics-hub/workshops-and-training/hacky-hour.html), outside training events (eg NVIDIA, Pawsey Center), and data/coding-related events. Look out for everthing happening on our [calander](https://sydney.edu.au/research/facilities/sydney-informatics-hub/workshops-and-training.html) or contact us to get some digital collaboration going.


# Acceleration/Paralleisation/Vectorising/Threading/make-Python-go-fast 

We will cover a few of the ways that you can potentially speed up Python. As we will learn there are multitudes of methods to make Python code more efficient, and also different implentations of libraries, tools, techniques that can all be utilised depending on how your code and/or data is organised. This is a rich and evolving ecosystem and there is no one perfect way to implement efficiencies.

Some key words that might come up:

* Vectorisation
* Multi threading/processing
* MPI message parsing interface
* GPU programming
* CPU, core, node, thread, process, worker, job, task
* Parallelisation
* MapReduce


# What does parallel mean
Seperate workers or processes exchange information and data.

### Embarrassingly parallel:
* requires no communication between processors
examples:
* running same algorithm for a range of input parameters
* rendering video frames in computer animation
* proof-of-work systems used in cryptocurrency


### Coarse-grained parallel:
Requires occasional communication between processors

### Fine-grained parallel:
Requires frequent communication between processors

examples:
* finite difference time-stepping on parallel grid
* domain decomposition modeling for finite element method


# MPI: Message Passing Interface
MPI is a standardized and portable message-passing system designed to function on a wide variety of parallel computers.
The standard defines the syntax and semantics of a core of library routines useful to a wide range of users writing portable message-passing programs in C, C++, and Fortran. There are several well-tested and efficient implementations of MPI, many of which are open-source or in the public domain.

http://openmpi.org

http://mpich.org

**MPI for Python**
mpi4py provides bindings of the MPI standard for the Python programming language, allowing any Python program to exploit multiple processors.



# Connect to Artemis

If you followed the [Setup]({{ page.root }}/setup) instructions, then you should already have the required software installed. If not, _please go do this now_!

<h2 data-toc-text="via SSH command line"> Connect via SSH in a terminal (recommended)</h2>

Fire up your **terminal emulator** and connect to Artemis HPC via **SSH**. When you use Artemis for your research, these will be your **Unikey** and **Unikey password**; however, for this training course we'll be using _training accounts_, which are:

* Username: **ict_hpctrain\<N\>**, with N from 1-20 (replace _**\<N\>**_ with your assigned number)
* Password: _will be written on the whiteboard!_

~~~
ssh -X ict_hpctrain<N>@hpc.sydney.edu.au
~~~
{: .bash}

or, if using XQuartz on a Mac

~~~
ssh -Y ict_hpctrain<N>@hpc.sydney.edu.au
~~~
{: .bash}

The ```-X``` or ```-Y``` flags tell **ssh** to enable X-forwarding, which lets GUI programs on Artemis serve you graphical windows back on your local machine.


<figure>
  <img src="{{ page.root }}/fig/01_bash.png" style="margin:10px;width:600px"/>
  <figcaption> An iTerm2 terminal window on Mac</figcaption>
</figure><br>


If connecting for the first time on this machine, you may get the following output, requesting authorisation to connect to a new **host** server:

~~~
The authenticity of host 'hpc.sydney.edu.au (10.250.96.203)' can't be established.
RSA key fingerprint is SHA256:qq9FPWBcyvvOWOMdFs8uZES0tF3SVzJsNx1cdn56GSE.
Are you sure you want to continue connecting (yes/no)?
~~~
{: .output}

Enter 'yes'. You will then be asked for your password: type it and press 'enter', and you should then be logged in!

<figure>
  <img src="{{ page.root }}/fig/01_granted.png" style="margin:10px;width:630px"/>
  <figcaption> Access granted! </figcaption>
</figure><br>


<h2 data-toc-text="via SSH GUI apps"> Connecting via an SSH GUI (common for Windows users) </h2>

If you're on Windows, and followed the [Setup]({{ page.root }}/setup) guide, then you will likely be connecting through an X-window or shell client program, like 'X-Win32' or 'PuTTY'. Following the instructions in the [Setup]({{ page.root }}/setup) guide:
* Open your installed program
* Select the "Artemis" session you configured earlier
* Click 'Launch' (X-Win32) or 'Open' (PuTTY)

If this is the first time connecting to Artemis, you will be asked to authorise it as a trusted **host** server; click 'Accept' (X-Win32) or 'Yes' (PuTTY).

<figure>
  <img src="{{ page.root }}/fig/01_xwinhosts.png" style="margin:10px;height:240px"/>
  <img src="{{ page.root }}/fig/01_puttyhosts.png" style="margin:10px;height:250px"/>
  <figcaption> Unknown host challenges: X-Win32 (top), PuTTY (bottom) </figcaption>
</figure><br>

* If using 'X-Win32', enter your **password** and once entered, a terminal window connected to Artemis should open.

* If using 'PuTTY', enter your **username**, and then your **password** in the terminal window that appears. You should now be logged in to Artemis.

<figure>
  <img src="{{ page.root }}/fig/01_xwin.png" style="margin:10px;height:220px"/>
  <img src="{{ page.root }}/fig/01_putty.png" style="margin:10px;height:350px"/>
  <figcaption> Access granted! X-Win32 (top) cuts the welcome messages, PuTTY (bottom) </figcaption>
</figure><br>

<br>


# Get the input data

We'll now retrieve the data we'll use for the examples in this course. Since it won't take very long, we'll also use this as an opportunity to demonstrate an **interactive PBS job** on Artemis, rather than the **batch** (script) jobs we performed in the [‘_Introduction to Artemis HPC_’]({{ site.sih_pages }}/training.artemis.interhpc) course.

## Interactive jobs

**Interactive** jobs give us access to a terminal window on an Artemis **compute node** -- as opposed to the **login** nodes that we have all just logged in to. Normally, we'd need to wait a while for an interactive job to start, however since we are only doing _data transfer_ operations (eg getting our input data ready) we can use the **data transfer queue (dtq)**, and hence shouldn't have to wait too long.

First, change to your project directory. Since we are using the Training account, that will be the **Training** PROJECT

~~~
cd /project/Training
~~~
{: .bash}

Now request an interactive job (```-I```): 

~~~
qsub -I -P Training 
~~~
{: .bash}

For the workflow we are currently doing you would probably want to use ```qsub -I -P Training -q dtq```, but in the training environemnt ```dtq``` is only availble to one person at a time! So we just use the default training queue (by not specifying which queue to use.)

~~~
[jdar4135@login1 Training]$ qsub -I -P Training 
qsub: waiting for job 2595948.pbsserver to start
qsub: job 2595948.pbsserver ready

[jdar4135@hpc242 ~]$v
~~~
{: .output}

The last two lines will appear when your interactive job has connected. Note that the **host** indicated at the **command prompt** has changed:

~~~
[jdar4135@hpc242 ~]$
~~~
{: .output}

You should no longer be on a **login node** but instead on one of Artemis' **compute nodes**, in this case it was **hpc242**. Note also that I've once again been moved to my **/home** directory (```~```), which you may recall is the default behaviour when logging in to an Artemis machine.

Move back in to your project folder. Remember that we were _in_ our project folders when we submitted our ```qsub``` request, so you might also remember that because we are now technically _inside_ an Artemis job, the PBS system will have defined certain **environment variables** for us. Can you think of one that might be relevant right now?

> ## Answer
>
> The PBS variable **PBS_O_WORKDIR** records the directory you were in _when you called **qsub**_.
>
> Check this with ```echo```
> ~~~
> echo $PBS_O_WORKDIR
> ~~~
> {: .bash}
>
> Now use this variable to return to your project folder.
>
> ~~~
> cd $PBS_O_WORKDIR
> ~~~
> {: .bash}
{: .solution}

Now that we have returned to our project folders, create yourself a personal directory in which to work in and change in to that directory:

~~~
mkdir hayimdata 
cd hayimdata
~~~
{: .bash}

(_You can use many shortcuts in unix, like the command above could be done in one line with ```mkdir hayimdata && cd !$```. Where ```&&``` means 'and then do' and ```!$``` is a Bash shortcut referring to the last argument of the previous command._)

Now, download and extract the data archive below:

~~~
wget https://www.dropbox.com/s/b0m31e4cj9wudx9/Automation.tar.gz

tar -zxvf Automation.tar.gz
~~~
{: .bash}

You should see quite a bit of output, hopefully resulting in a successful download and extraction.
~~~
<snip>

HTTP request sent, awaiting response... 200 OK
Length: 1810727404 (1.7G) [application/octet-stream]
Saving to: “Automation.tar.gz”

100%[===============================================>] 1,810,727,404 32.7M/s   in 53s

2018-11-15 12:25:37 (32.4 MB/s) - “Automation.tar.gz” saved [1810727404/1810727404]


<snip>

Automation/Alignment/equcab2_chr20.fasta.pac
Automation/Alignment/equcab2_chr20.fasta.fai
Automation/Alignment/canfam3_chr20.fasta.fai
Automation/Alignment/equcab2_chr20.fasta.bwt
Automation/Alignment/canfam3_chr20.fasta.ann
Automation/Alignment/FM0238_D1A03ACXX_GATCAG_R2.fastq.gz
Automation/Alignment/CMW_USCF70_D09NUACXX_R1.fastq.gz
Automation/Alignment/BD394_C7RNWACXX_ATTCCT_L001_R2.fastq.gz
Automation/Alignment/BD394_C16NWHCXX_ATTCCT_L002_R1.fastq.gz

~~~
{: .output}

Delete the archive once you've unpacked it:

~~~
rm Automation.tar.gz
~~~
{: .bash}

Finally, we can now **exit** our interactive job:

~~~
exit
~~~
{: .bash}

~~~
[jdar4135@hpc242 hayimdata]$ exit
logout

qsub: job 2595955.pbsserver completed
~~~
{: .output}

<br>
