003 CKAD Pods: Modifying the Configuration of an Existing Pod

Ref
* Set default namespace with kubectl https://medium.com/@icheko/set-default-namespace-with-kubectl-7e878d3c0ea0
* Define Environment Variables for a Container https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/ 


```
function kubectl-set-ns(){
  kubectl config set-context --current --namespace="$@"
}
```




## Switching to a namespace
Switch to the namespace using the following command:

```
$ kubectl config set-context --current --namespace=business
Context "kubernetes-admin@kubernetes" modified.
Run the following command to query the current namespace:

$ kubectl config view --minify | grep namespace:
    namespace: business
List the Pods in the namespace.

$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          100s
You will find the Pod named nginx.
```



## Modifying an existing Pod
You won't be able to add or modify environment variables for an existing Pod. You'll have to delete and recreate it. Use the existing YAML definition as a starting point:

```
$ kubectl get pod nginx -o yaml > pod.yaml
Edit the file pod.yaml with vi or nano and add the environment variables to its specification. Remove the current object state. You should end up with the YAML definition shown here:

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: business
spec:
  containers:
  - name: nginx
    image: nginx:1.17.10
    ports:
    - containerPort: 80
    env:
    - name: DB_URL
      value: postgresql://mydb:5432
    - name: DB_USERNAME
      value: admin
Delete the existing Pod, and create it from the YAML manifest.

$ kubectl delete pod nginx
pod "nginx" deleted
$ kubectl create -f pod.yaml
pod/nginx created
Verify that the environment variables have been created correctly by printing them.

$ kubectl exec nginx -- env
...
DB_URL=postgresql://mydb:5432
DB_USERNAME=admin
...
```
Among the rendered environment variables, you should find the ones you defined.





##  My efforts

```
controlplane $ launch.sh
Waiting for Kubernetes to start...
Kubernetes started
controlplane $ 
controlplane $ kubectl create namespace business
namespace/business created
controlplane $ kubectl run nginx --image=nginx:1.17.10 --restart=Never --port=80 -n business
pod/nginx created
controlplane $ kubectl get namespaces
NAME              STATUS   AGE
business          Active   99s
default           Active   118s
kube-node-lease   Active   119s
kube-public       Active   119s
kube-system       Active   119s
controlplane $ kubectl config set-context --current --namespace=business
Context "kubernetes-admin@kubernetes" modified.
controlplane $ kubectl get pods

NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          3m23s
controlplane $ 
controlplane $ 
controlplane $ 
controlplane $ kubectl get pod nginx -o yaml > pod.yaml
controlplane $ vi pod.yaml 
controlplane $ diff pod.yaml pod
pod_with_env_variables.yaml  pod.yaml                     
controlplane $ diff pod.yaml pod
pod_with_env_variables.yaml  pod.yaml                     
controlplane $ diff pod.yaml pod_with_env_variables.yaml 
111a112,116
>     env:
>     - name: DB_URL
>       value: postgresql://mydb:5432
>     - name: DB_USERNAME
>       value: admin
```

but something not quite right
```
controlplane $ kubectl apply -f pod_with_env_variables.yaml
Warning: resource pods/nginx is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
The Pod "nginx" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
  core.PodSpec{
        Volumes:        {{Name: "default-token-srbt9", VolumeSource: {Secret: &{SecretName: "default-token-srbt9", DefaultMode: &420}}}},
        InitContainers: nil,
        Containers: []core.Container{
                {
                        ... // 5 identical fields
                        Ports:   {{ContainerPort: 80, Protocol: "TCP"}},
                        EnvFrom: nil,
-                       Env: []core.EnvVar{
-                               {Name: "DB_URL", Value: "postgresql://mydb:5432"},
-                               {Name: "DB_USERNAME", Value: "admin"},
-                       },
+                       Env:          nil,
                        Resources:    {},
                        VolumeMounts: {{Name: "default-token-srbt9", ReadOnly: true, MountPath: "/var/run/secrets/kubernetes.io/serviceaccount"}},
                        ... // 12 identical fields
                },
        },
        EphemeralContainers: nil,
        RestartPolicy:       "Never",
        ... // 25 identical fields
  }

```
As well as correcting my mistake ... I needed to delete the problem pod
```
pod_with_env_variables.yaml  pod.yaml                     
controlplane $ vi pod_with_env_variables.yaml 
controlplane $ kubectl apply -f pod_with_env_variables.yaml
Warning: resource pods/nginx is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
The Pod "nginx" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
  core.PodSpec{
        Volumes:        {{Name: "default-token-srbt9", VolumeSource: {Secret: &{SecretName: "default-token-srbt9", DefaultMode: &420}}}},
        InitContainers: nil,
        Containers: []core.Container{
                {
                        ... // 5 identical fields
                        Ports:   {{ContainerPort: 80, Protocol: "TCP"}},
                        EnvFrom: nil,
-                       Env: []core.EnvVar{
-                               {Name: "DB_URL", Value: "postgresql://mydb:5432"},
-                               {Name: "DB_USERNAME", Value: "admin"},
-                       },
+                       Env:          nil,
                        Resources:    {},
                        VolumeMounts: {{Name: "default-token-srbt9", ReadOnly: true, MountPath: "/var/run/secrets/kubernetes.io/serviceaccount"}},
                        ... // 12 identical fields
                },
        },
        EphemeralContainers: nil,
        RestartPolicy:       "Never",
        ... // 25 identical fields
  }

controlplane $ kubectl delete pod nginx
pod "nginx" deleted
controlplane $ kubectl apply -f pod_with_env_variables.yaml
pod/nginx created
controlplane $ kubectl exec nginx -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx
DB_URL=postgresql://mydb:5432
DB_USERNAME=admin
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
NGINX_VERSION=1.17.10
NJS_VERSION=0.3.9
PKG_RELEASE=1~buster
HOME=/root
```
