# Aptitude Test

For the purpose of this aptitude test, all docker images have already been built and push to the my personal Docker hub repository and puplicly available for download and use.

This stack was build using the following technologies:

    - Python 3.7
    - Flask the python MicroFramework
    - Vue JS and Node JS
    - PostgreSQL for the database
    - Docker for container image creation
    - Virtualbox to simulate Kubernetes and perform test
    - Minikube to emulate Kubernetes
    - Kubectl for docker Orchestration

# What is Container Orchestration?

As you move from deploying containers on a single machine to deploying them across a number of machines, you'll need an orchestration tool to manage (and automate) the arrangement, coordination, and availability of the containers across the entire system.

Orchestration tools help with:

    - Cross-server container communication
    - Horizontal scaling
    - Service discovery
    - Load balancing
    - Security/TLS
    - Zero-downtime deploys
    - Rollbacks
    - Logging
    - Monitoring

# Kubernetes Concepts

Before diving in, let's look at some of the basic building blocks that you have to work with from the Kubernetes API

    1. A **Node** is a worker machine provisioned to run Kubernetes. Each Node is managed by the Kubernetes master.
    2. A **Pod** is a logical, tightly-coupled group of application containers that run on a Node. Containers in a Pod are deployed together and share  resources (like data volumes and network addresses). Multiple Pods can run on a single Node.
    3. A **Service** is a logical set of Pods that perform a similar function. It enables load balancing and service discovery. It's an abstraction layer   over the Pods. Pods are meant to be ephemeral while services are much more persistent.
    4. **Deployments** are used to describe the desired state of Kubernetes. They dictate how Pods are created, deployed, and replicated.
    5. **Labels** are key/value pairs that are attached to resources (like Pods) which are used to organize related resources. You can think of them like   CSS selectors. For example:
       - Environment - dev, test, prod
       - App version - beta, 1.2.1
       - Type - client, server, db
    6. **Ingress** is a set of routing rules used to control the external access to Services based on the request host or path.
    7. **Volumes** are used to persist data beyond the life of a container. They are especially important for stateful applications like Redis and Postgres.
       - A PersistentVolume defines a storage volume independent of the normal Pod-lifecycle. It's managed outside of the particular Pod that it resides in.
       - A PersistentVolumeClaim is a request to use the PersistentVolume by a user.

## Want to use this project?

### Docker

Build the images and spin up the containers:

```sh
$ docker-compose up -d --build
```

Run the migrations and seed the database:

```sh
$ docker-compose exec server python manage.py recreate_db
$ docker-compose exec server python manage.py seed_db
```

Test it out at:

1. [http://localhost:8080/](http://localhost:8080/)
1. [http://localhost:5001/books/ping](http://localhost:5001/books/ping)
1. [http://localhost:5001/books](http://localhost:5001/books)

### Kubernetes

#### Minikube

Install and run [Minikube](https://kubernetes.io/docs/setup/minikube/):

1. Install a [Hypervisor](https://kubernetes.io/docs/tasks/tools/install-minikube/#install-a-hypervisor) (like [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or [HyperKit](https://github.com/moby/hyperkit)) to manage virtual machines
1. Install and Set Up [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to deploy and manage apps on Kubernetes
1. Install [Minikube](https://github.com/kubernetes/minikube/releases)

Start the cluster:

```sh
$ minikube start --vm-driver=virtualbox
$ minikube dashboard
```

#### Volume

Create the volume:

```sh
$ kubectl apply -f ./kubernetes/persistent-volume.yml
```

Create the volume claim:

```sh
$ kubectl apply -f ./kubernetes/persistent-volume-claim.yml
```

#### Secrets

Create the secret object:

```sh
$ kubectl apply -f ./kubernetes/secret.yml
```

#### Postgres

Create deployment:

```sh
$ kubectl create -f ./kubernetes/postgres-deployment.yml
```

Create the service:

```sh
$ kubectl create -f ./kubernetes/postgres-service.yml
```

Create the database:

```sh
$ kubectl get pods
$ kubectl exec postgres-<POD_IDENTIFIER> --stdin --tty -- createdb -U postgres books
```

#### Flask

Build and push the image to Docker Hub:

```sh
$ docker build -t thieryl/flask-kubernetes ./services/server
$ docker push thieryl/flask-kubernetes
```

> Make sure to replace `thieryl` with your Docker Hub namespace in the above commands as well as in _kubernetes/flask-deployment.yml_

Create the deployment:

```sh
$ kubectl create -f ./kubernetes/flask-deployment.yml
```

Create the service:

```sh
$ kubectl create -f ./kubernetes/flask-service.yml
```

Apply the migrations and seed the database:

```sh
$ kubectl get pods
$ kubectl exec flask-<POD_IDENTIFIER> --stdin --tty -- python manage.py recreate_db
$ kubectl exec flask-<POD_IDENTIFIER> --stdin --tty -- python manage.py seed_db
```

#### Ingress

Enable and apply:

```sh
$ minikube addons enable ingress
$ kubectl apply -f ./kubernetes/minikube-ingress.yml
```

Add entry to _/etc/hosts_ file:

```
minikube ip
<MINIKUBE_IP> vdia-test.world
```

Try it out:

1. [http://vdia-test.world/books/ping](http://vdia-test.world/books/ping)
1. [http://vdia-test.world/books](http://vdia-test.world/books)

#### Vue

Build and push the image to Docker Hub:

```sh
$ docker build -t thieryl/vue-kubernetes ./services/client \
    -f ./services/client/Dockerfile-minikube
$ docker push thieryl/vue-kubernetes
```

> Again, replace `thieryl` with your Docker Hub namespace in the above commands as well as in _kubernetes/vue-deployment.yml_

Create the deployment:

```sh
$ kubectl create -f ./kubernetes/vue-deployment.yml
```

Create the service:

```sh
$ kubectl create -f ./kubernetes/vue-service.yml
```

Try it out at [http://vdia-test.world/](http://vdia-test.world/).
