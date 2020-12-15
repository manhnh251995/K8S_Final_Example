PART 1
Question 1: Create a pod with image nginx called nginx and allow traffic on port 80 ?
Answer: 
```sh
kubectl run nginx --image=nginx --restart=Never --port=80 
```

Question 2: Change pod's image to nginx:1.7.1. Observe that the pod will be killed and recreated as
soon as the image gets pulled ?
Answer :
```sh
kubectl set image pod nginx nginx=nginx1.7.1 --record
kubectl describe pod/nginx
kubectl get pod nginx -w
```

Question 3: Get information about the pod, including details about potential issues (e.g. pod hasn't
started) ?
Answer :
```sh
kubectl get pod nginx -o wide
kubectl describe pod nginx 
```

Question 4: Get pod logs ?
Answer  : 
```sh
kubectl logs nginx
```
Question 5: If pod crashed and restarted, get logs about the previous instance ?
Answer  : 
```sh
kubectl get events --all-namespaces  | grep -i nginx
```

Question 6: Execute a simple shell on the nginx pod ?
Answer  : 
```sh
kubectl exec -it pod/nginx -- sh 
```

Question 7: Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence
within the pod?
Answer  : 
```sh
- create pod export to yaml file : kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > env.yaml
- add environment to yaml file : vim env.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    env:
      - name: var1
        value: val1
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
    
- apply file yaml: kubectl apply -f env.yaml
- result:

kubectl exec -it pod/nginx -- env | grep var1
var1=val1
```

Question 8: Create a Pod with two containers, both with image busybox and command "echo hello;
sleep 3600". Connect to the second container and run 'ls' ?\
    Answer:

vim two-container.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: two-container
  name: two-container
spec:
  containers:
  - image: busybox
    name: container1
    args: ["-c", "echo hello > /etc/hello.log;sleep 3600"]
    volumeMounts:
      - name: shared-data
        mountPath: /etc/
  - image: busybox
    name: container2
    volumeMounts:
      - name: shared-data
        mountPath: /etc/
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
    - name: shared-data
      emptyDir: {}
status: {}

```
```
Question 9: Create a deployment with image nginx:1.7.8, called nginx, having 2 replicas, defining port
80 as the port that this container exposes (don't create a service for this deployment) ?
    Answer  :  
```sh
kubectl create deployment nginx --image=nginx:1.7.8 --port=80 && sleep 10 && kubectl scale deploy nginx --replicas=2
```


Question 10: Update the nginx image to nginx:1.7.9 ?
    Answer  : 
```sh
kubectl set image deployment nginx nginx=nginx:1.7.9 --record
```

Question 11: Check the rollout history and confirm that the replicas are OK?
    Answer  :
- Check rollout history
```sh
kubectl rollout history deployment nginx 
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx nginx=nginx:1.7.9 --record=true
```
- confirm that replicas are OK 
```sh
kubectl get replicasets | grep nginx
nginx-54f57cf6bf             2         2         2       3m40s
nginx-64f4c47bf8             0         0         0       7m52s
```

Question 12: Undo the latest rollout and verify that new pods have the old image (nginx:1.7.8) ?
    Answer :
```sh
kubectl rollout undo deployment nginx --to-revision 1
```

Question 13: Do an on purpose update of the deployment with a wrong image nginx:1.91 ?
    Answer : kubectl set image deployment nginx nginx=nginx:1.91 --record
```sh
kubectl describe rs nginx-84fdd4f88c | grep "Pods Status"
Pods Status:    0 Running / 1 Waiting / 0 Succeeded / 0 Failed

kubectl get events | grep pod/nginx-84fdd4f88c-lgg5j
12m         Normal    Scheduled                pod/nginx-84fdd4f88c-lgg5j             Successfully assigned default/nginx-84fdd4f88c-lgg5j to k8s-woker1
11m         Normal    Pulling                  pod/nginx-84fdd4f88c-lgg5j             Pulling image "nginx:1.91"
11m         Warning   Failed                   pod/nginx-84fdd4f88c-lgg5j             Failed to pull image "nginx:1.91": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:1.91 not found: manifest unknown: manifest unknown
11m         Warning   Failed                   pod/nginx-84fdd4f88c-lgg5j             Error: ErrImagePull
12m         Normal    SandboxChanged           pod/nginx-84fdd4f88c-lgg5j             Pod sandbox changed, it will be killed and re-created.
7m16s       Normal    BackOff                  pod/nginx-84fdd4f88c-lgg5j             Back-off pulling image "nginx:1.91"
2m20s       Warning   Failed                   pod/nginx-84fdd4f88c-lgg5j             Error: ImagePullBackOff
```

Question 14: Verify that something's wrong with the rollout ?
    Answer
```sh
kubectl describe rs nginx-84fdd4f88c | grep "Pods Status"
Pods Status:    0 Running / 1 Waiting / 0 Succeeded / 0 Failed

kubectl get events | grep pod/nginx-84fdd4f88c-lgg5j
12m         Normal    Scheduled                pod/nginx-84fdd4f88c-lgg5j             Successfully assigned default/nginx-84fdd4f88c-lgg5j to k8s-woker1
11m         Normal    Pulling                  pod/nginx-84fdd4f88c-lgg5j             Pulling image "nginx:1.91"
11m         Warning   Failed                   pod/nginx-84fdd4f88c-lgg5j             Failed to pull image "nginx:1.91": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:1.91 not found: manifest unknown: manifest unknown
11m         Warning   Failed                   pod/nginx-84fdd4f88c-lgg5j             Error: ErrImagePull
12m         Normal    SandboxChanged           pod/nginx-84fdd4f88c-lgg5j             Pod sandbox changed, it will be killed and re-created.
7m16s       Normal    BackOff                  pod/nginx-84fdd4f88c-lgg5j             Back-off pulling image "nginx:1.91"
2m20s       Warning   Failed                   pod/nginx-84fdd4f88c-lgg5j             Error: ImagePullBackOff
```

Question 15: Return the deployment to the second revision (number 2) and verify the image is
nginx:1.7.9 ?
    Answer
```sh
kubectl rollout undo deployment nginx --to-revision 2
```

Question 16: Check the details of the fourth revision (number 4) ?
    Answer
```sh
kubectl rollout history deployment nginx --revision=4
deployment.apps/nginx with revision #4
Pod Template:
  Labels:       app=nginx
        pod-template-hash=84fdd4f88c
  Annotations:  kubernetes.io/change-cause: kubectl set image deployment nginx nginx=nginx:1.91 --record=true
  Containers:
   nginx:
    Image:      nginx:1.91
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```

Question 17: Scale the deployment to 5 replicas?
    Answer:
```sh
kubectl scale deployment nginx --replicas=5
```
Question 18: Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80% ?
    Answer:
```sh
kubectl autoscale deployment nginx --min=5 --max=10 --cpu-percent=80
```

Question 19: Pause the rollout of the deployment ?
    Answer:
```sh
kubectl rollout pause deployment nginx
```

Question 20: Update the image to nginx:1.9.1 and check that there's nothing going on, since we
paused the rollout ?
    Answer:
```sh
kubectl set image deployment nginx nginx=nginx:1.9.1 --record
kubectl rollout pause deployment nginx 
kubectl rollout status deployment nginx
    Waiting for deployment "nginx" rollout to finish: 1 out of 3 new replicas have been updated...
```

Question 21: Resume the rollout and check that the nginx:1.9.1 image has been applied ?
    Answer:
```sh
kubectl rollout resume deployment nginx 
kubectl rollout status deployment nginx
Waiting for deployment "nginx" rollout to finish: 3 out of 5 new replicas have been updated...
```

Question 22: Delete the deployment and the horizontal pod autoscaler you created?
    Answer:
```sh
kubectl delete deployment nginx
kubectl delete hpa nginx

```

Question 23: Create a job with the image busybox that executes the command 'echo hello;sleep
30;echo world' ?
    Answer:
```sh
kubectl create job job --image=busybox  -- /bin/bash -c "echo hello;sleep 30;echo world"
```

Question 24: Create a job but ensure that it will be automatically terminated by kubernetes if it takes
more than 30 seconds to execute ?
    Answer:
```sh
kubectl create job 2job --image=busybox  --dry-run -o yaml > job.yaml
vim job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: 2job
spec:
  activeDeadlineSeconds: 30
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: busybox
        name: 2job
        resources: {}
        args: ["echo StongNH is Best"]
      restartPolicy: OnFailure
status: {}

kubectl apply -f job.yaml
```

Question 25: Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes
'date; echo Hello from the Kubernetes cluster' to standard output ?
    Answer:
```sh
kubectl create cronjob cronjob --image=busybox --schedule="*/1 * * * *" -- /bin/bash -c "date; echo Hello from the Kubernetes cluster "
```

Question 26: Create a configmap named config with values foo=lala,foo2=lolo?
    Answer:
```sh
kubectl create cm config --from-literal=foo=lala --from-literal=foo2=lolo
```

Question 27: Create and display a configmap from a file ?
Create the file with
echo -e "foo3=lili\nfoo4=lele" > config.txt
    Answer:
```sh
echo -e "foo3=lili\nfoo4=lele" > config.txt | kubectl create cm config2 --from-file=config.txt
kubectl describe cm config2
Name:         config2
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
config.txt:
----
foo3="lili"
foo4="lele"

Events:  <none>
```

Question 28: Create and display a configmap from a .env file ?
Create the file with the command
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
    Answer:
```sh
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env | kubectl create cm config3 --from-file=config.env | kubectl describe cm config3
Name:         config3
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
config.env:
----
var1=val1
# this is a comment

var2=val2
#anothercomment

Events:  <none>
```

Question 29: Create a configMap called 'options' with the value var5=val5. Create a new nginx pod
that loads the value from variable 'var5' in an env variable called 'option' ?
    Answer
```sh
kubectl create cm options --from-literal=var5=val5 
kubectl run nginx-cm --image=nginx --restart=Never --dry-run -o yaml > nginx-cm.yaml | vim ngin-cm.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-cm
  name: nginx-cm
spec:
  containers:
  - image: nginx
    name: nginx-cm
    resources: {}
    env:
      - name: option
        valueFrom:
          configMapKeyRef:
           name: options
           key: var5
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl exec -it pod/nginx-cm -- env | grep option
option=val5

```
Question 30: Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this
configMap as env variables into a new nginx pod ?
    Answer:
```sh
kubectl create cm anotherone --from-literal=var6=val6 --from-literal=var7=val7 | kubectl run nginx-cm2 --image=nginx --restart=Never --dry-run -o yaml > nginx-cm2.yaml | vim nginx-cm2.yaml

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-cm2
  name: nginx-cm2
spec:
  containers:
  - image: nginx
    name: nginx-cm2
    resources: {}
    envFrom:
      - configMapRef:
          name: anotherone
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl exec -it nginx-cm2 -- env | grep -E 'var6|var7'
var6=val6
var7=val7
```

Question 31: Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a
volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory. ?
    Answer:
```sh
kubectl create cm cmvolume --from-literal=var8=val8  --from-literal=var9=val9
kubectl run nginx-cm3 --image=nginx --restart=Never --dry-run -o yaml > nginx-cm3.yaml | vim nginx-cm3.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-cm3
  name: nginx-cm3
spec:
  containers:
  - image: nginx
    name: nginx-cm3
    resources: {}
    volumeMounts:
    - name: volume-cm
      mountPath: /etc/lala
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
    - name: volume-cm
      configMap:
        name: cmvolume
status: {}

kubectl exec -it nginx-cm3 -- /bin/bash -c "cd /etc/lala/ && ls"
var8
var9 
```

Question 32: Create an nginx pod with requests cpu=100m,memory=256Mi and limits
cpu=200m,memory=512Mi?
    Answer:
```sh
kubectl run nginx-cpu-mem --image=nginx --restart=Never --requests=cpu=100m,memory=256Mi --limits=cpu=200m,memory=512Mi
```

Question 33: Create a secret called mysecret with the values password=mypass ?
    Answer:
```sh
kubectl create secret generic mysecret --from-literal=password=mypass
```

Question 34: Create a secret called mysecret2 that gets key/value from a file?
Create a file called username with the value admin:
echo -n admin > username
    Answer:
```sh
echo -n admin > username && kubectl create secret generic mysecret2 --from-file=username
```

Question 35: Get the value of mysecret2 ?
    Answer:
```sh
kubectl describe secret mysecret2

Name:         mysecret2
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
username:  5 bytes
```

Question 36: Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo?
    Answer:
```sh
kubectl run nginx-secret --image=nginx --restart=Never --dry-run -o yaml > nginx-secret.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-secret
  name: nginx-secret
spec:
  containers:
  - image: nginx
    name: nginx-secret
    resources: {}
    volumeMounts:
      - name: volume-secret
        mountPath: /etc/foo
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
    - name: volume-secret
      secret:
        secretName: mysecret2
status: {}

kubectl exec -it nginx-secret -- /bin/bash -c "cd /etc/foo && cat username"
adminroot
```

Question 37: Delete the pod you just created and mount the variable 'username' from secret mysecret2
onto a new nginx pod in env variable called 'USERNAME' ?
    Answer:
```sh
kubectl delete pod nginx-secret
kubectl run nginx-secret --image=nginx --restart=Never --dry-run -o yaml > nginx-secret.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-secret
  name: nginx-secret
spec:
  containers:
  - image: nginx
    name: nginx-secret
    resources: {}
    env:
      - name: USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret2
            key: username
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl exec -it nginx-secret -- env | grep USERNAME
USERNAME=admin

```

Question 38: Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its
YAML in pod.yaml. Run it, check its probe status, delete it. ?
    Answer:
```sh
kubectl run nginx-liveness --image=nginx --dry-run -o yaml > pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-liveness
  name: nginx-liveness
spec:
  containers:
  - image: nginx
    name: nginx-liveness
    resources: {}
    livenessProbe:
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

kubectl describe pod/nginx-liveness | grep Liveness
Liveness:       exec [ls] delay=0s timeout=1s period=10s #success=1 #failure=3
```

Question 39: Create a PersistentVolume of 10Gi, called 'myvolume'. Make it have accessMode of
'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'.
Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster?
    Answer:
```sh
vim pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: myvolume
spec:
  storageClassName: normal
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: /etc/foo

kubectl apply -f pv.yaml

kubectl get pv -o wide | grep myvolume
myvolume   10Gi       RWO            Retain           Available                         normal                      31s    Filesystem
```

Question 40: Create a PersistentVolumeClaim for this storage class, called mypvc, a request of 4Gi
and an accessMode of ReadWriteOnce, with the storageClassName of normal, and save it on
pvc.yaml. Create it on the cluster. Show the PersistentVolumeClaims of the cluster. Show the
PersistentVolumes of the cluster?
    Answer
```sh
vim pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
  storageClassName: normal

kubectl apply -f pvc.yaml
kubectl get pvc | grep mypvc
mypvc     Bound    myvolume   10Gi       RWO            normal             15s
```

Question 41: Create a busybox pod with command 'sleep 3600', save it on pod.yaml. Mount the
PersistentVolumeClaim to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to
'/etc/foo/passwd' ?
    Answer:
```sh
vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-pvc
  name: pod-pvc
spec:
  containers:
  - image: busybox
    name: pod-pvc
    resources: {}
    args: ["/bin/bash -c 'sleep 3600'"]
    volumeMounts:
      - name: volume-pvc
        mountPath: /etc/passwd
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
    - name: volume-pvc
      persistentVolumeClaim:
        claimName: mypvc
status: {}
kubectl apply -f pod.yaml 

```

Question 42: Create a second pod which is identical with the one you just created (you can easily do it
by changing the 'name' property on pod.yaml). Connect to it and verify that '/etc/foo' contains the
'passwd' file. Delete pods to cleanup ?
    Answer:
```sh
vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-pvc2
  name: pod-pvc2
spec:
  containers:
  - image: busybox
    name: pod-pvc2
    resources: {}
    args: ["/bin/bash -c 'sleep 3600'"]
    volumeMounts:
      - name: volume-pvc
        mountPath: /etc/passwd
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
    - name: volume-pvc
      persistentVolumeClaim:
        claimName: mypvc
status: {}

kubectl apply -f pod.yaml

```
Question 43: Create a pod with image nginx called nginx and expose its port 80?
    Answer:
```sh
kubectl run nginx-expose --image=nginx --restart=Never --port=80
kubectl expose pod nginx-expose --target-port=80 --type=ClutserIP
```

Question 44: Confirm that ClusterIP has been created. Also check endpoints?
    Answer:
```sh
kubectl get service | grep nginx-expose
nginx-expose      ClusterIP   10.102.224.56    <none>        80/TCP    100s

```

Question 45: Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget ?
    Answer:
```sh
kubectl run temppod --image=busybox -it -- sh
If you don't see a command prompt, try pressing enter.
/ # wget 10.102.224.56
Connecting to 10.102.224.56 (10.102.224.56:80)
saving to 'index.html'
index.html           100% |********************************************************************************************************************************************************************************|   612  0:00:00 ETA
'index.html' saved
```
