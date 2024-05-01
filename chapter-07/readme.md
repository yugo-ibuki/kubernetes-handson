# ヘルスチェック

3種類の Probe でヘルスチェックを行うことができる。

- Readiness Probe
- Liveness Probe
- Startup Probe

## Readiness Probe

定期的に Pod の状態を確認し、リクエストを受け付ける準備ができているかを判断する。

全ての Pod のライフサイクルに対して有効。

probe で失敗すると、Service リソースの接続対象から外され、トラフィックを受けなくなる。

## Liveness Probe

Readiness probe と似ているが、Probe が失敗したときの挙動が変わる。

Readiness probe は Service から接続を外すのに対し、Liveness probe は Pod を再起動する。

そのため、無限に再起動を行ってしまう可能性があるため、安易に設定するべきではない。

## Startup Probe

基本的に、Readiness/Liveness probe の併用と同じになる。

## リソースについて

アプリケーションの運用において、リソースの設定は重要。

CPU、メモリ・Ephemeral Storage などを設定することで、Pod のリソース使用量を制限することができる。

### Requests

コンテナごとに指定することができ

Kubernetes のスケジューラはこの値を見てスケジュールする Node を決める。

どの Node も Requests に書かれている量が確保できなければ、Pod がスケジュールされることはない。

### Limits

コンテナごとに指定することができ

Pod が使用できるリソースの上限を設定する。

メモリが上限値を超える場合、OOM(Out of Memory) が発生し、Pod が kill される。

ただ、即座に kill されるということはない。

その代わり、スロットリングが発生し、アプリケーションのパフォーマンスが低下する。

## リソースの単位

一般的に

### メモリ

1K は 1024 (10の2乗)
Ki は 1000 (10の3乗)

となる。

### CPU

指定しないと、1は CPU の1コアを意味する。

1m = 0.001 コアなので、通常整数もしくはミリコアで指定する。

## QoS (Quality of Service)

Kubernetes の機能で、Node のメモリが完全に使い切られた場合、全てのコンテナが動作できなくなってしまうのを防ぐために

OOM killer という役割のプログラムがいる。

Pod の優先順位を決め、優先度の低い Pod から OOMKill するために、QoS が使われる。

クラスには以下の3つがある。

- BestEffort
  - リソースの Requests/Limits が設定されていない
- Burstable
  - リソースの Requests/Limits のうち、少なくとも1つはが設定されている
- Guaranteed
  - リソースの Requests/Limits が全て設定されている

以下コマンドで確認することもできる。

```bash
$ kubectl get pod hello-server --output jsonpath='{.status.qosClass}'
Guaranteed⏎
```

## トラブルシューティング

```bash
$ kubectl get pod --namespace default
NAME                           READY   STATUS    RESTARTS   AGE
hello-server-9cbfdfd5c-6xnzj   0/1     Pending   0          37s
hello-server-9cbfdfd5c-b98td   1/1     Running   0          37s
hello-server-9cbfdfd5c-mgv29   0/1     Pending   0          37s
```

Pending になっているものを詳細を確認する。

```bash
$ kubectl describe pod hello-server-9cbfdfd5c-6xnzj --namespace default
Name:             hello-server-9cbfdfd5c-6xnzj
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           app=hello-server
                  pod-template-hash=9cbfdfd5c
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Controlled By:    ReplicaSet/hello-server-9cbfdfd5c
Containers:
  hello-server:
    Image:      blux2/hello-server:1.6
    Port:       8080/TCP
    Host Port:  0/TCP
    Limits:
      cpu:     10m
      memory:  5Gi
    Requests:
      cpu:        10m
      memory:     5Gi
    Liveness:     http-get http://:8080/health delay=10s timeout=1s period=5s #success=1 #failure=3
    Readiness:    http-get http://:8080/health delay=5s timeout=1s period=5s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-q5kgp (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-q5kgp:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  78s   default-scheduler  0/1 nodes are available: 1 Insufficient memory. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.
```

node のメモリが足りないため、Pod がスケジュールされていないことがわかる。

```bash
$ kubectl describe node --namespace default                        水  5/ 1 13:43:20 2024
Name:               kind-control-plane
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=arm64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=arm64
                    kubernetes.io/hostname=kind-control-plane
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Mon, 29 Apr 2024 13:30:51 +0900
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  kind-control-plane
  AcquireTime:     <unset>
  RenewTime:       Wed, 01 May 2024 13:43:18 +0900
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Wed, 01 May 2024 13:38:37 +0900   Mon, 29 Apr 2024 13:30:49 +0900   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Wed, 01 May 2024 13:38:37 +0900   Mon, 29 Apr 2024 13:30:49 +0900   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Wed, 01 May 2024 13:38:37 +0900   Mon, 29 Apr 2024 13:30:49 +0900   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Wed, 01 May 2024 13:38:37 +0900   Mon, 29 Apr 2024 13:31:09 +0900   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  172.22.0.2
  Hostname:    kind-control-plane
Capacity:
  cpu:                10
  ephemeral-storage:  61202244Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  hugepages-32Mi:     0
  hugepages-64Ki:     0
  memory:             8029368Ki   <==== ここ
  pods:               110
Allocatable:
  cpu:                10
  ephemeral-storage:  61202244Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  hugepages-32Mi:     0
  hugepages-64Ki:     0
  memory:             8029368Ki
  pods:               110
System Info:
  Machine ID:                 61e0b61e9a92413daffcaf165f7461b4
  System UUID:                61e0b61e9a92413daffcaf165f7461b4
  Boot ID:                    7a1baaf3-8870-44b3-90f9-b7a7db917d48
  Kernel Version:             6.6.12-linuxkit
  OS Image:                   Debian GNU/Linux 11 (bullseye)
  Operating System:           linux
  Architecture:               arm64
  Container Runtime Version:  containerd://1.7.1
  Kubelet Version:            v1.29.0
  Kube-Proxy Version:         v1.29.0
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
ProviderID:                   kind://docker/kind/kind-control-plane
Non-terminated Pods:          (10 in total)
  Namespace                   Name                                          CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                          ------------  ----------  ---------------  -------------  ---
  default                     hello-server-9cbfdfd5c-b98td                  10m (0%)      10m (0%)    5Gi (65%)        5Gi (65%)      2m38s < === ここ
  kube-system                 coredns-76f75df574-2p4cv                      100m (1%)     0 (0%)      70Mi (0%)        170Mi (2%)     2d
  kube-system                 coredns-76f75df574-z2xfw                      100m (1%)     0 (0%)      70Mi (0%)        170Mi (2%)     2d
  kube-system                 etcd-kind-control-plane                       100m (1%)     0 (0%)      100Mi (1%)       0 (0%)         2d
  kube-system                 kindnet-pp2fw                                 100m (1%)     100m (1%)   50Mi (0%)        50Mi (0%)      2d
  kube-system                 kube-apiserver-kind-control-plane             250m (2%)     0 (0%)      0 (0%)           0 (0%)         2d
  kube-system                 kube-controller-manager-kind-control-plane    200m (2%)     0 (0%)      0 (0%)           0 (0%)         2d
  kube-system                 kube-proxy-zw6zg                              0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d
  kube-system                 kube-scheduler-kind-control-plane             100m (1%)     0 (0%)      0 (0%)           0 (0%)         2d
  local-path-storage          local-path-provisioner-6f8956fb48-9849m       0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                960m (9%)     110m (1%)
  memory             5410Mi (68%)  5510Mi (70%)
  ephemeral-storage  0 (0%)        0 (0%)
  hugepages-1Gi      0 (0%)        0 (0%)
  hugepages-2Mi      0 (0%)        0 (0%)
  hugepages-32Mi     0 (0%)        0 (0%)
  hugepages-64Ki     0 (0%)        0 (0%)
Events:
```

今回は、kind のクラスタで1つの Node に Control Plane も乗せているので、kube-apiserver なども同じ Node に乗っていることが分かる。

```bash
$ kubectl get deployment hello-server -o=jsonpath='{.spec.template.spec.containers[0].resources.requests}' --namespace default
{"cpu":"10m","memory":"5Gi"}
```

これを、64Mi にする。

```bash
$ kubectl get deployments hello-server --namespace default --output=jsonpath='{.spec.template.spec.containers[0].resources.requests}'
{"cpu":"10m","memory":"64Mi"}⏎
```

変更後、確認する。

```bash
$ kubectl get pod --namespace default
NAME                            READY   STATUS    RESTARTS   AGE
hello-server-76d79d7889-642pf   1/1     Running   0          42s
hello-server-76d79d7889-bdtmc   1/1     Running   0          67s
hello-server-76d79d7889-njf5r   1/1     Running   0          93s
```

Pod を確認すると、全て Running になっていることがわかる。

### メモリーリークを起こす

メモリーリークを起こした Pod を見てみる。

```bash
$ kubectl describe pod hello-server-856d5c6c7b-xcd8z --namespace default
Name:             hello-server-856d5c6c7b-xcd8z
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-control-plane/172.22.0.2
Start Time:       Wed, 01 May 2024 14:11:40 +0900
Labels:           app=hello-server
                  pod-template-hash=856d5c6c7b
Annotations:      <none>
Status:           Running
IP:               10.244.0.37
IPs:
  IP:           10.244.0.37
Controlled By:  ReplicaSet/hello-server-856d5c6c7b
Containers:
  hello-server:
    Container ID:   containerd://af8e7226930ac9c7dbc2159721a94b29573cf192f03f5addd742dfaf3a41e295
    Image:          blux2/hello-server:1.7
    Image ID:       docker.io/blux2/hello-server@sha256:e34bb060e65c7f5cc58001c7e373e781e481b8875426227c3e1e4ac7709059af
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 01 May 2024 14:12:58 +0900
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    137
      Started:      Wed, 01 May 2024 14:12:03 +0900
      Finished:     Wed, 01 May 2024 14:12:46 +0900
    Ready:          True
    Restart Count:  1
    Limits:
      cpu:     10m
      memory:  100Mi
    Requests:
      cpu:        10m
      memory:     100Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-jvr8j (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-jvr8j:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age                   From               Message
  ----    ------     ----                  ----               -------
  Normal  Scheduled  2m54s                 default-scheduler  Successfully assigned default/hello-server-856d5c6c7b-xcd8z to kind-control-plane
  Normal  Pulling    2m49s                 kubelet            Pulling image "blux2/hello-server:1.7"
  Normal  Pulled     2m42s                 kubelet            Successfully pulled image "blux2/hello-server:1.7" in 7.398s (7.398s including waiting)
  Normal  Created    107s (x2 over 2m42s)  kubelet            Created container hello-server
  Normal  Pulled     107s                  kubelet            Container image "blux2/hello-server:1.7" already present on machine
  Normal  Started    96s (x2 over 2m31s)   kubelet            Started container hello-server
```

しかし、タイムアウトしか出ていないので、正確な理由が分からない。

そこで、Last State と Reason を参照してみる。

```bash
$ kubectl get pod hello-server-856d5c6c7b-xcd8z --output=jsonpath='{.status.containerStatuses[0].lastState}' --namespace default | jq .
{
  "terminated": {
    "containerID": "containerd://677613bdc1e2d1c9152350e8445c2ab73fe34eea59ba6a4659dbf4d6bdc63e21",
    "exitCode": 137,
    "finishedAt": "2024-05-01T05:12:46Z",
    "reason": "OOMKilled",  <== ここ
    "startedAt": "2024-05-01T05:12:03Z"
  }
}
```

lastState が terminated で、reason が OOMKilled となっている。

イメージタグを 1.8 にする。

port-forward で疎通確認して完了。

## スケジュールについて

同じ Node に Pod を乗せないことで、Node の障害に備えたり、特定のようとに使う Pod 専用に Node を立ち上げたりすることができる。

### Node Selector

特定の Node に対してのみ、制御を行う仕組み。

Node にラベルが付与されてる場合、次のように SSD を使っている Node にのみ Pod をスケジュールすることができる。

`disktype: ssd`

### Affinity / Anti-Affinity

Node と Pod の関係性を定義する仕組み。

近くなるように、または、近づかないようにスケジューリングすることができる。

- Node Affinity
  - Node Selector と似ているが、「可能ならスケジュールする」という選択ができる。
  - Node Selector は対応する Node がない場合、Pod がスケジュールされないが、Node Affinity は対応する Node がない場合、他の Node にスケジュールされる。
  - 以下の指定をすることで、マニフェストの書き方が変わる
    - requiredDuringSchedulingIgnoredDuringExecution
      - 一致する Node がない場合、Pod はスケジュールされない
    - preferredDuringSchedulingIgnoredDuringExecution
      - 一致する Node がない場合、他の Node にスケジュールされる
- Pod Affinity
  - ある Pod が特定の Pod と同じ Node にスケジュールされるようにする。
  - Node Affinity と同じように、Pod に対しても指定できる。
    - requiredDuringSchedulingIgnoredDuringExecution
      - 一致する Pod がない場合、Pod はスケジュールされない
    - preferredDuringSchedulingIgnoredDuringExecution
      - 一致する Pod がない場合、他の Node にスケジュールされる
- Pod Anti-Affinity
  - ある Pod が特定の Pod と同じ Node にスケジュールされないようにする。
  - Node Affinity と同じように、Pod に対しても指定できる。
    - requiredDuringSchedulingIgnoredDuringExecution
      - 一致する Pod がない場合、Pod はスケジュールされない
    - preferredDuringSchedulingIgnoredDuringExecution
      - 一致する Pod がない場合、他の Node にスケジュールされる

topologyKey というものを指定することで、どのようにマッチングするかを指定することができる。

## Pod を分散するための設定: Pod Topology Spread Constraints


