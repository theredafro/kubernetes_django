# Kubernetes, Local to Production with Django and PostgreSQL

## Introduction

This repo is largely a result of the work done by [Mike Gituma](https://markgituma.medium.com/kubernetes-local-to-production-with-django-1-introduction-d73adc9ce4b4) and a clone from his repo. Please have a look at his article to get a more thorough explanation to how the process goes.

I will be making a few changes, in line with my interest and also using Docker Desktop. This comes in the Mac or Windows flavours. You should be able to follow along, whichever you pick.

### 1. Requirements

[Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac/)
[kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

If you are on an a Linux OS, you should look at [Microk8s](https://microk8s.io/) by Canonical who also build Ubuntu and [Minikube](https://minikube.sigs.k8s.io/docs/start/) as a way to spin up kubernetes clusters.

Once you have installed the K8s tools of choice, check which `kubectl` environment you are in.

```
$ kubectl config current-context
docker-desktop
```

To confirm that the `docker` command line interface is using the docker-desktop docker daemon, run:

```
$ docker info | grep Name
Name: docker-desktop 
```
The rest of the instructions to this tutorial are in the following files:

[ Containerised Django](./docs/part2_containerise_django.md)
