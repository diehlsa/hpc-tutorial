# HPC tutorial

## How to connect to a remote session

### 1. Get a terminal application

> Definition: A terminal is an interface in which you can type and execute text based commands.

#### Download your Terminal if you don't have one already.


+ **Windows**
  + MobaXterm [download](http://mobaxterm.mobatek.net/download-home-edition.html)

  > Remarks: In Windows, there is a default command line, called command prompt. However, the names of many commands is different and some commands do not exist. For that reason, use another terminal, such as MobaXterm.

+ **Mac**
  + iTerm2 [download](https://www.iterm2.com/downloads.html)
  + Terminal
+ **Linux**
  + Terminal

### 2. Access with SSH

#### Background information

+ **What is SSH?**

<img src="images/ssh_connection.png" width=600>

* **What is a SSH key pair?**

<img src="images/525px-Public_key_encryption.svg.png" width=400>

#### Step 2a: Connect to UL HPC (Linux / Mac OS / Unix)

Run the following commands in a terminal (substituting *studentXX* with the name of the key file you received from us):

        (laptop)$> ssh -p 8022 -i /path/to/studentXX.key studentXX@access-gaia.uni.lu

#### Step 2b: Connect to UL HPC (Windows)

* Open MobaXterm: **Start** > **Program Files** > **MobaXterm**
* Click on **Session**
  * In **SSH Session**:
    * Remote host: `access-gaia.uni.lu`
		* Check the **Specify username** box
		* Username: `studentXX`
    * Port: 8022
  * In **Advanced SSH Settings**
	  * Check `Use private key` box
		* Select the `studentXX.key` file you received.
  * Click on **Save**


## How to reserve resources  on the cluster


### Web monitoring interfaces

Each cluster offers a set of web services to monitor the platform usage:

* [**Monika**](https://hpc.uni.lu/gaia/monika), the visualization interface of the OAR scheduler, which  display the status of the clusters as regards the jobs running on the platform.
* A [pie-chart overview of the platform usage](https://hpc.uni.lu/status/overview.html)
* [DrawGantt](https://hpc.uni.lu/status/drawgantt.html), the Gantt visualization of jobs scheduled on OAR
* [Ganglia](https://hpc.uni.lu/status/ganglia.html), a scalable distributed monitoring system for high-performance computing systems such as clusters and Grids.

### Reserving resources with OAR

#### The basics

* [reference documentation](https://hpc.uni.lu/users/docs/oar.html)

[OAR](http://oar.imag.fr/) is an open-source batch scheduler which provides simple yet flexible facilities for the exploitation of the UL HPC clusters.

* it permits to schedule jobs for users on the cluster resource
* a _OAR resource_ corresponds to a node or part of it (CPU/core)
* a _OAR job_ is characterized by an execution time (walltime) on a set of resources.
  There exists two types of jobs:
  * _interactive_: you get a shell on the first reserved node
  * _passive_: classical batch job where the script passed as argument to `oarsub` is executed **on the first reserved node**

For simplicity we will only cover interactive jobs in this tutorial.

We will now see the basic commands of OAR.

Connect to the frontend of the Gaia cluster (access-gaia.uni.lu). You can request resources in interactive mode with the following command:

	(access)$> oarsub -I

Notice that with no parameters, oarsub gave you one resource (one core) for two hours. You were also directly connected to the node you reserved with an interactive shell.
  Now exit the reservation:

    (node)$> exit      # or CTRL-D

When you run exit, you are disconnected and your reservation is terminated.

#### Job management

You can check the status of your running jobs using `oarstat` command:

		(access)$> oarstat      # access all jobs
		(access)$> oarstat -u   # access all your jobs

#### Hierarchical filtering of resources

OAR features a very powerful resource filtering/matching engine able to specify resources in a **hierarchical**  way using the `/` delimiter. The resource property hierarchy is as follows:

		enclosure -> nodes -> cpu -> core


*  Reserve interactively 2 cores on 3 different nodes belonging to the same enclosure (**total: 6 cores**) for 3h15:

		(access)$> oarsub -I -l /enclosure=1/nodes=3/core=2,walltime=3:15


* Reserve interactively two full nodes belonging to different enclosures for 6 hours:

		(access)$> oarsub -I -l /enclosure=2/nodes=1,walltime=6

## Software environment


On the ULHPC clusters the software is managed with [Environment Modules](http://modules.sourceforge.net/). This is a software package that allows us to provide a [multitude of applications and libraries in multiple versions](http://hpc.uni.lu/users/software/) on the UL HPC platform. The tool itself is used to manage environment variables such as `PATH`, `LD_LIBRARY_PATH` and `MANPATH`, enabling the easy loading and unloading of application/library profiles and their dependencies.

* [Introduction to Environment Modules by Wolfgang Baumann](https://www.hlrn.de/home/view/System3/ModulesUsage)
* [Modules tutorial @ NERSC](https://www.nersc.gov/users/software/nersc-user-environment/modules/)
* [UL HPC documentation on modules](https://hpc.uni.lu/users/docs/modules.html)

## Test data

**Important:** This is already done for the `studentXX` accounts, so you can skip this section and go ahead to run mpiBLAST.

This tutorial relies on several input files for the bioinformatics packages, thus you will need to download them
before following the instructions in the next sections:

    (access)$> mkdir -p ~/bioinfo-tutorial/tophat ~/bioinfo-tutorial/mpiblast
    (access)$> cd ~/bioinfo-tutorial
    (access)$> wget --no-check-certificate https://raw.github.com/ULHPC/tutorials/devel/advanced/Bioinformatics/tophat/test_data.tar.gz -O tophat/test_data.tar.gz
    (access)$> wget --no-check-certificate https://raw.github.com/ULHPC/tutorials/devel/advanced/Bioinformatics/tophat/test2_path -O tophat/test2_path
    (access)$> wget --no-check-certificate https://raw.github.com/ULHPC/tutorials/devel/advanced/Bioinformatics/mpiblast/test.fa -O mpiblast/test.fa

Or simply clone the full ULHPC tutorials repository and make a link to the Bioinformatics tutorial:

    (access)$> git clone https://github.com/ULHPC/tutorials.git
    (access)$> ln -s tutorials/advanced/Bioinformatics/ ~/bioinfo-tutorial

Additionally you need a `.ncbirc` file containing the paths to the blast databases (too big to download quickly). The file can be downloaded from [here](https://raw.github.com/ULHPC/tutorials/devel/advanced/Bioinformatics/mpiblast/.ncbirc)
and needs to be placed in your $HOME directory (make sure to backup an existing $HOME/.ncbirc before overwriting it with the one in this tutorial).

## Run mpiBLAST

Characterization: data intensive, little RAM overhead, native parallelization

### Description

__mpiBLAST__: Open-Source Parallel BLAST

mpiBLAST is a freely available, open-source, parallel implementation of NCBI BLAST. By efficiently utilizing distributed computational resources through database fragmentation, query segmentation, intelligent scheduling, and parallel I/O, mpiBLAST improves NCBI BLAST performance by several orders of magnitude while scaling to hundreds of processors  [\[\*\]](http://www.mpiblast.org/).

### Example

This example will be run in an interactive session, with batch-mode executions
being proposed later on as exercises.

```
# Connect to Gaia (Linux/OS X):
(yourmachine)$> ssh -p 8022 -i /path/to/studentXX.key studentXX@access-gaia.uni.lu

# Request 1 full node in an interactive job:
(access-gaia)$> oarsub -I -l nodes=1,walltime=00:30:00

# Load the bioinfo software set
(node)$> module use $RESIF_ROOTINSTALL/bioinfo/modules/all

# Check the mpiBLAST versions installed on the clusters:
(node)$> module avail 2>&1 | grep -i mpiblast

# Load the default mpiBLAST version:
(node)$> module load bio/mpiBLAST

# Check that it has been loaded, along with its dependencies:
(node)$> module list

# The mpiBLAST binaries should now be in your path
(node)$> mpiformatdb --version
(node)$> mpiblast --version
```


mpiBLAST requires access to NCBI substitution matrices and pre-formatted BLAST databases. For the purposes of this tutorial, a FASTA (NR)
database has been formatted and split into 12 fragments, enabling the parallel alignment of a query against the database.

We will run a test using mpiBLAST. Note that mpiBLAST requires running with at least 3 processes, 2 dedicated for scheduling tasks and
coordinating file output, with the additional processes performing the search.



	# Go to the test directory and execute mpiBLAST with one core for search
	(node)$> cd ~/bioinfo-tutorial/mpiblast
	(node)$> mpirun -hostfile $OAR_NODEFILE -np 3 mpiblast -p blastp -d nr -i test.fa -o test.out

	# Note the speedup when using 12 cores
	(node)$> mpirun -hostfile $OAR_NODEFILE -np 12 mpiblast -p blastp -d nr -i test.fa -o test.out


### Exercises

Launch jobs with 8, 14 and 24 cores across two nodes and measure the speedup obtained.

**Hint**: To reserve 8 cores across two nodes, use `oarsub -I -l nodes=2/core=4,walltime=00:30:00`

## TopHat (and Bowtie2)

Characterization: data intensive, RAM intensive

### Description

__TopHat__ : A spliced read mapper for RNA-Seq

TopHat is a program that aligns RNA-Seq reads to a genome in order to identify exon-exon splice junctions. It is built on the ultrafast short read mapping program Bowtie [\[\*\]](http://ccb.jhu.edu/software/tophat/index.shtml).

__Bowtie2__: Fast and sensitive read alignment

Bowtie 2 is an ultrafast and memory-efficient tool for aligning sequencing reads to long reference sequences. It is particularly good at aligning reads of about 50 up to 100s or 1,000s of characters, and particularly good at aligning to relatively long (e.g. mammalian) genomes [\[\*\]](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml).

### Example

This example will show you how to use TopHat in conjunction with Bowtie2.


```
# Connect to Gaia (Linux/OS X):
(yourmachine)$> ssh -p 8022 -i /path/to/studentXX.key studentXX@access-gaia.uni.lu

# Request 1 full node in an interactive job:
(gaia-frontend)$> oarsub -I -l nodes=1,walltime=00:30:00

# Load the bioinfo software set
(node)$> module use $RESIF_ROOTINSTALL/bioinfo/modules/all

# Check the Tophat versions installed on the clusters:
(node)$> module avail 2>&1 | grep -i tophat

# Load the default Tophat version:
(node)$> module load bio/TopHat

# Load the corresponding Bowtie2 version:
(node)$> module load bio/Bowtie2/2.3.2-foss-2017a

# Check that both have been loaded:
(node)$> module list

# The Tophat and Bowtie2 binaries should now be in your path
(node)$> tophat --version
(node)$> bowtie2 --version
```

Now we will make a quick TopHat test, using the provided sample files:

    # Go to the test directory, unpack the sample dataset and go to it
    (node)$> cd ~/bioinfo-tutorial/tophat
    (node)$> tar xzvf test_data.tar.gz
    (node)$> cd test_data


    # Launch TopHat in serial mode
    (node)$> tophat -r 20 test_ref reads_1.fq reads_2.fq

    # Launch TopHat in parallel mode
    (node)$> tophat -p 12 -r 20 test_ref reads_1.fq reads_2.fq

We can see that for this fast execution, increasing the number of threads does not improve the calculation time due to the relatively high overhead of thread creation.
Note that TopHat / Bowtie are not MPI applications and as such can take advantage of at most one compute node.

Next, we will make a longer test, where it will be interesting to monitor the TopHat pipeline (with `htop` for example) to see the transitions between the serial
and parallel stages (left as an exercise).

```
# Load the file which will export $TOPHATTEST2 in the environment
(node)$> source ~/bioinfo-tutorial/tophat/test2_path

# Launch TopHat in parallel mode
(node)$> tophat2 -p 12 -g 1 -r 200 --mate-std-dev 30 -o ./  $TOPHATTEST2/chr10.hs $TOPHATTEST2/SRR027888.SRR027890_chr10_1.fastq $TOPHATTEST2/SRR027888.SRR027890_chr10_2.fastq
```

To monitor the run of Tophat, open a new session or terminal window and connect to the Gaia cluster.

```
# Check the running jobs
(access)$> oarstat -u

# Connect to your running job
(access)$> oarsub -C <jobid>

# Monitor CPU and RAM usage
(node)$> htop
```

The input data for the first test corresponds to the [TopHat test set](http://ccb.jhu.edu/software/tophat/tutorial.shtml),
while the second test is an example of aligning reads to the chromosome 10 of the human genome [as given here](http://www.bigre.ulb.ac.be/courses/statistics_bioinformatics/practicals/ASG1_2012/rnaseq_td/rnaseq_td.html).

### Proposed exercises

Launch jobs with 1, 2, 4, 8 and 10 cores on one node, using the second test files, and measure the speedup obtained.

***

**References**

* Image 1: by Saleh Salem on [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-log-into-a-vps-with-putty-windows-users) licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/)
* Image 2: https://commons.wikimedia.org/wiki/File:Public_key_encryption.svg
* Text for Section 1 modified from [LCSB R3 Linux 101 tutorial](https://git-r3lab.uni.lu/tutorial/2015-06-19-R3-Linux101)
* Text for Section 2 and 3 modified from [ULHPC "Getting started" tutorial](https://github.com/ULHPC/tutorials/tree/devel/basic/getting_started)
* Text for Section 4-6 modified from [ULHPC "Bioinformatics" tutorial](https://github.com/ULHPC/tutorials/tree/devel/advanced/Bioinformatics)
