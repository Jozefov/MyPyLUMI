# Using Python on CSC Supercomputers

Some important aspects of working with the Python programming language are notably different on CSC supercomputers compared to usage on a personal device or in other HPC environments. To make the most of the computational resources available to you, it is helpful to be aware of the differences.

See the *Python application page* for general information on the Python language and pre-installed Python environments on CSC supercomputers.

## Creating and Managing Python Environments

See also our *FAQ section* with common questions related to Python environments on supercomputers.

### Installing Python Packages to Existing Modules

If there is a CSC-provided module that covers almost everything you need, but it is missing a few Python packages, you can try installing those yourself with the `pip` package manager.

See the package lists on our *Python application page* to find out which packages are installed in existing modules. If you think that some important package should be included by default in a module provided by CSC, do not hesitate to contact our *Service Desk*.

### Using `venv`

**Using `pip install --user`**

The recommended way to add packages on top of an existing environment is to use `venv`, which is a standard Python module for creating a lightweight "virtual environment". You can have multiple virtual environments, for example one for each project.

For example, to install a package called `whatshap` on top of the CSC-provided `python-data` module:

```bash
cd /projappl/<your_project>  # change this to the appropriate path for your project
module load python-data
python3 -m venv --system-site-packages <venv_name>
source <venv_name>/bin/activate
pip install whatshap
```

Unlike for example Tykky, `venv` creates a new directory for the environment, so there is no need for you to create one beforehand. **Do not forget to use the `--system-site-packages` flag** when creating the virtual environment, otherwise the environment will not find the pre-installed packages from the base module (for example `numpy` from `python-data`).

Later when you wish to use the virtual environment, you only need to load the module and activate the environment:

```bash
module load python-data
source /projappl/<your_project>/<venv_name>/bin/activate
```

Likewise, when using the virtual environment, make sure to actually have the base module loaded. Naturally, this also applies to Slurm job scripts.

## Problems with Virtual Environments

In some specific cases CSC modules may not work properly with Python virtual environments. Try first to run:

```bash
export CW_FORCE_CONDA_ACTIVATE=1
```

before activating the venv. If that does not help, try to use the pip install --user approach described on the other tab.

Creating your own Python environments
It is also possible to create your own Python environments.


pip
conda
Pip is a good choice for managing Python environments that do not rely on complex dependency relationships.

The easiest way to create a custom pip environment is by using the venv module discussed in the previous section, which actually shows precisely how to do this. If you do not wish to use packages from one of the existing modules, simply do not include the
--system-site-packages flag when creating the virtual environment.

Another option is to create a pip environment inside a container. The most straightforward way to do so is by using the Tykky container wrapper. To find out how to easily containerize your environment, see the Tykky instructions for pip-based installations.

An alternative to using Tykky is creating a pip environment inside a custom Apptainer container. This is a practical choice if, for example, you know of a suitable ready-made Apptainer or Docker container. For more information about using Apptainer containers, please see the related documentation:

Running Apptainer containers
Creating Apptainer containers, including how to convert Docker containers to Apptainer containers.

Python development environments
Python scripts can be edited directly on a CSC supercomputer using a console-based text editor like vim or emacs. In addition to these terminal-based editors, several graphical programming environments, such as Jupyter notebooks, Visual Studio Code and Spyder, can be used on a supercomputer through our web interface.

In addition to editing code directly on a supercomputer, it is also possible to develop code remotely using some locally installable editors like Visual Studio Code. Please note, however, that this way of connecting to CSC supercomputers is prone to issues and thus not fully supported.

Finally, one can of course edit code on a local device and copy it to a supercomputer with command-line tools like scp and rsync, or by using graphical file transfer tools.

Jupyter
Jupyter notebooks provide an interactive programming environment where one can write and run Python code in individual cells. The notebooks combine code, equations, visualizations and narrative text in a single document.

The Jupyter interactive application on our web interface allows using Jupyter on CSC supercomputers. Many of our Python environments, including python-data, geoconda as well as deep learning modules like pytorch include the main Jupyter packages, so they can be used in the application. The documentation page for the application includes a list of supported environments.

Visual Studio Code
Visual Studio Code (VSCode) is a widely-used source code editor developed by Microsoft. Unlike the other two development environments introduced here, it does not rely on any Python packages, so it can be used by default with all CSC- and custom-made Python environments.

There are two ways to run VSCode on CSC supercomputers:

As an interactive app on our web interface
Remotely using the Remote-SSH plugin (unsupported)
Using custom environments in VSCode

Since only CSC modules are offered in the VSCode session launch form, using custom Python environments with built-in VSCode functions like debugging requires changing the path of the Python interpreter after the session has launched. This can be done by clicking on the Python version information displayed in the lower right corner of the VSCode window.

Spyder
Spyder is a scientific Python development environment. The python-data and geoconda modules have Spyder included. The best option for using it is through the Puhti web interface remote desktop.

Python parallel jobs
There are several Python libraries for parallel computing. Below are a few suggestions:

multiprocessing – process-based parallelism
joblib – running Python functions as pipeline jobs
dask – general purpose parallel programming solution
mpi4py – MPI bindings for Python
The multiprocessing package is likely the easiest to use. Being part of the Python standard library, it is included in all Python installations by default. joblib provides some more flexibility in comparison. These two packages are suitable for single-node parallelization (max. 40 cores).

dask is the most versatile of the bunch and has several options for parallelization. Please see the CSC Dask tutorial for examples of both single-node (max. 40 cores) and multi-node parallelization.

In addition, there are examples of using different parallelization options on Puhti on our CSC Training GitHub organization. Of the above four packages, examples are provided for multiprocessing, joblib and dask.

The mpi4py package is included in our PyTorch environment. It is generally the most efficient option for multinode jobs with non-trivial parallelization. For a short tutorial on mpi4py, along with other approaches for improving the performance of Python programs, please see the free Python in High Performance Computing online course.