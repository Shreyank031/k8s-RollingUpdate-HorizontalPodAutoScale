# Nginx Deployment with Rolling Update

This repository contains a Kubernetes Deployment manifest for deploying an Nginx web server with a rolling update strategy.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Deployment](#deployment)
- [Rolling Update](#rolling-update)
- [Cleanup](#cleanup)

## Overview

The `env-deployment.yaml` file defines a Kubernetes Deployment resource that deploys four replicas of an Nginx web server. The Deployment uses a rolling update strategy to ensure zero downtime during updates.

## Prerequisites

Before deploying the Nginx application, ensure that you have the following prerequisites:

- A Kubernetes cluster up and running
- The `kubectl` command-line tool installed and configured to communicate with the cluster

## Deployment

To deploy the Nginx application, follow these steps:

1. Apply the Deployment manifest:

```bash
kubectl apply -f env-deployment.yaml
```
The above command will create the Deployment resource and spin up four replicas of the Nginx Pods.

2. Verify that the Deployment and Pods are running:
```bash
kubectl get deployment env-deployment
kubectl get pods
```

You should see the `env-deployment` Deployment and four Nginx Pods in the running state.


## Rolling Update

The `env-deployment.yaml` manifest defines a rolling update strategy for the Deployment. When you update the Deployment, such as changing the Nginx image version or modifying the container configuration, Kubernetes will perform a rolling update without causing any downtime.

###  Here are the key parameters defined for the rolling update strategy:

- `maxSurge: 1` : Allows one extra Pod to be created during the update process. The new ReplicaSet is created with one extra `Pod` running updated nginx:1.17 image. The old ReplicaSet will be as it is running 4 pods with nginx:1.16 version.

- `maxUnavailable: 0` : Ensures that all Pods are available during the update process. Assume if 3 pods are running out of 4 pods (the desired number of replicas), then k8s won't update. It's typically `4 - 0 = 4` where all the 4 pods should be up and running.

- `minReadySeconds: 10` : Specifies that a new Pod should be ready for at least 10 seconds before considering it available. Then only the one of the pod in old ReplicaSet will be deleted automatically. Suppose if the pod is not running as expected in newly created ReplicaSet, then k8s waits for 10, checks in again waits, doesn't udpate if the pod is not running with new image.
