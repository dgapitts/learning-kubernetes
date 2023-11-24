## OReilly-CKAD-001 Pods: Creating and Interacting with a Pod


Ref:
* https://kubernetes.io/docs/reference/kubectl/cheatsheet/
* https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/
* https://jamesdefabia.github.io/docs/user-guide/simple-nginx/


## Highlights

Use `loop` to keep checking the pod status
```
kubectl get pod loop --namespace=ckad
```

To download the logs, use a simple logs command:
```
$ kubectl logs nginx --namespace=ckad
```



## Creating a namespace

The Pod you are about to create should live in a custom namespace.
Start by creating a new namespace with the name ckad.
Verify that the namespace has been created properly by querying for it.

```
controlplane $ launch.sh
Waiting for Kubernetes to start...
Kubernetes started
controlplane $ kubectl get namespaces --show-labels
NAME              STATUS   AGE   LABELS
default           Active   61s   <none>
kube-node-lease   Active   62s   <none>
kube-public       Active   62s   <none>
kube-system       Active   62s   <none>
controlplane $ vi namespace-ckad.yaml
controlplane $ cat namespace-ckad.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ckad
  labels:
    name: ckad
controlplane $ kubectl apply -f namespace-ckad.yaml 
namespace/ckad created
controlplane $ kubectl get namespaces --show-labels
NAME              STATUS   AGE     LABELS
ckad              Active   11s     name=ckad
default           Active   2m9s    <none>
kube-node-lease   Active   2m10s   <none>
kube-public       Active   2m10s   <none>
kube-system       Active   2m10s   <none>
```


##  Creating a Pod
Create a new Pod named nginx running the image nginx:1.17.10.
Expose container port 80.
The Pod should live in the namespace ckad.
Wait until the Pod transitions into the "Running" status.


```
controlplane $ kubectl run nginx --image=nginx:1.17.10 --port=80 --namespace ckad --dry-run=client -o yaml > nginx_pod.yaml
controlplane $ cat nginx_pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: ckad
spec:
  containers:
  - image: nginx:1.17.10
    name: nginx
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
controlplane $ kubectl apply -f nginx_pod.yaml 
pod/nginx created
controlplane $  kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
ckad          nginx                                  1/1     Running   0          49s
kube-system   coredns-74ff55c5b-crp8d                1/1     Running   0          3m8s
kube-system   coredns-74ff55c5b-w7kvb                1/1     Running   0          3m8s
kube-system   etcd-controlplane                      1/1     Running   0          3m15s
kube-system   kube-apiserver-controlplane            1/1     Running   0          3m15s
kube-system   kube-controller-manager-controlplane   1/1     Running   0          3m15s
kube-system   kube-flannel-ds-amd64-754b9            1/1     Running   0          3m7s
kube-system   kube-flannel-ds-amd64-zv6gg            1/1     Running   1          2m58s
kube-system   kube-proxy-gcqm2                       1/1     Running   0          3m8s
kube-system   kube-proxy-jn5nw                       1/1     Running   0          2m58s
kube-system   kube-scheduler-controlplane            1/1     Running   0          3m15s
controlplane $ kubectl get pods  --namespace ckad
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          96s
```

##  Interacting with the Pod
Get the details of the Pod, including its IP address.
Create a temporary Pod that uses the busybox image to execute a wget command inside the container.
The wget command should access the endpoint exposed by the nginx container.
You should see the HTML response body rendered in the terminal.

```
controlplane $ kubectl get pod nginx -n ckad -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          4m45s   10.244.1.2   node01   <none>           <none>
controlplane $ kubectl run busybox --image=busybox --rm -it --restart=Never -n ckad -- wget 10.244.1.2:80
Connecting to 10.244.1.2:80 (10.244.1.2:80)
saving to 'index.html'
index.html           100% |********************************|   612  0:00:00 ETA
'index.html' saved
pod "busybox" deleted
```


## Delete the namespace to delete all objects in it.

```
controlplane $ kubectl delete namespace ckad
namespace "ckad" deleted
controlplane $ kubectl get  namespace ckad
Error from server (NotFound): namespaces "ckad" not found
controlplane $ 
```