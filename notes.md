
## 1. Resource Allocation on a Node

## One GPU, One-Eighth of a Node

- **Resources per GPU Slice**:
	**CPU Memory**: Each compute node has 512GB of CPU memory, but only 480GB are available per user. When divided into 8 equal slices (since 1 node is effectively split into 8 slices for the GPUs/GCDs), each slice gets 60GB of CPU memory.
- **CPUs**: Each GPU slice is allocated 7 CPU cores.
- **SLURM Option**:
•	In the SLURM script, you request --mem-per-gpu=60G because 60GB is the fair share per GPU slice (1/8th of 480GB).
•	**Note**: This memory is CPU RAM, not the GPU’s onboard memory (GPUs memory you have 64GB per GPU).


### Example SBATCH Script:
```bash
#!/bin/bash
#SBATCH --account=project_num
#SBATCH --partition=small-g # this allow you reserver just numbers of GPUs note whole node
#SBATCH --gpus-per-node=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=7
#SBATCH --mem-per-gpu=60G
#SBATCH --time=0:15:00
```

## 2. Available GPU Partitions

- **standard-g**:
  -   Maximum runtime: ≤ 48h
  -   **Reserves whole nodes only**
  -   Up to 1024 nodes per job
- **small-g**:
    -   Maximum runtime: ≤ 72h
    -   **Allocates individual GCDs (good for single-GPU jobs)**
    -   Up to 4 nodes per job
- **dev-g**:
    -   Maximum runtime: ≤ 3h
    -   Allocates individual GCDs
    -   Up to 32 nodes per job (with a maximum of 2 jobs running concurrently)

## 3. Checking GPU Utilization and PyTorch Verification
- **Checking Utilization**:
    - Launch an interactive session on your job’s compute node:
```bash
srun --overlap --pty --jobid=<jobid> bash
```
- Then, run:
```bash
rocm-smi
```
This command displays GPU status and utilization.

- **Verifying in PyTorch**:
  - Even though LUMI uses ROCm, PyTorch uses the cuda namespace. To confirm your GPUs are visible:
  - But first you have to git torch in! Solved later, now we reserved GPUs, but have not placed torch in.
```bash
singularity exec $SIF python -c "import torch; print(f'Number of GPUs: {torch.cuda.device_count()}'); print(torch.cuda.get_device_properties(0))"
```

## 4. Understanding Singularity and Modules

## What is Singularity?

- **Singularity (also known as Apptainer)**:
    - It is a container technology designed for HPC environments.
    - **Container**: A container is a runtime instance of an image. In Singularity, container images are stored as .sif files (Singularity Image Files).
  - **Image vs. Container**:
    - **Image**: The file (e.g., lumi-pytorch-rocm-6.0.3-python-3.12-pytorch-v2.3.1.sif) that contains the entire software stack (Python, PyTorch, ROCm libraries, etc.).
    - **Container**: The running instance created when you execute the image. You might run the same image many times, each time creating a new container.
      - When you execute (or “run”) an image using a tool like Singularity, you create a container—an isolated environment in which your application runs
      - `singularity exec $SIF python your_script.py` here, $SIF is an environment variable set by the module that contains the full path to the correct Singularity image.
  - **Why Use a Container?**
    - It bundles all dependencies (including specific versions of PyTorch, CUDA/ROCm libraries, etc.) so that you don’t have to install packages on the shared system. This reduces stress on the network file (Conda created tens of thousands small files, which has to be queried each time) system and ensures consistency across runs.


- To find images of containers at prebuild torch-roc, see:
```bash
ls /appl/local/containers/sif-images
```
![types_of_nodes](assets/img_7.png)

## What are Modules?

- **Modules**:
  - HPC systems like LUMI use Environment Modules to manage different software versions, more info [here](https://docs.lumi-supercomputer.eu/runjobs/lumi_env/Lmod_modules/#module-avail).
  - Modules let you quickly switch between different software versions (e.g., Python, compilers, or libraries) and ensure that the right dependencies are in your environment.
  - **module load <module-name>**: Loads a specific software package (or a set of packages) into your environment.
  - **module use <path>**: Adds a directory to the module search path.
    - How to create own modules can be found here, [Lmod](https://lmod.readthedocs.io/en/latest/index.html).
  - **module purge**: Removes all currently loaded modules. This ensures that your environment starts clean, reducing conflicts between different software versions.
- **Module Spider**:
  - The command module spider searches for modules and shows you all the available versions and any additional steps needed to load them.
  - Running in bash `module spider pytorch` might output:
    - ![types_of_nodes](assets/img_8.png)
    - 
- **Comparison of Modules and Singularity images**:
  - **module spider pytorch**:
    - Searches through the module system.
    **Returns a module name (e.g., PyTorch/2.2.0-rocm-5.6.1-python-3.10-singularity-20240209) that, when loaded, sets up your environment (including pointing to the proper container image).
  - **ls /appl/local/containers/sif-images**:
    - Lists all Singularity image files available on the system.
    - You might see an image like lumi-pytorch-rocm-5.7.3-python-3.12-pytorch-v2.2.2.sif here.
    - The module system abstracts this detail for you, so you don’t have to remember the full path or exact name when setting up your job.

## **EXAMPLE**

In this example we start an interactive session, load the required modules, and run a Python command to verify that PyTorch sees the GPUs. Follow the steps below:

### 1. Start an Interactive Session

Run the following command to reserve a node with 2 GPUs and 2 CPU cores using the dev-g partition:
```bash
srun --export=ALL --preserve-env --account=project_<project_number> --partition=dev-g --time=0:30:00 --nodes=1 --cpus-per-node=2 --pty bash
```

- **srun**: Launches a job via SLURM.
- **–export=ALL –preserve-env**: Ensures your current environment variables are passed along.
- **–account, –partition, –time, –nodes, –cpus-per-node**: Specify your project account, partition type, job duration, number of nodes, and CPU allocation.
- **–pty bash**: Opens an interactive bash shell on the compute node.

### 2. Load the Required Modules
Once in your interactive session, prepare your environment by loading the necessary modules:
```bash
module purge
module load CrayEnv
module load LUMI/22.08 partition/G
module load PyTorch/2.2.0-rocm-5.6.1-python-3.10-singularity-20240209
```
- **module purge**: Clears any preloaded modules for a clean environment.
- **module load CrayEnv**: Loads the base Cray environment.
- **module load LUMI/22.08 partition/G**: Loads the appropriate LUMI software stack for your partition (required by pytorch, found by `model spider pytorch` command).
- module load PyTorch/…: Loads the PyTorch module that provides a Singularity container image along with the necessary environment variables (e.g., SIF and WITH_CONDA).

You can verify that the PyTorch module has set the Singularity image path by running:
```bash
echo $SIF
```
Thi will return somethign like: `/users/your_account/EasyBuild/SW/container/PyTorch/2.2.0-rocm-5.6.1-python-3.10-singularity-20240209/lumi-pytorch-rocm-5.6.1-python-3.10-pytorch-v2.2.0-dockerhash-f72ddd8ef883.sif`

### 3. Execute a Python Command Inside the Container

Run the following command to activate the container’s Conda environment and execute a Python command that checks the available GPUs:

```bash
singularity exec $SIF bash -c 'eval "$WITH_CONDA" && python -c "import torch; print(f\"Number of GPUs: {torch.cuda.device_count()}\"); print(torch.cuda.get_device_properties(0))"'
```

- **singularity exec $SIF bash -c ‘…’**: Runs a bash shell inside the Singularity container specified by the SIF environment variable.
- **eval “\$WITH_CONDA”**: Activates the internal Conda environment within the container.

**Why this is necessary**:

Unlike some containers that automatically set up the environment, the PyTorch container on LUMI uses a Conda environment that isn’t active by default. Running eval "$WITH_CONDA" ensures that the python executable (and all other dependencies) become available in the container’s PATH. This information were find by `model spider pytorch` command.

### OUTPUT
```bash
Number of GPUs: 2
_CudaDeviceProperties(name='AMD Instinct MI250X', major=9, minor=0, gcnArchName='gfx90a:sramecc+:xnack-', total_memory=65520MB, multi_processor_count=110)
```
This confirms that the container is running correctly, the Conda environment is activated, and PyTorch can detect the available GPUs.


    