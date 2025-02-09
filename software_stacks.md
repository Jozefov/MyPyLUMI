
# LUMI Software Stacks and Module System

## 1. What Are Software Stacks?

On LUMI, the globally installed software is organized into [software stacks](https://docs.lumi-supercomputer.eu/runjobs/lumi_env/softwarestacks/#crayenv) that you access using the [LMOD](https://docs.lumi-supercomputer.eu/runjobs/lumi_env/Lmod_modules/) module system. These stacks define which compilers, libraries, and tools (including Python) are available to you. The main types are:

- **CrayEnv**:
Provides the base Cray Programming Environment (PE) with a limited selection of additional tools. It sets up the minimal, hardware-aware context. Including default compilers, MPI, and core libraries, that you need to compile and run applications on LUMI
- **LUMI (e.g., LUMI/22.08)**:
Building on CrayEnv, the LUMI software stack is a more complete, curated collection of software packages (like Python, MPI, and scientific libraries) that are pre-installed. For example, when you load a module like:
    
    ```bash
    module load LUMI/22.08 partition/G
    ```
- **EasyBuild**:
EasyBuild is a tool that allows you to extend the central LUMI software stack by installing additional software in your own space (/users/<username>, /scratch/<project>,...).


## 2. Python, Conda, and Virtual Environments on LUMI

- **Pre-installed Python and Conda**:
The LUMI software stacks include a pre-configured Python installation. Many modules (such as the PyTorch module) not only provide the software but also bundle a Conda environment inside a Singularity container. 
- This means:
  - You don’t have to create your own virtual environment manually.
  - Environment variables (such as SIF for the Singularity image and WITH_CONDA for activating the Conda environment) are automatically set when you load the module.
- **Why Use the Container’s Conda Environment**?
The containerized Conda environment is tailored to work with the pre-installed libraries and versions on LUMI. It ensures that all dependencies (like PyTorch, DeepSpeed, FlashAttention and others) are compatible. When you run your jobs, you’ll typically execute Python commands inside this container. Then you do not need to manage a separate Python virtual environment.
- **Custom Software Installations (Advanced)**:
    If you need software that isn’t in the central stack, you can use EasyBuild to install additional packages in your home or project directory. For more help, you can ask [LUST team](https://lumi-supercomputer.github.io/LUMI-training-materials/2day-20241210/M05-SoftwareStacks/).

## 3. How Does This Affect Your Workflow?


```bash
module purge
module load CrayEnv
module load LUMI/22.08 partition/G
module load PyTorch/2.2.0-rocm-5.6.1-python-3.10-singularity-20240209
```

- your environment is automatically configured with:
  - Correct system libraries and compilers (from CrayEnv and the LUMI stack).
  - A pre-built Singularity container that includes Python and PyTorch, with a Conda environment that’s activated via the `WITH_CONDA` variable.

Then you can execute:
```bash
singularity exec $SIF bash -c 'eval "$WITH_CONDA" && python -c "import torch; print(f\"Number of GPUs: {torch.cuda.device_count()}\"); print(torch.cuda.get_device_properties(0))"'
```
