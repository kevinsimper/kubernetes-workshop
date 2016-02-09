# Create your first container in Kubernetes

You can also do something similar to docker run in Kubernetes.

```
$ kubectl run nginx --image=nginx --port=80
```
This creates a replicationController, but we will get back to that.

```
$ kubectl get pods
```

```
$ kubectl describe pod nginx
…
IP: 172.17.0.3
```

```
$ curl 172.17.0.3
...
<h1>Welcome to nginx!</h1>
```

That was easy, but usual we need a bit more config for our containers.

The awesome part of docker-machine is that we have mounted our files inside
the machine, so we can use our favorite editor to edit the file.

So open your favorite editor and create a new file called `pod-nginx.yml`
and we then save it in our user folder, that means somewhere in `~`.

An example
```
~/kubernetes-workshop/pod-nginx.yml
```

and in that file we write

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

Now we can create that Pod with

```
$ kubectl create -f /Users/kevinsimper/kubernetes-workshop/pod-nginx.yml
pod "nginx" created
```

We can now see it created

```
$ kubectl get pods
```

and if we want to get more information about that pod

```
$ kubectl describe pod nginx
```

we can now delete that container by doing

```
$ kubectl delete pod nginx
```

ANOTHER EXAMPLE
- arguments
