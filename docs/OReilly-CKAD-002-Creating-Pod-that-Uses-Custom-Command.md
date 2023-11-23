## Creating a Pod that Uses a Custom Command


### Creating a Pod
It's easiest to generate the YAML manifest with a dry-run execution of the run command.
```
$ kubectl run loop --image=busybox -n team-red -o yaml --dry-run=client --restart=Never -- /bin/sh -c 'for i in 1 2 3 4 5 6 7 8 9 10; do echo "Welcome $i times"; done' > pod.yaml
```
Alternatively, you can create the Pod by using the YAML manifest defined in the file pod.yaml.

```
apiVersion: v1
kind: Pod
metadata:
  name: loop
  namespace: team-red
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - for i in 1 2 3 4 5 6 7 8 9 10; do echo "Welcome $i times"; done
    image: busybox
    name: loop
```

Execute the following command to create the Pod:

```
$ kubectl apply -f pod.yaml
pod/loop created
```

The Pod should be created without problems.



Getting the container logs
Retrieve the logs of the Pod.

Verify that the custom command executed in the container produces the following output:

```
Welcome 1 times
Welcome 2 times
Welcome 3 times
Welcome 4 times
Welcome 5 times
Welcome 6 times
Welcome 7 times
Welcome 8 times
Welcome 9 times
Welcome 10 times
```
The status should say "Completed", since the custom command finished after executing 10 times. Therefore, the container could transition into this status.

Solution:

The following command writes the status to the file:
```
$ echo "Completed" > pod-status.txt
```

### lab output

initial config
```
controlplane $ kubectl create namespace team-red
namespace/team-red created
```
setup `pod.yaml` via `--dry-run=client`
```
controlplane $ kubectl run loop --image=busybox -n team-red -o yaml --dry-run=client --restart=Never -- /bin/sh -c 'for i in 1 2 3 4 5 6 7 8 9 10; do echo "Welcome $i times"; done' > pod.yaml
controlplane $ cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: loop
  name: loop
  namespace: team-red
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - for i in 1 2 3 4 5 6 7 8 9 10; do echo "Welcome $i times"; done
    image: busybox
    name: loop
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

create pod and review logs
```
controlplane $ kubectl apply -f pod.yaml 
pod/loop created
controlplane $ kubectl get pods --namespace team-red
NAME   READY   STATUS      RESTARTS   AGE
loop   0/1     Completed   0          22s
controlplane $ kubectl logs loop  --namespace team-red
Welcome 1 times
Welcome 2 times
Welcome 3 times
Welcome 4 times
Welcome 5 times
Welcome 6 times
Welcome 7 times
Welcome 8 times
Welcome 9 times
Welcome 10 times
```
and finally (this part I didn´t get)
```
controlplane $ echo "Completed" > pod-status.txt
controlplane $ 
```