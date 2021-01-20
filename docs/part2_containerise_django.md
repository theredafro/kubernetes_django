### 2. Containerised Django
Kubernetes expects a containerised application, we will be using `docker` to get started.

We will use the `Dockerfile` that is in the repo `getting-started` branch.
```
FROM python:3.8-slim
LABEL maintainer="jimmygitts@gmail.com"

ENV PROJECT_ROOT /app
WORKDIR $PROJECT_ROOT

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD python manage.py runserver 0.0.0.0:8000
```
• The Dockerfile first defines the base image to build from, where in this case it’s the python:3.8-slim image.

• The LABEL instruction is then used to add metadata to the image. This is the recommended way to specify the package maintainer as the MAINTAINER instruction has been deprecated.

• The ENV `<VARIABLE> <var>` directive sets the project root environmental variable, where the variable can be reused in several places as `$<VARIABLE>`. This allows for one point of modification in case some arbitrary variable needs to be changed.

• The current working directory is then set using the WORKDIR instruction. The instruction resolves the `$PROJECT_ROOT` environmental variable previously set. The working directory will be the execution context of any subsequent `RUN, COPY, ENTRYPOINT` or `CMD` instructions, unless explicitly stated.

• The `COPY` instruction is then used to copy the requirements.txt file from the current directory of the local file system and adds them to the file system of the container. Copying the individual file ensures that the `RUN pip install` instruction’s build cache is only invalidated (forcing the step to be re-run) if specifically the `requirements.txt` file changes, leading to an efficient build process. See the docker documentation for further details. It’s worth noting the `COPY` as opposed to the `ADD` instruction is the recommended command for copying files from the local file system to the container file system.

• The required python packages are then installed using the RUN pip install instruction.

• The rest of the project files are then copied into the container file system. This should be one of the last steps as the files are constantly changing leading to more frequent cache invalidations resulting in more frequent image builds.

• The final instruction executed is `CMD `which provides defaults for an executing container. In this case the default is to start the python web server.

The command used to build the docker image is:

`$ docker build -t theredafro/django_k8s:v1`

The built image within the docker-desktop environment can be viewed with:
```
$ docker images                                                                                                                                                                                                              
REPOSITORY                           TAG        IMAGE ID       CREATED         SIZE
theredafro/django_k8s                v1         165cb2bd3ee8   2 minutes ago   165MB
python                               3.8-slim   be5d294735c6   8 days ago      113MB
k8s.gcr.io/kube-scheduler            v1.19.3    aaefbfa906bd   3 months ago    45.7MB
k8s.gcr.io/kube-apiserver            v1.19.3    a301be0cd44b   3 months ago    119MB
k8s.gcr.io/kube-controller-manager   v1.19.3    9b60aca1d818   3 months ago    111MB
k8s.gcr.io/etcd                      3.4.13-0   0369cf4303ff   4 months ago    253MB
k8s.gcr.io/pause                     3.2        80d28bedfe5d   11 months ago   683kB
```
The other images we have in the cluster belong to our docker-desktop.

### 3. Deployments
Kubernetes uses the concept of pods (i.e. a grouping of co-located and co-scheduled containers running in a shared context) to run applications. To deploy these pods, Kubernetes commands can be executed by an imperative or declarative approach.
Imperative commands specify how an operation needs to be performed, a declarative approach is done by using configuration files which can be stored in version control. The preferred method is the declarative approach as the steps can be tracked and audited.

**Imperative**
To deploy _imperatively_, we use the following command:
```
$ kubectl run kubernetes-django --image=theredafro/django_k8s --port=8000
pod/kubernetes-django created
```
This command creates a `Deployment` controller, then the controller creates pods, which consist of containers based on the defined image. The pods are then deployed to the docker-desktop Kubernetes cluster.
```
$ kubectl get pods                                                                                              
NAME                READY   STATUS             RESTARTS   AGE
kubernetes-django   0/1     ImagePullBackOff   0          32s
```
But when we look for the deployment on the terminal, we use:
```
$ kubectl get deployments
No resources found in default namespace.
```
The pod with the containers is not assigned to any namespace and it was not automatically assigned to the `default` namespace.
Interesting. Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces. What [`namespaces`](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) do we have?
```
$ kubectl get namespaces                                                                                        
NAME              STATUS   AGE
default           Active   14m
kube-node-lease   Active   14m
kube-public       Active   14m
kube-system       Active   14m
```
Perhaps these namespces have been given a specific label?
```
$ kubectl get namespaces --show-labels                                                                                       
NAME              STATUS   AGE   LABELS
default           Active   18m   <none>
kube-node-lease   Active   18m   <none>
kube-public       Active   18m   <none>
kube-system       Active   18m   <none>
```
**NOTE:**
Minikube and Microk8s use VM drivers making them different from Docker Desktop that runs on computers that support the Hypervisor framework on Macs and Hyper-V on Windows. With `minikube start`, we get this:
```
...
Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
Let us make sure we are running in the `minikube` environment:
```
$ minikube status

minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
timeToStop: Nonexistent

$ kubectl config current-context
minikube

$ docker info | grep Name
Name: docker-desktop
```
Oops!

Let us configure the docker command line in the host machine  to utilize the docker daemon within minikube by running, `$ eval $(minikube docker-env)`.

And now:
```
docker info | grep Name
Name: minikube
```
Let us deploy our image using `minikube`:
```
$ kubectl run kubernetes-django --image=theredafro/django_k8s --port=8000
pod/kubernetes-django created

$ kubectl get pods
NAME                READY   STATUS         RESTARTS   AGE
kubernetes-django   0/1     ErrImagePull   0          66s

$ kubectl get deployments
No resources found in default namespace.
```
We can use the following _imperative_ command to delete the deployment:
```
$ kubectl delete deployment/kubernetes-django
Error from server (NotFound): deployments.apps "kubernetes-django" not found
```
=> So it is clear that if we use an _imperative_ command, the deployments are NOT assigned a `namespace`, not even the `default` namespace. What we can access are the pods in the single node cluster of both `docker-desktop` and `minikube`.

**Declarative**
This type of deployment uses a <deployment.yaml> file that _declares_ the deployment in a configuration file. This is the preferred method as the steps can be tracked and audited.

Our `deployment.yaml` looks like so:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-django
  labels:
    app: django
spec:
  replicas: 1
  selector:
    matchLabels:
      app: django-container
  template:
    metadata:
      labels: 
        app: django-container
    spec:
      containers:
        - name: django-web
          image: theredafro/django_k8s:v1
          ports:
            - containerPort: 8000
```
We can use the command:
```
$ kubectl apply -f ./deploy/kubernetes/django/deployment.yaml
deployment.apps/kubernetes-django created 

$ kubectl get deployments
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-django   1/1     1            1           77s
```
To delete the deployment we could use the command:
```
$ kubectl delete -f ./deploy/kubernetes/django/deployment.yaml
deployment.apps "django" deleted
```
### 4. Services
When a deployment is created, each pod in the deployment has a unique IP address within the cluster. However, we need some kind of mechanism to allow the access of the pod IP address from outside the cluster.

A Service, sometimes called a _micro-service_, routes traffic across pods which have dynamic IP addresses, and thus being less stable. This means pods can die and be recreated and thus their IP addresses can change, and yet the traffic will always route to the right pods. The Service needs to have a unique static IP with the outside world, abstracting the pods' dynamic IP addresses.

**Imperative**
We can issue a command:
```
$ kubectl expose deployment kubernetes-django --type=NodePort
service/kubernetes-django exposed
```
To view the existing services, we run:
```
$ kubectl get svc
NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
kubernetes          ClusterIP   10.96.0.1     <none>        443/TCP          24m
kubernetes-django   NodePort    10.99.44.15   <none>        8000:32470/TCP   67s
```
Once we open our browser on `http://localhost:32470/`, we get our default Django 3 webpage.

To delete the service imperatively, the command is:
```
$ kubectl delete svc/kubernetes-django
service "kubernetes-django" deleted
```
**Declarative**
Our _declaration_ of the service can be found in the `./deploy/kubernetes/django/service.yaml` file that looks like so:
```
kind: Service
apiVersion: v1
metadata:
  name: kubernetes-django-service
spec:
  selector:
    app: django-container
  ports:
  - protocol: TCP
    port: 8000
    targetPort: 8000
  type: NodePort
```
So we run the command:
```
$ kubectl apply -f ./deploy/kubernetes/django/service.yaml
service/kubernetes-django-service created

$ kubectl get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes                  ClusterIP   10.96.0.1        <none>        443/TCP          45m
kubernetes-django-service   NodePort    10.100.189.139   <none>        8000:31323/TCP   69s
```
We open our browser at `http://localhost:31323/`, we should get our Django 3 web page.