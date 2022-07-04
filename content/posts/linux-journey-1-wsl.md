+++
title = "My Linux Journey 1: WSL"
date = "2022-07-03"
description = "Chronicling my linux journey"

[taxonomies]
tags = ["linux-journey"]
+++

This is the first blog post in a series of posts that will be documenting my Linux journey. Let me start with how I discovered Linux and started using it.

## Beginnings
I have been a Windows user for my whole life. My first operating system was either Windows 98 or Windows 2000. My daily driver is an HP Spectre x360 laptop running on Windows 10. My first exposure to Linux was using it during an internship for CFD simulations back in 2014. I had to remote onto a Linux server to run my complex CFD simulations that took days to run. The next time I used Linux is to run [PyBaMM](https://www.pybamm.org/) a python based battery modeling software using WSL in October 2021. This is because some of the solvers don't work in Windows and the documentation recommends using WSL for people who want to develop PyBaMM. 

## Installing WSL
From Microsoft's [documentation](https://docs.microsoft.com/en-us/windows/wsl/): 
> WSL or Windows Subsystem for Linux lets users run a Linux environment directly on Windows without the overhead of a traditional virtual machine or dual-boot setup. WSL is a type 1 hypervisor VM, which means no compatibility layer between itself and hardware. So, you are getting 100% of the performance, and it's as close to bare metal as a VM can be. 

To get started just enter this command in a PowerShell:
```powershell
wsl --install
```
To see a list of available distros:
```powershell
wsl --list --online

NAME            FRIENDLY NAME
Ubuntu          Ubuntu
Debian          Debian GNU/Linux
kali-linux      Kali Linux Rolling
openSUSE-42     openSUSE Leap 42
SLES-12         SUSE Linux Enterprise Server v12
Ubuntu-16.04    Ubuntu 16.04 LTS
Ubuntu-18.04    Ubuntu 18.04 LTS
Ubuntu-20.04    Ubuntu 20.04 LTS
```
Then you can install your choice of distro (for example Ubuntu):
```powershell
wsl --install -d Ubuntu
```
Finally, ensure that you are using WSL 2:
```powershell
wsl --set-version <distro name> 2
```
You can then enter wsl with this simple command:
```powershell
wsl
```
This takes you to Ubuntu's command line in the terminal you are using on Windows. 

I chose Ubuntu as my distro as it is the most popular Linux out there and I've heard its name before. Also, for WSL the best available choice is Ubuntu as it was the first distro supported by WSL and is currently the distro with the best support and documentation. 


## Running Ubuntu on WSL
The first command I learnt and used in Ubuntu is the update command:
```bash
sudo apt update && sudo apt dist-upgrade
```
Afer this, the first thing I installed was `software-properties-common` and then installing a PPA to get `python 3.9`. I didn't know what a PPA was or that they have to be used with care. I just ran commands I needed to get the python version I needed to run PyBaMM. 
```bash
sudo apt install software-properties-common 
sudo add-apt-repository ppa:deadsnakes/ppa
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3.9 get-pip.py
```
I will spare you the other details about what I did in Ubuntu and move on to one of the best features of WSL in my opinion. 

## VS Code Integration
From the VS Code [documentation](https://code.visualstudio.com/docs/remote/wsl):
> The Visual Studio Code Remote - WSL extension lets you use the Windows Subsystem for Linux (WSL) as your full-time development environment right from VS Code. You can develop in a Linux-based environment, use Linux-specific toolchains and utilities, and run and debug your Linux-based applications all from the comfort of Windows.

It is as simple as installing `Visual Studio Code Remote - WSL extension` in your VS Code on Windows and then inside WSL navigate to your project folder and type:
```powershell
code .
```
This command opens your project folder that is on WSL in VS Code running on Windows. The best thing about this is that the experience is almost the same as working in VS Code normally on your project in Windows. All your VS Code settings, themes and extensions are there for your to use. The except is that some extensions like Python extension have to be installed seperately in WSL but that is sas simple as clicking a button. 

## Conclusion
As shown in this blog post, we can see that WSL is a useful tool for running a Linux environment in Windows and doing some coding using VS Code. Using WSL made me familiar with the common Linux commands for updating the system, installing packages, removing packages and other basic bash commands like `cd`,`ls`,`mkdir`,`rm`,`cp` and the like. I had a good time using WSL and it made me more interested in Linux. Thus WSL was a starting point for my Linux Journey. 
