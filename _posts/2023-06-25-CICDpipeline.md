---
layout: post
title: "Deploy Application, CICD pipeline"
date: 2023-06-25
description: "Deploy Application"
tag: Microservices
---

# Deploy Application

## IntelliJ -> GitHub -> Jenkins -> DockerHub -> Kubernetes -> AWS

[How To Push a Docker Image To Docker Hub Using Jenkins](https://medium.com/codex/how-to-push-a-docker-image-to-docker-hub-using-jenkins-487fb1fcbe25)

[Jenkins 将 Docker 映像部署到 Kubernetes](https://blog.51cto.com/u_15976398/6166363)

1. write code in IntelliJ and then push to GitHub

2. Jenkins stages (jenkinsfile):

   - pull code from GitHub and build, run, test the project

   - Building a Docker image

   - Pushing the Docker image to DockerHub

   - Pulling the Docker image from DockerHub and creating a container

   - Deploying the container to Kubernetes or (EKS elastic Kubernetes service on AWS)

## Jenkins see [here](https://chriszzhong.github.io/2023/06/jenkins/)

## Docker & DockerHub

Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and wrap it all out as one package.

DockerHub is a Cloud service that allows you to upload and download Docker images.

## Kubernetes

Kubernetes is an open-source container-orchestration system for automating computer application deployment, scaling, and management.

## AWS EKS

Amazon Elastic Kubernetes Service (Amazon EKS) is a fully managed Kubernetes service.
