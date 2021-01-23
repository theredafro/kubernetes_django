
# Kubernetes: Local to Production with Django and PostgreSQL

## 1. Introduction

This repo is largely a result of the work done by [Mike Gituma](https://markgituma.medium.com/kubernetes-local-to-production-with-django-1-introduction-d73adc9ce4b4) and a clone from his repo. Please have a look at his articles to get a more thorough explanation to how the process goes.

I will be making a few changes, in line with my interest in using Docker Desktop. This comes in the Mac or Windows flavours. You should be able to follow along, whichever you pick.

### Requirements

[Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac/)

[Minikube](https://minikube.sigs.k8s.io/docs/start/)

[kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

If you are on an a Linux OS, you should look at [Microk8s](https://microk8s.io/) by Canonical who also build Ubuntu as a way to spin up kubernetes clusters.

[Mike](https://twitter.com/MarkGituma) uses `minikube`, an industry standard and after looking through it, I see why. I will be using `minikube` that will utilise the Docker Desktop drivers to create a container where it will run. Minikube has other driver choices such as `hyperkit, virtualbox, ssh`.

Once you have installed the K8s tools of choice, check which `kubectl` environment you are in.
```
$ kubectl config current-context
docker-desktop
```
We will change the context so that we can run inside `minikube`.  

`$ eval $(minikube docker-env)`

To confirm that the `docker` command line interface is using the minikube docker daemon, run:
```
$ docker info | grep Name
Name: minikube

$ kubectl config current-context
minikube
```
We are ready. The rest of the instructions to this tutorial are in the following files:

[2. Containerised Django](../docs/part2_containerise_django.md)

[3. Setting up PostgreSQL ](../docs/part3_setting_up_postgres.md)