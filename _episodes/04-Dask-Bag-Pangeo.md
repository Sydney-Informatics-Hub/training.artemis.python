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



### Some links:

on dask fundamentals
https://docs.dask.org/en/latest/bag.html

on limitations
https://docs.dask.org/en/latest/shared.html


## The PBS **job name** as a variable

In the previous Episode we used the **PBS_ARRAY_INDEX** variable as a tool to run slightly different jobs using the same **PBS script**. This works because the array index of a subjob is an _arbitrary_ variable that nothing in the PBS execution process depends on, so we can use it for our needs.

Another such arbitrary variable is the **PBS job name**, the human-readable name we have been giving our jobs so that we could more easily keep track of them. The key here is that we don't need to set the job name via a **PBS directive** in a PBS script -- we can also just _pass it_ to PBS as an an **option** in the call to ```qsub```.

To see this in action, navigate to the **Povray** directory, and open **single_image.pbs** in your preferred editor.

~~~
cd ../Povray

nano single_image.pbs
~~~
{: .bash}

<figure>
  <img src="{{ page.root }}/fig/04_povray.png" style="margin:00px;width:600px"/>
  <figcaption> <b>single_image.pbs</b> in nano </figcaption>
</figure><br>

Note that there is no job name directive in this script -- no ```#PBS -N Name``` line. However, the script does refer to a variable ```PBS_JOBNAME```, which PBS creates for us when we run ```qsub```. This script enters a folder called **$PBS_JOBNAME** and then uses the file **$PBS_JOBNAME.pov** as input.

What values will **$PBS_JOBNAME** need to take?

> ## Answer
> ~~~
> ls */*.pov
> ~~~
> {: .bash}
>
> ~~~
> castle/castle.pov      glass/glass.pov              plants/plants.pov
> escargot/escargot.pov  plants/exgrass3.pov          snow/snow.pov
> fridge/fridge.pov      plants/plants_demo_pano.pov
> ~~~
> {: .output}
>
> The values **$PBS_JOBNAME** should take are: _castle, escargot, fridge, glass, plants_, and _snow_.
>
> What will happen if we included _exgrass3_ or _plants_demo_pano_?
{: .solution}

Make any needed changes to the **single_image.pbs** script, and submit it with the first value for name above, passed to the ```qsub``` command with ```-N```:

~~~
qsub -N castle single_image.pbs
~~~
{: .bash}

Monitor your job as usual, and when it is done check that it was successful. In addition to the log files and **Exit Status: 0**, there should now be a **.png** image file created in the **castle** directory.

If you have enabled x-window forwarding (ie you used ```ssh -X``` or _X-Win32_ on _Windows_), then you should be able to display the image. Use the ```display``` command from the _ImageMagick_ image processing suite:

~~~
module load imagemagick
display castle/castle.png &
~~~
{: .bash}

<figure>
  <img src="{{ page.root }}/fig/04_castle.png" style="margin:10px;width:300px"/>
  <figcaption> The image <b>castle.png</b> displayed in <i>ImageMagick</i> and served by <i>XQuartz</i></figcaption>
</figure><br>

### _FOR_ loops

Now that we have convinced ourselves this works, we really don't have to manually run ```qsub``` for each image we wish to render. In fact, we can very easily write a _Bash_ script to loop over the images and run ```qsub``` for us.

An example of looping in _Bash_ can be seen in the script **loop.sh**. Display its contents:

~~~
cat loop.sh
~~~
{: .bash}

~~~
[jdar4135@login2 Povray]$ cat loop.sh
#! /bin/bash

# Iterate over list of words/strings
words=(One Two Three '4 on the floor')
for i in "${words[@]}"
do
	echo The string is $i
done
printf "\n"

# Iterate over range of letters
for i in {a..e}
do
   echo The letter is $i
done
printf "\n"

# Iterate over range of numbers
for i in {1..4}
do
   echo The number is $i
done
printf "\n"

# Iterate over range of non-sequential numbers
weeks=(2 3 {5..10})
for i in "${weeks[@]}"
do
	echo The week is $i
done
printf "\n"

# Iterate over range of floating point numbers between 2 and 3 with a step value of 0.1
for i in $(seq 2.0 0.1 3.0)
do
	echo The decimal is $i
done
printf "\n"
~~~
{: .output}

Run this script with ```bash```. What does it do? Note the syntax of each loop construct:

~~~
for VAR in VAR_LIST
do
  FOO BAR
done
~~~
{: .bash}

This is why we call these '**_FOR_**' loops; they iterate over a list of variables, once for each value the variable can take.

<br>
Now, let's return to our **single_image.pbs** script. Can you write a _FOR_ loop to submit a job for each of the images we identified earlier?

> ## Answer
> ~~~
> #!/bin/bash
>
> images=(castle escargot fridge glass plants snow)
>
> for image in ${images[@]}
> do
>     qsub -N $image single_image.pbs
> done
> ~~~
> {: .bash}
>
> Did you remember to include the ```#!/bin/bash``` '_hashbang_' to let the OS know what language your script is in?
{: .solution}

In the examples above, a couple of different _Bash_ elements are used. There is the 'brace' sequence expansion ```{i..f..s}```; there is the use of the ```seq``` function inside a command substitution  ```$(..)```; and there is the use of a _Bash_ _array_ ```VAR=(A B C ..)```.

<br>
Have a look at the script **povray.sh**. Does the loop there match the one you wrote? What _Bash_ features are used?

~~~
cat povray.sh
~~~
{: .bash}

Make any required changes to **povray.sh** and run it with ```bash povray.sh```. This runs ```qsub``` with each iteration of the loop and submits all the jobs. When the job has completed, view each of the images to make sure they were rendered.

Note that the syntax used to retrieve elements from a _Bash array_ of length **_N_** is ```${VAR[i]}``` where **_i_** runs from _**0**_ to _**N-1**_ (ie, _Bash_ uses 'zero-indexing'). The entire array can be accessed with ```${VAR[*]}```, and an _m_ element range starting with the _j_-th by ```${VAR[*]:j:m}```.<sup id="a1">[1](#f1)</sup>

<br>
### Indexing _Bash_ arrays

_FOR_ loops are very handy, and generally efficient structures in low-level languages like _Bash_. However, for submitting a large number of jobs to the **PBS Scheduler** an **array job** is _preferable_. This is because it's less work for the scheduler to manage; it's also easier to keep track of for you!

Have a look again at **single_image.pbs**:

~~~
cat single_image.pbs
~~~
{: .bash}

~~~
[jdar4135@login2 Povray]$ cat single_image.pbs
#!/bin/bash

#PBS -P Training
#PBS -l select=1:ncpus=1:mem=1gb
#PBS -l walltime=0:05:00
#PBS -q small-express

module load povray

cd $PBS_O_WORKDIR/$PBS_JOBNAME
povray res $PBS_JOBNAME.pov
~~~
{: .output}

How could you adapt this script to run as an **array job**? What could you replace ```$PBS_JOBNAME``` with?   
How would you set it for each **array index**?   
(Hint: Look back at the _FOR_ loop you wrote above)

<br>
> ## Extra hint (only if you're stuck!)
> What would the following _Bash_ script do?
> ~~~
> #!/bin/bash
> images=(castle escargot fridge glass plants snow)
>
> for i in {0..5}
> do
>     echo ${images[i]}
> done
> ~~~
> {: .bash}
{: .solution}

> ## Answer
> ~~~
> #!/bin/bash
>
> #PBS -P Training
> #PBS -l select=1:ncpus=1:mem=1gb
> #PBS -l walltime=0:05:00
> #PBS -q small-express
> #PBS -J 0-5
>
> module load povray
>
> images=(castle escargot fridge glass plants snow)
>
> image=${images[$PBS_ARRAY_INDEX]}
>
> cd $PBS_O_WORKDIR/$image
> povray res $image.pov
> ~~~
> {: .bash}
>
> Did you remember that _Bash_ arrays index from 0?
{: .solution}

<br>
Aside: If you had *lots* of images in named directories to process, you could use **globbing** or the ```find``` function to get a list of directory names to make your '**images**' _Bash_ array variable. Eg, with globbing (using __*__ wildcards) you could write:
~~~
images=(`echo */ | xargs -n1 basename`)
~~~
{: .bash}

The ```*/``` wildcard expands to list all the _directories_ in the current folder (the trailing **/** selects directories). We also have to pipe the results to the ```basename``` function (.. to remove that trailing **/** !). ```xargs``` does this piping, and the ```-n1``` flag tells **xargs** to only pipe one ```echo```'d directory at a time.

Other solutions to this little problem might be

~~~
ls -d */ | xargs -n1 basename
~~~
{: .bash}
~~~
find . -maxdepth 1 -mindepth 1 -type d -exec basename {} \;
~~~
{: .bash}
~~~
find . | egrep -o '(\w+)\/\1\.pov' | xargs -n1 dirname
~~~
{: .bash}

:wink:

(That last one will actually _only_ match directories containing **.pov** files _with the same name_, so it's a bit safer than the others!)

<br>
## Further exercises

### 1. povray **array job** with a _config_ file

Our solution above using a PBS **array job** was pretty neat, if I do say so myself. However, it may not prove to be very flexible.

Write another PBS script using an **array job** to render all of the images in the **Povray** example, but this time use a _config_ file instead. Look back at the _config_ file examples in the previous Episode if you need a reminder!

> ## Solution
> First we need a _config_ file. Let's call it **ex1.config**:
>
> ~~~
> # ArrayID Image
> 1 castle
> 2 escargot
> 3 fridge
> 4 glass
> 5 plants
> 6 snow
> ~~~
> {: .bash}
>
> Now we need a **PBS script**. Open **single_image.pbs** in your preferred editor, but don't forget to save it as new file! Call it **ex1.pbs** (In _nano_, type a new name on the entry line when you press <kbd>Ctrl</kbd>+<kbd>o</kbd>):
>
> ~~~
> #!/bin/bash
> #PBS -P Training
> #PBS -l select=1:ncpus=1:mem=1gb
> #PBS -l walltime=0:05:00
> #PBS -q small-express
> #PBS -J 1-6
>
> module load povray
>
> cd $PBS_O_WORKDIR
> config=ex1.config
>
> image=$(awk -v taskID=$PBS_ARRAY_INDEX '$1==taskID {print $2}' config)
>
> cd $image
> povray res $image.pov
> ~~~
> {: .bash}
>
> Did you remember to set the indexing to match your _config_ file?
{: .solution}


If you like, submit this script to Artemis with ```qsub```. Did it work?


<br>
### 2. povray **array job** with extra parameters

Having a _config_ file allows greater ease and flexibility to add extra parameters or options. Suppose you wanted to render the **Povray** example images at different resolutions? The ```povray``` function takes argument flags ```-W``` and ```-H``` to set the _width_ and _height_ of the rendered image.

Adapt your solution to _Exercise 1_ above to render the images in the following sizes:

| Image | Width | Height |
|:---:|:---:|:---:|
| Castle | 480 | 360 |
| Escargot | 600 | 400 |
| Fridge | 768 | 576 |
| Glass | 1024 | 768 |
| Plants | 480 | 360 |
| Snow | 320 | 120 |


> ## Solution
> First we need a _config_ file. Let's call it **ex2.config**:
>
> ~~~
> # ArrayID Image Width Height
> 1 castle 480 360
> 2 escargot 600 400
> 3 fridge 768 576
> 4 glass 1024 768
> 5 plants 480 360
> 6 snow 320 120
> ~~~
> {: .bash}
>
> Now we need a **PBS script**. Open **ex1.pbs** in your preferred editor, but don't forget to save it as new file! Call it **ex2.pbs** (In _nano_ type a new name on the entry line when you press <kbd>Ctrl</kbd>+<kbd>o</kbd>):
>
> ~~~
> #!/bin/bash
> #PBS -P Training
> #PBS -l select=1:ncpus=1:mem=1gb
> #PBS -l walltime=0:05:00
> #PBS -q small-express
> #PBS -J 1-6
>
> module load povray
>
> cd $PBS_O_WORKDIR
> config=ex2.config
>
> image=$(awk -v taskID=$PBS_ARRAY_INDEX '$1==taskID {print $2}' $config)
> width=$(awk -v taskID=$PBS_ARRAY_INDEX '$1==taskID {print $3}' $config)
> height=$(awk -v taskID=$PBS_ARRAY_INDEX '$1==taskID {print $4}' $config)
>
> cd $image
> povray -W$width -H$height $image.pov
> ~~~
> {: .bash}
>
> Did you remember to select the right _config_ file?
{: .solution}

If you like, submit this script to Artemis with ```qsub```. Did the images render at the correct resolutions?



<br>

___
**Notes**   
<sup id="f1">1[↩](#a1)</sup>In this construction, only either _j_ or _m_ are required, so ```:j``` will retrieve the _j_-th element and beyond, whilst ```::m``` will retrieve the first _m_ elements (up to element _m-1_, as the indexing starts with _0_).

___
<br>
