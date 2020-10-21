---
layout: default
title: Running Jobs on Expanse
parent: Expanse 101
nav_order: 6
---

## Running Jobs on Expanse <a name="running-jobs"></a>
Expanse manages computational work via the Simple Linux Utility for Resource Management (SLURM) batch environment. Expanse places limits on the number of jobs queued and running on a per group (allocation) and partition basis. Submitting a large number of jobs (especially very short ones) can impact the overall  scheduler response for all users. If you are anticipating submitting a lot of jobs,  contact the SDSC consulting staff before you submit them. We can work to check if there are bundling options that make your workflow more efficient and reduce the impact on the scheduler

For more details, see the section on Running job in the Expanse User Guide:
http://www.sdsc.edu/support/user_guides/expanse.html#running


### The Simple  Linux Utility for Resource Management  (SLURM) <a name="running-jobs-slurm"></a>

<img src="images/slurm.png" alt="Simple Linux Utility for Resource Management" width="500px" />

* “Glue” for parallel computer to schedule and execute jobs
* Role: Allocate resources within a cluster
* Nodes (unique IP address)
* Interconnect/switches
* Generic resources (e.g. GPUs)
* Launch and otherwise manage jobs

* Functionality:
* Prioritize queue(s) of jobs;
* decide when and where to start jobs;
* terminate job when done;
* Appropriate resources;
* Manage accounts for jobs

* All jobs must be run via the Slurm scheduling infrastructure. There are two types of jobs:
* [Interactive Jobs](#running-jobs-slurm-interactive)
* [Batch Jobs](#running-jobs-slurm-batch-submit)

[Back to Top](#top)
<hr>

### Interactive Jobs: <a name="running-jobs-slurm-interactive">
Interactive HPC systems allow *real-time* user inputs in order to facilitate code development, real-time data exploration, and visualizations. An interactive job (also referred as interactive session) will provide you with a shell on a compute node in which you can launch your jobs. On Expanse, use the ```srun``` command:
```
srun --pty --nodes=1 --ntasks-per-node=24 -p debug -t 00:30:00 --wait 0 /bin/bash
```

For more information, see the interactive computing tutorial [here](https://github.com/sdsc/sdsc-summer-institute-2020/blob/master/0_preparation/interactive_computing/README.md).

### Batch Jobs using SLURM: <a name="running-jobs-slurm-batch"></a>
When you run in the batch mode, you submit jobs to be run on the compute nodes using the ```sbatch``` command (described below).

Batch scripts are submitted from the login nodes. You can set environment variables in the shell or in the batch script, including:
* Partition (also called the qeueing system)
* Time limit for a job (maximum of 48 hours; longer on request)
* Number of nodes, tasks per node
* Memory requirements (if any)
* Job name, output file location
* Email info, configuration

Below is an example of a basic batch script, which shows key features including
naming the job/output file, selecting the SLURM queue partition, defining the
number of nodes and ocres, and the length of time that the job will need:

```
[mthomas@expanse-ln3 IBRUN]$ cat hellompi-slurm.sb
#!/bin/bash
#SBATCH --job-name="hellompi"
#SBATCH --output="hellompi.%j.%N.out"
#SBATCH --partition=compute
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=24
#SBATCH --export=ALL
#SBATCH -t 00:30:00

#Define user environment
source /etc/profile.d/modules.sh
module purge
module load intel
module load mvapich2_ib

#This job runs with 2 nodes, 24 cores per node for a total of 48 cores.
#ibrun in verbose mode will give binding detail

ibrun -v ../hello_mpi
```

Note that we have included configuring the user environment by purging and
then loading the necessary modules. While not required, it is a good habit
to develop when building batch scripts.

[Back to Top](#top)
<hr>

### Slurm Partitions <a name="slurm-partitions"></a>
Expanse places limits on the number of jobs queued and running on a per group (allocation) and partition basis. Please note that submitting a large number of jobs (especially very short ones) can impact the overall  scheduler response for all users.

<img src="images/expanse-queue-names.png" alt="Expanse Queue Names" width="500px" />

Specified using -p option in batch script. For example:
```
#SBATCH -p gpu
```
[Back to Top](#top)
<hr>

### Slurm Commands: <a name="slurm-commands"></a>
Here are a few key Slurm commands. For more information, run the `man slurm` or see this page:

* To Submit jobs using the `sbatch` command:

```
$ sbatch Localscratch-slurm.sb 
Submitted batch job 8718049
```
* To check job status using the squeue command:
```
$ squeue -u $USER
             JOBID PARTITION     NAME     USER      ST       TIME  NODES  NODELIST(REASON)
                        8718049   compute       localscr mahidhar   PD       0:00       1               (Priority)
                        ```
                        * Once the job is running, you will see the job status change:
                        ```
                        $ squeue -u $USER
                                     JOBID PARTITION     NAME     USER    ST       TIME  NODES  NODELIST(REASON)
                                                8718064     debug        localscr mahidhar   R         0:02      1           expanse-14-01
                                                ```
                                                * To cancel a job, use the `scancel` along with the `JOBID`:
                                                *   $scancel <jobid>
                                                
                                                ### Command Line Jobs <a name="running-jobs-cmdline"></a>
                                                The login nodes are meant for compilation, file editing, simple data analysis, and other tasks that use minimal compute resources. <em>Do not run parallel or large jobs on the login nodes - even for simple tests</em>. Even if you could run a simple test on the command line on the login node, full tests should not be run on the login node because the performance will be adversely impacted by all the other tasks and login activities of the other users who are logged onto the same node. For example, at the moment that this note was written,  a `gzip` process was consuming 98% of the CPU time:
                                                ```
                                                [mthomas@expanse-ln3 OPENMP]$ top
                                                ...
                                                PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                      
                                                19937 XXXXX     20   0  4304  680  300 R 98.2  0.0   0:19.45 gzip
                                                ```
                                                
                                                Commands that you type into the terminal and run on the sytem are considered *jobs* and they consume resources.  <em>Computationally intensive jobs should be run only on the compute nodes and not the login nodes</em>.
                                                
                                                [Back to Top](#top)
                                                <hr>
