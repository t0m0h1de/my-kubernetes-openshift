# Kubernetes OpenShift 備忘録

## MinikubeによるKubernetesハンズオン

### 前提条件

* Minikubeがインストールされている
* kubectlがインストールされている

### 初期化・起動

以下のコマンドでminikubeクラスターを作成・初期化・起動することができる。

```
minikube start
```

以下のコマンドでクラスター情報を得ることができる。

```
kubectl cluster-info
```

```
minikube stop
```

### おまけ

```
kubectl get componentstatuses
```

```
NAME                 STATUS    MESSAGE   ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   ok
```

```
kubectl get nodes
```

```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   10d   v1.31.0
```

```
kubectl describe nodes minikube
```

`kubectl describe nodes ノード名`

```
Name:               minikube
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=minikube
                    kubernetes.io/os=linux
                    minikube.k8s.io/commit=210b148df93a80eb872ecbeb7e35281b3c582c61
                    minikube.k8s.io/name=minikube
                    minikube.k8s.io/primary=true
                    minikube.k8s.io/updated_at=2024_09_24T11_44_01_0700
                    minikube.k8s.io/version=v1.34.0
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/cri-dockerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 24 Sep 2024 11:43:58 +0900
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  minikube
  AcquireTime:     <unset>
  RenewTime:       Fri, 04 Oct 2024 18:58:46 +0900
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  ... (省略)
Addresses:
  InternalIP:  192.168.39.143
  Hostname:    minikube
Capacity:
  cpu:                2
  ephemeral-storage:  17734596Ki
  hugepages-2Mi:      0
  memory:             3712076Ki
  pods:               110
Allocatable:
  cpu:                2
  ephemeral-storage:  17734596Ki
  hugepages-2Mi:      0
  memory:             3712076Ki
  pods:               110
System Info:
  Machine ID:                 5dc19b27e7384105a0d0b3345a9f2d82
  System UUID:                5dc19b27-e738-4105-a0d0-b3345a9f2d82
  Boot ID:                    66ba4766-1c3e-4047-a0e3-7515e7ec5e1a
  Kernel Version:             5.10.207
  OS Image:                   Buildroot 2023.02.9
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://27.2.0
  Kubelet Version:            v1.31.0
  Kube-Proxy Version:
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
Non-terminated Pods:          (14 in total)
  Namespace                   Name                                         CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                         ------------  ----------  ---------------  -------------  ---
  ... (省略)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                750m (37%)  0 (0%)
  memory             170Mi (4%)  170Mi (4%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:              <none>
```

kube proxyを確認してみるコマンド。

Kubernetes ProxyはロードバランスされたServiceにネットワークトラフィックをルーティングする役割を持つ。

```
kubectl get daemonSet kube-proxy -n kube-system
```

```
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   10d
```

ちなみに、kube-systemの名前空間を覗くとkube-proxy以外にも色々見える。

```
kubectl get all -n kube-system
```

```
NAME                                   READY   STATUS    RESTARTS      AGE
pod/coredns-6f6b679f8f-q7tws           1/1     Running   2 (68m ago)   10d
pod/etcd-minikube                      1/1     Running   2 (68m ago)   10d
pod/kube-apiserver-minikube            1/1     Running   2 (68m ago)   10d
pod/kube-controller-manager-minikube   1/1     Running   2 (68m ago)   10d
pod/kube-proxy-jh258                   1/1     Running   2 (68m ago)   10d
pod/kube-scheduler-minikube            1/1     Running   2 (68m ago)   10d
pod/storage-provisioner                1/1     Running   5 (67m ago)   10d

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   10d

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   10d

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           10d

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-6f6b679f8f   1         1         1       10d
```

Kubernetes DNSを確認するコマンド。

Kubernetes DNSはServiceの名前解決を行う。

```
kubectl get service kube-dns -n kube-system
```

```
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   10d
```
