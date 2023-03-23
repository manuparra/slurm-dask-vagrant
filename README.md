# Slurm + Dask with Vagrant

Slurm and dask cluster deployed with vagrant

Instructions:

1. Clone this repo:

```
git clone ....
```

2. Change to the folder:

```
cd slurm-dask-vagrant
```

3. Start vagrant VMs:

```
vagrant up
```

And wait ...

4. Once VMs are Up, connect to the main node:

```
vagrant ssh rm1
```

5. Test slurm:

```
sinfo
srun -n2 hostname
```

