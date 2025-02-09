#!/bin/bash
#SBATCH --account=project_465001707
#SBATCH --partition=standard-g
#SBATCH --gpus-per-node=8
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=56
#SBATCH --mem=480G
#SBATCH --time=1:00:00
Full node = we can also use
standard-g

Do we need to change the Python code?
● For plain PyTorch: yes, use DistributedDataParallel (DDP)
● For higher level frameworks, mostly no:
– transformers.Trainer is automatically set up for distributed
training when WORLD_SIZE & RANK environment variables are set
– Similar for other high-level frameworks like PyTorch Lightning or
Accelerate
● **BUT: Pay attention to global batch size vs per device batch size!
– Example: global batch size = 32 for one GPU,
split over 8 GPUs, per-device batch size is 4**
● Cosmetic: You might want to print some things only on rank 0

If I ask for one complete node and use “ddp”, how many maximum num_workers should I use ? should it still be num_workers == cpus-per-task=7

I assume you mean the number of workers for the dataloaders. These are "orthogonal" to the processes for the GPU. For each GPU, you will have one "main" process on the CPU, and torchrun will take care of running these for you. Each of those will then spawn the number of dataloader worker processes that you set up. These just handle preprocessing of data to be ready to be send to the GPU. Since the throughput of data at each GPU shouldn't change (much) compare to running with a single GPU, if you needed 7 dataloader processes (num_workers), it makes sense to keep it the same when running with multiple GPUs. However, in general, if you really want to find out just how many you need, you should try different values and see how that affects performance or profile you run.

Which container has lightning module? lightning handle this ddp automatically I guess. Can I install this in existing containers?
singularity exec lumi-pytorch-rocm-5.7.3-python-3.12-pytorch-v2.2.2.sif bash -c '$WITH_CONDA; pip list'

all others should be solved by torch lightning, it can differ, you can implement your model without option for multiple gpus, that you have to adjsut it. Prefer building models with help of torch lightning that takes care of all these things.