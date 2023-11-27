## Creating a Secret

Create a Secret named ext-service-secret in the namespace secret-ops.

Then, provide the key-value pair API_KEY=LmLHbYhsgWZwNifiqaRorH8T as literal.

```
controlplane $ launch.sh
Waiting for Kubernetes to start...
Kubernetes started
controlplane $ 
controlplane $ kubectl create namespace secret-ops
namespace/secret-ops created
controlplane $ 
controlplane $ 
controlplane $ kubectl create secret generic ext-service-secret --from-literal=API_KEY=LmLHbYhsgWZwNifiqaRorH8T -n secret-ops 
secret/ext-service-secret created
```

## Consuming the Secret
Create a Pod named consumer with the image nginx in the namespace secret-ops and consume the Secret as an environment variable.

```
controlplane $ kubectl run consumer --image=nginx --restart=Never -n secret-ops --dry-run=client -o yaml > pod.yaml
controlplane $ vi pod.yaml 
controlplane $ cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: consumer
  name: consumer
  namespace: secret-ops
spec:
  containers:
  - image: nginx
    name: consumer
    envFrom:
      - secretRef:
          name: ext-service-secret
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

and

```
controlplane $ kubectl apply -f pod.yaml 
pod/consumer created
controlplane $ kubectl exec consumer -n secret-ops -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=consumer
API_KEY=LmLHbYhsgWZwNifiqaRorH8T
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
NGINX_VERSION=1.25.3
NJS_VERSION=0.8.2
PKG_RELEASE=1~bookworm
HOME=/root
controlplane $ .
```