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
To copy file **from HPC cluster** (e.g., `yourfile` from home directory on HPC to home directory on your computer), use:
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
