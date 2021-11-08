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

## Map your samples to the reference

> ## Organise yourself
> 
> 1. Get back to `gbs/`
> 
> 2. In the folder `gbs/` create an output folder for this analysis, `refmap_output/` as well as the folder `samples_mapped`
> 
> 3. Copy the reference catalog below that we will use as a pretend reference genome from inside the `gbs/` folder you should currently be in.
>  
>  `/nesi/project/nesi02659/obss_2021/resources/gbs/reference_catalog.fa`
>  
>  Have a quick look inside the first few lines of reference_catalog.fa using `head -n 10 reference_catalog.fa`. It contains each loci identified in the denovo approach as a complete sequence.
>  
>>  ## Solution
>> ```bash
>> $ mkdir samples_mapped
>> $ mkdir refmap_output
>> $ cp  /nesi/project/nesi02659/obss_2021/resources/gbs/reference_catalog.fa .
>> $ head -n 10 reference_catalog.fa  
>> ```
> {: .solution}
{: .challenge}


It is about time to remind ourselves about a couple of mapping softwares. [BWA](https://github.com/lh3/bwa) is a piece of software that is commonly used for mapping. `bwa mem` is the ideal algorithm to align our short reads to the pretend reference genome. We will also use [Samtools](http://www.htslib.org/), a suite of tools designed to interact with the sam/bam alignment format (i,e, sorting, merging, splitting, subsetting, it is all there). 

> ## Load the software
> Identify the 2 modules you need for bwa and Samtools and load them.
> You need to index your reference genome. This is basically creating a map of your genome so that BWA can navigate it extra fast instead of reading it entirely for each read. Use bwa index YOURGENOME.fa for that.
>> ## Solution
>> 
>> ```bash
>> $ module spider BWA
>> $ module load BWA
>> $ module spider samtools
>> $ module load SAMtools
>> $ bwa index reference_catalog.fa  
>> ```
> {: .solution}
{: .challenge}

Well done, we are now ready to do the mapping!

## Mapping command

For each sample, we will now take the raw reads and map them to the reference genome, outputting a sorted `.bam` file inside the folder `samples_mapped`,

For a single sample, the command looks like this:

```bash
$ bwa mem -t 4 reference_catalog.fa  samples/MYSAMPLE.fq.gz   |  samtools view -b | samtools sort --threads 4 > samples_mapped/MYSAMPLE.bam 
```

> Explanations of this code: bwa mem use 4 threads to align samples/MYSAMPLE.fq.gz to the reference catalog. The  output is piped using the | symbol into  the next command instead of being printed to the screen, where samtools view create a bam file using `-b`. That bam output is piped into the sorting command of samtools before finally being outputted as a file  using `>` into sample mapping.
{: .callout}

Now this is all good and well, but we don't want to do it manually for each sample. The `for` loop below is doing it for all samples by going through all the `samples/*.fq.gz` files.

This is the chunkiest piece of code today. So no problem if you don't soak it all in. If you are in a classroom right now, we'll probably look at loops together. Otherwise, you could have a look at how they work [here](https://swcarpentry.github.io/shell-novice/05-loop/index.html).

```bash
$ for filename in samples/*fq.gz
	do base=$(basename ${filename} .fq.gz)
 	echo $base
  	bwa mem -t 4 reference_catalog.fa  samples/${base}.fq.gz | samtools view -b | samtools sort --threads 4 > samples_mapped/${base}.bam 
 done
```

> Explanations of this code:  for each filename in the folder samples that ends with .fq.gz, extract only the prefix of that filenamme: samples/PREFIX.fq.gz with the basename function. int the filename we are currently working with using echo. Use the bwa + samtools mapping explained above, using the base name to output a file `prefix.bam`.
{: .callout}

Well done, we only have `ref_map.pl` to run now.

## Run the ref_map pipeline

`ref_map.pl` has fewer parameters since the mapping takes care of many of the steps from `denovo_map.pl`, such as the creation of loci for each individual before a comparison of all loci across all individuals. `ref_map.pl`  uses the mapped files to identify the variable positions.

> ## build your ref_map.pl command
> Use the [online help](https://catchenlab.life.illinois.edu/stacks/comp/ref_map.php) to build your refmap command. you can also check `ref_map.pl --help`.
> • Unlike in the previous exercise, ask for 2 threads 
>
> • Specify the path to the input folder `samples_mapped/`
>
>  • Specify the path to the output folder `refmap_output/`
>  
> • Specify the path to the population map. We will be able to use the same popmap as for the denovo analysis, since we are using the same samples. 
>
> • We only want to keep the loci that are found in 80% of individuals. This is done by passing specific arguments to the `populations` software inside Stacks.
> 
> • Is your command ready? Run it briefly to check that it starts properly, once it does **stop it using ctrl + c**, we'll run it as a job.
>> ## Solution
>> ```bash
>> ref_map.pl -T 2 --samples samples_mapped/  -o refmap_output/ --popmap popmap.txt -r 0.8
>> ```
> {: .solution}
{: .challenge}

> ## Create the job file 
> • Time to put that into a job script. You can use the job script from the last exercise as a template. We will simply make a copy of it under a different name. In practice, we often end up reusing our own scripts. Let's copy the `denovojob.sh` into a new `refmapjob.sh`:
>
>```bash
> cp denovojob.sh refmapjob.sh
>```
>
> • Open `refmapjob.sh` using a text editor. Adjust the number of threads/cpus to 2, adjust the running time to `10mn`, and the job output to `refmap.log` instad of `denovo.log`. Most importantly, replace the `denovo_map.pl` command with your `ref_map.pl` command.
>
> • Save it, and run the job: `sbatch refmapjob.sh`
{: .challenge}
That should take about 5 minutes. Remember you can use `squeue -u <yourusername>` to check the status of the job. Once it is not there anymore, it has finished. This job will write out an output log called `output_refmap/refmap.log` that you can check using `less`.


### Analyse your results

>  ## Looking into the output
>  • Examine the output of the populations program in the file `ref_map.log` inside your `refmap_output` folder. (*hint*: use the `less` command).
>    
>  • How many SNPs were identified?
>   
>  • Why did `refmap.pl` run  much faster than `denovo_map.pl` ?
{: .challenge}

Well done, you now know how to call SNPs with or without a reference genome. It is worth re-iterating that even a poor-quality reference genome should improve the quality of your SNP calling by avoiding *lumping* and *splitting* errors. But beware of some applications. Inbreeding analyses are one example of applications that are sensitive to the quality of your reference genome.


Only one thing left, the cool biology bits.



{% include links.md %}

