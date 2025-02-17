The tools CSC provides for scientific computing and data management create a stepping stone for breakthroughs. We support Finland’s unique educative system. We connect Finnish higher education institutions, researchers, and students to each other and to the whole world. In 1988 we connected Finland to the Internet. And is also managing LUMI, Puhti, Mahti and provide much easier documentation to get started.

Load the CSC module tree into use with:
module use /appl/local/csc/modulefiles

there you have also torch, and from my perspective looks more convenient. Docs and version release can be found [here](https://docs.csc.fi/apps/pytorch/).

All modules are based on containers using Apptainer (previously known as Singularity). Wrapper scripts have been provided so that common commands such as python, python3, pip and pip3 should work as normal. So it is not that complicated as with your own calls of singularity exec commands.

module use /appl/local/csc/modulefiles/ # this enables to list modules provided by CSC organization
module load pytorch

if specific version:
module load pytorch/2.4

Each of this modeuls come also with environment, and can be checked, [here](https://a3s.fi/python-pkg-lists/pytorch.txt), all python modules are [here](https://docs.csc.fi/apps/python/#pre-installed-python-environments). 

To check the exact packages and versions included in the loaded module you can run:
list-packages

run interactive:
srun --export=ALL --preserve-env --account=project_465001738 --partition=dev-g --time=1:00:00 --nodes=1 --gpus-per-node=1 --cpus-per-task=7 --pty bash

sbatch script for csc version:
#!/bin/bash
#SBATCH --account=<project>
#SBATCH --partition=small-g
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=7
#SBATCH --gpus-per-node=1
#SBATCH --mem=60G
#SBATCH --time=1:00:00

module use /appl/local/csc/modulefiles/
module load pytorch/2.4
srun python3 myprog.py <options>

The recommended way to add packages on top of an existing environment is to use venv, which is a standard Python module for creating a lightweight "virtual environment". You can have multiple virtual environments, for example one for each project.

For example to install a package called whatshap on top of the CSC-provided python-data module:

cd /projappl/<your_project>  # change this to the appropriate path for your project
module load python-data
python3 -m venv --system-site-packages <venv_name>
source <venv_name>/bin/activate
pip install whatshap

Later when you wish to use the virtual environment you only need to load the module and activate the environment:
module load python-data
source /projappl/<your_project>/<venv_name>/bin/activate

In general, keep your code and software in projappl and datasets, logs and calculation outputs in scratch. The home directory is not intended for data analysis and computing, and you should only store small personal files there.