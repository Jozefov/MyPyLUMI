Absolutely! Here’s a complete LUMI Python ML project quickstart tutorial tailored to your workflow, designed for easy reuse.
Feel free to save, edit, or share as needed!

⸻

🧑‍💻 LUMI Python Project Setup – Practical Guide

This is a future-proof reference for setting up a Python deep learning environment on LUMI (CSC), handling dependencies and working efficiently!

⸻

1. Connect to LUMI

From your local terminal:

ssh <your_lumi_username>@lumi.csc.fi

or the actual hostname you normally use.

⸻

2. Move to Your Project Folder

cd /scratch/project_465002061  # or /projappl/project_465002061
ls
# Example: Go into your project
cd DreaMS_MIMB


⸻

3. Start an Interactive GPU Session

Ask for a GPU node for interactive work (testing, installs, debugging):

srun --export=ALL --preserve-env --account=project_465002061 --partition=dev-g --time=1:00:00 --nodes=1 --gpus-per-node=1 --cpus-per-task=7 --pty bash

You’ll be dropped into a shell on a GPU node.

⸻

4. Load the Python & ML Modules

module use /appl/local/csc/modulefiles/
module load pytorch   # Loads the latest supported PyTorch (with Jupyter etc.)
# Optional: module load pytorch/2.7  # If you need a specific version


⸻

5. Create a Python Virtual Environment

Use venv, not conda!

python3 -m venv --system-site-packages dreams_mimb
source dreams_mimb/bin/activate


⸻

6. (Recommended) Upgrade pip

pip install --upgrade pip


⸻

7. Install Your Project-Specific Python Packages

Don’t try to install/upgrade torch—use the system’s!

pip install --no-deps git+https://github.com/pluskal-lab/DreaMS.git
pip install --no-deps massspecgym

Or, for many pip packages:

pip install --no-deps -r requirements.txt


⸻

8. Check Everything is Installed and Importable

python -c "import torch; print(torch.__version__)"
python -c "import massspecgym, dreams; print('All good!')"


⸻

9. Run Your Code!

Train, test, or launch your notebook as needed:

python train.py
# or for jupyter
jupyter lab

(On LUMI, for heavy jobs, use SLURM batch scripts—see LUMI docs for job script examples.)

⸻

10. Next Time: Reactivate Your Environment

Every new session:

module use /appl/local/csc/modulefiles/
module load pytorch
source /scratch/project_465002061/DreaMS_MIMB/dreams_mimb/bin/activate


⸻

Troubleshooting
	•	Dependency conflict? Use --no-deps with pip for packages that require torch/ML libraries—let the LUMI module handle those.
	•	Need a different module? Use module avail or check documentation.
	•	Permission error? Make sure you’re in your own project space (/projappl, /scratch, etc).

⸻

Template for SLURM Batch Script (for training jobs)

#!/bin/bash
#SBATCH --account=project_465002061
#SBATCH --partition=small-g
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=7
#SBATCH --gpus-per-node=1
#SBATCH --mem=60G
#SBATCH --time=2:00:00
#SBATCH --job-name=dreams_train

module use /appl/local/csc/modulefiles/
module load pytorch
source /scratch/project_465002061/DreaMS_MIMB/dreams_mimb/bin/activate

srun python3 train.py  # or your entrypoint script


⸻

Best Practices
	•	Use /scratch for large, temporary data. Use /projappl for code/envs.
	•	Back up important data! /scratch can be purged.
	•	For more, check LUMI official documentation and CSC ML Guide.

⸻

Copy, Bookmark, or Adapt This for Your Future Projects!

If you want this as a Markdown, PDF, or shell script, just say the word.
Let me know if you want to add extra tips for Jupyter, SLURM array jobs, etc!