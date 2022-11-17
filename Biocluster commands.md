### Biocluster help page

https://help.igb.illinois.edu/Biocluster#Application_Lists

When first log in, the Terminal, the ID will be shown as "netid@biologin-1 ~", this means you're on the login node.
Please note that only run sbatch file when you're on login node.

When you are going to do interactive work such as running R script, you mush log in to a work node, after login to the worker node, the ID will be shown as "netid@compute-0-2 ~"

- To login to the worker node, type the follow code in termial to start interactive session:

```
srun --pty /bin/bash
```

- Viewing Job Status

squeue -u userid

```
squeue -u yifeik3
```

- Deleting Jobs
  You will need to use scancel, for example to delete a job with ID number 5523 you would type:

```
scancel 5523
```

Delete all of your jobs
scancel -u userid

```
scancel -u yifeik3
```
