---
title: "De-novo assembly without a reference genome"
teaching: 0
exercises: 0
questions:
- "Key question (FIXME)"
objectives:
- "Learn to call SNPs without a reference genome and to optimise the procedure"
- "Learn to run SLURM scripts"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

**Developed by:** Ludovic Dutoit

**Adapted from:** Julian Catchen, Nicolas Rochette

## Introduction 

In this exercise we will identify genetic variants for our 12 common bullies (AKA: Lake fish).

Without access to a reference genome, we will assemble the RAD loci *de novo* and examine population structure. However, before we can do that, we want to explore the *de novo* parameter space in order to be confident that we are assembling our data in an optimal way.

The detailed information of the stacks parameter can be found [in this explanation](http://catchenlab.life.illinois.edu/stacks/param_tut.php).

In summary, Stack (i.e. locus) formation is controlled by three main parameters: 

-m :  the minimum amount of reads to create a locus (default: 3)

**-M : the number of mismatches allowed between alleles of the same locus (i.e. The one we want to optimise)**

-n : The number of mismatches between loci between individuals. (should be set equal to `-M`)

-r: minimum percentage of individuals in a population sequences to consider that locus for that population 

If that does not make sense or you would like to know more, have a quick read of [this explanation from the manual](http://catchenlab.life.illinois.edu/stacks/param_tut.php).

Here, we will optimize the parameter `-M` (description in bold above) using the collaborative power of all of us here today! We will be using the guidelines of parameter optimization outlined in [Paris et al. (2017)](https://besjournals.onlinelibrary.wiley.com/doi/epdf/10.1111/2041-210X.12775) to assess what value for the `-M` parameter recovers the highest number of polymorphic loci. [Paris et al. (2017)](https://besjournals.onlinelibrary.wiley.com/doi/epdf/10.1111/2041-210X.12775) described this approach:  

*"After putative alleles are formed, stacks performs a search to match alleles together into putative loci. This search is governed by the M parameter, which controls for the maximum number of mismatches allowed between putative alleles [...] Correctly setting `-M` requires a balance – set it too low and alleles from the same locus will not collapse, set it too high and paralogous or repetitive loci will incorrectly merge together."*

As a giant research team, we will run the *denovo* pipeline with different parameters. The results from the different parameters will be shared using [this Google sheet](https://docs.google.com/spreadsheets/d/13qm_fFZ4yoegZ6Gyc_-wobHFb7HZxp27mrAHGPmnjRU/edit#gid=0).  After optimising these parameters, we'll be able to use the best dataset downstream for population genetics analyses and to compare it with a pipeline that utilises a reference genome. 


In reality, you'd probably do that optimisation with just a few of your samples as running the entire pipeline for several parameters combinations would be something that would consume much resources. But for today, I made sure our little dataset of 12 fishes run fast enough. 

## Optimisation exercise

In groups of 2, it is time to run an optimisation. So have a sip until your neighbour catch up or help them along. There are three important parameters that must be specified to denovo_map.pl, the minimum stack/locus depth (`m`), the distance allowed between stacks/loci (`M`), and the distance allowed between catalog loci (`n`). Choose which values of `M`  you want to run (M<10), not overlapping with parameters other people have already chosen, and insert them into [this google sheet](https://docs.google.com/spreadsheets/d/13qm_fFZ4yoegZ6Gyc_-wobHFb7HZxp27mrAHGPmnjRU/edit#gid=0). 

If you find most M values already entered in the spreadsheet, we will take the chance to optimise `-r`,   the number of samples that have to be sequenced for a particular locus to be kept it in the dataset (i.e. percentage of non-missing genotypes). Jump onto the second page of that Google sheet and vary `-r` away from 0.8 (80% of individuals covered).


> ## Organise yourself
>
> 1. In your `gbs/` workspace, create a directory called `denovo_output_optimisation` for this exercise.
> 
> 2. Make sure you load the `Stacks` module (you can check if you have already loaded it using `module list`)
> 
> 3. We want Stacks to understand which individuals in our study belong to which population. Stacks uses a so-called population map. The file contains sample names as well as populations. The file should be formatted in 2 columns like [this](http://catchenlab.life.illinois.edu/stacks/manual/#popmap). All 12 samples are at the file path below. Copy it into the `gbs/` folder you should currently be in. Note that all the samples have been assigned to a single "dummy" population, 'SINGLE_POP', just while we are establishing the optimal value of M in this current exercise.
>
>    `/nesi/project/nesi02659/obss_2021/resources/gbs/popmap.txt`
>
>> ## Solution
>> ```bash
>> ### create folder
>> $ cd ~/obss2021/gbs/
>> $ mkdir denovo_output_optimisation
>> ### load Stacks
>> $ module list
>> $ module load Stacks
>> ### copy the population map
>> cp  /nesi/project/nesi02659/obss_2021/resources/gbs/popmap.txt
>> ```
> {: .solution}
{: .challenge}
> ## Build the `denovo_map.pl` command
> 
> 1. Build your Stacks’ denovo_map.pl pipeline program according to the following set of instructions. Following these instructions you will bit by bit create the complete `denovo_map.pl` command:
> 
>   • Information on denovo_map.pl and its parameters can be found [online](http://catchenlab.life.illinois.edu/stacks/comp/denovo_map.php). You will use this information below to build your command.
>
>   • Specify the path to the directory containing your sample files (*hint* use your `samples/` link here!). The denovo_map.pl program will read the sample names out of the population map, and look for associated `fastq` in the samples directory you specify.
> 
>   • Make sure you specify this population map to the denovo_map.pl command (use the [manual](http://catchenlab.life.illinois.edu/stacks/comp/denovo_map.php)).
> 
>   • Set the `output_denovo_optimisation` directory as the output directory.
>  
>   • Set `-M` and `-n`. `n` should be equal to `M`, so if you set `M` at 3, set `n` at 3.
>
>   • Set `m` at 3, it is the default parameter, we are being explicit here for anyone (including ourselves), reading our code later.
>
>   • Set -r to 0.8, unless you are doing the -r optimisation, then set -r to your chosen value 
>  
>   • Finally, use 4 threads (4 CPUs: so your analysis finishes faster than 1!).
>
>   • Your command should be ready, try to execute denovo_map.pl (part of the Stacks pipeline). 
>
>   • Is it starting alright?  Good, now  **Use `control + c` to stop your command, we'll be back to it soon**
> 
>> ## Solution
>> This command is for someone optimising `-M` and setting `-M` at 3
>> ```bash
>> $ denovo_map.pl --samples samples/  --popmap popmap.txt -o denovo_output_optimisation/  -M 3 -n 3 -m 3 -r 0.8 -T 4
>> ```
> {: .solution}
{: .challenge}


## Running your first job

Running the commands directly on the screen is not common practice. You now are on a small server which has a reserved amount of resources for this workshop and this allows us to run our commands directly. You might want to run more than one thing at once or run things for longer or with way more resources than you have on your jupyter account. The way to do this is running your *job* using a *batch script*. The batch script (or submission script) accesses all the computing resources that are tucked away from the Mahuika login node. This allows your commands to be run as jobs that are sent out to computing resources elsewhere on Mahuika, rather than having to run jobs your little jupyter server itself. That way you can run many things at once and also use loads more [resources](https://support.nesi.org.nz/hc/en-gb/articles/360000204076-Mahuika-Slurm-Partitions). We will use this denovo_map.pl command as a perfect example to run our first job using a batch script.  

>  ## Your first job
>  
> • copy the example job ![image](https://user-images.githubusercontent.com/4376065/140674571-52a7795c-632f-4214-94e1-6df5de731f64.png)
file into this directory `gbs/`. The example is at: `/nesi/project/nesi02659/obss_2021/resources/gbs/denovojob.sh`  
>
> • Open it with a text editor, have a look at what is there. The first line is `#!/bin/bash -e`: this is a [shebang line](https://en.wikipedia.org/wiki/Shebang_(Unix)) that tells the computing environment that language our script is written in. Following this, there are a bunch of lines that start with `#SBATCH`, which inform the system about who you are, which type of resources you need, and for how long. In other words, your are telling mahuika how big of a computer you want to run that job.  
>
> • For a few of the `#SBATCH` lines, there are some spaces labelled up like `<...>` for you to fill in. These spaces are followed by a comment starting with a `#` that lets you know what you should be putting in there. With this information, fill in your job script. These are the resources you are requesting for this job. 
>
> • Once you are done, save it. Then run it using:  
>
> ```bash
> sbatch denovojob.sh
> ```
>   You should see `Submitted batch job` and then a random ID number.
>   
>  You can check what the status of your job is using:
>
> ```bash
> squeue -u <yourusername>
> ```
>  
> If your job is not  listed in `squeue`. It has finished running. It could be have been successful or unsuccessful (i.e. a dreaded bug), but if it is not in the queue, it has run. What would have printed to your screen has instead printed into the file `denovo.log`. If your job finished running immediately, go check that file to find a bug. If your job appear in the queue for more than 10s, you proably set it up correctly, sit back, it is probably time for a break.
{: .challenge}
 
 We used a few basic options of sbatch, including time, memory, job names and output log file. In reality, there are many, many, more options. Have a quick look at `sbatch --help` out of interest. NeSI also has its own handy guide on how to submit a job [here](https://support.nesi.org.nz/hc/en-gb/articles/360000684396-Submitting-your-first-job). 
  
 It is worth noting that we quickly test ran our command above before putting it in a job. This is a very important geek hack. Jobs can stay in the queue not running for some amount of time. If they fail on a typo, you'll have to start again. Quickly testing a command before stopping it using `ctrl + c` is a big time saver.

## Analysing the result of our collective optimisation

> ## Analysing the main output files
>Now that your execution is complete, we will examine some of the output files. You should find all of this info in `output_denovo_optimisation/`
>   
>   • After processing all the individual samples through ustacks and before creating the catalog with cstacks, denovo_map.pl will print a [table containing the depth of coverage](http://catchenlab.life.illinois.edu/stacks/manual/#cov) of each sample. Find this table in `output_denovo_optimisation/denovo_map.log`: what were the depths of coverage? 
>    
>   • Examine the output of the populations program in the file `populations.log` inside your `output_denovo_optimisation` folder (*hint*: use the `less` command).
>    
>   • How many loci were identified?
>
>   • How many variant sites were identified?
>  
>   • Enter that information in the collaborative [Google Sheet](https://docs.google.com/spreadsheets/d/13qm_fFZ4yoegZ6Gyc_-wobHFb7HZxp27mrAHGPmnjRU/edit#gid=0)
>    
>   • How many loci were filtered?
>      
>   • Familiarize yourself with the population genetics statistics produced by the populations component of stacks `populations.sumstats_summary.tsv` inside the `output_denovo_optimisation` folder
>    
>   • What is the mean value of nucleotide diversity (π) and FIS across all the individuals? [*hint*: The less -S command may help you view these files easily by avoiding the wrapping]
{: .challenge}
  
Congratulations, you went all the way from raw reads to genotyped variants. We'll have a bit of a think as a class compiling all this information.


{% include links.md %}

