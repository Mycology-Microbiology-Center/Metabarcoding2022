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
