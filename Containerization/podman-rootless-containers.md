
# Rootful and Rootless Containers

Podman enables "rootless containers," allowing regular, non-privileged users to create, run, and manage containers without sudo or root privileges. This approach significantly enhances security by preventing container escape attacks from gaining host-level root access, utilizing user namespaces and daemonless architecture to map container root users to the host user

### Run a rootfull container and check the processes running as root

```bash
sudo podman run \
  --name web-rootfull \
  -d -p 80:80 docker.io/httpd
```

```bash
sudo ps aux | grep httpd
```

### Run a rootless container and check the processes running a regular user

```bash
podman run –name web-rootless \
  -d -p 8080:80 docker.io/httpd
```

```bash
sudo ps aux | grep httpd
```
