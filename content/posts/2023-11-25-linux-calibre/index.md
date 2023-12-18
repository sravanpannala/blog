+++
title = "Creating a Calibre Web Server"
date = "2023-11-25"
description = "Using Calibre to create a web server to access books anywhere in your local network"

[taxonomies]
tags = ["guides","books"]
categories = ["Linux"]
[extra]
toc = true
keywords = "Guides, Tutorials, Books, calibre, Web server, Online Library"
+++

After spending almost 2 months exploring the wonderful world of Linux, I got back to the reason why I started my Linux Journey. That is to create a (Calibre Web Server)[https://manual.calibre-ebook.com/server.html] which I can access on any device at home and optionally anywhere in the world. For that the first step is to install calibre using the Arch Linux package manager/ AUR helper `yay`:
```bash
yay -S calibre
```
Then you have to copy your previous Library to your Linux computer that you want to use as the server. Then import this library during the initial configuration process. Calibre makes this process very easy. Now let us setup the options for the Calibre server before turning it on. Go to
> Preferences -> Sharing -> Sharing over the net

In the options dialog you can optionally set the port and after that enable `Run server automatically when calibre starts`. Then, under the user accounts tab add a user and password and then enable `Allow user to make changes`. Apply settings and restart calibre. Now the server is running. 
Now open the calibre server in other computer on the local network by entering the server address (which can be found when clicking the Connect/Share button in the calibre topbar) in a new browser window. The browser will ask you for the username and password, enter those to access your Calibre library. Voila, you can read your calibre books everywhere in your local network. The advantage of this is that the reading position and the highlights/notes are synced to your library which I feel is a really cool feature.

What we want to do next to enable the calibre server to start at boot. To do this create a service to run calibre by create a file `/etc/systemd/system/calibre-server.service` with the following contents:
```
[Unit]
Description=calibre Content server
After=network.target

[Service]
Type=simple
User=mylinuxuser
Group=mylinuxgroup
ExecStart=/usr/bin/calibre-server 

[Install]
WantedBy=multi-user.target
```
Now run the server and check its status:
```bash
sudo systemctl start calibre-server
sudo systemctl status calibre-server
```
If it is running properly, enable the service to start at boot using:
```bash
sudo systemctl enable calibre-server
```
Congratulations, you have a calibre server which runs on startup. You may need to disable the service using 
```bash
sudo systemctl disable calibre-server
```
to open the Calibre GUI to make modifications to your library on the Linux computer. 

For accessing your server anywhere accross the world enable port forwarding on your router. This [YouTube Video](https://www.youtube.com/watch?v=mLLKtO-qlNM&t=30s) explains how to do this step. You can use services like [no-ip](https://www.noip.com/) to setup a normal address instead of remembering your IP address.
After completing the previous step you can access your Calibre server anywhere in the world by going to 
```
yourserveraddress:port
```

I wasn't done after doing the above steps. I was wondering that if I can access my calibre-server remotely why can't I login to my computer remotely. One option I was aware of which could accomplish this is `ssh`. Now I will explain how to enable ssh in your linux computer and access it remotely. What we have to do is to edit the ssh configuration file by using the text editor of your choice (I use [Micro](https://micro-editor.github.io/)).
```
sudo micro /etc/ssh/sshd_config
```
In this file uncomment the port line and change the port number from `22` to `2222`. Save the file and start the `sshd service` using
```bash
sudo systemctl start sshd
```
You can also enable the service to start at boot by using
```bash
sudo systemctl enable sshd
```
To access your linux computer from your other computer (I use a Windows 10 PC as my daily driver), make sure that ssh is installed in your computer and then type this line in the terminal (I use Powershell on Windows Terminal):
```bash
 ssh user@computer-name.local -p 2222     
```
The `.local` has to be added to the computer to indicate that the computer is in the local network. Now you are logged into your linux computer. You can use your Windows Terminal as regular linux shell and perform any commands as you'd like. You can't access the GUI yet. That would be a project to do another day. You can also add the host to your config file to access your linux computer more easily. Add these lines to your `ssh config file` (create if not yet created):
```
Host hostname
  HostName computer-name.local
  User username
  Port 2222
```
Now you can access this computer by using:
```
ssh hostname
```
I hope you found my experience setting up calibre-server and remote login to linux computer using ssh helpful. Until next time, Caio!!!

