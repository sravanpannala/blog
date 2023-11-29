+++
title = "Distributed Cross-compiling using Icecream"
date = "2022-08-24"
description = "Using icecream for distributed compiling to speed up package compilation times on ARM machines"

[taxonomies]
tags = ["linux","guides","arm"]

+++

This blog post is a guide on using [Icecream](https://github.com/icecc/icecream) for both distributed compiling and distributed cross compiling to speed up package compilation times on your linux machines. Even though the commands used are specific to Arch Linux based systems, this guide can be adopted to any Linux distribution.

## Introduction
I'm a developer for EndeavourOS ARM. I have to build some ARM SBC specific kernels to support those devices. But compiling Linux kernel on the Raspberry Pi takes about 2 hours. I wanted to reduce this compile time. A common way to do it is to use distributed cross-compiling. This means that we will use an x86_64 machine in addition to the ARM computer to build packages. Most people use [distcc](https://wiki.archlinux.org/title/Distcc) to perform this task. There are a few issues with distcc including my inability to make [ccache](https://wiki.archlinux.org/title/ccache) work (ccache is a compiler cache which further improves build times), made me look for an alternative and thus I found `icecream`.

## Why Icecream?
There are a few issues I have with `distcc`:
- You have to edit the configuration in all the computers in the cluster if even if you're adding a single new node. This makes the configurations hard to maintain and to scale. There is no need to do this in `icecream`. We just have to configure the new node being added. The node automatically gets detected by other computers in the cluster.
- There is no load scheduler in `distcc` i.e. the jobs get distributed to all computers in the order decided by the configuration even if the computer listed first in the config is already under heavy load. In `icecream` there is a centralized scheduler. This scheduler decides which computer to send the job to depending on several factors including node speed and availability. From `icecream`'s readme:
>  Icecream uses a central server that dynamically schedules the compile jobs to the fastest free server. 
- I wasn't able to get ccache working with `distcc` when I'm using `makepkg` (the tool used to build packages on Arch Linux based systems). I tried reading several wikis and tutorials and couldn't figure it out. Then Peter from CachyOS told me that `icecream` supports ccache and it's easier to configure it. This was the main reason I started exploring `icecream`. 
 
I hope that this guide will help you get started on your distributed compiling/cross-compiling endeavour with `icecream`.

## Setup
Lets go through the initial setup of `icecream`.
1. Install icecream on all the computers you want to include in the compilation cluster. On Arch Linux based systems `icecream` is available in the AUR.
```bash
yay -S icecream
```
Optionally install these monitoring tools (that are also available in the AUR): `icemon`: a GUI monitor tool or `icecream-sundae`: a TUI monitor tool.
2. Once you install icecream, choose a computer that you want to act as a scheduler and then start and enable `icecream-scheduler.service` on that computer.
```bash
sudo systemctl enable --now icecream-scheduler.service
```
3. After this, add the ip of the scheduler to the configuration file on the computers you want to be included in the cluster. The configuration file for `icecream` can be found on `/etc/icecream.conf`. In that file edit this line by adding the ip.
```
ICECREAM_SCHEDULER_HOST="scheduler ip here"
```
4. In the same configuration file you can optionally configure the max number of jobs that can be allocated to that node parallelly. The default number used by `icecream` is the number of cores of the node.
```
ICECREAM_MAX_JOBS="number"
```
5. After finishing the configuration, now start and enable `icecream.service`
```bash
sudo systemctl enable --now icecream.service
```
Now you have `icecream` and `icecream-scheduler` running on your network. But you can't start using it for building packages before more configuration that I will show in the next section.

## Configuration
If you want to cross compile using `icecream` use the instructions in the next subsection. If all the nodes in your cluster are of the same architecture, you can skip it.

### Configure toolchain for cross compiling
The most common cross-compiling scenario is using `x64` computers to compile for `aarch64` since ARM computers are generally not powerful. To compile `aarch64` on `x64` you first need to configure a cross-compiler toolchain. There is an AUR package that does most of it for you. You can install it by running
```
yay -S distccd-alarm-armv8
```
### Creating icecream environment
Next you need to create an `icecream` environment to tell `icecream` that it has to use the cross-compiler instead of the normal x64 compiler. That can be done using
```
/usr/lib/icecream/bin/icecc-create-env --gcc /opt/x-tools8/aarch64-unknown-linux-gnu/bin/aarch64-unknown-linux-gnu-gcc /opt/x-tools8/aarch64-unknown-linux-gnu/bin/aarch64-unknown-linux-gnu-g++
```
This will create a tarball with the environment, which you then have to copy over to each ARM computer where you will run jobs from.
If you want to use your local node for compiling you need to also generate a icecream environment for that. This step is not given in the official documentation but the local node wouldn't take part in compiling otherwise. You can do that by using:
```
/usr/lib/icecream/bin/icecc --build-native
```
### Configuring Path
Next, you need to configure the `path` in your computer to tell it to use icecream for compiling instead of the inbuilt compiler. This configuration will also setup `icecream` to use `ccache`. So first install `ccache`:
```
yay -S ccache
```
Then modify your path like so:
```
export CCACHE_PREFIX=icecc
export PATH=~/scripts:$PATH
export PATH=/usr/lib/icecream/bin:$PATH
export PATH=/usr/lib/ccache/bin/:$PATH
export ICECC_VERSION=<path to local icecream environment>,x86_64:<path to cross icecream environment>
```

### Configuring makepkg
Finally you need to configure `makepkg` to run more job parallelly. This can be done by editing `/etc/makepkg.conf` like so:
```
MAKEFLAGS="-jN"
```
where `N` is the number of jobs. A rule of thumb here is to set it as number of total threads available in the cluster.

## Monitoring
Congratulations on getting this far. Now you can use `makepkg` to build packages using `icecream`. You can monitor it using either `icemon` or `icecream-sundae` that you may have installed earlier. `icemon` screen looks like this:
![](./dist_comp_1.png)
One of the common issues is the firewall blocking the ports for `icecream` so make sure that this port is open on all nodes. 
```
UDP Source Port 8765
```

## Conclusion
I hope you enjoy your newfound power to build packages quickly even on less powerful computers. One advantage on the above setup is that even building of AUR packages using AUR helpers like `yay` or `paru` will now use distributed compiling which will make your normal updates faster.

Final words:
> Compile Responsibly

## Resources
- [GitHub - icecc/icecream: Distributed compiler with a central scheduler to share build load](https://github.com/icecc/icecream)
- [Archwiki - distcc cross compiling](https://wiki.archlinux.org/title/Distcc#Cross_compiling_with_distcc)
- [Cross-compiling with icecream - Samuel Iglesias Gonsálvez's blog](https://blogs.igalia.com/siglesias/2021/12/09/Cross-compiling-with-icecream/)
- [The C/C++ Developer’s Guide to Avoiding Office Swordfights – Part 2: icecream](https://www.methodpark.de/blog/the-c-c-developers-guide-to-avoiding-office-swordfights-part-2-icecream/)
- [icecc setup how to](https://savago.wordpress.com/2012/11/23/icecc-setup-how-to/)