# k8s-demo
A Demo to deploy applications in k8s


## Prerequisits

* [Docker](https://docs.docker.com/engine/install/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)


### Minikube

To install [minikube](https://minikube.sigs.k8s.io/docs/start/) on ubuntu you will need:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Check instalation

```bash
user@user-laptop1:~$ minikube version
minikube version: v1.13.0
commit: 0c5e9de4ca6f9c55147ae7f90af97eff5befef5f-dirty
```

Before start check if you have [drivers](https://minikube.sigs.k8s.io/docs/drivers/) available. Can use docker as a defualt driver on linux.


Start your cluster

```bash
user@user-laptop1:~$ minikube start 
😄  minikube v1.13.0 on Ubuntu 20.04
✨  Automatically selected the docker driver
👍  Starting control plane node minikube in cluster minikube
🔥  Creating docker container (CPUs=2, Memory=3900MB) ...
🐳  Preparing Kubernetes v1.19.0 on Docker 19.03.8 ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass, storage-provisioner

❗  /usr/bin/kubectl is version 1.23.4, which may have incompatibilites with Kubernetes 1.19.0.
💡  Want kubectl v1.19.0? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" by default
```

Check cluster status

```bash
user@user-laptop1:~$ minikube kubectl -- get po -A
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
default       simple-server-api-59595d46d9-b9kn7   1/1     Running   0          41m
kube-system   coredns-f9fd979d6-cc79p              1/1     Running   0          57m
kube-system   etcd-minikube                        1/1     Running   0          57m
kube-system   kube-apiserver-minikube              1/1     Running   0          57m
kube-system   kube-controller-manager-minikube     1/1     Running   1          57m
kube-system   kube-proxy-4mnms                     1/1     Running   0          57m
kube-system   kube-scheduler-minikube              1/1     Running   0          57m
kube-system   storage-provisioner                  1/1     Running   0          56m
```

# Deploy simple python server

Move to server folder

```console
cd python-simple-server/
```

Follow readme file instructions. Then when you have the docker image ready and you minikube up and running.

Create deployment and service objects using k8s cli or kubectl as follow:

* Deployment
```console
kubectl apply -f deployment.yaml 
```

* Service
```console
kubectl apply -f service.yaml 
```

* Ingress
```console
kubectl apply -f ingress.yaml 
```

# Using a Local Registry with Minikube

## Install a local Registry

These instructions include running a local registry accessible from Kubernetes as well as from the 
host development machine at `registry.dev.svc.cluster.local:5000`.

1.  Use the docker CLI to run the `registry:2` container from Docker, listening on port `5000`, and persisting images in the `~/.registry/storage` directory.

    ```
    docker run -d -p 5000:5000 --restart=always --volume ~/.registry/storage:/var/lib/registry registry:2
    ````

2.  Edit the `/etc/hosts` file on your development machine, adding the name `registry.dev.svc.cluster.local` on the same line as the entry for `localhost`.

3.  Validate that the registry is running.

    ```
    docker ps
    ```

    ```
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    02ea46d51f58        registry:2          "/entrypoint.sh /etc…"   About an hour ago   Up About a minute   0.0.0.0:5000->5000/tcp   sharp_pike
    ```

4.  Validate that the registry at `registry.dev.svc.cluster.local:5000` is reachable from your development machine.

    ```
    curl registry.dev.svc.cluster.local:5000/v2/_catalog
    ```
    
    ```
    {"repositories":[]}
    ```

5.  Configure the docker daemon with an insecure registy at `registry.dev.svc.cluster.local:5000`. 

    On macOS in `~/.docker/daemon.json` on Linux in `/etc/docker/daemon.json` (craete the file if it does not exist)
    
    ```
    {
      "insecure-registries": ["registry.dev.svc.cluster.local:5000"]
    }
    ```

## Start Minikube

```
minikube start --cpus 4 --memory 4096 --insecure-registry registry.dev.svc.cluster.local:5000
```

## Configure a fixed IP address

This IP address will allow processes in Minikube to reach the registry running on your host. Configuring a fixed IP address avoids the problem of the IP address changing whenever you connect your machine to a different network. If your machine already uses the 172.16.x.x range for other purposes, choose an address in a different range e.g. 172.31.x.x..

```
export DEV_IP=172.16.1.1
```

Create an alias on MacOS:

```
sudo ifconfig lo0 alias $DEV_IP
```

Create an alias on Linux:

```
sudo ifconfig lo:0 $DEV_IP
```

Note that the alias will need to be reestablished when you restart your machine.
This can be avoided by using a launchdeamon on MacOS or by editing `/etc/network/interfaces` on Linux.

## Minikube /etc/hosts

Add an entry to `/etc/hosts` inside the minikube VM, pointing the registry to the IP address of the host.
This will result in `registry.dev.svc.cluster.local` resolving to the host machine allowing the docker daemon in minikube to pull images from the local registry.
This uses the `DEV_IP` environment variable from the previous step.

```
export DEV_IP=172.16.1.1
minikube ssh "echo \"$DEV_IP       registry.dev.svc.cluster.local\" | sudo tee -a  /etc/hosts"
```

## Kubernetes Service and Endpoint

Create a kubernetes service without selectors called registry in the dev namespace and a kubernetes endpoint with the same name pointing to the static IP address of your development machine.
This will result in registry.dev.svc.cluster.local resolving to the host machine, allowing container builds running in the cluster, to work with the local registry.

```
kubectl create namespace dev
```

```
cat <<EOF | kubectl apply -n dev -f -
---
kind: Service
apiVersion: v1
metadata:
  name: registry
spec:
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
---
kind: Endpoints
apiVersion: v1
metadata:
  name: registry
subsets:
  - addresses:
      - ip: $DEV_IP
    ports:
      - port: 5000
EOF
```