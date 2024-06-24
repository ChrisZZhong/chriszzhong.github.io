---
layout: post
title: "OpenShift"
date: 2024-05-24
description: "intro to Openshift"
tag: CI/CD
---

## Architect

![72b820f0422704dd0db458c3751b3a3](https://github.com/ChrisZZhong/chriszzhong.github.io/assets/90562417/47b49a00-e115-4fd7-9ee7-ccd12746de24)

InfraNodes are only responsible for the routing and use as much resource as possible to process more request in the same time. Pods are running inside of the workerNode, when traffic comes, it first comes tp the infraNodes -> service -> pod and the response comes to the caller back this way.
