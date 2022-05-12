# Simple python server

To deploy this sever first build docker image

## Bare python


Run main.py as follow:

```console
user@user-laptop1:~$ python main.py 
INFO:root:Starting httpd...
```


## Docker

Build

```console
user@user-laptop1:~$ docker build -t registry.dev.svc.cluster.local:5000/simple-server:dev-python3.8 .
```

> Note: Using a local registry: registry.dev.svc.cluster.local at 5000 port

Push 

```console
user@user-laptop1:~$ docker push registry.dev.svc.cluster.local:5000/simple-server:dev-python3.8
```
Run

```console
user@user-laptop1:~$ docker run --name python-api -p 8080:8080 registry.dev.svc.cluster.local:5001/simple-server:dev-python3.8 main.py 8080
INFO:root:Starting httpd...
```

## Testing the server

1. make a request

```console
user@user-laptop1:~$ curl localhost:8080
GET request for /
```
2. On log server you will get request info

```log
INFO:root:GET request,
Path: /
Headers:
Host: localhost:8080
User-Agent: curl/7.68.0
Accept: */*



172.17.0.1 - - [12/May/2022 05:12:49] "GET / HTTP/1.1" 200 -
```