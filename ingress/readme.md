# Cluster Ingress

## Config

Add ingress to minikube

```console
user@user-laptop1:~$ minikube addons enable ingress  
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

Create certificates

```console
user@user-laptop1:~$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls-key.key -out tls-cert.crt  
```

Setting to minikube

```console
user@user-laptop1:~$ kubectl create secret tls tls-certificate --key tls-key.key --cert tls-cert.crt  
```

## Apply ingess manifesto


```console
user@user-laptop1:~$ kubectl apply -f ingress.yaml 
ingress.networking.k8s.io/cluster-ingress created
```