
## Introduction

Podman is an open source tool that you can use to manage your containers locally. 

By default, Podman is daemonless. 

- It does not use a daemon to proxy the requests, which means no single point of failure.

- It does not require elevated privileges.

A container is an isolated runtime environment where applications are executed as isolated processes.

A container image contains a packaged version of your application, with all the dependencies necessary for the application to run. You create your containers from container images.


## Installing Podman

```bash
sudo yum install container-tools -y
```

Checking the version

```bash
podman -v
```


## Working with Podman

### Pulling and displaying images

```bash
podman pull docker.io/nginx:latest
```

```bash
podman images
```

### Running Containers

```bash
podman run docker.io/nginx:latest
```

```bash
podman ps
```

Run a container named web-1 in detached mode

```bash
podman run --name web-1 -d docker.io/nginx:latest
```

Run a container named web-2 in detached mode and map port 8080 in the host to port 80 in the container from the `docker.io/nginx:latest` image

```bash
podman run --name web-2 \
  --detached \
  -p 8080:80 \
  docker.io/nginx:latest
```



### Guided Exercise - Create a basic container

Using Podman, create a container with the following settings:
  
- Name the container __web-c1__ using the `--name` parameter.
- Run the container in __detached__ mode using the `--detached` parameter.
- Redirect port __8080__ in the port to port __80__ in the container using the `-p` parameter.
- Use the __Apache httpd__ image from Docker Hub located in `docker.io/httpd` with the lastest tag.



```bash
podman run --name web-c1 -p 8080:80 \
  --detached \
  --image=docker.io/httpd
```

Check the created containers.

```bash
podman ps
```

Check the access to the web server inside the container through port `8080` in the ost.

```bash
curl localhost:8080
```




    
