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


## Useful links

https://realpython.com/python-concurrency/


~~~
#PBS -J 1-100
~~~
{: .bash}

~~~
#PBS -J 4-25:3
~~~
{: .bash}

The first example **directive** above will launch 100 copies of the job defined in the **PBS script**, with each copy having an **array index** equal to an integer from _1 to 100_. The second example will launch 8 copies of the job defined in the script, with each copy having an **array index** equal to an integer from _4 to 25_ in steps of _3_.

In general, the argument to the ```-J``` directive is ```i-f[:s]``` for non-negative integers _i<f, s_, which creates rising indices starting from **initial value _i_**, to a **maximum final value _f_**, in **steps of _s_**. The step argument is optional, and defaults to _1_.

A quick and pointless example of an array job could be submitting the following script, **hello_array.sh**

~~~
#!/bin/bash
echo "Hello, world. I am job $PBS_ARRAY_INDEX"
~~~
{: .bash}

via

~~~
qsub -P Training -J 1-3 hello_array.sh
~~~
{: .bash}

This would simply submit 3 jobs in an array, with each echoing the string _"Hello, world! I am number $PBS_ARRAY_INDEX."_

Job arrays are marked in the job queue with a '**B**' for 'batch' rather than an '**R**' when running. They also have slightly different **JobID**s than single jobs, in that their ID numbers are followed by a pair of square brackets **[]**. Ie, an array job might have JobID ```1578913[]```, where as a single job would have JobID ```1578913```. This is important, and the **[]** _must be included_ when identifying the job to a command such as ```qstat```.

<br>   


## Submitting an array job

Let's now practice submitting an actual array job. Move into the data directory we extracted earlier, and then into the **Demo-a** folders

~~~
cd Automation/Demo-a
~~~
{: .bash}

Open the script **array_demo.pbs** in ```nano``` or your preferred editors

~~~
nano array_demo.pbs
~~~
{: .bash}

<figure>
  <img src="{{ page.root }}/fig/02_demo_a.png" style="margin:00px;width:900px"/>
  <figcaption> <b>array_demo.pbs</b> in nano </figcaption>
</figure><br>

Make any required edits necessary to run this script, and then submit it via ```qsub```.

> ## Change #1
> Specify your **project**.
>
> Use the ```-P``` PBS directive to specify the _**Training**_ project, using its _short name_.
> ~~~
> #PBS -P Training
> ~~~
> {: .bash}
{: .solution}

> ## Change #2
> Give your job a **name**
>
> Use the ```-N``` PBS directive to give your job an easily identifiable name. You might run **lots** of jobs at the same time, so you want to be able to keep track of them!
>
> ~~~
> #PBS -N DemoHayA
> ~~~
> {: .bash}
> Substitute a job name of your choice!
{: .solution}

> ## Change #3
> Tailor your **resource** requests.
>
> Use the ```-l``` PBS directive to request appropriate compute **resources** and **walltime** for your job.
>
> This script isn't doing much! Leave as is, with **1 minute** of walltime, **1 GB** of RAM (the minimum on Artemis), and **1 CPU**.
> ~~~
> #PBS -l select=1:ncpus=1:mem=1GB
> #PBS -l walltime=00:01:00
> ~~~
> {: .bash}
{: .solution}

> ## Change #4
> If you are using the **Training Scheduler**<sup id="a1">[1](#f1)</sup> then you do not have access to all the queues. You can submit jobs to **defaultQ** and **dtq** _only_.
>
> In the _normal Artemis environment_ you can submit to **defaultQ**, **dtq**, **small-express**, **scavenger**, and possibly some strategic allocation queues you may have access to.
> ~~~
> #PBS -q defaultQ
> ~~~
> {: .bash}
{: .solution}

> ## Change #5 (optional)
> Set up **email notifications** for your job.
>
> Use the ```-M``` and ```-m``` PBS directive to specify a destination email address, and the events you wish to be notified about. You can receive notifications for when your job **(b)**egins, **(e)**nds or **(a)**borts.
> ~~~
> #PBS -M hayim.dar@sydney.edu.au
> #PBS -m abe
> ~~~
> {: .bash}
{: .solution}
<br>

What will this job do? Firstly, note the array job directive initialising 10 copies of this job, with indices 1,2,...,10:

~~~
#PBS -J 1-10
~~~
{: .bash}


Secondly, can you see how each copy of this job will execute differently? How is this achieved?

> ## Answer
> With the ```PBS_ARRAY_INDEX``` variable.
>
> Commands in the **array_demo.pbs** script refer to the job's **array index** to differentiate between the copies.
> ~~~
> mkdir $PBS_O_WORKDIR/Demo$PBS_ARRAY_INDEX
> ~~~
> {: .bash}
> ~~~
> cd $PBS_O_WORKDIR/Demo$PBS_ARRAY_INDEX
> ~~~
> {: .bash}
> ~~~
> printf "This is job number $PBS_ARRAY_INDEX in the array\n" > job$PBS_ARRAY_INDEX.log
> ~~~
> {: .bash}
{: .solution}

Submit the job and then take a look at your job status:

~~~
qstat -u $USER
~~~
{: .bash}

~~~
[jdar4135@login3 Demo-a]$ qstat -u $USER

pbsserver:
                                                            Req'd  Req'd   Elap
Job ID          Username Queue    Jobname    SessID NDS TSK Memory Time  S Time
--------------- -------- -------- ---------- ------ --- --- ------ ----- - -----
2596901[].pbsse jdar4135 small-ex DemoHayA      --    1   1    1gb 00:00 Q   --
~~~
{: .output}

What do you notice? What would be the output of this command? (user _your_ jobID!)

~~~
qstat 596901
~~~
{: .bash}

~~~
[jdar4135@login3 Demo-a]$ qstat 2596901
qstat: Unknown Job Id 596901.pbsserver
~~~
{: .output}

<br>
### qstat for array jobs

What went wrong? We left off the ```[]```. Now query all **subjobs** in your array with ```qstat -t```.

~~~
qstat -t 596901[]
~~~
{: .bash}

~~~
[jdar4135@login3 Demo-a]$ qstat -t 2596901[]
Job id            Name             User              Time Use S Queue
----------------  ---------------- ----------------  -------- - -----
2596901[].pbsserv DemoHayA         jdar4135                 0 B small-express
2596901[1].pbsser DemoHayA         jdar4135                 0 X small-express
2596901[2].pbsser DemoHayA         jdar4135                 0 X small-express
2596901[3].pbsser DemoHayA         jdar4135                 0 X small-express
2596901[4].pbsser DemoHayA         jdar4135                 0 X small-express
2596901[5].pbsser DemoHayA         jdar4135                 0 X small-express
2596901[6].pbsser DemoHayA         jdar4135                 0 X small-express
2596901[7].pbsser DemoHayA         jdar4135                 0 X small-express
2596901[8].pbsser DemoHayA         jdar4135                 0 X small-express
2596901[9].pbsser DemoHayA         jdar4135          00:00:00 R small-express
2596901[10].pbsse DemoHayA         jdar4135          00:00:00 R small-express
~~~
{: .output}

Note that for _subjobs_ of an array, complete status is not indicated by '**F**' but '**X**', meaning 'exited'. You can query individual subjobs by their specific index within the array:

~~~
qstat -t 2596901[4]
~~~
{: .bash}

~~~
jdar4135@login3 Demo-a]$ qstat -t 2596901[4]
Job id            Name             User              Time Use S Queue
----------------  ---------------- ----------------  -------- - -----
2596901[4].pbsser DemoHayA         jdar4135                 0 X small-express
~~~
{: .bash}

<br>
As usual, once your array job has completed, have a look to see that it ran as expected:
- Check for errors:

~~~
cat DemoHayA.e*
~~~
{: .bash}

- Check for _Exit Status_ of 0:

~~~
grep -se "Exit Status" *
~~~
{: .bash}

- Check for log files:

~~~
[jdar4135@login1 Demo-a]$ ls
array_demo.pbs  DemoHayA.e2596901.1   DemoHayA.o2596901.10        DemoHayA.o2596901.6
Demo1           DemoHayA.e2596901.10  DemoHayA.o2596901.10_usage  DemoHayA.o2596901.6_usage
Demo10          DemoHayA.e2596901.2   DemoHayA.o2596901.1_usage   DemoHayA.o2596901.7
Demo2           DemoHayA.e2596901.3   DemoHayA.o2596901.2         DemoHayA.o2596901.7_usage
Demo3           DemoHayA.e2596901.4   DemoHayA.o2596901.2_usage   DemoHayA.o2596901.8
Demo4           DemoHayA.e2596901.5   DemoHayA.o2596901.3         DemoHayA.o2596901.8_usage
Demo5           DemoHayA.e2596901.6   DemoHayA.o2596901.3_usage   DemoHayA.o2596901.9
Demo6           DemoHayA.e2596901.7   DemoHayA.o2596901.4         DemoHayA.o2596901.9_usage
Demo7           DemoHayA.e2596901.8   DemoHayA.o2596901.4_usage
Demo8           DemoHayA.e2596901.9   DemoHayA.o2596901.5
Demo9           DemoHayA.o2596901.1   DemoHayA.o2596901.5_usage
~~~
{: .output}

Note how each **_subjob_** has its own set of log files, indicated by the **_.N_** suffix corresponding to its **array index**.

- Check to see if expected files were created:

~~~
ls Demo*

cat Demo*/*.log
~~~
{: .bash}

~~~
[jdar4135@login1 Demo-a]$ ls Demo*/*.log
Demo10/job10.log  Demo1/job1.log  Demo2/job2.log  Demo3/job3.log  Demo4/job4.log  Demo5/job5.log  Demo6/job6.log  Demo7/job7.log  Demo8/job8.log  Demo9/job9.log

[jdar4135@login1 Demo-a]$ cat Demo*/*.log
This is job number 10 in the array
This is job number 1 in the array
This is job number 2 in the array
This is job number 3 in the array
This is job number 4 in the array
This is job number 5 in the array
This is job number 6 in the array
This is job number 7 in the array
This is job number 8 in the array
This is job number 9 in the array
~~~
{: .output}

<br><br>
## Another array job!

We're going to try that again, but with a small modification. The outputs from our first array job were all _**piped**_ to different locations, using the ```>``` redirect

~~~
printf "This is job number $PBS_ARRAY_INDEX in the array\n" > job$PBS_ARRAY_INDEX.log
~~~
{: .bash}


This meant that the standard output log files generated by the PBS Scheduler were _empty_, since the output of ```printf``` was sent elsewhere. However, we can customise PBS log files directly using the subjob array indices themselves.

Head to the **Demo-b** folder and open the **array_demo.pbs** script
~~~
cd ../Automation/Demo-b
~~~
{: .bash}

~~~
nano array_demo.pbs
~~~
{: .bash}

<figure>
  <img src="{{ page.root }}/fig/02_demo_b.png" style="margin:00px;width:900px"/>
  <figcaption> <b>array_demo.pbs</b> in nano </figcaption>
</figure><br>

Make any required edits necessary to run this script, and then submit it via ```qsub```.

> ## Change #1
> Specify your **project**.
>
> Use the ```-P``` PBS directive to specify the _**Training**_ project, using its _short name_.
> ~~~
> #PBS -P Training
> ~~~
> {: .bash}
{: .solution}

> ## Change #2
> Give your job a **name**
>
> Use the ```-N``` PBS directive to give your job an easily identifiable name. You might run **lots** of jobs at the same time, so you want to be able to keep track of them!
>
> ~~~
> #PBS -N DemoHayB
> ~~~
> {: .bash}
> Substitute a job name of your choice!
{: .solution}

> ## Change #3
> Tailor your **resource** requests.
>
> Use the ```-l``` PBS directive to request appropriate compute **resources** and **wall-time** for your job.
>
> This script isn't doing much! Leave as is, with **1 minute** of walltime, **1 GB** RAM (the minimum on Artemis), and **1 CPU**.
> ~~~
> #PBS -l select=1:ncpus=1:mem=1GB
> #PBS -l walltime=00:01:00
> ~~~
> {: .bash}
{: .solution}

> ## Change #4
> If you are using the **Training Scheduler**<sup id="a1">[1](#f1)</sup> then you do not have access to all the queues. You can submit jobs to **defaultQ** and **dtq** _only_.
>
> In the _normal Artemis environment_ you can submit to **defaultQ**, **dtq**, **small-express**, **scavenger**, and possibly some strategic allocation queues you may have access to.
> ~~~
> #PBS -q defaultQ
> ~~~
> {: .bash}
{: .solution}

> ## Change #5 (optional)
> Set up **email notifications** for your job.
>
> Use the ```-M``` and ```-m``` PBS directive to specify a destination email address, and the events you wish to be notified about. You can receive notifications for when your job **(b)**egins, **(e)**nds or **(a)**borts.
> ~~~
> #PBS -M hayim.dar@sydney.edu.au
> #PBS -m abe
> ~~~
> {: .bash}
{: .solution}
<br>

What has changed compared to the last job we ran?

> ## Answer
> The ```PBS_ARRAY_INDEX``` variable has been used to modify the naming of the PBS job logs, and the job output is now printed to the output log file, instead of being piped to a new file as in the previous example.
>
> Commands in the **array_demo.pbs** script refer to the job's **array index** to differentiate between the copies.
> ~~~
> #PBS -o Demo^array_index^/stdout
> #PBS -e Demo^array_index^/stderr
> ~~~
> {: .bash}
> ~~~
> printf "This is job number $PBS_ARRAY_INDEX in the array\n"
> ~~~
> {: .bash}
{: .solution}

Submit the job and then monitor it with ```qstat -t```:

~~~
qstat -t 2597100[]
~~~
{: .bash}

~~~
[jdar4135@login1 Demo-b]$ qstat -t 2597100[]
Job id            Name             User              Time Use S Queue
----------------  ---------------- ----------------  -------- - -----
2597100[].pbsserv DemoHayB         jdar4135                 0 B small-express
2597100[1].pbsser DemoHayB         jdar4135          00:00:00 R small-express
2597100[2].pbsser DemoHayB         jdar4135          00:00:00 R small-express
2597100[3].pbsser DemoHayB         jdar4135          00:00:00 R small-express
2597100[4].pbsser DemoHayB         jdar4135          00:00:00 R small-express
2597100[5].pbsser DemoHayB         jdar4135          00:00:00 R small-express
2597100[6].pbsser DemoHayB         jdar4135          00:00:00 R small-express
2597100[7].pbsser DemoHayB         jdar4135          00:00:00 R small-express
2597100[8].pbsser DemoHayB         jdar4135          00:00:00 R small-express
2597100[9].pbsser DemoHayB         jdar4135          00:00:00 R small-express
2597100[10].pbsse DemoHayB         jdar4135          00:00:00 R small-express
~~~
{: .output}

Practis checking an individual subjob:

~~~
[jdar4135@login1 Demo-b]$ qstat -t 2597100[7]
Job id            Name             User              Time Use S Queue
----------------  ---------------- ----------------  -------- - -----
2597100[7].pbsser DemoHayB         jdar4135          00:00:00 R small-express
~~~
{: .output}

<br>
### The array_state_count

Information about subjobs can also be found in the job's 'full display' ```-f``` status report:

~~~
qstat -f 2597100[] | grep array
~~~
{: .bash}

~~~
[jdar4135@login1 Demo-b]$ qstat -f 2597100[] | grep array
    Submit_arguments = array_demo.pbs
    array = True
    array_state_count = Queued:0 Running:10 Exiting:0 Expired:0
    array_indices_submitted = 1-10
~~~
{: .output}

The **array_state_count** entry is an especially useful way to see how many of your subjobs are in which state.

<br>
Once your jobs are done, check for errors -- but wait, there are no log files!

~~~
[jdar4135@login1 Demo-b]$ ls
array_demo.pbs  Demo1  Demo10  Demo2  Demo3  Demo4  Demo5  Demo6  Demo7  Demo8  Demo9
~~~
{: .output}

But there _are_ three files in each of the new folders created:
~~~
[jdar4135@login1 Demo-b]$ ls Demo2
stderr  stdout  stdout_usage
~~~
{: .output}

And they are in fact the PBS log files, just named and saved differently.

~~~
cat Demo2/*
~~~
{: .bash}

~~~
[jdar4135@login1 Demo-b]$ cat Demo2/*
This is job number 2 in the array
-- Job Summary -------------------------------------------------------
Job Id: 2597100[2].pbsserver for user jdar4135 in queue small-express
Job Name: DemoHayB
Project: RDS-CORE-Training-RW
Exit Status: 0
Job run as chunks (hpc056:ncpus=1:mem=1048576kb)
Array id: 2597100[].pbsserver array index: 2
Walltime requested:   00:00:20 :      Walltime used:   00:00:03
                               :   walltime percent:      15.0%
-- Nodes Summary -----------------------------------------------------
-- node hpc056 summary
    Cpus requested:          1 :          Cpus Used:    unknown
          Cpu Time:    unknown :        Cpu percent:    unknown
     Mem requested:      1.0GB :           Mem used:    unknown
                               :        Mem percent:    unknown

-- WARNINGS ----------------------------------------------------------

** Low Walltime utilisation.  While this may be normal, it may help to check the
** following:
**   Did the job parameters specify more walltime than necessary? Requesting
**   lower walltime could help your job to start sooner.
**   Did your analysis complete as expected or did it crash before completing?
**   Did the application run more quickly than it should have? Is this analysis
**   the one you intended to run?
**
-- End of Job Summary ------------------------------------------------
~~~
{: .output}

Or to see all the outputs:

~~~
cat Demo*/stdout
~~~
{: .bash}

~~~
[jdar4135@login1 Demo-b]$ cat Demo*/stdout
This is job number 10 in the array
This is job number 1 in the array
This is job number 2 in the array
This is job number 3 in the array
This is job number 4 in the array
This is job number 5 in the array
This is job number 6 in the array
This is job number 7 in the array
This is job number 8 in the array
This is job number 9 in the array
~~~
{: .output}

<br>
The PBS directives ```-o``` and ```-e``` in our **array_demo.pbs** script told the scheduler how to name our standard output and standard error logs:

~~~
#PBS -o Demo^array_index^/stdout
#PBS -e Demo^array_index^/stderr
~~~
{: .bash}

In our case, we saved log files called **stdout** and **stderr** in folders called **Demo****_N_**, where **_N_** is the **array index** of the job. We didn't use the _Bash_ variable syntax ```$PBS_ARRAY_INDEX``` because these lines are _PBS directives **not** Bash commands_. Instead, PBS has a special syntax of it's own ```^array_index^``` for referring to the index of each array subjob inside a _directive_.


<br>

___
**Notes**   
<sup id="f1">1[↩](#a1)</sup>As you should recall from the [Introduction to Artemis HPC]({{ site.sih_pages }}/training.artemis.introhpc/) course, the **scheduler** is the software that runs the cluster, allocating jobs to physical compute resources. Artemis HPC provides us with a separate 'mini-cluster' for _Training_, which has a separate PBS scheduler instance and dedicated resources.

___
<br>
