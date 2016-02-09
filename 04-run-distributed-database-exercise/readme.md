# Run a database in Kubernetes

Lets try to run redis inside Kubernetes. Redis is a simpel Key-Value store.

First we need to create a redis master. Let's start by creating the file `redis-master.yml` and save it in the same directory as we have used before.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: redis
    redis-sentinel: "true"
    role: master
  name: redis-master
spec:
  containers:
    - name: master
      image: kubernetes/redis:v1
      env:
        - name: MASTER
          value: "true"
      ports:
        - containerPort: 6379
      resources:
        limits:
          cpu: "0.1"
      volumeMounts:
        - mountPath: /redis-master-data
          name: data
    - name: sentinel
      image: kubernetes/redis:v1
      env:
        - name: SENTINEL
          value: "true"
      ports:
        - containerPort: 26379
  volumes:
    - name: data
      emptyDir: {}
```
Notice how here we have two containers inside the same Pod definition.

```
$ kubectl create -f /Users/kevinsimper/kubernetes-workshop/redis-master.yml
```

Then we need to create a service. Start with the file `redis-service.yml` and save it into the same directory.

```
apiVersion: v1
kind: Service
metadata:
  labels:
    name: sentinel
    role: service
  name: redis-sentinel
spec:
  ports:
    - port: 26379
      targetPort: 26379
  selector:
    redis-sentinel: "true"
```

```
$ kubectl create -f /Users/kevinsimper/kubernetes-workshop/redis-service.yml
```

Next we create a ReplicationController. Start with the file `redis-controller.yml` and save it to the same directory as we've used before.

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    name: redis
  template:
    metadata:
      labels:
        name: redis
    spec:
      containers:
      - name: redis
        image: kubernetes/redis:v1
        ports:
        - containerPort: 6379
        resources:
          limits:
            cpu: "0.1"
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
      volumes:
        - name: data
          emptyDir: {}
```

```
$ kubectl create -f /Users/kevinsimper/kubernetes-workshop/redis-controller.yml
```

Then we can create our sentinel redis. Start with the file `redis-sentinel-controller.yml` and save it to the same directory as we've used before.

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-sentinel
spec:
  replicas: 1
  selector:
    redis-sentinel: "true"
  template:
    metadata:
      labels:
        name: redis-sentinel
        redis-sentinel: "true"
        role: sentinel
    spec:
      containers:
      - name: sentinel
        image: kubernetes/redis:v1
        env:
          - name: SENTINEL
            value: "true"
        ports:
          - containerPort: 26379
```

```
$ kubectl create -f /Users/kevinsimper/kubernetes-workshop/redis-sentinel-controller.yml
```

Then we can try to scale up our redis cluster

```
$ kubectl scale rc redis --replicas=3
$ kubectl scale rc redis-sentinel --replicas=3
```

Now we want to find out the IP address and the port of our redis service.

```
$ kubectl describe service redis
```

Look for "IP" and "Port".

We can then try to access our redis cluster.

```
$ docker run --rm -it --net=host redis bash
```
then we simply connect to it by first accessing the redis sentinel service.

```
$ redis-cli -h REPLACE_WITH_IP -p REPLACE_WITH_PORT
```

Then we ask it for a master

```
10.0.0.13:26379> SENTINEL get-master-addr-by-name mymaster
1) "172.17.0.2"
2) "6379"
```

First run exit
```
$ exit
```

then we can connect to that machine
```
redis-cli -h REPLACE_WITH_IP -p REPLACE_WITH_PORT
```

and then put some data in

```
172.17.0.2:6379> SET KUBERNETES AWESOME
OK
172.17.0.2:6379> KEYS *
1) "KUBERNETES"
```

Run exit two times
```
$ exit
$ exit
```

Lets try to delete our originally redis-master
```
$ kubectl delete pods redis-master
```

and now a slave will be promoted as a master. You can run the above redis commands to test it!
