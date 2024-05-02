## Multi-node Clusters

```bash
$ kind create cluster -n kind-multinode --config kind/multinode-config.yaml --image=kindest/node:v1.29.0
```

確認してみる。

```bash
$ kubectl get node
NAME                           STATUS   ROLES           AGE   VERSION
kind-multinode-control-plane   Ready    control-plane   68s   v1.29.0
kind-multinode-worker          Ready    <none>          45s   v1.29.0
kind-multinode-worker2         Ready    <none>          45s   v1.29.0
```

## ハンズオン

Pod がスケジュールできないハンズオンを開始。

```bash
$ kubectl apply -f chapter-07/deployment-schedule-handson.yaml --namespace default
deployment.apps/hello-server created
```

確認すると、pending になっている pod が一つあるので、NAME をコピーして詳細を見てみる。

```bash
$ kubectl describe pod hello-server-9c5ff67bd-zrq9c --namespace default
Name:             hello-server-9c5ff67bd-zrq9c
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           app=hello-server
                  pod-template-hash=9c5ff67bd
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Controlled By:    ReplicaSet/hello-server-9c5ff67bd
Containers:
  hello-server:
    Image:        blux2/hello-server:1.8
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-v6k5d (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-v6k5d:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  68s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 2 node(s) didn't match pod anti-affinity rules. preemption: 0/3 nodes are available: 1 Preemption is not helpful for scheduling, 2 No preemption victims found for incoming pod.
```

toleration と affinity を確認してみる。

```bash
$ kubectl get node kind-multinode-control-plane -o jsonpath='{.spec.template.spec.tolerations}'
$ kubectl get deployment hello-server --output=jsonpath='{.spec.template.spec.affinity}' --namespace default | jq
{
  "podAntiAffinity": {
    "requiredDuringSchedulingIgnoredDuringExecution": [
      {
        "labelSelector": {
          "matchExpressions": [
            {
              "key": "app",
              "operator": "In",
              "values": [
                "hello-server"
              ]
            }
          ]
        },
        "topologyKey": "kubernetes.io/hostname"
      }
    ]
  }
}
```

Node の Toleration を見てみる。

```bash
$ kubectl get nodes -o custom-columns='NAME:.metadata.name,TAINTS-KEY:.spec.taints[*].key'
NAME                           TAINTS-KEY
kind-multinode-control-plane   node-role.kubernetes.io/control-plane
kind-multinode-worker          <none>
kind-multinode-worker2         <none>
```

kind-multinode-control-plane には node-role.kubernetes.io/control-plane という taint が付いている。

いくつか直し方はある。

1. Toleration をつけて Taint: {node-role.kubernetes.io/control-plane: } がついている Node にスケジュール可能とする
2. Node を増やし、Pod anti-affinity が守られるようにする
3. Deployment の Replicas を減らし、Pod anti-affinity が守られるようにする
4. requredDuringSchedulingIgnoredDuringExecution を preferredDuringSchedulingIgnoredDuringExecution に変更し、Pod anti-affinity が守られなくても問題ないようにする

今回は、3 を選択する。

```bash
$ kubectl scale deployment hello-server --replicas=2 --namespace default
deployment.apps/hello-server scaled
```

確認すると、Pod が running になっている。

## HPA を行う

HPA とは

- Horizontal Pod Autoscaler
    - Pod の数を自動で増減させる機能

```bash
$ kubectl patch --namespace kube-system deployment metrics-server --type=json --patch '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
deployment.apps/metrics-server patched
```

以下で、metrics-server が正常に起動していることを確認。

```bash
$ kubectl get deployment metrics-server --namespace kube-system
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           5m58s
```

## averageUtilization を使った HPA

averageUtilization とは

- CPU 使用率の平均値が指定した値を超えた場合に Pod の数を増やす
  - 50 だと、CPU 使用率の平均値が 50% を超えた場合に Pod の数を増やす

## Node 退役に備える

Node がシャットダウンしても安全にサービスを稼働し続けるための機能がいくつかある。

### PodDisruptionBudget

Deployment でカバーできるのは、あくまで Pod を更新するときだけ。

本番の運用環境では、Node をメンテナンスするために、Node から Pod を退避させるなど、Pod が増えたり減ったりするケースがよくあります。

しかし、Pod が増えたり減ったりすることで、サービスが停止してしまうことがあります。

そこで利用するのが、PodDisruptionBudget です。

PodDisruptionBudget は、Pod の更新や削除を行う際に、最低限稼働している Pod の数を指定することができます。

- minAvailable
    - 最低限稼働している Pod の数
- maxUnavailable
    - 稼働していない Pod の数
