# Running Flask on Kubernetes

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
