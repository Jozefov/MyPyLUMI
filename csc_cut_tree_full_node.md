Below is a summary of the final working setup for multi-GPU training on LUMI using PyTorch Lightning with Distributed Data Parallel (DDP). You can keep this as a reference for future jobs.

1. SLURM Batch Script

This script reserves one node with 8 GPUs, 56 CPU cores, 480 GB of memory, and a 48-hour runtime. It uses the PyTorch launcher (torch.distributed.run) to automatically spawn one process per GPU. For example:

#!/bin/bash
#SBATCH --account=project_{num}
#SBATCH --partition=standard-g
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=56
#SBATCH --gpus-per-node=8
#SBATCH --mem=480G
#SBATCH --time=48:00:00

# Load necessary modules and activate your environment:
export WANDB_API_KEY=${WANDB_API_KEY}
cd /scratch/project_{num}/jozefov_147/PhantoMS

module --force purge
module use /appl/local/csc/modulefiles/
module load pytorch/2.4
export PYTHONNOUSERSITE=1
source phantoms_venv/bin/activate

# Launch the training script with torch.distributed.run,
# which will spawn 8 processes (one per GPU).
srun python3 -m torch.distributed.run --standalone --nnodes=1 --nproc_per_node=8 scripts/run_cut_trees_LUMI.py

Key Points:
	•	--ntasks-per-node=1: Launch a single main process.
	•	--gpus-per-node=8: Allocate 8 GPUs to that process.
	•	--nproc_per_node=8: torch.distributed.run spawns 8 child processes (one for each GPU).
	•	The environment variable WANDB_API_KEY is passed, and the working directory and Python environment are correctly set.

2. YAML Configuration for PyTorch Lightning

In your configuration YAML (e.g., config_skip_connection_LUMI.yml), set up the Trainer for multi-GPU training with DDP:

trainer:
  accelerator: "cuda"                  # Use GPUs
  devices: 8                           # Number of GPUs per node (should match --nproc_per_node)
  num_nodes: 1                         # Number of nodes
  max_epochs: 30                       # Train for 30 epochs
  log_every_n_steps: 20                # Logging frequency
  check_val_every_n_epoch: 10          # Validate every 10 epochs
  limit_train_batches: 1.0             # Use 100% of training data
  limit_val_batches: 1.0
  limit_test_batches: 1.0
  checkpoint_monitor: "val_loss"       # Monitor validation loss
  save_top_k: 2                        # Save top 2 models
  checkpoint_mode: "min"               # Lower validation loss is better
  checkpoint_dir: "checkpoints/"       # Where to save checkpoints
  strategy: "ddp"                      # Use Distributed Data Parallel

Key Points:
	•	devices: 8 matches the number of GPUs allocated on the node.
	•	strategy: "ddp" instructs Lightning to spawn one process per GPU and handle gradient synchronization automatically.
	•	Global hyperparameters (e.g., batch size, epochs) remain as configured.

3. How It Works
	•	Resource Allocation:
The SLURM script reserves the resources (8 GPUs, 56 CPUs, 480 GB memory) on one node.
	•	Distributed Launch:
Using torch.distributed.run, the job spawns 8 child processes (one per GPU). The launcher sets environment variables (like LOCAL_RANK) so that each child process uses one GPU.
	•	Lightning DDP:
With the YAML config set to devices: 8 and strategy: "ddp", PyTorch Lightning internally wraps your model with DDP. Each process handles a portion of the batch, and gradients are synchronized across GPUs automatically.
	•	Monitoring:
You can verify GPU usage with monitoring tools (e.g., rocm-smi on AMD systems) and via logs from your training script (Lightning’s internal logs show the distributed setup).

Summary Recap
	•	SLURM Script:
Use one task with 8 GPUs per node and launch with torch.distributed.run to automatically spawn 8 processes.
	•	YAML Config:
Set devices: 8 and strategy: "ddp" so Lightning uses all GPUs.
	•	Distributed Training:
Lightning’s DDP automatically sets the correct GPU for each spawned process using environment variables set by the launcher.

This configuration has been tested to correctly utilize all 8 GPUs on the node for distributed training, ensuring proper network updates and expected speedup.

Save this summary for future reference. It provides a blueprint to return to when setting up multi-GPU training on CSC’s LUMI system using PyTorch Lightning and DDP.