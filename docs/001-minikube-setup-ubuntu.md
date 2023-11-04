## Download and starting minikube



No need to setup docket
```
[~] # which docker
/usr/bin/docker
[~] # docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[~] # 
```

Download minikube
```
[~] # curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 82.4M  100 82.4M    0     0  17.7M      0  0:00:04  0:00:04 --:--:-- 20.2M
[sudo] password for dpitts: 
[~] # sudo install minikube-linux-amd64 /usr/local/bin/minikube
[~] # minikube start
😄  minikube v1.31.2 on Ubuntu 22.04
✨  Automatically selected the docker driver. Other choices: ssh, none
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.27.4 preload ...
    > preloaded-images-k8s-v18-v1...:  393.21 MiB / 393.21 MiB  100.00% 13.70 M
    > gcr.io/k8s-minikube/kicbase...:  447.62 MiB / 447.62 MiB  100.00% 11.08 M
🔥  Creating docker container (CPUs=2, Memory=3900MB) ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...

🚫  Exiting due to HOST_KUBECONFIG_PERMISSION: Failed kubeconfig update: Error reading file "/home/dpitts/.kube/config": open /home/dpitts/.kube/config: permission denied
💡  Suggestion: Run: 'sudo chown $USER $HOME/.kube/config && chmod 600 $HOME/.kube/config'
🍿  Related issue: https://github.com/kubernetes/minikube/issues/5714
```

Unfortunately
```
[~] # sudo chown $USER $HOME/.kube/config && chmod 600 $HOME/.kube/config
chown: cannot access '/home/dpitts/.kube/config': No such file or directory
```

Fortunately as per  https://github.com/kubernetes/minikube/issues/5714 
```
[~] # sudo chown -R $USER ~/.kube
[~]
```
and now we can start minikube
```
[~] # minikube start
😄  minikube v1.31.2 on Ubuntu 22.04
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🏃  Updating the running docker "minikube" container ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner

❗  /usr/local/bin/kubectl is version 1.21.5+k3s2, which may have incompatibilities with Kubernetes 1.27.4.
    ▪ Want kubectl v1.27.4? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
[~] # 
```

