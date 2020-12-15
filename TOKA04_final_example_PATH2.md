
**LAB1:**

Create Namespace
```sh
kubectl create ns web
```

Create Deploymnet

```sh
kubectl create deployment --image=kienbt/podofminerva:latest --dry-run -o yaml > lab1.yaml
vim lab1.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: web
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 70%
      maxUnavailable: 25%
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: web
        image: kienbt/podofminerva:latest
        ports:
        - containerPort: 8081
        livenessProbe:
          httpGet:
            path: /
            port: 8081
          initialDelaySeconds: 5
          periodSeconds: 5
```
```
Create service NodePort
```sh
kubectl expose deployment test-cm-mount --port=80 --target-port=8081 --type=NodePort --dry-run -o yaml > service.yaml
vim service.yaml

apiVersion: v1
kind: Service
metadata:
  labels:
    app: web-service
  name: web-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8081
    nodePort: 30080
  selector:
    app: webapp
  type: NodePort

kubectl apply -f service.yaml
````
```

**LAB2**

Crea PV

```sh 
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lab2-volume
spec:
  storageClassName: local-storage
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: /etc/data
EOF 
```

Create PVC 
```sh
cat <<EOF | kubectl apply -f -
> apiVersion: v1
> kind: PersistentVolumeClaim
> metadata:
>   name: data-pvc
>   namespace: web
> spec:
>   accessModes:
>     - ReadWriteOnce
>   resources:
>     requests:
>       storage: 256Mi
>   storageClassName: local-storage
> EOF
```sh
```

Create Pod 
```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: data-pod
  name: data-pod
spec:
  containers:
  - image: busybox:1.28
    name: data-pod
    resources: {}
    volumeMounts:
      - name: temp-data
        mountPath: /tmp/data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
    - name: temp-data
      persistentVolumeClaim:
        claimName: data-pvc
EOF
```
``

**LAB3**

Create deployment name webfront-deploy
```sh 
kubectl create deployment webfront-deploy --image=nginx:1.7.8 --port=80

expose expose port 80, target port 80. The service should be exposed externally by listening on port 30080 on each node.
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  labels:
    app: web-service
  name: web-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30080
  selector:
    app: webfront-deploy
  type: NodePort
EOF
```
```

Create Pod db-redis 
```sh
kubectl run db-redis --image=redis:latest
```

Apply the label role=frontend to the web application pods 

```sh
kubectl label deployment webfront-deploy role=frontend
```

Apply the label role=db to the database pod 

```sh
kubectl label pod db-redis role=db
```

Create a network policy that will apply an ingress rule for the pods labeled with role=db to allow
traffic on port 6379 from the pods labeled role=frontend
```sh
cat <<EOF | kubectl app -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-rule
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
EOF
```
