---
title: "Assembly with a reference genome"
teaching: 0
exercises: 0
questions:
- "Key question (FIXME)"
objectives:
- "Learn to call SNPs with a reference genome"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

**Developed by:** Ludovic Dutoit, Alana Alexander
**Adapted from:** Julian Catchen, Nicolas Rochette

### Introduction

Obtaining an assembly without a reference genome is easy and possible. However, having a reference genome allows us to avoid several issues. We do not have to make assumptions about the "best" value for the `-M` parameter, and we reduce the risk of collapsing different loci together ("lumping") or separating one "real" locus into several "erroneous loci" ("splitting"). Studies have demonstrated that having some kind of reference genome is the single best way you can improve GBS SNP calling (see for example [Shafer et al. 2016](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.12700)). Using a related species genome might be good enough in several cases too (but it sometimes depend on your application).

The stacks pipeline for samples with a reference genome is [ref_map.pl](https://catchenlab.life.illinois.edu/stacks/comp/ref_map.php). It skips the creation of loci and the catalog steps as reads belong to the same stack/locus if they map to the same location of the reference genome. This means we don't have to rely on assumptions derived from genetic distances (`-M` and `-n` of `denovo_map.pl`) about whether reads belong to the same locus or not. 

We don't have a genome for our common bullies yet. Instead, we will map the reads to the catalog of loci that stacks as created for one of our denovo run. You would never do that in the real world, but that gives us a chance to run the ref_map.pl pipeline and 

## Map your samples

> ## Organise yourself
> 
> 1. Get back to `gbs/`
> 
> 2.In the folder `gbs/` create an output folder for this analysis, `refmap_output/`.
> 
> 3. Copy the reference catalog that we will use as a pretend reference genome from inside the `gbs/` folder you should currently be in.
>> ## Solution
>> 
>> ```bash
>> $ Get back to gbs, if you re lost ...
>> $ mkdir refmap_output
>> $ cp ...
>> ```
> {: . challenge}
{: . solution}


It is about time to remind you about a couple of softwares in. `BWA` [!!!link!!!] is a piece of software that is commonly used for mapping. `bwa mem` is the ideal algorithm to align our short reads to the pretend reference genome. We will also use samtools, a suite of tools designed to interact with the sam/bam alignment format (sorting, merging, splitting, subsetting, it is all there). 

> ## Load the software
> Identify the 2 module respectively you need for bwa and Samtools and load them  ( https://support.nesi.org.nz/hc/en-gb/articles/360000360576-Finding-Software
> You need to index your reference genome. This is basically creating a map of your genome so that BWA can navigate it extra fast insread of reading it entirely for each read. use bwa index YOURGENOME for that.

>> ## Solution
>> 
>> ```bash
>> $ Get back to gbs, if you re lost ...
>> $ mkdir refmap_output
>> $ cp ...
>> ```
> {: . challenge}
{: . solution}
 
 
 Well done, we are now ready to mapping

## Run the ref_map pipeline.

`ref_map.pl` has less options since the mapping takes care of many of the steps from `denovo_map.pl`, such as the creation of loci for each individual before a comparison of all loci across all individuals. Use the [online help](https://catchenlab.life.illinois.edu/stacks/comp/ref_map.php) to build your refmap command.

• Unlike in the previous exercise, ask for 2 threads 

• Specify the path to the output folder `output_refmap/`

• Specify the path to the input folder `samples_mapped/`

• Specify the path to the population map. We will be able to use the same popmap as for the denovo analysis, since we are using the same samples. 

• We only want to keep the loci that are found in 80% of individuals. This is done by passing specific arguments to the `populations` software inside Stacks. The following should be added to your command: `-X "populations:  -r 0.80"`. Make sure you include the quotes.

• Is your command ready? Run it briefly to check that it starts properly, once it does **stop it using ctrl + c**

• Time to put that into a job script. You can use the job script from the last exercise as a template. We will simply make a copy of it under a different name. In practice, we often end up reusing our own scripts.

    cp denovojob.sh refmapjob.sh

• Open `refmapjob.sh` using a text editor. Adjust the number of cpus to 2, adjust the running time to `10mn`, and the job name to `refmap`. Most importantly, replace the `denovo_map.pl` command with your `ref_map.pl` command.

• Save it, and run the job:
  
    sbatch refmapjob.sh

That should take about 5 minutes. Remember you can use `squeue -u <yourusername>` to check the status of the job. Once it is not there anymore, it has finished. This job will write out an output log called `output_refmap/refmap.log` that you can check using `less`.


### Analyse your results.


   • Examine the output of the populations program in the file `ref_map.log` inside your `output_refmap` folder. (*hint*: use the `less` command).
    
  • How many SNPs were identified?
   
 • Why did it run so much faster than `denovo_map.pl` ?
 
 • As a thought exercise: Do you have any idea how we could check whether the same loci have been found as in the *de novo* run or not?

Well done, you now know how to call SNPs with or without a reference genome. It is worth re-iterating that even a poor-quality reference genome should improve the quality of your SNP calling by avoiding "lumping" and "splitting" errors. But that some applications ( for example inbreeding analyses are sensitive to the quality of your reference genome).


Only one thing left, the cool biology bits!



{% include links.md %}

