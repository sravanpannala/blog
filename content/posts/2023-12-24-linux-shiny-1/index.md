+++
title = "How I started making Shiny apps for visualizing NBA Data, Part I: Shiny Server"
date = "2023-12-24"
description = "Documenting my journeys in developing Shiny apps, starting with self-hosting Shiny apps using a custom-built Docker image."

draft = false

[taxonomies]
tags = ["docker", "shiny", "apps"]
categories = ["Linux"]
[extra]
toc = true
keywords = "docker, docker-compose, shiny, shiny-server, R, python, Data Science"
+++


## Introduction

I keep seeing good Shiny apps from the NBA Analytics community and find them cool. Some of them are:
- [Darko](https://apanalytics.shinyapps.io/DARKO/) by [Kostya Medvedovsky]([kmedved](https://twitter.com/kmedved)) & [Andrew Patton](https://twitter.com/anpatt7)
- [Game of Runs](https://ramirobentes.shinyapps.io/gameofruns/) by [Ramiro Bentes](https://twitter.com/NbaInRstats)

### Learning R

But these apps were in `R`, and I didn't know `R` then. I started learning R on Nov 7, 2023, after [interacting with another Twitter user](https://twitter.com/SravanNBA/status/1721997698906415168) who wanted someone to recreate a cool-looking Owen Phillips [Bad Calls visualization](https://twitter.com/owenlhjphillips/status/1392475252945350656). I got the code from Owen's [Substack](https://thef5.substack.com/) and modified it just enough to recreate it for the latest data within a [few hours](https://twitter.com/georgemikan/status/1722086399317348702). Hardest part was setting up the `R` development environment in VSCode. 

Because I knew what `R` was, Shiny didn't seem that distant to me anymore. Through Shiny's documentation, I discovered that there is [Shiny for Python](https://Shiny.posit.co/py/). This was excellent news for me since I was much more comfortable writing code in Python as I was a beginner in `R`. Still, I didn't start immediately, for the main reason that I didn't have any ideas for cool apps and didn't have background of existing NBA Analytics works which could then be converted into apps. 

Since that day, I have published many NBA analyses and visualizations on Twitter (you can find the collection of tweets [here](https://twitter.com/SravanNBA/timelines/1355559613777797122)). So now I have a catalog of works I can build apps from. 

Also, between Nov 8 and Dec 21, 2023, I found more apps I liked:
- [Kill/Death Ratio Tracker](https://swishlistanalytics.shinyapps.io/kd_tracker/) by [Swishlist](https://twitter.com/RealSwishList) 
- [Player Combos](https://saurabhrane.shinyapps.io/playerCombos/) by [Saurabh Rane](https://twitter.com/SaurabhOnTap)
- [Positive Residual's collection of Apps](https://positiveresidual.com/apps/) So, a few days ago (Dec 21, 2023), I found enough motivation to start making Shiny apps.

## Getting Started with Shiny
I started my Shiny journey by reading documentation and other resources to get Shiny set up. Shiny has a easy way to generate examples automatically through the command line. So, I tried some of them and was able to play with them a little. Next, I wanted to see how to deploy the example apps to the web, as it would be a trial run for when I publish the apps I would make in the future. 

### Deploying Considerations

Most of the apps I've shared in the introduction of this blog are hosted by [shinyapps.io](https://www.shinyapps.io/). Posit, the company behind Shiny and RStudio, makes it easy to deploy apps to shinyapps.io v[shinyapps.io](https://www.shinyapps.io/) directly via RStudio. But I don't use RStudio because I'm developing in python. Looking at their [documentation](https://Shiny.posit.co/py/docs/deploy-cloud.html) for Python they have a package called `rsconnect-python` that enables you to push your Python Shiny app to shinyapps.io. Then I went to sign up at shinyapps.io and looked at the free tier. Here are the *perks* of a free account:
  - 5 application limit
  - 25 active hours
For a free option that is not good, at least compared to Heroku [(R.I.P)] (https://www.techtarget.com/searchsoftwarequality/news/252524336/Heroku-to-end-free-tiers-creating-platform-void-for-devs) I used for my [WNBA RAPM Application](https://github.com/sravanpannala/WNBA) (also R.I.P) around 2.5 years ago.   
But the difference in me from back then is that I now have experience self hosting stuff on my Single Board Computers (SBC) like [Raspberry Pi](https://www.raspberrypi.org/) running Linux. I became so interested in Linux's flexibility as an operating system that I became a developer for the SBC (also called [ARM](https://www.arm.com/) machines) version of a popular Linux distribution called [EndeavourOS](https://endeavouros.com/).  

Now, getting back to Shiny (before running off on another tangent), there is good [documentation](https://Shiny.posit.co/py/docs/deploy-on-prem.html) on self-hosting Shiny apps. As an open-source advocate, my eyes immediately landed on the [open-source option](https://shiny.posit.co/py/docs/deploy-on-prem.html#deploy-to-shiny-server-open-source).

## Building an ARM Server
I faced a huge issue right away. The documentation didn't have a way to deploy a Shiny server for ARM devices.  
So, I immediately did a web search, which landed me at [hvalev/shiny-server-arm-docker](https://github.com/hvalev/shiny-server-arm-docker), which had a docker image for Shiny-Server on ARM.  
Again, there was a small (well, maybe big) issue. It was shipped with Python 3.7 and didn't have Python 3.11, since it's based on [Debian Buster](https://www.debian.org/releases/buster/) and didn't come with any preinstalled packages required for data science (like ggplot, etc.).   
There was a way to add the packages later, but the Python version was a deal breaker since I needed at least Python 3.9 for the packages I use. Therefore, my only option was to build my image based on `hvalev/shiny-server-arm-docker` with my custom modifications.

### Features I needed:
- `python3.11`: Recent version for future-proofing.
- Standard data handling and visualization packages:
    - `python`:
        - `shiny`: for running Python Shiny apps
        - `plotnine`: `ggplot` for Python
        - `seaborn`: plotting
        - `plotly`: interactive visualization
        - `scikit-learn`: statistical analysis
        - `pyarrow`: for reading data stored in `parquet` files (a better alternative to `csv`)
    - `R`:
        - `tidyverse` : data science
        - `gt` : great looking tables
        - `gtExtras` : additional features for `gt`
        - `ggimage` : add images to `ggplot`
        - `ggtext` : modifying text in `ggplot`
        - `scales` : axis scales for `ggplot`

### Extending Existing Image
Docker makes it easy to extend existing images by using a `Dockerfile`. A `Dockerfile` usually looks like this:
```
FROM base_docker_image_name

Do operations
```
In our case, this would be:
```
FROM hvalev/shiny-server-arm:latest

Do operations
```
You can have the `Dockerfile` execute a list of operations to configure the image to your needs. Then, a docker image can be created from the `Docker file` using `docker build` command. In the directory with the `Dockerfile`, run this command:
```
docker build -t image_name .
```
It's as simple as that. 

### Dockerfile

Lets look at the operations I performed to modify the image. We start by pulling the existing image and update it. In docker you can get the working directory using `WORKDIR` instruction and run a command using the default shell using `RUN` instruction. These instructions are similar to the `FROM` instruction we've seen before.
```
FROM hvalev/shiny-server-arm:latest
WORKDIR /root
RUN apt update -y && apt upgrade -y
```
Since we have to build python3.11, we need to install these packages:
```
RUN apt install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev
```
Let's download the Python source file and extract it for compiling:
```
RUN wget https://www.python.org/ftp/python/3.11.6/Python-3.11.6.tgz &&\
    tar -xvf Python-3.11.6.tgz
```
We can then build Python using these commands adopted from [this site](https://linuxize.com/post/how-to-install-python-3-9-on-debian-10/):
```
WORKDIR /root/python-3.11.6
RUN ./configure --enable-optimizations
RUN make -j 4
RUN make altinstall
```

I running make with `-j4` makes compiling Python faster using 4 threads instead of 1.
I then run some cleanup to make sure the image size remains as small as possible
```
RUN rm -rf ./python-3.11.6*
RUN apt clean -y
RUN apt autoremove -y
```
Next, let's install the Python packages, starting by upgrading pip:
```
RUN pip3.11 install --no-cache-dir --upgrade pip
RUN pip3.11 install --no-cache-dir shiny plotnine seaborn plotly pyarrow scikit-learn
```
We are getting to the last stages now. Let's install the R packages. These need additional dependencies to compile correctly, so let's install those first. I made this list of dependencies by looking at the output of failed builds. 
```
RUN apt install -y libv8-dev libharfbuzz-dev libfribidi-dev libmagick++-dev
RUN R -e "install.packages(c('tidyverse','gt','gtExtras','ggimage','ggtext','scales'), Ncpus = 4, repos='https://repo.miserver.it.umich.edu/cran/')"
```

Similar to compiling python, we set `Ncpus = 4` to utilize 4 threads and speed up the builds. During my testing, it cut down build time from 45 mins to 15 minutes (3x speedup). Finally, let's create a shared folder to store data
```
RUN mkdir -p /var/data/
```
We need to run `docker build` now to build the image:
```
docker build -t shiny-server-arm-python .
```
## Deploying the Server

Yay, we have an image with Shiny-Server, which can run on ARM devices and has both Python and R support. But, we don't have a server yet. First, we need to configure the server, which we can do by creating a `docker-compose.yml` file in an empty folder where you want your server to reside, with the following contents:

```
version: "0.1"
services:
  shiny-server:
    image: sradjoker/shiny-server-arm-python:latest
    container_name: shiny-server
    ports:
      - 3838:3838
    volumes:
       - ~/shiny-server/apps:/srv/shiny-server/
       - ~/shiny-server/logs:/var/log/shiny-server/
       - ~/shiny-server/conf:/etc/shiny-server/
     # Optional: Mount additional data directory
     #  - ~/nbadata/:/var/data/
    restart: always
```
After saving the file run the following command:
```
docker-compose up -d
```
The server will be available at:
```
http://localhost:3838/
```
Remember that this is just a local server available on your host computer and other computers in the network. Connecting it to your domain, as I did at: 
> [https://shiny.sradjoker.cc/](https://shiny.sradjoker.cc/)

needs tunneling your port to your domain. I tunnel through Cloudflare. You can find the instructions on how to do it [here](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/). To use this Cloudfare method to deploy the server to the web, you must first let Cloudflare manage your website, which can be done by following the instructions [here](https://developers.cloudflare.com/fundamentals/setup/account-setup/add-site/). 

# Conclusion  
We have seen my journey to start making Shiny apps, including me starting with example apps but immediately transitioning into making a server to host my apps on (I find it funny, do you). 
I thought the server image that I've created would be a tool that might be helpful for other people interested in  self-hosting their Shiny apps, so I released it on Github:


[https://github.com/sravanpannala/shiny-server-arm-python](https://github.com/sravanpannala/shiny-server-arm-python). 

You can find instructions on how to make your Shiny server there. 

That's it for Part I. In Part II, I will talk about my first original Shiny Apps. Keep an eye out for it.