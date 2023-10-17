This note aims to provide a quick guide through the Advanced Research Computing (ARC) for members of the Bogacz group at BNDU. It only covers the basic functions of ARC, to make sure our **first** script submissions to the ARC are as seamless as possible. Please follow the [ARC User Guide](https://arc-user-guide.readthedocs.io/en/latest/index.html) for more advanced usages and a deeper understanding of the whole system.

# Log-in

For a start, contact Rafal (our project owner) for setting up your account within our project `ndcn-computational-neuroscience`. After the account setup, in a terminal/powershell window type (make sure you are connected to Oxford WIFI or are using Oxford VPN): 

```ssh -X abcd1234@gateway.arc.ox.ac.uk``` 

where `abcd1234` is your Oxford email account name. Then follow the instructions on screen to log into the ARC cluster (only CPUs) or HTC cluster (also with GPUs). After this second login, you are in the "login/submit node" mentioned in the user guide. By default, after logging into the server, you will be located in the `$HOME` directory of this login node. This is your "personal" directory within the `ndcn-computational-neuroscience` project, which has a 15GB storage. You may also use the following command to go to the `$DATA` directory for a 5TB storage shared with other members of the Bogacz group:

```cd /data/ndcn-computational-neuroscience/abcd1234```

As suggested, the submit/login node is not for any computational task. For running your code, you can either use the interactive node or the computing nodes (see below for how to use them).

**Oct/2022 Update**: The official [ARC User Guide](https://arc-user-guide.readthedocs.io/en/latest/index.html) updated new instructions for graphical user guide.

# Data transfer between ARC and your local machine

The [ARC User Guide](https://arc-user-guide.readthedocs.io/en/latest/connecting-to-arc.html#copying-to-the-arc-systems) provides the basic syntax on this topic. It uses the following command to copy the data from your local machine to the ARC:

```scp local_path/filename arcuserid@gateway.arc.ox.ac.uk:/path_to_destination_directory/``` 

However, if you are copying your data into the `$HOME` area, this will copy your file to the gateway server and you won't be able to find your file in you specified directory in ARC or HTC clusters. This is because the gateway server (`gateway.arc.ox.ac.uk`) cannot access the `$HOME` area in either clusters. Replace the ```gateway``` above with ```htc-login``` or `arc-login` if you do not want to worry about this issue:

```scp local_path/filename arcuserid@arc-login.arc.ox.ac.uk:/path_to_destination_directory/``` 

For copying your data from the server to your local machine, simply swap the local path and the remote path in the command above. Notice that you should run both commands in a separate terminal/powershell window located in your local directory for copying data to/from your local computer.

Example: say I want transfer a `test.txt` file on my desktop to my own folder in the `$DATA` directory:

`scp desktop/test.txt abcd1234@arc-login.arc.ox.ac.uk:/data/ndcn-computational-neuroscience/abcd1234 `

# Submitting and running code

After copying your data and code into you directory in the ARC server, you can then run the code by submitting it to the computing node, or testing/preprocessing data within the interactive node. Here the instructions for Python and MATLAB are separate. However, some of them are interchangeable between these two languages (e.g. job submission scripts), so make sure you read them both! 

## Python

### Creating virtual environments (for Python users)

ARC uses Anaconda for environment management. [This page](https://arc-software-guide.readthedocs.io/en/latest/python/anaconda_venv.html) already provided some instructions for basic virtual environment creation, you may also refer to [this Anaconda page](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html) for other functions related to virtual environments. ARC recommended using `source activate` rather than `conda activate` for activating your created virtual environment due to some conflicts with the job submission system.

In terms of installing libraries in your own virtual environment, for most libraries you can simply install them as usual using `conda install`. However, since most of our group members using Python need deep learning frameworks, here's a list of available CUDA versions on the ARC server:

```bash
11.4.1-GCC-10.3.0
9.2.88-GCC-7.3.0-2.30
10.1.243-GCC-8.3.0
11.4.1
11.3.1-GCC-10.3.0
11.1.1-iccifort-2020.4.304
10.1.243-iccifort-2019.5.281
11.2.2
11.3.1
11.0.2-GCC-9.3.0
11.2.2-GCC-10.3.0
11.1.1-GCC-10.2.0
```

This is particularly important because some DL frameworks like PyTorch will ask you to specify the version of CUDA during installation. 

### Submitting a job

To run your code on the computing nodes, you have to submit it to a queue where you wait until you are assigned a node. For Python virtual environment users, it is also required that you deactivate the current virtual environment and then submit the job, because you are going to specify the virtual environment during submission. The submission is through a `.sh` script specifying what resources you need. For example let's look at the following submission script `test.sh`:

```shell
#!/bin/bash
# set the number of nodes
#SBATCH --nodes=1
# set max wallclock time
#SBATCH --time=00:20:00
# set name of job
#SBATCH --job-name=test1
# set gpu 
#SBATCH --gres=gpu:1
# mail alert at start, end and abortion of execution
#SBATCH --mail-type=ALL
# send mail to this address
#SBATCH --mail-user=your.name@bndu.ox.ac.uk

# Specifying virtual envs
module load Anaconda3
source activate $DATA/myenv

# run the application
python test_model.py
```

This submission script requires 1 GPU computing node for 20 minutes. The name of this job is test1, and the system will send you an email whenever the job starts, ends, fails or is requeued. The last few lines are for submitting Python scripts, which specify the virtual environment and run a script called `test_model.py`. 

Notice that you will need to prepare this `test.sh` file on your local machine along with your code, and copy it to the remote server. You can then run your code using the following `sbatch` command in your working directory:

```sbatch test.sh``` 

After this command your script is in the queue, waiting to be assigned a node/nodes. If you asked for email notifications in your `.sh` file, you will receive an email telling you your script is beginning to run. You can check the status of your job using the `squeue` command.

I haven't found the way to monitor the real-time output of your code when running in the computing nodes on ARC. For example, you may want your code to print the loss value for every training iteration on screen, but you won't be able to see this with the computing nodes. However, after your code starts to run, you will find a file called `slurm-123456.out` in your current working directory, which keeps a record of your real-time outputs. The `123456` here is your job id, which will be shown to you after your submission. You can also change the location of the saved output file by adding the following line to your submission `.sh` file:

```shell
#SBATCH --output=/data/ndcn-computational-neuroscience/abcd1234/YOUR_DESIRED_LOCATION/%j.out
```

### Using the interactive session 

Run the command to initialize an interactive session, which is basically a computing node assigned to you but with limited computing power:

```srun -p interactive --pty /bin/bash```

When running code in this interactive session, you will be able to see the real-time output of your code, as well as any runtime errors. This is particularly useful for testing and debugging your code before submitting it to the computing nodes. You may have already debugged it on your local machine first, but the system configuration of the ARC server may be very different from that of your local machine, so it is better to see if it runs well on the interactive nodes, which shares the system configuration with the computing nodes. This is also an ideal place to pre/post process your code and model. For example, having trained the model and logged training losses, I will use the interactive session to visualize the logged loss values. Notice that you can also specify your requirements for the interactive nodes using a `.sh` file:

```srun -p interactive --pty /bin/bash test.sh```

You can also use GPUs in an interactive session in the HTC clusters. To use it, start the interactive session using:

```srun -p interactive --gres=gpu:1 --pty /bin/bash```

Also, notice that the maximum time on the interactive sessions is 4 hours, and after that your script will be terminated. So make sure you only test and debug your code there!

### Using VScode to connect to the server

If you have successfully executed your code on the ARC and obtained your results, say a plot, you will need to `scp` the plot to your local machine to see if everything is in place. If, for example, you find your plot colored incorrectly by your code, you will have to modify your code and `scp` it to ARC, run it, generate the plot and then `scp` the plot to your local machine to view it again. This sounds a bit exhausting. One way to get away from this is to use VScode to connect to the ARC and you can then view your plot, modify your code and view your plot again on the remote folder, without the need to `scp` everything back and forth. The official instructions for doing so is [here](https://code.visualstudio.com/docs/remote/ssh). 

## MATLAB

🎈*Full credit to Charlotte for writing this section for MATLAB users*🎈

There are two key methods to run MATLAB on the arc, which take advantage of the parallel processing inbuilt into the GUI – using command line (clunky) or using the ARC’s graphical interface. If your job is quite a simple one, and doesn’t require more than 1 node, you can just use the inbuilt [parfor function](https://uk.mathworks.com/help/parallel-computing/parfor.html), and the command line submission which ARC User Guide goes into [here](https://arc-software-guide.readthedocs.io/en/latest/apps/arc_matlab.html). This is probably the easiest approach for those already familiar with SLURM.

For those unfamiliar with SLURM (or who just want to avoid it), I recommend starting with the second to get to grips with it. This is the method I describe below.

This method assumes that you have already created an ARC account, and have your password to hand. I am also working on Windows 11, and the exact process may be different for other devices.

### Connect to the NoMachine Client

The ARC guidelines for how to do this can be found [here](https://arc-user-guide.readthedocs.io/en/latest/arc-nx-web.html).

The NoMachine Client is a browser-based linux desktop on a server. You log in using your ARC credentials.

Then, once you have opened a desktop (you will need one per job running simultaneously), you can open MATLAB 2022B (current latest version available).

### Set up parallel clusters

ARC already has the Matlab default cluster settings available in their files. The step-by-step for adding these to your NoMachine Matlab session can be found [here](https://arc-software-guide.readthedocs.io/en/latest/apps/arc_matlab_mps.html), under ‘import cluster profiles’.

Repeat these steps for whichever ARC/HTC resource type you would like to use (devel, short, medium, long).

Remember to validate the profile, and then set it as your default.

#### Note: 

There is a work around to this through using MATLAB functions, where you can begin your script with the following: 

```bash
arc_profile = parallel.importProfile ('/apps/common/commercial/MATLAB/mps_profiles/R2022b/arc_devel.mlsettings');

parallel.defaultClusterProfile ('arc_devel');
```

Your NoMachine desktop is now set up to run Matlab scripts.

### Transfer your job script to the machine

Using whatever method you prefer (I like WinSCP), transfer the MATLAB job script and all other required scripts into your `$DATA/userID` folder.

You can create the job script in the NoMachine client, but I find that it’s often slow and buggy, so this is my preferred method.

### The Skeleton Script

Here’s a skeleton script which you can run using parfor over multiple nodes:

```
%
%% ARC Parallel MATLAB 
 
% Have arc-xxx / htc-xxx cluster profile set as default in GUI or uncomment the
% following two lines:
 
% parprof = parallel.importProfile("/apps/common/commercial/MATLAB/mps_profiles/R2022b/xxxxxxx.mlsettings")
% parallel.defaultProfile(parprof)
 
%%% Create all initial parameters and serial code
 
%%% Create parallel pool
 
poolObj=parpool('arc_short',96); % name the cluster profile you want to use and the number % of workers
 
parfor w = 1:xxxx
%         Run the work in parallel (e.g. over a set of parameters)
end
 
% Shutdown parallel pool.
delete(poolObj);
 
% Then whatever you want to do with the parallelised data
% I recommend saving the workspace!
 
 
```

It’s really that simple! You then just run the code from within the NoMachine GUI as you would any other MATLAB script. It will automate the number of nodes/CPUs to run over.

The parpool function creates the SLURM script and submits it for you.

Note: You may also notice a series of ‘Jobx’ files opening in your current folder – do not delete these!

### Checking on your code

If you want to find your jobID and progress, I find the easiest way is to go back to your normal desktop (you can close the browser – nx.arc.ox.ac.uk continues running without you), then log into the ARC using the normal ssh method.

Then if you type `squeue -u -userID`, you will be able to see the details of any jobs you are currently running. 

# Usage and credit management

By default, users consume credits from a pool belonging to their project (e.g., our `ndcn-computational-neuroscience` project) when they run jobs. Our project is initialized with 25,000 core hours, and I have consumed a few hours to write this note. The credit we have is basically core hours $\times$ 3,600. After you log in to your login node, you can use the command `mybalance` to see our current available credits, which shows you something like this:

```shell
Detailed account balance:
Id    Name                                     Amount   Reserved Balance  CreditLimit Available 
----- ---------------------------------------- -------- -------- -------- ----------- --------- 
93646 ndcn-computational-neuroscience                 0        0        0           0         0 
93647 ndcn-computational-neuroscience-priority        0        0        0           0         0 
93648 ndcn-computational-neuroscience-basic    89990684        0 89990684           0  89990684
```

There are 3 levels of services on ARC: **basic, standard, priority**. Details of these levels can be found [here](https://www.arc.ox.ac.uk/arc-service-level-agreements). As you can see above our project only has credits on the basic level, which has the lowest priority in the queue. Based on my testing, a job submission with a short duration (e.g., 20mins) will have no problem in terms of priority and will run immediately after submission. 

**Purchasing credits with different levels of service:** as a group within the Medical Sciences Division, we can request basic-level credits for free by emailing [support@arc.ox.ac.uk](mailto:support@arc.ox.ac.uk). However, anything beyond this requires additional costs. The current pricing is £0.01/Core.h for the standard service, £0.02/Core.h for the priority service, and £0.08/Core.h for using GPUs. [This website](https://www.arc.ox.ac.uk/arc-accounting) gives some helpful examples of calculating the credit usage. Notice that it says the standard service is free to request - this is not the case for MSD from my understanding, but it does no harm asking.

**Running jobs with priority service**: [Here](https://arc-user-guide.readthedocs.io/en/latest/arc-priority-jobs.html) is a guideline.

Another important issue with basic service level is that we can only run one job at a time and it is not even possible to open an interactive session to test our scripts when another job is running. **Good news Oct/2022:** Rafal has purchased the standard service, so this is no longer a concern now.

# Miscellaneous

This website provides a full list of installed libraries and software on ARC: https://arc-module-list.readthedocs.io/en/latest/

The commands `sbatch`, `srun` etc. we used before all belong to the workload management software called SLURM. There are many advanced usages of these commands for you to add more specifications to your submission, and a summary of SLURM commands can be found in this pdf: https://slurm.schedmd.com/pdfs/summary.pdf

Cancelling a job: `scancel 123456` where `123456` is your job id.

`ps; kill -9 PID` to terminate a process

