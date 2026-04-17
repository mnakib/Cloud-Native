# Creating container images to run on OpenShift 

OpenShift does run containers as non-root users by default, which is a core security feature. If your container fails to run on OpenShift, it is likely due to one of these common restrictions: 

- __Restricted Security Context Constraints (SCC):__ By default, OpenShift applies the `restricted` or `restricted-v2` SCC. This forces containers to run with a __randomly assigned, high-number UID__ (e.g., 1000670000) rather than the UID defined in the Dockerfile.

- __Permission Denied Errors:__ Many community images (like standard Nginx or MariaDB) expect to run as root or a specific UID like 1001. When OpenShift forces a random UID, the container may lack permissions to write to its own directories (e.g., `/var/cache/nginx`).

- __Privileged Ports:__ Containers on OpenShift cannot bind to "privileged" ports (below 1024) by default. If your application tries to listen on port 80, it will fail unless you change it to a higher port like 8080.


To fix those potential restrictions you may either:

- __Modifying the Image:__ You may modify the image and make it "OpenShift-ready". by ensuring all required directories (logs, cache, data) are owned by the root group (`gid=0`) and have group-write permissions. Because OpenShift sets the random user's group to 0, the user itself will have the necessary permissions to access those directories. It's worth noting that making a user part of the `root` group in RHEL does not make them a true `root` user (UID 0). This membership only gives the user read/write access to specific files that happen to be owned by that group. It does not allow you to bypass general system security.

- __Using a Specific SCC:__ If you absolutely must run as a specific user or as root, an administrator can grant your ServiceAccount access to a different SCC, such as `anyuid` or `privileged`, but this is generally not recommended.

- __Using OpenShift-Optimized Images:__ Images from the [Red Hat Ecosystem Catalog](https://catalog.redhat.com/en/search?searchType=Containers) (like Red Hat UBI) are pre-configured to work with OpenShift's security constraints out of the box.

In this guide, we'll take a default Docker httpd image and tweak to serve on port 8080 instead of port 80, and provide the necessary permissions to the root group on the folders used by Apache, in this case; `/usr/local/apache2/logs`.


## The Containerfile content

This Dockerfile contains the instructions to create a custom image exposing an `httpd` container on port `8080` 

```Dockerfile
FROM docker.io/httpd

RUN sed -i 's/Listen 80/Listen 8080/g' /usr/local/apache2/conf/httpd.conf

RUN chgrp -R 0 /usr/local/apache2/logs && chmod -R g=u /usr/local/apache2/logs

# USER root

EXPOSE 8080

CMD ["httpd-foreground"]
```


## Create the image and push it Quay.io

We'll need to create an image from above Containerfile, then push to an image registry, public or private, that is accessible to OpenShift. In below example, we'll be using [quay.io](https://quay.io/) registry to push our image.

```bash
podman build -t apache-server .
```

```bash
podman login quay.io -u <username> -p <password>
```

```bash
podman push localhost/apache-server quay.io/<repository-username>/apache-server
```

## Deploy the image from OpenShift

Because this is a private image (unless you make it public), you will need to create a secret containing the private image registry credentials and link it to the account configured inside the deployment to be able to pull the image. The deployment is using the `default` account by default, so you'll just have to link to it. If you're using a different service account than `default`, the secret has to be linked to that custom service account, but in this example, we'll just make it simple and link the secret to the `default` service account.

Create the secret

```bash
oc create secret docker-registry quay-creds \
    --docker-username=<registry-username> \
    --docker-password=<registry-password> \
    --docker-server=quay.io
```

Link the secret to the service account

```bash
oc secrets link default quay-creds --for=pull
```

Create the deployment

```bash
oc create deployment apache-server --image=quay.io/<repository-username>/apache-server
```
