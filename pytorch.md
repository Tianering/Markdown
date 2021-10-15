单机多卡

多张显卡训练问题

python -m torch.distributed.launch --nproc_per_node=NUM_GPUS_YOU_HAVE main.py

torch.nn.parallel.DistributedDataParallel(my_model, find_unused_parameters=True)

https://zhuanlan.zhihu.com/p/86441879

