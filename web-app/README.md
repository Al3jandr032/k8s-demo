# Simple web app

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in your browser.

The page will reload when you make changes.\
You may also see any lint errors in the console.


## Docker

Build

```console
user@user-laptop1:~$ docker build -t registry.dev.svc.cluster.local:5000/web-app:nginx .
```

> Note: Using a local registry: registry.dev.svc.cluster.local at 5000 port

Push 

```console
user@user-laptop1:~$ docker push registry.dev.svc.cluster.local:5000/web-app:nginx
```
Run

```console
user@user-laptop1:~$ docker run --name web-app -p 5050:80 registry.dev.svc.cluster.local:5000/web-app:nginx
```


Port-forward

```console
user@user-laptop1:~$ kubectl port-forward web-app-58bfd7d74-59wch 3000:80
```