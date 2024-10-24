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

クラスターを停止させたい場合は以下のコマンドで止める。

```
minikube stop
```

### Kubernetesの基本オブジェクト

* Pod
* Service
* Volume
* Namespace

###　アプリのデプロイ

以下でminikube上にアプリを作成する。

`kubectl create deploy`コマンドで新しいアプリを作成してみる。

```
kubectl create deploy my-first-app --image=localhost/rust-web-sample
```

```
deployment.apps/my-first-app created
```

TODO: `--image=localhost/rust-web-sample`をgcrに変更

ここで`kubectl get all`コマンドを実行すると、以下のようにpodやdeployment、replicasetが作成されていることがわかる。

```
NAME                               READY   STATUS         RESTARTS   AGE
pod/my-first-app-544d4dcb8-s7fzg   0/1     ErrImagePull   0          9s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-first-app   0/1     1            0           9s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/my-first-app-544d4dcb8   1         1         0       9s
```

### Podハンズオン

一番簡単にPodを作成する方法として、`kubectl run`コマンドを用いる方法がある。

以下の例では、rust製のサンプルwebアプリ[https://github.com/t0m0h1de/rust-webapp-sample](https://github.com/t0m0h1de/rust-webapp-sample)のPodを作成している。

```
kubectl run rust-webapp --image=ghcr.io/t0m0h1de/rust-webapp-sample/rust-webapp-sample:latest
```

以下のコマンドで作成されたPodを確認する。

```
kubectl get pods
```

Note: `get pods`としているが、`get pod`としても同様な動作ととなる。

出力は以下のようになる。

```
NAME          READY   STATUS    RESTARTS   AGE
rust-webapp   1/1     Running   0          35s
```

Podの動作を確認するために、以下のコマンドでポートフォワードを行い、ローカルから8080ポートでアクセスしてみる。

```
kubectl port-forward pod/rust-webapp 8080:8000
```

この状態でブラウザから[http://localhost:8080](http://localhost:8080)にアクセスすると"Sample Web Page"という文字列が表示されることが確認できる。

Note: `pod/rust-webapp`という表記になっているが、`pod rust-webapp`としても同様の動作となる

このとき、Pod自体のログは以下のコマンドで確認することができる。

```
kubectl logs pod/rust-webapp
```

ログを見るとPod自体は8000ポートでリッスンしていることがわかる。

Note: ログをリアルタイムに確認したいときには`kubectl logs -f pod/rust-webapp`とする

Note: `logs`コマンドでは`pod rust-webapp`とするとエラーとなる。

また、`kubectl describe pod/rust-webapp`コマンドでPodが作成された際の記録等のより詳細な情報が確認できる。

TODO: describeコマンドで確認できる情報の説明を足す。

また、`kubecl get`コマンドで作成されたPodのmanifestが確認できる。

```
kubectl get pod/rust-webapp -o yaml
```

`-o yaml`がyamlファイルとしてmanifestを取得することを指定している。

Note: `-o yaml`は`-oyaml`としても同様の動作となる。

出力は以下のようになる。

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-10-10T02:18:37Z"
  labels:
    run: rust-webapp
  name: rust-webapp
  namespace: default
  resourceVersion: "73080"
  uid: 50c87b4d-0f7d-4d77-854a-7b763c0e3812
spec:
  containers:
  - image: ghcr.io/t0m0h1de/rust-webapp-sample/rust-webapp-sample:latest
    imagePullPolicy: Always
    name: rust-webapp
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-w8fnm
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  ...
```

ここで、以下のコマンドで一度Podを削除する。

```
kubectl delete pod rust-webapp
```

次に、yamlファイルとしてmanifestを作成し、applyすることでPodを作成してみる。

以下のような`rust-webapp-pod.yaml`を作成する。

`rust-webapp-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rust-webapp
spec:
  containers:
    - image: ghcr.io/t0m0h1de/rust-webapp-sample/rust-webapp-sample:latest
      name: rust-webapp
      ports:
        - containerPort: 8000
          name: http
          protocol: TCP
```

作成が完了したら以下のコマンドでmanifestを流す。

```
kubectl apply -f rust-webapp-pod.yaml
```

以下のコマンドで作成されたPodのmanifestを見てみると、kubernetes側で自動的に様々な情報が付与されていることがわかる。

```
kubectl edit pod rust-webapp
```

Note: `kubectl edit`コマンドは`kubectl get -oyaml`でyamlファイルを出力し、編集後、`kubectl apply`する部分までを一気通貫で行うことができる。

ここで、`kubectl exec`コマンドを使用して、Pod内でコマンドを実行してみる。

```
kubectl exec rust-webapp -- pwd
```

出力は以下のようになる。

```
/app
```

更に`-it`オプションをつけることで実行中のPodのシェルに入ることができる。

```
kubectl exec -it rust-webapp -- /bin/bash
```

最後に以下のコマンドでPodを削除する。

```
kubectl delete -f rust-webapp-pod.yaml
```

TODO: `kubectl cp`コマンドについての説明でrust-webappではtarコマンドが無くて失敗しることの説明と、代替コマンドを説明する。

### Podハンズオン2

KubernetesはPodを監視、異常があればPodをkillして新たにPodを再作成する。

このようなPodが正常に動作しているのかどうかのヘルスチェックには、liveness probeまたはReadness probeで監視することができる。

まずは以下のようなyamlファイルのmanifestを作成し、kubenetesにliveness probeの監視を適用したPodを作成する。

`toggle-health-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toggle-health-app
spec:
  containers:
    - image: ghcr.io/t0m0h1de/wildfly-toggle-health-app/toggle-health-app:latest
      name: toggle-health-app
      livenessProbe:
        httpGet:
          path: /health
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
        timeoutSeconds: 1
        failureThreshold: 3
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

上記のmanifestでPodを作成する。

```
kubectl apply -f toggle-health-pod.yaml
```

`kubectl get pod`コマンドで無事Podが作成されていることが確認できたら、ブラウザから[http://localhost:8080/health/](http://localhost:8080/health/)にアクセスし、"healthy"という文字列が表示されることを確認する。

この状態で、[http://localhost:8080/health/confuse](http://localhost:8080/health/confuse)にアクセスするとPodはレスポンスコード400のレスポンスを返し、それ以降いかなるエンドポイント(URL)にアクセスしてもレスポンスコード400を返すようになる。

実際に`kubectl port-forward toggle-health-app 8080:8080`を実行した状態で、上記の[http://localhost:8080/health/confuse](http://localhost:8080/health/confuse)にアクセスし、以下の`kubectl describe`コマンドを実行すると出力にヘルスチェックに失敗し、PodがKillされ、再作成されることがわかる。

```
kubectl describe pod/toggle-health-app
```

出力
```
...
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  ...
  Warning  Unhealthy  60s                  kubelet            Liveness probe failed: Get "http://10.244.0.46:8080/health": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
  Warning  Unhealthy  42s (x2 over 52s)    kubelet            Liveness probe failed: HTTP probe failed with statuscode: 400
  Normal   Killing    42s                  kubelet            Container toggle-health-app failed liveness probe, will be restarted
  Normal   Pulling    41s (x2 over 4m21s)  kubelet            Pulling image "ghcr.io/t0m0h1de/wildfly-toggle-health-app/toggle-health-app:latest"
  Normal   Created    39s (x2 over 4m20s)  kubelet            Created container toggle-health-app
  Normal   Started    39s (x2 over 4m19s)  kubelet            Started container toggle-health-app
  ...
```

この時、`kubectl get pod/toggle-health-app`コマンドを実行すると以下のようにリスタートが一回実行されたことがわかる。

出力
```
NAME                READY   STATUS    RESTARTS        AGE
toggle-health-app   1/1     Running   1 (3m10s ago)   6m51s
```

最後に以下のコマンドで作成したPodを削除する。

```
kubectl delete -f toggle-health-pod.yaml
```

TODO: Readness probeの説明を追加する。

Podのリソース管理をするには`spec`上の`resouces`に`requests`や`limits`を記載する。

`requests`にCPUやメモリについてPodが必要とするリソースを定義し、Podを作成してみる。

以下のように`rust-webapp-required-resources-pod.yaml`を作成する。

`rust-webapp-request-resources-pod.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: rust-webapp-required-resources
spec:
  containers:
    - image: ghcr.io/t0m0h1de/rust-webapp-sample/rust-webapp-sample:latest
      name: rust-webapp-requred-resources
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
      ports:
        - containerPort: 8000
          name: http
          protocol: TCP
```

`kubectl apply -f rust-webapp-request-resources-pod.yaml`でPodを作成すると、作成されるPodは0.5CPUと128MiBのメモリが割り当てられるノード上に作成される。

ここで今度は以下の`rust-webapp-limit-resource-pod.yaml`のように`requests`に加えて、`limits`を記載することで、Podの消費リソースを制限することができる。

```
apiVersion: v1
kind: Pod
metadata:
  name: rust-webapp-limit-resources
spec:
  containers:
    - image: ghcr.io/t0m0h1de/rust-webapp-sample/rust-webapp-sample:latest
      name: rust-webapp-limit-resources
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
      ports:
        - containerPort: 8000
          name: http
          protocol: TCP
```

`requests`がリソースの下限、`limits`がリソースの上限を設定する形となる。

TODO: リソース制限はアディショナルに移動



### Serviceハンズオン

### KubernetesのAdditionalなオブジェクト

* Deployment
* ReplicaSet
* Job
* DaemonSet
* StatefulSet

### Deploymentハンズオン

### ReplicaSetハンズオン

### Contextの切り替え

minikubeで異なるプロファイルを作成したり、minikube以外のKubernetesクラスターに接続するには、Contextを切り替える必要がある。

ここでは例として、以下のようにminikubeプロファイルを作成し、切り替える。

minikubeプロファイルの一覧は以下のコマンドで確認できる。

```
minikube profile list
```

kubectl側のContextの一覧は以下のコマンドで確認できる。

```
kubectl config get-contexts
```

現在のプロファイルの確認は`minikube`コマンドまたは`kubectl`コマンドで行える。

`minikube`コマンドを用いる方法

```
minikube profile
```

`kubectl`コマンドを用いる方法

```
kubectl config current-context
```

次に、以下のコマンドで新規のプロファイルを作成する。

`second-mkube`が新しく作成するプロファイル名

```
minikube start -p second-mkube
```

プロファイルを新規作成すると、自動的に現在のContextが切り替わる。

以下のコマンドでkubectl側での現在のContextが切り替わっていることが確認できる。

```
# 現在のContextが`second-mkube`になっていることを確認
kubectl config current-context

# Context一覧で`second-mkube`が新規作成されていることを確認
kubectl config get-contexts
```

また、同様に以下のコマンドでminikube側のプロファイルも切り替わっていることが確認できる。

```
# 現在のプロファイルが`second-mkube`に変更されていることを確認
minikube profile

# プロファイル一覧で`second-mkube`が新規作成されていることを確認
minikube profile list
```

ここで、以下のコマンドでkubectl側のContextを`minikube`に切り替えてみる。

```
kubectl config use-context minikube
```

以下のコマンドでminikube, kubectlの両方でプロファイルとContextが切り替えられていることが確認できる。

```
# 現在のプロファイルが`minikube`に変更されていることを確認
minikube profile

# プロファイル一覧で`minikube`が新規作成されていることを確認
minikube profile list

# 現在のContextが`minikube`になっていることを確認
kubectl config current-context

# Context一覧で`minikube`が新規作成されていることを確認
kubectl config get-contexts
```

確認できたら、以下のコマンドで`second-mkube`を削除する。

```
minikube stop -p second-mkube
minikube delete -p second-mkube
```

また、クラスター切り替えと同様にnamespaceの切り替えもContextで行うことができる。

例えば、現在のクラスタで`novel-namespace`というnamespaceのContextに切り替えたい場合には、以下のコマンドを実行する。

```
kubectl config set-context $(kubectl config current-context) --namespace=novel-namespace
```

TODO: kubernetes-labへの接続情報を扱うセクションを追加する

### kubernetesのシステムコンポーネント

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

### Minikubeのadditionalな使い方

* kubernetesのバージョンを指定して、新規minikubeプロファイルを作成する

```
minikube start -p 新規プロファイル名 --kubernetes-version=v1.xx.x
```
