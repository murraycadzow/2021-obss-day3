---
title: "SNP filtering and Population genetics analyses"
teaching: 0
exercises: 0
questions:
- "How do I navigate filtering my SNPs?"
- "How can I quickly visualise my data"
objectives:
- "Learn to navigate SNP filtering using Stacks"
- "Learn basic visualisation of population genetic data"
keypoints:
- "SNP filtering is about balancing signal vs noise"
- "Populations is the stacks implemented software to deal with filtering of SNPs"
- "Principal componant analysis (PCA) and Structure are an easy visualisation tools for your samples"
---
**Developed by:** Ludovic Dutoit, Alana Alexander

**Adapted by:** Julian Catchen, Nicolas Rochette


## Introduction

In this exercise, we'll think a about filtering the variants before applying two performing two population genetics structure analyses. We'll visualise our data using a Principal Component Analysis (PCA), and using the software [Structure](https://web.stanford.edu/group/pritchardlab/structure.html).

## Part 1: Data filtering
### Getting the data

We will work with the best combination of parameters identified from our [collective optimisation exercise](https://murraycadzow.github.io/2021-obss-day3/02-denovo-assembly/index.html#optimisation-exercise).

> ## Obtain the optimised dataset
> 
>   • Get back to the `gbs/` folder 
>   
>   • The optimised dataset can be optained from the link below, use the `-r` parameter of the `cp` command. That stands for recursive and allow you to copy folders and their content.
>  `/nesi/project/nesi02659/obss_2021/resources/gbs/denovo_final/`  
>  
>   • We'll also copy a specific population map that is encoding the 4 different possible populations ( 2 lakes x 2 depths). That population map is at `/nesi/project/nesi02659/obss_2021/resources/gbs/complete_popmap.txt`. 
>   
>   •  Once you copied it, have a quick look at the popmap using `less`
>   
>> ## Solution
>> ```
>> $ cp -r /nesi/project/nesi02659/obss_2021/resources/gbs/denovo_final/ .
>> $ cp /nesi/project/nesi02659/obss_2021/resources/gbs/complete_popmap.txt .
>> $ less complete_popmap.txt  # q to quit
>> ```
> {: .solution}
{: .challenge}


### Filtering the Data

You might have noticed that we have not really looked at our genotypes yet. Where are they? Well, we haven't really picked the format we want them to be written in yet. The last module of Stacks is called `populations`. `populations` allows us to export quite a few different formats directly depending on which analyses we want to run .Some of the format you might be familiar with are called `.vcf` `.phylip` or `.structure` (see [File Output Options](https://catchenlab.life.illinois.edu/stacks/comp/populations.php)).  The `populations` module also allows filtering of the data in many different ways as can be seen in the [manual](https://catchenlab.life.illinois.edu/stacks/comp/populations.php).`populations` is run at the end of `ref_map.pl` or `denovo_map.pl` anyway, but it can be re-run as a separate step to tweak our filtering or add output files.

SNP filtering is a bit of a tricky exercise that is approached very differently by different people. Here are my three pieces of advice. Disclaimer: This is just how I approach things, other people might disagree.

1. A lot of things in bioinformatics can be seen in terms of **signal to noise ratio**. Filtering your variants can be seen as maximising your signal (true genotypes) while reducing your noise (erroneous SNPs, missing data). 

2. **Think of your questions**. Some applications are sensitive to data filtering, some are less sensitive. Some applications are sensitive to some things but not others. They might be sensitive to missing data but not the absence rare variants. In general, read the literature and look at their SNP filtering. 

3. *Don't lose too much time hunting the perfect dataset, try creating one that can answer your question.* I often see people that try optimising every single step of the pipeline before seeing any of the biological outputs. I find it is often easier to go through the whole pipeline with reasonable guesses on the parameters to see if it can answer your questions before going back if needed and adapting it. If you then need to go back, you'll be a lot more fluent with the program anyway, saving yourself a lot of time.

Right, now that we went through this, Let's create our populations command using the [manual](https://catchenlab.life.illinois.edu/stacks/comp/populations.php).

> ## Building the populations command
> 
> • Specify `denovo_final/` as the path to the directory containing the Stacks files.
> 
> • Also specify `denovo_final/` as the output folder.
> 
> • Specify the population map that you just copied
> 
> • Specify the minimum proportion number of individuals covered to 80%
> 
> • In an effort to keep statistically independent SNPs, Specify that only one random SNP per locus should be outputted.
> 
> • Make sure to output a `structure file` and a `.vcf` file as well. We might be coming back to that `.vcf` file later today.
>> ## Solution
>> ```bash
>> $ populations -P denovo_final/ -O denovo_final/ -M complete_popmap.txt  -r 0.8 --write-random-snp  --structure --vcf
>> ``` 
> {: .solution}
{: .challenge}


How many SNPs do you have left? 

**Creating a white list**

In order to run a relatively quick set of population genetics analysis (i.e. for today), we will save a random list of 1000 loci into a file and then ask `populations` to only output data from these `loci`. Such a list is named a `white list`, intuitively the opposite of a `black list`. We can create a random white list by using the shell given a list of catalog IDs from the output of`populations`. We will use the file `denovo_final/populations.sumstats.tsv`. Quickly check it out using:

```bash
$ less -S denovo_final/populations.sumstats.tsv # use q to quit
```

> ## Create the white list.
> With the help of the instructions below, use the `cat`, `grep`, `cut`, `sort`, `uniq`, `shuf`, and `head` commands to generate a list of 1,000 random loci. Save this list of loci as `whitelist.txt`, so that we can feed it back into `populations`. This operation can be done in a single shell command. That sounds challenging, but the instructions below should help you create one command with several pipes to create that `whitelist.txt` file. The idea of a pipe is to connect commands using `|` e.g. `command 1 | command 2` outputs the content of command 1 into command 2 instead of outputting it to the screen.
> 
> At each step, you can run the command to see that it is outputting what you want, before piping it into the next step. Remember, you can access your last command in the termianl by using the up arrow key.
> 
> • First, use `cat` to concatenante `denovo_final/populations.sumstats.tsv`. 
> 
> • Then, stream it into the next command using a `|`.
> 
> • Use the command `grep` with `-v` to exclude all headers (i.e. `-v` stands for exclude, we want to exclude all the lines with "#").
> 
> • Add another `|` and select the first column with `cut -f 1`
> 
> • Then using *two* more pipes, use `sort` before using `uniq`. `uniq` will collapse succesive identical lines into single lines, so that you have one line per locus. Lucky us, those two commands don't require any arguments. We have to use `sort` first to make sure identical lines are next to each other, otherwise they won't collapse.
> 
> • Now try adding `shuf` which will mix all these lines all over. 
> 
> • Select the first one thousand lines with `head -n 1000` 
> 
> • Well done, redirect the output of `head` into the file `whitelist.txt`. (*hint*: use ">" to redirect into a file). Do not worry about the shuf: `write error: Broken pipe`, it is simply because head stops the command before the end.
> 
> • You did it! If you are new to bash, I am sure writing this command seemed impossible on Monday, so take a minute to congratulate yourself on the progress made even if you required a little help.
> 
>> ## Solution
>>```bash
>>cat denovo_final/populations.sumstats.tsv | grep -v "#" | cut -f 1 | sort | uniq | shuf | head -n 1000 > whitelist.txt
>>```
> {: .solution}
{: .challenge}


> ## Execute `populations` with the white list
> 
> Now we will execute `populations` again, this time feeding back in the whitelist you just generated. Check out the [help](https://catchenlab.life.illinois.edu/stacks/comp/populations.php) of `populations` to see how to use a white list. This will cause populations to only process the loci in the `whitelist.txt`. 
> 
>
>The simple way to do this is to start with the same command than last time:
>
>```bash
> $ populations -P denovo_final/ -O denovo_final/ -M complete_popmap.txt  -r 0.8 --write-random-snp  --structure --vcf
>```
> with one modification:
>
> • Specify the `whitelist.txt` file that you just generated as a white list.
>
> • Ready? Hit run!
>
cp  /nesi/project/nesi02659/obss_2021/resources/gbs/complete_popmap.txt .
>
>> ## Solution
>> ```bash
>> populations -P denovo_final/ -O denovo_final/ -M complete_popmap.txt  -r 0.8 --write-random-snp  --structure --vcf -W whitelist.txt
>> ```
> {: .solution}
{: .challenge}


• Did you really need to specify `-r`?

• What about --write-random-snp, did we need it?

We've run commands to generate the `structure` file two times now, but how many `structure` files are there in the stacks directory? Why?

If you wanted to save several different `vcf` and `structure` files generated using different `populations` options, what would you have to do?


## Part 2: Population genetics analyses


## Principal Component analysis (PCA)

> ## Let's get organised
> 
> • Create a `pca/` folder inside your `gbs/` folder
> 
> • Copy the `.vcf/` file inside denovo_final inside `pca/` (if you're not sure how it is called, remember it ends with `.vcf`)
> 
> • Get inside the pca folder to run the rest of the analyses
>> ## Solution
>> ``` bash
>> $ mkdir pca
>> $ cp denovo_final/*.vcf pca/
>> $ cd pca
>> ```
> {: .solution}
{: .challenge}
For the rest of the exercise, we will use R. You can run R a few different ways, you can use it from within the unix terminal directly or by selecting a R console or an R notebook from the `jupyter launcher` options. As our R code is relatively short, we'll save ourselves the pain of clicking around and we'll launch R from within the terminal. If you want to go to the R console or notebook, feel free to do so but make sure you select `R 4.1.0`.

```bash
$ module load R/4.1.0-gimkl-2020a
$ R
```

You are now running R, first off, let's load the `pcadapt` package.

```r
> library("pcadapt") 
```
let's read in our vcf and our popmap as a metadata file to have population information

```r
> vcf <- read.pcadapt("populations.snps.vcf", type = "vcf")
> metadata<-read.table("../complete_popmap.txt",h=F)
> colnames(metadata)<-c("sample","pop")
```

Finally, let's run the PCA.

```
> pca <- pcadapt(input = vcf, K = 10) # computing 10 PC axes
> png("pca.png") # open an output file
> plot(pca, option = "scores",pop=metadata$pop)
> dev.off() # close the output file
```

• Use the navigator on the left to go find the file `pca.png` inside the folder `pca/`

• What do you see, is there any structure by lake? by depth?

We really did the simplest PCA, but you can see how it is a quick and powerful to have an early look at your data. You can find a longer tutorial of pcadapt [here](https://bcm-uga.github.io/pcadapt/articles/pcadapt.html).

## Structure

Our goal now is to use our dataset in [Structure](https://web.stanford.edu/group/pritchardlab/structure.html), which analyzes the distribution of multi-locus genotypes within and among populations in a Bayesian framework to make predictions about the most probable population of origin for each individual. The assignment of each individual to a population is quantified in terms of Bayesian posterior probabilities, and visualized via a plot of posterior probabilities for each individual and population.

A key user defined parameter is the hypothesized number of populations of origin which is represented by K. Sometimes the value of K is clear from from the biology, but more often a range of potential K-values must be explored and evaluated using a variety of likelihood based approaches to decide upon the ultimate K. In the interests of time we won’t be exploring different values of K here, but this will be a key step for your own datasets. We'll just see what we get with K = 4 as we have up to 4 different populations. In addition, Structure takes a long time to run on the number of loci generated in a typical RAD data set because of the MCMC algorithms involved in the Bayesian computation. We therefore want to choose a random subset of loci that are well represented across our three populations. Despite downsampling, this random subset contains more than enough information to investigate population structure.

> ## Run structure
> • Leave R using `ctrl+z` Get back in the `gbs/` directory 
>
> • Create a structure directory
>  
> • Copy the `denovo_final/populations.structure` file inside the `structure/` directory
> 
> • Get inside the `structure directory`
>
> • Edit the `structure` output file to remove the comment line (i.e. first line in the file, starts with “#”). 
>
> • Find and load the structure module using the usual module commands.
> 
> • The parameters to run `structure` (with value of K=4) have already been prepared, you can find them here: `/nesi/project/nesi02659/obss_2021/resources/gbs/mainparams` and `/nesi/project/nesi02659/obss_2021/resources/gbs/extraparams`. Copy them into your `structure` directory as well. 
>
> • So far, when we've gone to run programs, we've been able to use `module spider` to figure out the program name, and then module load program_name to get access to the program and run it. Do it one more time for `structure`> 
>
> •  Run `structure` by simply typing `structure` > 
>
> •  Do you see `WARNING! Probable error in the input file.?` In our mainparams file it says that we have 1000 loci, but due to filters, it is possible that the populations module of Stacks actually output less than the 1000 loci we requested in `whitelist.tx`t. In the output of `populations.log` in your `denovo_final/` directory, how many variant sites remained after filtering? This is the number of loci actually contained in your `structure` file. You will need to adjust the number of loci in the mainparams file to match this exact Stacks output.
>
> I could have saved you that bug, but it is a very common one and it is rather obscure, so I wanted to make sure you get to fix it here for the first time rather than spending a few hours alone on it.
> 
>> ## Solution
>>```bash
>> $ mkdir structure
>> $ cp denovo_final/populations.structure structure
>> $ cd structure
>>```
>> Use nano or a text editor of your choise to remove the first line of the `populations.structure` file.
>>```bash
>> $ module spider structure
>> $ module load Structure
>> $ cp /nesi/project/nesi02659/obss_2021/resources/gbs/mainparams .
>> $ cp /nesi/project/nesi02659/obss_2021/resources/gbs/extraparams .
>> structure
>>```
> {: .solution}
{: .challenge}

### Structure Visualisation

Now that we  you ran `structure` successfully. It is time to visualise the results. We will do that in R. Open R using either a launcher careful to open R 4.1.0 or by typing `R`. Then use the code below to generate a structure plot.

```r
> library("pophelper")# load plotting module for structure 
> data<-readQ("populations.structure.out_f",filetype = "structure",indlabfromfile=TRUE) # Don't worry about 
> metadata<-read.table("../complete_popmap.txt",h=F)
> rownames(data[[1]])<-metadata[,1]
> plotQ(data,showindlab = TRUE,useindlab=TRUE,exportpath=getwd())  
```

That should create a structure image `populations.structure.out_f.png` in your structure folder. Use the path navigator on the left to reach your folder to be able to visualise the picture by double-clicking on it. Eah column is a sample. Individuals are assigned to up to 4 different clusters to maximise the explained genetic variance. They are represented in 4 different colours. 

• Do you see a similar structuring than in the PCA analysis?


Congrats, you just finished our tutorial for the assembly of RAD-Seq data.
Here are a few things you could do to solidify your learning from today.


### Extra activities

• You could play a bit with the parameters of `plotQ()` at [http://www.royfrancis.com/pophelper/reference/readQ.html](http://www.royfrancis.com/pophelper/reference/readQ.html) to make a prettier structure plot.

• You could spend a bit of time going through and making sure you understand the set of steps we did in those few exercises.

• You could try running this set of analyses on the less optimised dataset to see if it makes any difference or the one with reference based mapping.

• You could have a look at this [tutorial](populationstructure_tuto/populationstructure_tuto.md). It is a small tutorial I wrote previously that goes over a different set of population genetics analyses in R. You could even try reproducing it using the `vcf` you generated above. The `vcf` for the set of analyses presented in that code can be downloaded [here](https://github.com/ldutoit/bully_gbs/blob/master/output_files/populations.snps.vcf) should you want to download it.

{% include links.md %}
