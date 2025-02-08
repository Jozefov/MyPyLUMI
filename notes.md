

one gpu, whcih is 1/8 of nodes has 60gm ram and 7 cpus
Why in --mem-per-gpu=60G it is 60 GB, not 64?

Because the nodes have only 480 GB available per user of the 512 GB, so ⅛th of that is a fair use per GPU. Which gives 60 GB. Note that this is CPU memory and not GPU memory!
looks like this example:
#!/bin/bash
#SBATCH --account=project_num
#SBATCH --reservation=AI_workshop_1   # comment this out if the reservation is no longer available
#SBATCH --partition=small-g
#SBATCH --gpus-per-node=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=7
#SBATCH --mem-per-gpu=60G
#SBATCH --time=0:15:00

to find images of containers at prebuild torch-roc, see:
ls /appl/local/containers/sif-images

Available GPU partitions
• standard-g
≤ 48h, whole nodes only, max 1024 nodes/job small-g
≤ 72h, individual GCDs, max 4 nodes/job
• dev-g
≤ 3h, individual GCDs, max 32 nodes/job, max 1 job running

check GPU utilizations:
srun --overlap --pty --jobid=num_of_your_job bash

tutorials:
https://github.com/Lumi-supercomputer/Getting_Started_with_AI_workshop/tree/main?tab=readme-ov-file

videos and slides:
https://lumi-supercomputer.github.io/LUMI-training-materials/ai-20250204/index.html
