# Slurm <!-- {docsify-ignore} -->
This page serves as point of introduction to understanding and making best use of Slurm. 

Slurm is a scheduler is a free and open-source job scheduler for Linux and Unix-like kernels, used by many supercomputers and computer clusters.

Slurm serves the purpose of being the scheduler of jobs and the manager of the resources on both, MERCED and Pinnacles clusters. It does it in three ways: 
- Provides exclusive and/or non-exclusive access to the resources on the compute nodes to the users for a certain amount of time. 
- Provides a framework to start, execute, and check the work on the set of allocated compute nodes.
- Slurm manages the queue of pending jobs based on the availability of resources.



## Slurm Commands

Basic Slurm Commands to interact via the command line interface with the queueing and resource scheduler/ 
The table below shows the list and descriptions of the mostly used Slurm commands.


| Commands   | Syntax                        | Description                                              |
|------------|-------------------------------|----------------------------------------------------------|
| `sbatch`   | `sbatch <job-id>`             | Submit a batch script to Slurm for processing.           |
| `squeue`   | `squeue -u`                   | Show information about your job(s) in the queue. The command when run without the `-u` flag, shows a list of your job(s) and all other jobs in the queue. |
| `srun`     | `srun <resource-parameters>`  | Run jobs interactively on the cluster.                   |
| `skill/scancel` | `scancel <job-id>`        | End or cancel a queued job.                              |
| `sacct`    | `sacct`                       | Show information about current and previous jobs.        |
| `sinfo`    | `sinfo`                       | Get information about the resources on available nodes that make up the HPC cluster. |

### `sbatch` Command <!-- {docsify-ignore} -->

The `sbatch` command is used to sumbit job scripts to the SLURM scheduler to be placed on the respective queue and to then begin with the resources are available. 

#### Key Features of the `sbatch` command <!-- {docsify-ignore} -->

1. Job Submission
    Submit jobs for batch processing.

2. Schedule jobs for future execution.
3. Submit jobs with dependencies, ensuring that certain jobs run only after the completion of other jobs.
4. Combine standard output and error into a single file.


!> **NOTE:** `sbatch` command is different from `SBATCH` directives. 




## Slurm Script Main Parts 

In creating a Slurm script, there are 4 main parts that are mandatory in order for your job to be successfully processed. 

Shebang The Shebang command tells the shell (which interprets the UNIX commands) to interpret and run the Slurm script using the bash (Bourne-again shell) shell.

This line should always be added at the very top of your SBATCH/Slurm script.

    !/bin/bash


### Common `SBATCH` Directives in scripts <!-- {docsify-ignore} -->

The `SBATCH` directives must be used in the following manner for Slurm to properly recognize them: 

Template:
    #SBATCH --directive

Example: 

    #SBATCH --job-name=EXAMPLEJOB
    #SBATCH --nodes=2


Common Options

    --job-name=NAME: Specifies the name of the job.
    --partition=PARTITION: Submits the job to a specific partition.
    --nodes=N: Requests N nodes for the job.
    --ntasks-per-node=N: Specifies the number of tasks to launch per node.
    --mem=MB: Requests a specific amount of memory (in megabytes).
    --time=MINUTES: Sets a limit on the total run time of the job.
    --output=FILE_NAME: Designates a file to capture the standard output.
    --error=FILE_NAME: Specifies a file to capture the standard error output.
    --mail-type=TYPE: Configures when SLURM sends email notifications about the job.
    --mail-user=USER_EMAIL: Specifies the email address to receive job notification




!> Lines that begin with `#SBATCH` in all caps is treated as a directive by Slurm. This means that to comment out a Slurm command, you need to append a second another pound sign # to the SBATCH command (#SBATCH means Slurm command, # #SBATCH means comment).
Final Part of the job submssion script is to include: 

    --export=all 

Conclude by including your list of commands to execute your script.


### Job Examples <!-- {docsify-ignore} -->

Job Examples can be found [here](running_jobs.md)

### SBATCH directives for job script <!-- {docsify-ignore} -->

| Directives      | Description   |
|-----------------|---------------|
| `--job-name`    | Specifies a name for the job allocation. The specified name will appear along with the job id number when querying running jobs on the system. The default is the name of the batch script, or just sbatch if the script is read on sbatch’s standard input. |
| `--output`      | Instructs Slurm to connect the batch script's standard output directly to the filename. If not specified, the default filename is `slurm-jobID.out`. |
| `--partition`   | Requests a specific partition for the resource allocation (`gpu`, `interactive`, `normal`). If not specified, the default partition is normal. |
| `--ntasks`      | This option advises the Slurm controller that job steps run within the allocation will launch a maximum of number tasks and offer enough resources. The default is 1 task per node, but note that the `--cpus-per-task` option will change this default. |
| `--cpus-per-task` | Advises the Slurm controller that ensuing job steps will require ncpus number of processors per task. Without this option, the controller will just try to assign one processor per task. For instance, consider an application that has 4 tasks, each requiring 3 processors. If the HPC cluster is comprised of quad-processors nodes and simply ask for 12 processors, the controller might give only 3 nodes. However, by using the `--cpus-per-task=3` options,
| `--mem-per-cpu`  | This is the minimum memory required per allocated CPU. Note: It’s highly recommended to specify `--mem-per-cpu`. If not, the default setting of 500MB will be assigned per CPU. |
| `--time`         | Sets a limit on the total run time of the job allocation. If the requested time limit exceeds the partition’s time limit, the job will be left in a PENDING state (possibly indefinitely). The default time limit is the partition’s default time limit. A time limit of zero requests that no time limit be imposed. The acceptable time format is days-hours:minutes:seconds. Note: It’s mandatory to specify a time in your script. Jobs that don’t specify a time will be given a default time of 1-minute after which the job will be killed. This modification has been done to implement the new backfill scheduling algorithm and it won’t affect partition wall time. |
| `--mail-user`    | Defines user who will receive email notification of state changes as defined by `--mail-type`.                          |
| `--mail-type`    | Notifies user by email when certain event types occur. Valid type values are BEGIN, END, FAIL. The user to be notified is indicated with `--mail-user`. The values of the `--mail-type` directive can be declared in one line like so: `--mail-type BEGIN, END, FAIL` |
| `--get-user-env` | Tells `sbatch` to retrieve the login environment variables. Be aware that any environment variables already set in `sbatch` environment will take precedence over any environment variables in the user’s login environment. Clear any environment variables before calling `sbatch` that you don’t want to be propagated to the spawned program. |


### Slurm Output Environment Variables <!-- {docsify-ignore} -->

When a job scheduled by Slurm starts, it needs to know some information about its execution environment. For example, It needs to know the working directory, and what nodes allocated to it. Slurm passes this information to the running job via what so-called environment variables. The following is the most common-used environment variable.

| SLURM Environment Variable | Description                                       |
|----------------------------|---------------------------------------------------|
| `SLURM_CLUSTER_NAME`       | Name of the cluster on which the job is executing.|
| `SLURM_CPUS_ON_NODE`       | Number of CPUs on the allocated node.             |
| `SLURM_CPUS_PER_TASK`      | Number of CPUs requested per task.                |
| `SLURM_GPUS`               | Number of GPUs requested.                         |
| `SLURM_GPUS_PER_NODE`      | Requested GPU count per allocated node.           |
| `SLURM_GPUS_PER_TASK`      | Requested GPU count per allocated task.           |
| `SLURM_JOB_ID`             | The ID of the job allocation.                     |
| `SLURM_JOB_CPUS_PER_NODE`  | Count of processors available to the job on this node. |
| `SLURM_JOB_NAME`           | Name of the job.                                  |
| `SLURM_JOB_NODELIST`       | List of nodes allocated to the job.               |
| `SLURM_JOB_NUM_NODES`      | Total number of nodes in the job's resource allocation. |
| `SLURM_JOB_PARTITION`      | Name of the partition in which the job is running.|
| `SLURM_MEM_PER_CPU`        | Minimum memory required per allocated CPU.        |
| `SLURM_MEM_PER_GPU`        | Requested memory per allocated GPU. |
| `SLURM_MEM_PER_NODE`       | Total amount of memory per node that the job needs. |
| `SLURM_NODELIST`           | List of nodes allocated to the job. |
| `SLURM_NPROCS`             | Total number of CPUs allocated |
| `SLURM_NTASKS`             | Maximum number of MPI tasks (that's processes). |
| `SLURM_NTASKS_PER_CORE`    | Number of tasks requested per core. |
| `SLURM_NTASKS_PER_GPU`     | Number of tasks requested per GPU. |
| `SLURM_NTASKS_PER_NODE`    | Number of tasks requested per node. |
| `SLURM_PRIO_PROCESS`       | The scheduling priority (nice value) at the time of job submission. This value is propagated to the spawned processes. |
| `SLURM_PROCID`             | The MPI rank (or relative process ID) of the current process. |
| `SLURM_SUBMIT_DIR`         | The directory from which SBATCH was invoked. |
| `SLURM_SUBMIT_HOST`        | The Hostname of the computer from which SBATCH was invoked. |
| `SLURM_TASK_PID`           | The process ID of the corresponding task. |

## Slurm - Job Management 

Job management is critical before running or scaling jobs and computations within a HPC enviroment. We have created a manual page that can be found [here](Manage_job.md). The documentation goes over common Slurm commands that help debug job errors and overall performance using `sacct` and `scontrol`. 


## Job Arrays 

Job arrays offer a mechanism for submitting and managing collections of similar jobs quickly and easily that utliizes only one job script. Job arrays are great at "automating" a repetetive job script that each time may only have input changes. Click [here](job_array.md) for the job array




