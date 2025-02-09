• Main singularity commands, do i need them actually?:
o Getting (pulling) a container from a container registry
singularity pull my_container.sif docker://ubuntu:22.04
o Opening a shell inside the container
singularity shell my_container.sif
o Executing a command inside the container
singularity exec my_container.sif python3 my_script.py

you bind files like this (that is old version, still supported):
When running a container on LUMI, where is
/project, /scratch, etc.?
• You may "inject" parts of the host (LUMI) file
system into the container by bind mounting it
singularity exec --bind /project/<project_ID>
my_containr.sif tree –L 1 /

but now you do:
You typically want to bind mount your project folders
(/project, /scratch, /flash). A shortcut is:
module use /appl/local/containers/ai-modules
module load singularity-AI-bindings

explanation:
The singularity-AI-bindings module shown in the slides sets up the directory mounts of /scratch/, /project/, /flash/, etc from the LUMI filesystem into your container. It also binds some essential system libraries into the container
to see what the module does, you can also do module show singularity-AI-bindings

For the LUMI application containers, you need to run $WITH_CONDA in the container to
activ ate the conda env ironment in which the application, e.g. PyTorch, is installed

On lumi login node, it can look like, container was found in `/appl/local/containers/sif-images/` :

$ module use /appl/local/containers/ai-modules
$ module load singularity-AI-bindings
$ singularity exec /appl/local/containers/sif-images/lumi-pytorch-rocm-6.2.1-python-3.12-pytorch-20240918-vllm-4075b35.sif bash -c "\$WITH_CONDA; python3 your_code.py"

On a LUMI-G, it can looks like:
module use /appl/local/containers/ai-modules
$ module load singularity-AI-bindings
$ srun --account=project_465001707 --partition=small-g --time=00:00:30 --nodes=1 --gpus=4 singularity exec /project/project_465001707/containers/lumi-pytorch-rocm-6.2.1-python-3.12-pytorch-20240918-vllm-4075b35.sif bash -c "\$WITH_CONDA; python3 your_code.py"

**It is a good idea to copy the lumi-pytorch-rocm-6.2.1-python-3.12-pytorch-20240918-vllm-4075b35.sif container to your project folder and run it from there to enable you to reproduce your results. We may remove or replace lumi-pytorch-rocm-6.2.1-python-3.12-pytorch-20240918-vllm-4075b35.sif at any point in time!**

Also, good example from [tutorial_5](https://github.com/Lumi-supercomputer/Getting_Started_with_AI_workshop/blob/ai-20250204/05_Running_containers_on_LUMI/reference_solution/reference_solution.md).
task:
Open an interactive shell in the LUMI TensorFlow+Horovod container

Open an interactive Python interpreter in the interactive container shell and (successfully) import horovod.tensorflow
Watch out, you have to activate conda environment!

code looks like:
module use /appl/local/containers/ai-modules
$ module load singularity-AI-bindings
$ singularity shell /appl/local/containers/sif-images/lumi-tensorflow-rocm-6.2.0-python-3.10-tensorflow-2.16.1-horovod-0.28.1.sif
Singularity> $WITH_CONDA
(tensorflow) Singularity> python3
Python 3.10.13 (main, Sep 11 2023, 13:44:35) [GCC 11.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.