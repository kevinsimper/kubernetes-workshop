# Run a database in Kubernetes

Lets try to run redis inside Kubernetes. Redis is a simpel Key-Value store.

First we need to create a redis master. Let's start by creating the file `redis-master.yml`.

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

Then we need to create a service `redis-service.yml`

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

We create a `redis-controller.yml`

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

Then we can create our sentinel redis `redis-sentinel-controller.yml`

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

Then we can try to scale up our redis cluster

```
$ kubectl scale rc redis --replicas=3
$ kubectl scale rc redis-sentinel --replicas=3
```

We can then try to access our redis cluster now.

```
$ docker run --rm -it --net=host redis bash
```
then we simply connect to it by first accessing the redis sentinel service.

```
$ redis-cli -h 10.0.0.13 -p 26379
```

Then we ask it for a master

```
10.0.0.13:26379> SENTINEL get-master-addr-by-name mymaster
1) "172.17.0.2"
2) "6379"
```

then we can connect to that machine
```
redis-cli -h 172.17.0.2 -p 6379
```

and then put some data

```
172.17.0.2:6379> SET KUBERNETES AWESOME
OK
172.17.0.2:6379> KEYS *
1) "KUBERNETES"
```

Lets try to delete our originally redis-master
```
$ kubectl delete pods redis-master
```

and now will a slave be promoted as a master. You can run the above redis commands to test it!
