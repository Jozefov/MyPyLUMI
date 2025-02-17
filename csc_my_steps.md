Since you’re not using conda on LUMI, you’ll need to install your dependencies using pip inside your new venv. Assuming you have your venv activated (which you already did) and your requirements file (or environment file) is in your project directory, here’s how you can proceed:

1. Verify Your Virtual Environment is Active

Your prompt should show (phantoms_venv) and both which python and which pip should point inside your venv (as you already confirmed):

which python
# /pfs/lustrep3/scratch/project_465001738/jozefov_147/PhantoMS/phantoms_venv/bin/python

2. Prepare to Install Dependencies

You have two options for specifying your dependencies:
	•	Using the requirements_lumi.txt file
	•	Using the list from your environment.yml file

Since you’re not using conda and pip is your package manager here, the simplest approach is to use your requirements_lumi.txt file. Make sure that file is in your current directory (/scratch/project_465001738/jozefov_147/PhantoMS).

The contents of your requirements_lumi.txt are:

# Do not install torch here because LUMI provides an optimized PyTorch module
numpy
pandas
pyyaml
pytorch-lightning
torch-geometric
wandb
tensorboard
rdkit==2024.03.5
umap-learn==0.5.7

# Optional: Pin versions for packages that might conflict
# For example, if gensim requires scipy<1.14.0, you could add:
# scipy==1.13.1
# lxml==4.9.4

# Install massspecgym (with notebooks extras) from your private repository
massspecgym[notebooks] @ git+ssh://git@github.com/Jozefov/MassSpecGymMSn.git@main

# Install your local package in editable mode
-e .

3. Install the Dependencies with pip

With your venv active, run:

pip install -r requirements_lumi.txt

This command will do the following:
	•	Skip installing torch: Since the file comments that torch is provided by the module.
	•	Install packages like numpy, pandas, etc.
	•	Install your massspecgym package from GitHub (with the [notebooks] extra).
Make sure your SSH key is loaded (e.g., using ssh-add ~/.ssh/id_rsa) so that pip can access your private repository.
	•	Install your local package in editable mode: This means that if you make changes to your local code (the one with your setup.py), they’ll be reflected immediately in your environment.

4. Verify the Installation

After the installation finishes, check that everything is installed in your venv:

pip list

Also, check your massspecgym installation:

pip show massspecgym

The Location should now be within your venv or point to your local source (for the editable install).

5. Updating Your Package in the Future

If you later update your GitHub repository and want to update the package inside your venv, you can do one of the following:
	•	If using the GitHub install (non-editable):

pip install --upgrade git+ssh://git@github.com/Jozefov/MassSpecGymMSn.git@main


	•	If using editable mode:
Simply pull the latest changes in your local repository (the one pointed to by -e .), and the changes will be immediately available in your venv. If necessary, you can reinstall with:

pip uninstall massspecgym -y
pip install -e .



	Note: Be cautious about having both the GitHub installation and the editable installation at the same time. They might conflict. Decide whether you want to use the package directly from GitHub or work on a local copy. If you’re actively developing the package, the editable install (-e .) is recommended.

Summary of Steps
	1.	Start an interactive session:
(You already did this with your srun command.)
	2.	Load the module and create/activate your venv:

module use /appl/local/csc/modulefiles/
module load pytorch/2.4
python3 -m venv --system-site-packages phantoms_venv
source phantoms_venv/bin/activate


	3.	Install dependencies:

pip install -r requirements_lumi.txt


	4.	Verify installations and update as needed.

Following these steps should give you a clean, pip-managed environment on LUMI that includes your optimized PyTorch module from the CSC module and all your project-specific dependencies, including your private GitHub package.

then run script:
python3 scripts/run_cut_trees_LUMI.py


AFTER EVERYTHING INSTALLED USE:
module use /appl/local/csc/modulefiles/
module load pytorch/2.4
export PYTHONNOUSERSITE=1
source phantoms_venv/bin/activate

then after
PYTHONNOUSERSITE=1
bash scripts/update_massspecgym.sh