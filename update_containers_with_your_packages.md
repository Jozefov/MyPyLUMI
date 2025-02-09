created by:
https://462000265.lumidata.eu/ai-20250204/files/LUMI-ai-20250204-07-Extending_containers.pdf

it is when you have conda yml, example:
name: minimal_pytorch
channels:
- conda-forge
dependencies:
- filelock=3.15.4
- fsspec=2024.9.0
- jinja2=3.1.4
- markupsafe=2.1.5
- mpmath=1.3.0
- networkx=3.3
- numpy=2.1.1
- pillow=10.4.0
- pip=24.0
- python=3.12.3
- sympy=1.13.2
- typing-extensions=4.12.2
- pip:
- --extra-index-url https://download.pytorch.org/whl/rocm6.0
- pytorch-triton-rocm==3.0.0
- torch==2.4.1+rocm6.0
- torchaudio==2.4.1+rocm6.0
- torchvision==0.19.1+rocm6.0.6

and you build it with:
module load LUMI/24.03 cotainr
cotainr build minimal_pytorch.sif --base-image=/appl/local/containers/sif-images/lumi-
rocm-rocm-6.0.3.sif --conda-env=minimal_pytorch.yml --accept-license

and --base-image is can be lumi-g as in conda_container.md or you can find your version in `ls /appl/local/containers/sif-images`

and then you run:
module use /appl/local/containers/ai-modules
module load singularity-AI-bindings
singularity shell minimal_pytorch.sif
and you will see same packages as in your env yml

and now we have container from minimal_pytorch.sif and want to extend it with lets say torchmetrics. So we do:
Create a virtual environment via venv
Inside the container, create a virtual environment via venv, it is run in bash of container from now on, all this commands below:
python -m venv myenv --system-site-packages
The --system-site-packages flag gives the virtual environment access to the packages inside, it is important, otherwise it will create new empy env and will not take into accounts one from minimal_pytorch.sif
the container.
Activate the environment via
source myenv/bin/activate

Install custom packages via pip
pip install torchmetrics
The new package will then be available alongside the packages in the container, this way it will be saved in virtual env and not home directory, which is small 20Gb only.

We can check the location of the installed files via
Location of installed packages
LUMI
We can check the location of the installed files via
(myenv) Singularity> python
Python 3.12.3 | packaged by conda-forge | (main, Apr 15 2024, 18:38:13) [GCC 12.3.0] on linux
Type "help"
", "copyright", "credits" or "license" for more information.
>> import os
>>> import torchvision
>>> import torchmetrics
»>> os. path.abspath(torchvision.__file__)
'/opt/conda/envs/conda_container_env/lib/python3.12/site-packages/torchvision/__init__.py'
file
»>> os. path. abspath(torchmetrics.__file_
'/pfs/lustrep3/projappl/project_462000002/decristoforo/myenv/lib/python3.12/site-packages/torchmetrics/__init__.py'
The new package is installed in our virtual environment whereas the other packages are installed in the container.

Warning
You should not stop here, as this way of installing python packages creates typically thousands of
small files. This puts a lot of strain on the Lustre file system and might exceed your file quota.
Once you have a complete set of python packages and their versions, choose one of the following
options:
⚫ Create a new container with cotainr and delete virtual environment
⚫ Turn myenv into a SquashFS file and bind mount it to the container


Option 1: Create a new container with cotainr
After having found all needed packages, add them to the conda
environment file and create a new container:
module load LUMI/24.03 cotainr
cotainr build updated_pytorch.sif --base-image=/appl/local/containers/sif-images/lumi-
rocm-rocm-6.0.3.sif --conda-env=updated_pytorch.yml --accept-license
The virtual environment should then be deleted:
rm -rf myenv
name: updated_pytorch
channels:
- conda-forge
dependencies:
- filelock=3.15.4
- fsspec=2024.9.0
- jinja2=3.1.4
- markupsafe=2.1.5
- mpmath=1.3.0
- networkx=3.3
- numpy=2.1.1
- pillow=10.4.0
- pip=24.0
- python=3.12.3
- sympy=1.13.2
- typing-extensions=4.12.2
- pip:
- --extra-index-url https://download.pytorch.org/whl/rocm6.0
- pytorch-triton-rocm==3.0.0
- torch==2.4.1+rocm6.0
- torchaudio==2.4.1+rocm6.0
- torchvision==0.19.1+rocm6.0.6
- torchmetrics==1.6.0
- 
Option 2: Turn myenv into a SquashFS file
Turn the myenv directory into a SquashFS file and bind mount it to the container:
mksquashfs myenv myenv.sqsh
rm -rf myenv
export SINGULARITYENV_PREPEND_PATH=/user-software/bin
singularity exec -B myenv.sqsh:/user-software:image-src=/ minimal_pytorch.sif python my_script.py
This is much better for the file system as it regards the myenv.sqsh as a single file.
For advanced users:
This approach is compatible with packages that cannot be installed via cotainr (e.g. packages
that require manual compilation)

LUMI application containers
12
venv approach may also be used with the LUMI application containers that are not built with cotainr,
e.g. /appl/local/containers/sif-images/lumi-pytorch-rocm-6.2.1-python-3.12-pytorch-20240918-vllm-4075b35.sif
For these containers it is required to activate the conda environment ($WITH_CONDA) before creating
the venv
CONTAINER=/appl/local/containers/sif-images/lumi-pytorch-rocm-6.2.1-python-3.12-pytorch-20240918-vllm-
4075b35.sif
srun singularity exec $CONTAINER bash -c '$WITH_CONDA && source myenv/bin/activate && python my_script.py'
Building a (final) container from LUMI application containers + a venv is not directly supported by
cotainr

Pros and Cons
Pros:
⚫ Quick (and easy) approach for installing additional packages to existing containers
Cons:
⚫ Additional packages are installed directly on Lustre file system which can lead to bad performance
and exceed your file limit (if SquashFS approach is not used)
⚫ Required to keep manually track of which venv matches which container for which use case
⚫ Necessary to source the venv every time you run the container to get access to the packages in the
virtual environment

Summary of steps
Open shell inside container
singularity shell --bind /pfs,/scratch,/projappl,/project,/flash,/appl container_image.sif
If no virtual environment present, create a new one
python -m venv myenv --system-site-packages
Activate virtual environment
source myenv/bin/activate
Install custom packages
pip install new_package