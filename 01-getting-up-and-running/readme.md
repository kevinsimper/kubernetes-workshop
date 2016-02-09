# Getting Kubernetes up and running

First we create a docker-machine. This way we can ensure that everybody has the same platform.

```
$ docker-machine create --driver virtualbox kubernetes-master
```

Then we ssh into the machine:

```
$ docker-machine ssh kubernetes-master
```

Then we download the kubectl CLI tool that we will use to control Kubernetes.

```
$ curl -O https://storage.googleapis.com/kubernetes-release/release/v1.1.4/bin/linux/amd64/kubectl
$ chmod +x kubectl
$ sudo mv kubectl /usr/local/bin
```

Now we need to start ETCD, which is a key value store.

```
$ docker run \
  --net=host \
  -d gcr.io/google_containers/etcd:2.0.12 \
  /usr/local/bin/etcd \
  --addr=127.0.0.1:4001 \
  --bind-addr=0.0.0.0:4001 \
  --data-dir=/var/etcd/data
```

Now we can start Kubernetes master

```
$ docker run \
    --volume=/:/rootfs:ro \
    --volume=/sys:/sys:ro \
    --volume=/dev:/dev \
    --volume=/var/lib/docker/:/var/lib/docker:ro \
    --volume=/var/lib/kubelet/:/var/lib/kubelet:rw \
    --volume=/var/run:/var/run:rw \
    --net=host \
    --pid=host \
    --privileged=true \
    -d \
    gcr.io/google_containers/hyperkube:v1.1.4 \
    /hyperkube kubelet --containerized \
    --hostname-override="127.0.0.1" --address="0.0.0.0" \
    --api-servers=http://localhost:8080 \
    --config=/etc/kubernetes/manifests
```
Now we start a proxy that makes it possible to run commands against kubernetes from the outside.

```
$ docker run -d \
  --net=host \
  --privileged \
  gcr.io/google_containers/hyperkube:v1.1.4 \
  /hyperkube proxy --master=http://127.0.0.1:8080 --v=2
```

Now we can query kubernetes and see if it works. It can take some time before the services has started so if you get an error message, just try again after a couple of minutes.

```
$ kubectl cluster-info
Kubernetes master is running at http://localhost:8080
```
