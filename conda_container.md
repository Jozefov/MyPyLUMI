you need your conda_env.yml and send it to container that build a container on LUMI

Instead of
conda env create -f my_AI_env.yml
you run*
cotainr build my_AI_container.sif --system=lumi-g
--conda-env=my_AI_env.yml

it has to be conda env file, pip does not work.

The --system=lumi-g is a shortcut to getting a proper base
image. Alternatively, specify --base-image=. 

In container info `cotainr info`, you got options:
Available system configurations:
    - lumi-g
    - lumi-c
as we want to train on GPU we choose lumi-g, and lumi-c is for cpu nodes.
to run code your have to first:

module purge
module load CrayEnv cotainr
cotainr build my_AI_container.sif --system=lumi-g
--conda-env=my_AI_env.ym

Cotainr does not do any magic conversion of the packages specified in the Conda environment to make sure they fit the hardware of LUMI. It simply installs the packages exactly as listed in the my_conda_env.yml file.

• To avoid putting stress on the login-nodes, you may consider
running cotainr non-interactively on a compute node
srun --output=cotainr.out --error=cotainr.err --
account=<project_ID> --time=00:30:00 --mem=64G --cpus-per-
task=32 --partition=debug cotainr build minimal_pytorch.sif
--system=lumi-g --conda-env=minimal_pytorch.yml --accept-
licenses

**Self-built containers
Not straighforward to built conda environment inside container. Especially
difficult to get communication right** It is LUMI specific


To build a container using cotainr on LUMI, we must remember to:

Load the cotainr module on LUMI
Determine the relevant system (LUMI-C/LUMI-G) or, alternatively, pick a suitable base image
Run cotainr using srun, redirect stdout/stderr, and accept all licenses up-front when building non-interactively on a compute node
Since the python312.yml environment only contains Python 3.12, we don't need ROCm or other special system libraries, so using --system=lumi-c with cotainr is sufficient for getting a fairly minimal base image.

On a login node, we may build the container interactively by:

$ module purge
$ module use /appl/local/training/modules/AI-20241126
$ module load cotainr
$ cotainr build python312.sif --system=lumi-c --conda-env=python312.yml

On a LUMI-C compute node, we may build the container non-interactively by:
$ module purge
$ module use /appl/local/training/modules/AI-20241126
$ module load cotainr
$ srun --output=cotainr.out --error=cotainr.err --account=project_465001363 --time=00:05:00 --mem=60G --cpus-per-task=8 --partition=dev-g cotainr build python312.sif --system=lumi-c --conda-env=python312.yml --accept-licenses

python312.yml looks like this:
name: python312_extra
channels:
  - conda-forge
dependencies:
  - python=3.12.3
  - pip=24.0
  - python=3.12.3
  - pandas=2.2.0
  - scikit-learn=1.4.2
  - pip:
    - env-var==1.0.1

**to run python on login node:**
singularity exec python312.sif python3 -c "import sys; print(sys.executable); print(sys.version)"
$ singularity exec python312.sif python3 -c "import sys; print(sys.executable); print(sys.version)"
/opt/conda/envs/conda_container_env/bin/python3
3.12.7 | packaged by conda-forge | (main, Oct  4 2024, 16:05:46) [GCC 13.3.0]

whereas directly on LUMI we get:
$ python3 -c "import sys; print(sys.executable); print(sys.version)"
/usr/bin/python3
3.6.15 (default, Sep 23 2021, 15:41:43) [GCC]

which shows that within the container we directly have access to the Python 3.12 we installed as part of our conda environment instead of the Python 3.6 provided by the OS. Note that if you run python3 -c "import sys; print(sys.executable); print(sys.version)" after having run module load cotainr, you will get

$ python3 -c "import sys; print(sys.executable); print(sys.version)"
/opt/cray/pe/python/3.9.13.1/bin/python3
3.9.13 (main, Aug 10 2022, 17:20:06)
[GCC 9.3.0 20200312 (Cray Inc.)]
since the cotainr module loads the cray-python module to get a Python >= 3.8 which is needed for running cotainr.

**We don't need to activate the conda environment in the container as this is done automatically.**

Even though we pin the versions of the added packages, their dependencies are not pinned and may change if building the container again at a later point in time. To be able to build a new container with the exact same set of all packages (including dependencies), you need to use the output of conda env export (in the container) as the conda environment file provided to cotainr (or specify all dependencies manually). The output of conda env export in the container looks something like:
Singularity> conda env export
name: conda_container_env
channels:
  - conda-forge
dependencies:
  - _libgcc_mutex=0.1=conda_forge
  - _openmp_mutex=4.5=2_gnu
  - bzip2=1.0.8=hd590300_5
  - ca-certificates=2024.2.2=hbcca054_0
  - joblib=1.4.2=pyhd8ed1ab_0
  - ld_impl_linux-64=2.40=h55db66e_0
  - libblas=3.9.0=22_linux64_openblas
  - libcblas=3.9.0=22_linux64_openblas
  - libexpat=2.6.2=h59595ed_0
  - libffi=3.4.2=h7f98852_5
  - libgcc-ng=13.2.0=h77fa898_7
  - libgfortran-ng=13.2.0=h69a702a_7
  - libgfortran5=13.2.0=hca663fb_7
  - libgomp=13.2.0=h77fa898_7
  - liblapack=3.9.0=22_linux64_openblas
  - libnsl=2.0.1=hd590300_0
  - libopenblas=0.3.27=pthreads_h413a1c8_0
  - libsqlite=3.45.3=h2797004_0
  - libstdcxx-ng=13.2.0=hc0a3c3a_7
  - libuuid=2.38.1=h0b41bf4_0
  - libxcrypt=4.4.36=hd590300_1
  - libzlib=1.2.13=hd590300_5
  - ncurses=6.5=h59595ed_0
  - numpy=1.26.4=py312heda63a1_0
  - openssl=3.3.0=hd590300_0
  - pandas=2.2.0=py312hfb8ada1_0
  - pip=24.0=pyhd8ed1ab_0
  - python=3.12.3=hab00c5b_0_cpython
  - python-dateutil=2.9.0=pyhd8ed1ab_0
  - python-tzdata=2024.1=pyhd8ed1ab_0
  - python_abi=3.12=4_cp312
  - pytz=2024.1=pyhd8ed1ab_0
  - readline=8.2=h8228510_1
  - scikit-learn=1.4.2=py312h394d371_0
  - scipy=1.13.0=py312hc2bc53b_1
  - setuptools=69.5.1=pyhd8ed1ab_0
  - six=1.16.0=pyh6c4a22f_0
  - threadpoolctl=3.5.0=pyhc1e730c_0
  - tk=8.6.13=noxft_h4845f30_101
  - tzdata=2024a=h0c530f3_0
  - wheel=0.43.0=pyhd8ed1ab_1
  - xz=5.2.6=h166bdaf_0
  - pip:
      - arrow==1.3.0
      - decorator==5.1.1
      - env-var==1.0.1
      - isoduration==20.11.0
      - rfc3339-validator==0.1.4
      - rfc3986-validator==0.1.1
      - types-python-dateutil==2.9.0.20240316
      - validators==0.18.2
prefix: /opt/conda/envs/conda_container_env

EXEMPLE when package has no pip version but just git (here panopticapi, does not matter):
Create a conda environment file for installing panopticapi
Use the conda environment file to build a container for LUMI-C using cotainr
To build a container for panopticapi using cotainr on LUMI, we must remember to:

Include as many of panopticapi's dependencies (in this case listed in the setup.py file) as possible as Conda packages in our conda environment file.
List the panopticapi GitHub repo master branch as a pip dependency since no conda/pip packages exist for panopticapi.
Add git as a conda dependency for pip to be able to pull the panopticapi GitHub repo.
A panopticapi.yml conda environment file may look like:

name: panopticapi
channels:
  - conda-forge
dependencies:
  - git=2.45.1
  - numpy=1.26.4
  - pillow=10.3.0
  - pip=24.0
  - python=3.9
  - pip:
    - git+https://github.com/cocodataset/panopticapi.git
Note

panopticapi is a somewhat old package that does not have any specific release and no pinned versions of dependencies. Thus, one may need some trial & error to install versions of numpy and pillow that are compatible with the current master branch of panopticapi. Maybe the above works...

Now we can build a container the usual way:

$ module use /appl/local/training/modules/AI-20241126
$ module load cotainr
$ cotainr build panopticapi.sif --system=lumi-c --conda-env=panopticapi.yml
Note

When you install directly from (private) Git(Hub) repos, you may need to install extra Conda packages needed by pip to connect to the repo, e.g. git and openssh. Alternatively, you can also install directly from a zip archive of the repo, e.g. specifying https://github.com/cocodataset/panopticapi/archive/master.zip instead of git+https://github.com/cocodataset/panopticapi.git. See the cotainr conda environment documentation for a more elaborate example.
