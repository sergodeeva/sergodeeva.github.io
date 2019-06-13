---
layout: post
title: "Dockerizing Next.js application"
date: 2019-06-13 23:15:00 +0800
tags: coding devops
description: "How to get a Next.js application into a Docker container"
image: "/assets/post_images/2019-06-13-docker.png"
---

One former colleague of mine used to say that even a monkey can become a DevOps engineer if put some effort. So I asked him to make DevOps engineer out of me. "Sure", he said and gave me my first assignment: to read a book "How Linux Works: What Every Superuser Should Know" by Brian Ward. I abandoned the book in the middle, not because it was a boring book, but because it was hard to read the pure theory without applying it. However since then I got some interest in devops stuff.

Yesterday I tryied to dockerize my next.js application and was surprised how easy it was. If you the same as me before think that Docker is too complex, I can say that it's not (or maybe I have not got into complexity yet). Below are the steps I performed to get my app into a docker container.

## Install Docker

You can download Docker installer from the [official site](https://docs.docker.com/install/). Installation went smooth on my Mac machine, however I got into minor issues when was doing the same on Linux CentOS.
Here are the installation steps for CentOS:

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce
sudo usermod -aG docker $(whoami)
sudo systemctl enable docker.service
sudo systemctl start docker.service
```

If there is an error during `docker-ce` installation, this command might help:

```
sudo yum install http://vault.centos.org/centos/7.3.1611/extras/x86_64/Packages/container-selinux-2.9-4.el7.noarch.rpm
```

## Create Dockerfile

Create Dockerfile and place it in the root of your Next.js application. Here is my Dockerfile:

```
FROM node:alpine

# Set working directory. All the paths will be relative to WORKDIR
WORKDIR /opt/dist

# Install PM2 globally
RUN npm install --global pm2

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy source files
COPY . .

# Build the app
RUN npm run build

# Run the app
CMD [ "pm2-runtime", "start", "npm", "--", "start" ]
```

Btw, do not repeat my mistake: originally the first line of my docker file was `FROM node:12`. As a result a final docker image was huge, about 1.3 Gb. I spent almost 1 hour trying to understand what makes the image so large, then found [the github issue](https://github.com/nodejs/docker-node/issues/311) which recommends to use `node:slim` and `node:alpine` variants that are much more reasonable in size.

## Build Docker image

Run this command to create a docker image:

```
docker build -t my_app .
```

## Run Docker container from the image

This is the last step. To run Docker container type:

```
docker run -d -p 3000:3000 my_app:latest
```

After that your application will be available at localhost:3000.
Easy, right?

## A few useful Docker commands

To wrap it up, here are few Docker commands that I found useful during setup:

```
docker ps # to see a list of running containers
docker stop [container_id] # to stop given container
docker images # to see a list of docker images
docker logs [first 3 characters of container id] # to see a log of a given container
docker rmi -f $(docker images -a -q) # to remove all docker images
```

Coming back to the beginning of the article, have I became a DevOps after getting my Next app into Docker? Of course, no! It's just baby steps, but I enjoy it a lot.
