---
layout: default
title: Overview
---

This workshop is based on the [Docker 101 Tutorial](https://www.docker.com/101-tutorial/){:target="_blank"}, created by Docker and others. I wanted to focus on different aspects hence the fork. 

For this workhop I used the original Docker 101 Todo app and a great deal of the original texts from the [Docker 101 Github repo](https://github.com/docker/getting-started){:target="_blank"}. A big thanks to Docker, Inc. for placing their work under an Apache 2.0 license.

## Objectives

In this workshop, you will learn some Docker or Container basics, like:

* Creating a Dockerfile
* Creating a container image
* Starting a container
* Container networking
* Mounting external "volumes" onto containers
* A little bit about docker-compose

## Prerequisites

* [git](https://git-scm.com/downloads){:target="_blank"}
* [Docker](https://docs.docker.com/desktop/){:target="_blank"}

Docker can be Docker CE for Linux or Docker Desktop for Mac, Windows, or Linux. We will not cover installation of Docker (Desktop) in this workshop.  

> **Note: bwLehrpool** has all the required software installed. Change into the PERSISTENT directory before cloning the repository in the next step.

## Labs

We are going to deploy a ToDo app based on Node.js. It can run "stand-alone" using a built in SQLite database or it can connect with a external MySQL database. 

- [1 - Deploy ToDo stand-alone](workshop/lab1.md)
- [2 - Update the app, build a new image](workshop/lab2.md)
- [3 - Persisiting the data, Volumes](workshop/lab3.md)
- [4 - Add MySQL DB, Multi-Container apps](workshop/lab4.md)
- [5 - Image Building Best Practises](workshop/lab5.md)
- [6 - Docker Compose](workshop/lab6.md)
- [7 - Tips and useful commands](workshop/lab7.md)

