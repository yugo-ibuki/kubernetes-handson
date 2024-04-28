## 章

壊して直す編を進める

## 手順

1. kubectl get でリソースを確認
2. kubectl describe でリソースの詳細を確認
3. kubectl edit でリソースを編集

(1) で壊れているのを確認
```bash
kubectl get pod myapp --namespace default                                        日  4/28 13:42:49 2024
NAME    READY   STATUS             RESTARTS   AGE
myapp   0/1     ImagePullBackOff   0          11m
```

ImagePullBackOff はイメージが見つからないときに発生するエラー

(2) で詳細を確認

```bash
kubectl describe pod myapp --namespace default
```

```bash
Name:             myapp
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-control-plane/172.22.0.2
Start Time:       Sun, 28 Apr 2024 13:31:51 +0900
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.7
IPs:
  IP:  10.244.0.7
Containers:
  hello-server:
    Container ID:   containerd://bd15e07e874c4d4036157b4bf3c8326bac61c4480d006fbcf75259e467ba54e9
    Image:          blux2/hello-server:1.1
    Image ID:       docker.io/blux2/hello-server@sha256:35ab584cbe96a15ad1fb6212824b3220935d6ac9d25b3703ba259973fac5697d
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    2
      Started:      Sun, 28 Apr 2024 13:31:52 +0900
      Finished:     Sun, 28 Apr 2024 13:36:59 +0900
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-c9vml (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-c9vml:
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
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Normal   Scheduled  13m                    default-scheduler  Successfully assigned default/myapp to kind-control-plane
  Normal   Pulled     13m                    kubelet            Container image "blux2/hello-server:1.0" already present on machine
  Normal   Created    13m                    kubelet            Created container hello-server
  Normal   Started    13m                    kubelet            Started container hello-server
  Normal   Killing    8m42s                  kubelet            Container hello-server definition changed, will be restarted
  Warning  Failed     7m54s (x3 over 8m41s)  kubelet            Failed to pull image "blux2/hello-server:1.1": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/blux2/hello-server:1.1": failed to resolve reference "docker.io/blux2/hello-server:1.1": docker.io/blux2/hello-server:1.1: not found
  Warning  Failed     7m54s (x3 over 8m41s)  kubelet            Error: ErrImagePull
  Normal   BackOff    7m15s (x3 over 8m40s)  kubelet            Back-off pulling image "blux2/hello-server:1.1"
  Warning  Failed     7m15s (x3 over 8m40s)  kubelet            Error: ImagePullBackOff
  Warning  BackOff    6m8s (x7 over 7m41s)   kubelet            Back-off restarting failed container hello-server in pod myapp_default(eeab0ebf-b4f5-43ba-873e-8a36596f1766)
  Normal   Pulling    3m41s (x5 over 8m42s)  kubelet            Pulling image "blux2/hello-server:1.1"
```

Reason が ImagePullBackOff であることがわかる。

```bash
  Warning  Failed     7m54s (x3 over 8m41s)  kubelet            Failed to pull image "blux2/hello-server:1.1": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/blux2/hello-server:1.1": failed to resolve reference "docker.io/blux2/hello-server:1.1": docker.io/blux2/hello-server:1.1: not found
```

これにより、イメージが見つからないことがわかる。

2つの仮説
- リポジトリが存在しない
- タグが存在しない

今回は、タグが存在しなかった模様。

(3) で編集

```bash
kubectl edit pod myapp --namespace default
```

を実行し、1.1 を 1.0 に変更する。

```bash
pod/myapp edited
```

これで変更が確認された。

再度、pod の状態を確認する。

```bash
$ kubectl get pod myapp --namespace default                               3749ms  日  4/28 13:57:03 2024
NAME    READY   STATUS    RESTARTS      AGE
myapp   1/1     Running   1 (20m ago)   25m
```

running になっていることが確認できた。

最後に、掃除をする。

```bash
$ kubectl delete --filename chapter-05/pod-destruction.yaml --namespace default
pod "myapp" deleted
```

再度確認
    
```bash
$ kubectl get pod --namespace default
No resources found in default namespace.
```

これで完了。
