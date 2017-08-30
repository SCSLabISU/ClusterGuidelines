# Cluster Guidelines
This page is for providing some guidelines on how to use SCSLab Cluster effectively, without any glitches.

SCSLab is maintained by the [High Performance Computing Group](hpc.iastate.edu) who maintain all the clusters at ISU. SCSLab Cluster is equipped with one head node and two compute nodes. Both the compute nodes have 4 GPUs each. It is running on Red Hat Enterprise Linux system (version 7.4).

## How to login to cluster

To access the compute nodes of the cluster, you should first login to the head node of the cluster. You can SSH to head node by the following command in bash terminal (In windows, you can either use the linux subsystem beta version of bash or use softwares like putty to access the cluster.)

`SSH netID@scslab01.me.iastate.edu`

For more details on how to logon is provided [here](cluster_usage_basics.pdf). This is also found in [http://www.hpc.iastate.edu/guides](http://www.hpc.iastate.edu/guides)

First time when you login, you would receive an email for registering the Two Factor authentication. You need to use google authenticator or authy for generating the code.

You should ssh to the compute nodes (node2 and node3) for all your computations. The head node doesnt have much of RAM to run your computations.

eg. `ssh node2` or `ssh node3`

Note that, If you are accessing the cluster from outside ISU campus, then you must use VPN.

## Basic Unix Instructions

Basic Instructions on how to access the cluster and navigate in the bash environment is provided [here](Notes_unix.pdf). VI editor is one of the fundamental editors for editing the codes in the cluster. There are some other editors like emacs, atom etc. which are not available in cluster. But gedit is available in the cluster. Simple instruction on how to use vi editor are provided [here](Notes_unix_vim.pdf).

## Hardware information

Besides the instructions above, It is recommended to keep track of the RAM usage of both CPU and GPU. 

`top` is used to see all the codes running currently in cpu. Most of the deep learning codes take a lot of memory in both CPU and GPU. Specifically when the dataset is huge. The memory in the CPU limits the total amount of data you can use for training.

`nvidia-smi` is useful for looking at the utilization of the GPU Memory. GPU memory doesn’t limit the total data which we can load but limits the model size of the Network and the batch size you can use for training.

When you initialize a deep learning code, it initializes the model and then loads the data and then sends model parameters to GPU and then in a small batch specified by batch size the training data is sent to the GPU for updating the model parameters. After the training is completed, the model parameters are copied back to the CPU.

You should be cautious about these “parameters” affecting your computations and others’ while choosing the appropriate nodes for a Job. Example: If you have a very small dataset which will use very less CPU and GPU RAM, it is preferable to use the titanX instead of P40. Since there is no formal scheduler (though we are working with HPC team to have a better allocation by a slurm job scheduler), it is upto each person’s wise choices on usage the resources and talk to others while to schedule things.

## Softwares Available
Every user has their own requirements on what softwares one wants to use and it wouldn’t be same for everyone. So we use something called as modules to standardize the environment for all the users and make all the softwares modular so that one can load the modules needed.

To check what are the modules you have currently loaded you could use: `module list`

To purge the modules you have currently loaded, use `module purge`

To check the available modules, use `module avail`

To load a particular module, use `module load m1 m2`, m1 and m2 are the modules you want to load.

### Best practices to work faster and efficiently

1. You could add the modules you want in your bashrc file
2. you could use a shell script file to set the modules while running your code.
3. Bash aliases is also another option for that.

Shell scripting is one of the powerful ways to automate a lot of efforts which people would do manually. Also ensures that the codes are reproducible.

Another important best practice to avoid loss of data while running the codes in the cluster is to use nohup.

`Nohup python filename.py arguments >run.out &`

This stores the output which was supposed to be printed in the screen to run.out file and runs the code in background ensuring that your code doesn’t stop when the connection to cluster is lost while working remotely.

Note: killing the background processes started by you also has to be done by you.

To identify what all processes are running because of your user, use `ps aux | grep username` 

Use the PID from the search to kill the process.

### X-Forwarding 

Some softwares would need X forwarding for using interactive window. For eg. Matlab can run in command prompt or you could get an interactive window to work on Matlab. Rviz or Gazebo would need X-forwarding as well.

So, while you login you need to use SSH -X instead of SSH for X-forwarding. Sometimes, we would even need port forwarding, which can be done by SSH -L. 




### Specific Platform examples
For almost all the packages you would use gpus irrespective of what platform you are using. Thus loading cuda module is necessary for all the below tools. This might be something you might want to add in your bashrc as a best practice.
The Cluster has few deep learning platforms installed for all the users:
1. Keras
This is installed in the module python/3.6.0
2. TensorFlow
This is installed in the module python/3.6.0 python/2 and python/3.4.3.
You could always load a python version and check if the package you want is installed by issuing `pip list` or `pip3 list` based on the python version you are using is python 2 or 3 respectively.

3. Theano
This is installed in module python/3.6.0

4. Torch
This is installed as a module named torch. You could start using it by issuing `module load torch`

5. Matlab

There are different versions of Matlab installed. One could search for the useful one by issuing `module avail` and load accordingly.

6. Tensorboard

Tensorboard is installed with tensorflow. But usage of tensorboard is a bit tricky.

You would need to launch a tensor board session and then use then use Firefox in the cluster with X Forwarding and visualize the performance of the model.

Most of the tools are installed in some versions of python and some of the tools are not installed yet in the cluster. Please see next section about how to install softwares which are specific to you alone.

## Installation of new packages

There are a lot of packages in the world wide web. Installing everything for every user and every version possible is an unintelligent thing to do. Further some of these packages intersect with others and causes problems for other users. For this reason, sudo access(admin privileges) is not provided to every user. Sudo access makes your installation affect other user’s environment.

Does that mean we can’t install anything at all? No.

We can install the same software such that it only works for your user. Eg. If the instruction given by the package is `pip/pip3 install XYZ` you would use `pip/pip3 install XYZ —-user`. That way you can install it in your environment without affecting other users.

Here are few packages I have tried and installed successfully to run the codes.

1. CNTK

This is a tricky installation.

The installation needs usage of module openmpi/1.10.X for the installation. You could traditionally install using `pip3 install —-user` But there is a small patch which needs to be applied to make it run in RHEL, the built version in /shared/hpc/cntk folder of the cluster, please use that wheel file for a successful installation.

2. Chainer
Chainer is not tricky at all. The dependency of cupy is tricky, you need to add a prefix environment variable of `CUDA_PATH=‘opt/rit/app/cuda/‘ pip3 install —-user cupy` (verify the location)

There might be a need to create a folder in /scratch/ for your username.
Here is a perfect example to demonstrate the power of modules which is there in the cluster. Chainer doesn’t work well with the module gcc/6.3.0. So, ensuring that this is not loaded when running Chainer is necessary.

3. ROS

ROS Indigo is installed in the cluster. Rviz is also installed with it. Gazebo will soon be available for usage.

Module load ros-indigo will load the module. Also, velodyne driver is also installed for the self driving car people to use the cluster remotely.


4. MXNet

Specific instructions for this will be updated soon

5. Caffe

Specific instructions will be updated soon.

If one would need surely sudo access (which is not the case for most of the times) then they can contact [hpc-help@iastate.edu](email:hpc-help@iastate.edu). 

### Best Practices

If one is working in docker containers or virtual environments, one wouldn’t need to worry about the sudo access as those will never affect other users.

## Docker and Virtual Environment

 



## List of interesting and useful repositories
This section shall be a curated list of useful repositories and tools which I would update as and when I encounter anything new or based on what you share with me.

[keras2cpp](https://github.com/pplonski/keras2cpp)

[kerasvis](https://github.com/raghakot/keras-vis)


## Contact
If this is platform specific questions, post it in Slack team so that everyone can have a look at it and anyone can answer it.
[slack account invite](https://join.slack.com/t/scslabteam/shared_invite/MjI4MDQwOTk1MDkwLTE1MDI5ODQ0NjYtNGM4OTU1ZTcyNQ)

If this is about running adding new platforms and installing new packages in cluster other than those mentioned above then, I would suggest do some homework and then contact hpc-help@iastate.edu only if there is something needing sudo access.

Final point, HPC community @ ISU is there for our complete support. Their only goal is that they will remove all the hardware and software issues from our head and let us focus on our research work. They are ready to help us to the extent of running our codes to verify if the installation is correct or not. Let’s respect them and take their help wisely so that we don’t waste our time and we don’t waste their time unnecessarily.

In case of any generic questions or any specific things to be added to this webpage, please contact [me](email:baditya@iastate.edu).





