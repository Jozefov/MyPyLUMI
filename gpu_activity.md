I started to write this tutorial because I had problem with GPU utilization, I reserved one node with 8 GPUs and send to pytorch lightning and what looking at log, it was laying to me :) I see it recognize GPUs and saying it is using them when training. But in reality gpu utilization look like this
-   ![unsuccessful_train](assets/img_9.png)
and all my computation was utilized purely by CPUs.

- 2 GPUs/GCD are for 500w, if power is around that level, it says GPUs are being leveraged
- if you use just single GPU then 300W is good indicator
- profiler rocprof, omnitrace, omniperf (kernel optimization)
- -   ![unsuccessful_train](assets/img_10.png)
- you also have option for profiling with torch.profiler, solved in torch lighting and wandb