# AMEB_HPC_Snakemake

## Welcome back

In this session we will be doing a phylogenomic analyses (mulitgene phylogeny) based on BUSCO genes, basically how I showed you in previous sessions, but this time we'll run it as a workflow using Snakemake on a HPC cluster.

### Prerequisites

Ideally, you should have followed along the previous demos:
 - [Phylogenomics_demo](https://github.com/chrishah/AMEB2020_Phylogenomics_demo)
 - [Snakemake_demo](https://github.com/chrishah/AMEB2020_Snakemake_demo)
 - [HPC_demo](https://github.com/chrishah/AMEB_HPC_demo)

You should be able to connect to the University's HPC cluster (Sauron) via ssh (e.g. PuTTY on windows).

You should have a conda environment with Snakemake set up on the cluster.

## Set the stage

Let's try if you can load your Snakemake conda environment.

```bash
(user@host)-$ conda activate snakemake
```
The prompt should change and now look like this:
```bash
(snakemake) (user@host)-$ #some command
```

We're going to use a bunch of software in our workflow that, as you know from the previous sessions, I have containerized, so they are portable and we don't have to install them locally. For these to be usable on the cluster we need to make sure [Singularity](https://sylabs.io/docs/) is installed and accessible. Singularity is an alternative to Docker for software containerization that is preferred on HPC clusters. You can assume it will be preinstalled on most modern HPC clusters, because containers are just so awesome..

On Sauron, Singularity is also preinstalled, but not globally. The `singularity` program can be executed like this:
```bash
(snakemake) (user@host)-$ /software/Singularity/3.5.3/bin/singularity
```

To use it in a convenient way, i.e. you don't have to type the full path to the program every time, you can to add it to your path. Do this once:
```bash
export PATH=$PATH:/software/Singularity/3.5.3/bin
```
Then `singularity` should be accessible just like so:
```bash
(snakemake) (user@host)-$ singularity
```

As a final step you can download this Repository, which contains a few useful files. We'll use git for that, should be installed on pretty much every Linux system.
```bash
(snakemake) (user@host)-$ git clone https://github.com/chrishah/AMEB_HPC_Snakemake.git
```

## Submit our analyses

The idea would be that each one of you submits a job that runs a phylogenomic analysis using a random subset of 30 BUSCO genes.

First move into the directory you've just downloaded:
```bash
(snakemake) (user@host)-$ cd AMEB_HPC_Snakemake
```

Now, create a file that contains a random list of 30 BUSCO genes that passed our criteria (I am sure you remember this from one of the [previous](https://github.com/chrishah/AMEB2020_Phylogenomics_demo) sessions). I have the results from the pre-filtering step deposited in `data/evaluate.all.tsv`. Some bash magic to extract 30 random BUSCOs.
```bash
(snakemake) (user@host)-$ cat data/evaluate.all.tsv | \
grep "pass$" | \
cut -f 1 | \
shuf | \
tail -n 30 > my_subset.txt
```

The repo also contains a `Snakefile`. It basically contains all teh steps we've discussed in the previous sessions (get the genes, do multiple sequence alignment, score and cut, post-filter, single gene tree), only organized as a snakemake workflow.

You can look into it with `less` (remember that `less` can be exited with `q`).
```bash
(snakemake) (user@host)-$ less Snakefile
```

Ok. The last thing we need is a submission script. I've deposited a template in `data/data/submission.template.sge.sh`. Copy it to your current working directory, renaming it in the process.
```bash
(snakemake) (user@host)-$ cp data/submission.template.sge.sh snakemake_phylo.sge.sh
```

Now, I want you to open the submission script and make a few changes.
 - give a new jobname
 - set the number of cores to 6
 - change the queue to `all.q@C146` or `all.q@C147` (I will give instructions)

Next, add the following code to the end of the file.
```bash
snakemake \
--use-singularity --singularity-args "-B /cl_tmp/hahnc/AMEB/AMEB2020_Phylogenomics_demo/data/checkpoints/BUSCO_results/" \
-j 6 -p \
--config \
dir=/cl_tmp/hahnc/AMEB/AMEB2020_Phylogenomics_demo/data/checkpoints/BUSCO_results/ \
ingroup="$(pwd)/data/ingroup.txt" outgroup="$(pwd)/data/outgroup.txt" \
files="$(cat my_subset.txt | tr '\n' ' ' | sed 's/ $//')"
```

Save and close the file.

Finally, submit your job.
```bash
(snakemake) (user@host)-$ qsub snakemake_phylo.sge.sh
```

Once, this is running we'll have plenty of time to discuss the details of what you just did.

### Well done!


