# Load-balance your container

This is where kubernetes really shines. Kubernetes is build up around pods and
pods can be load-balanced by using a `replicationController` in Kubernetes.

Now we create new file called `nginx-rc.yml`

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

and we can now see our two pods that has been created

```
$ kubectl create -f /Users/kevinsimper/kubernetes-workshop/nginx-rc.yml
```

We can now see that there is multiple pods

```
$ kubectl describe rc my-nginx
```

"but there is no ip?"

That is the where we use services.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  ports:
  - port: 8000
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
```
Create a service using the

```
$ kubectl create -f /Users/kevinsimper/kubernetes-workshop/nginx-service.yml

```
Now we can inspect our service.

```
kubectl describe svc nginx-service
```

Looks especially at `Endpoints:		172.17.0.4:80,172.17.0.5:80`

Now we can try to scale our `replicationController` by doing This

```
$ kubectl scale --replicas 4 rc my-nginx
replicationcontroller "my-nginx" scaled
```

And we can now see we have changed it

```
$ kubectl get rc
CONTROLLER   CONTAINER(S)   IMAGE(S)   SELECTOR    REPLICAS   AGE
my-nginx     nginx          nginx      app=nginx   4          21m
```

and we now have 4 pods

```
$ kubectl get pods
NAME                   READY     STATUS    RESTARTS   AGE
my-nginx-582ut         1/1       Running   0          22m
my-nginx-eptuh         1/1       Running   0          22m
my-nginx-j5jmb         1/1       Running   0          1m
my-nginx-lq17v         1/1       Running   0          1m
...
```

EXAMPLE
CURL pod

EXPLAIN
why is it not possible to access outside
you use ingress

rolling deploy
