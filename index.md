---
layout: default
title: Overview
---

This workshop is based on the [Docker 101 Tutorial](https://www.docker.com/101-tutorial/), created by Docker and others. I wanted to focus on different aspects hence the fork. I used the original todo app and a great deal of the original texts from [here](https://github.com/docker/getting-started).

## Objectives

In this workshop, you will learn some Docker or Container basics, like:

* creating a Dockerfile
* creating a container image
* starting a container
* container networking
* mounting external "volumes" onto containers

## Prerequisites

* [git](https://git-scm.com/downloads)
* [Docker](https://docs.docker.com/desktop/)

Docker can be Docker CE for Linux or Docker Desktop for Mac or Windows. We will not cover installation of Docker (Desktop) in this workshop.  

> **bwLehrpool** has all the required software installed. Change into the PERSISTENCE directory before cloning the repository in the next step.

## Get the code

```
git clone https://github.com/Harald-U/docker-101.git
cd docker-101/app
```

## Labs

We are going to deploy a ToDo app based on Node.js. It can run "stand-alone" using a built in SQLite database or it can connect with a external MySQL database. 

- [Lab 1](workshop/lab1.md) - Deploy ToDo stand-alone
- [Lab 2](workshop/lab2.md) - Update the app, build a new image
- [Lab 3](workshop/lab3.md) - Persisiting the data, Volumes
- [Lab 4](workshop/lab4.md) - Add MySQL DB, Multi-Container apps
- [Lab 5](workshop/lab5.md) - Image Building Best Practises

