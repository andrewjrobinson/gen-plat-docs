# HPC Updates

This document covers changes to the way we use LIMS-HPC

## Group directories

There are group directories for every user on the system.  They are located in the /home/group/LABGROUP 
where LABGROUP is usually the labhead's surname followed by 'lab'  e.g. /home/group/whelanlab.  Each lab 
group is restricted to only users who are in the labgroup.

## No home dir data

No data should be stored or processed within your home directory (e.g. /home/USERNAME).  While it works, 
it makes it difficult to cleanup/archive and for you to get help debugging your work.

## Fair usage

Please be fair and not request more that 32 cores.

## Partitions

* **8hour**: For small jobs requiring less than 8 hours of time to compute.  You should not use more than 
  8 cores.  It has access to all nodes of the HPC so should always be able to run a job (unless it's 
  really busy)
* **Compute**: Regular jobs up to 7 days of compute.  It has access to most nodes.
* **Long**: For jobs that are going to take a long time to complete; maximum is 200 days
* **Bigmem**: For jobs that require more than 128GB of memory (and hence need node 1)

### Time extensions

I can extend your jobs time setting if it looks like it won't finish in time up to 2x the maximum for its 
queue.  The exception to this is Node 1 which I will not extend as it is a specialist node and needs to be 
available for bigmem jobs.  To get this, please email your job number and expected time it will require to 
Genomics Platform [genomics@latrobe.edu.au](mailto:genomics@latrobe.edu.au)

## NTasks

The *--ntasks* option regularly gets used instead of *--cpus-per-task*.  If you are running programs one 
at a time then you should be using *--cpus-per-task*.  While it may produce the same result, the 
consequence of using *--ntasks* is that SLURM could give you some CPUs on one node and the rest on another
which your scripts are not likely to be able to support.

```sh
#!/bin/bash
#SBATCH --share
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=1024
#SBATCH --partition=8hour
#SBATCH --time=08:00

echo "Starting: $(date)"

some-awesome-program --threads=8 input.fa > output.txt

echo "Finished: $(date)"
``` 
**Figure**: example slurm script requesting 8 cores on a single node, for 8 hours, with 8GB of memory


## Job monitoring

### Munin

Munin is a webpage that shows graphs CPU usage over time.  It can be accessed from the following webpage:

http://munin-lims.latrobe.edu.au/lims-hpc.html

It is good for tracking how your job used the CPU over time however it only shows the whole node usage so 
in some situations it is hard to tell who used what.

If you click on one the graphs you can see the usage over longer time frames.

### Top

*top* us a unix command that shows the current usage of CPU and Memory of each process running on the current
node.  You will need to login to the compute nodes to see your job's usage.  See the "Topic 4: Job Monitoring" 
section in the [HPC Workshop material](http://andrewjrobinson.github.io/training_docs/tutorials/hpc/#topic-4-job-monitoring) 
for more details on using top.  To setup node login see [LIMS-HPC Node Login Setup](https://docs.google.com/document/d/1pv_ovxs9xoGtQfXFlbM632RLW2X75aGcXJZ5DFSr6QE)

### Is it correct?

Make sure you check your job is running correct when it starts and at regular intervals.