debian-ssh
==========

Debian Docker images with *passwordless* SSH access and sudo right for docker user, ready to ansible

Using
-----

The images are built by [Docker hub](https://hub.docker.com/r/bibiche42/debian-ssh).
Each Debian release corresponds to a tag.  To run an SSH daemon in a new Debian "buster"
container:

    docker run -d -p 2222:22 -e SSH_KEY="$(cat ~/.ssh/id_rsa.pub)" bibiche42/debian-ssh:wheezy

This requires a public key in `~/.ssh/id_rsa.pub`.

Two users exist in the container: `root` (superuser) and `docker` (a regular user
with passwordless `sudo`). SSH access using your key will be allowed for both
`root` and `docker` users.
To connect to this container as root:

    ssh -p 2222 root@localhost

To connect to this container as regular user:

    ssh -p 2222 docker@localhost

Change `2222` to any local port number of your choice.


### With Kubernetes
If you have a kubernetes cluster ready, you can install metallb loadbalancer and deploy debian-ssh image.

I. Install last version of metallb
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

II. Create configmap for IP range

Edit IPv4 range in file `metallb-configmap.yaml` (last line)

III. Deploy image

Edit env variable `SSH_KEY` in `kubernetes/deployment.yaml`
Launch deployment:
```
kubectl apply -f kubernetes
```

Enhancing
---------

Each Debian release corresponds to a Git branch, the branches differ only by
the `FROM` element in the `Dockerfile`.

To create the image `bibiche42/debian-ssh` e.g. for Debian "bullseye":

    git checkout bullseye
    make build

Use `make rebuild` to pull the base image and rebuild without caching.


Testing
-------

Execute `make test` to create a container and fetch all environment variables
via SSH.  This requires an `.ssh/id_rsa.pub` file in your home, it will be
passed to the container via the environment variable `SSH_KEY` and installed.
The `Makefile` is configured to run the container with the limited `docker`
account, this user is allowed to run `sudo` without requiring a password.
The SSH daemon will be always run with root access.  The `debug-*` targets
can help troubleshooting any issues you might encounter.
