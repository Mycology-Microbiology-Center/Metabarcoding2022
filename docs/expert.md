# "Expert mode" commands

## HPC basics

For demonstration purposes, we will use the [Rocket Cluster](https://hpc.ut.ee/services/HPC-services/Rocket) of the University of Tartu.

To connect to the head-node of the HPC cluster using the secure shell protocol, run:
```bash
ssh USER@SERVER
```
substitute `USER` with your login ID and `SERVER` with a hostname (IP or a domain name).<br/>
E.g., `ssh koljalg@rocket.hpc.ut.ee`.

To copy a file or multiple files **to the HPC cluster**, use:
```bash
scp yourfile koljalg@rocket.hpc.ut.ee:~/yourfile   # single file
scp file1 file2 koljalg@rocket.hpc.ut.ee:~/        # multiple files
```
To copy file **from the HPC cluster** (e.g., `yourfile` from home directory on HPC to home directory on your computer), use:
```bash
scp koljalg@rocket.hpc.ut.ee:~/yourfile ~/yourfile
```

If you have large files (or a large number of files), it's better to use [`rsync`](https://en.wikipedia.org/wiki/Rsync) program for file transfer, e.g.
```bash
rsync -avz Documents/* koljalg@rocket.hpc.ut.ee:~/all/
```

To end your session on the HPC cluster, run:
```bash
exit
```

## Setup working environment on HPC cluster

In general, one needs admin rights to install the software on HPC clusters. However, users may install software into their home directory where they have write permissions. To make life easier, you may use `Conda` - a package manager which helps you find and install the software and its dependencies.<br/>
To install [Miniconda](https://docs.conda.io/en/latest/index.html), run the following code:
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p $HOME/miniconda
~/miniconda/bin/conda init bash
source ~/.bashrc
conda update --all --yes -c bioconda -c conda-forge
conda install --yes -c conda-forge mamba
```

To install the software (e.g., [`seqkit`](https://bioinf.shenwei.me/seqkit/) program), run:
```bash
mamba install -c bioconda seqkit
```


### Conda environments

If the software you wish to use could not be installed in the base (default) environment due to the conflict of versions, or you want to use a specific version of the program, or just want to keep it independent, you may create a separate environment with:
```bash
mamba create --name VSEARCHENV -c bioconda -c conda-forge vsearch=2.21.1 blast=2.13.0
conda activate VSEARCHENV           # swith to the new environment we've created
```

Verify which software versions are installed:
```bash
vsearch --version
blastn -version
```

To switch to the base environment, run:
```bash
conda deactivate
```

### Module system

Alternatively, if the software you wish to use is pre-installed on the HPC cluster, you may load it as an environment module.<br/>
To list all available modules, use `module avail` command (scroll the list with `space` button, press `q` to quit).<br/>
To search for a particular module, use e.g. `module -r spider '.*singularity.*'`.<br/>
If the required software was found, you need to load the module, e.g.:
```bash
module load any/singularity/3.7.3
```


## Scheduling jobs on the HPC cluster

The Slurm Workload Manager, a.k.a. Simple Linux Utility for Resource Management (SLURM), is used to share the HPC resources between users.

Please note that users log in from their computers to the cluster **head node**.<br/>
**Do not run the analysis on the head node!**<br/>
You should use it only to schedule the tasks that would be distributed on the **cluster nodes**.

To run the program on the HPC cluster you should:
- Prepare a batch script with directives to SLURM about the number of CPUs<sup>\*</sup>, amount of RAM<sup>\*</sup>, and time duration requested for the job, along with commands which perform the desired calculations;<br/>
- Submit the script to the batch queue. SLURM will evaluate the task's priority and start executing the job when it reaches the front of the queue.<br/>
When the job finishes, you may retrieve the output files.

<sup>\*</sup> Unless specified otherwise, on the Rocket cluster, all jobs will be allocated 1 node with 1 CPU core and 2 GB of memory.

### Batch script

Here is a basic batch script which that contains a minimal number of SLURM options:
```bash
#!/bin/bash -l
#SBATCH --job-name=my_job
#SBATCH --cpus-per-task=4
#SBATCH --nodes=1
#SBATCH --mem=10G
#SBATCH --partition amd
#SBATCH --time=48:00:00

## If needed, you may load the required modules here
# module load X

## Run your code
some_program -i input.data -o output_1.data --threads 4
some_script.sh output_1.data > output_2.data
echo "Done" > output.log
```

The syntax for the SLURM directive in a script is `#SBATCH <flag>`, where `<flag>` could be:<br/>
- `--job-name`, the name of the job;
- `--cpus-per-task`, the number of CPUs each task should have (e.g., 4 cores);
- `--nodes`, requested number of nodes (each node could have multiple CPUs);
- `--mem`, requested amount of RAM (e.g., 10 gigabytes);
- `--partition`, the partition on which the job shall run
- `--time`, the requested time for the job (e.g., 48 hours).

To submit a job, save the code above to a file (e.g., `my_job.sh`) and run:
```bash
sbatch my_job.sh
```
