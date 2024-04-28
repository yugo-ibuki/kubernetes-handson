## pod 以外のリソース

実際の現場では、pod を直に触る事は推奨されない。

そこで Deployment を使用する。

## ReplicaSet

ReplicaSet も直接利用は推奨されていない。

が、紹介されている。

ReplicaSet は pod の数を管理するリソース。

Pod を複製する時に使う。

```bash
$ kubectl get replicaset --namespace default                                       日  4/28 14:41:09 2024
NAME         DESIRED   CURRENT   READY   AGE
httpserver   3         3         3       51s
```

```bash
$ kubectl get pod --namespace default                                      200ms  日  4/28 14:41:22 2024
NAME               READY   STATUS    RESTARTS   AGE
httpserver-bpgtp   1/1     Running   0          56s
httpserver-btjg2   1/1     Running   0          56s
httpserver-gwf68   1/1     Running   0          56s
```

## Deployment

推奨されている方法として、Deployment がある。

ReplicaSet だと、v1、v2 というようにバージョンを管理する際に、ローリングアップデートができないという点がある。

そこで、バージョンを管理するための、さらに上位の概念として Deployment がある。

```bash
$ kubectl apply --filename chapter-06/deployment.yaml --namespace default          日  4/28 14:47:38 2024
deployment.apps/nginx-deployment created
```

```bash
$ kubectl get pod --namespace default                                              日  4/28 14:48:21 2024
NAME                                READY   STATUS              RESTARTS   AGE
httpserver-bpgtp                    1/1     Running             0          7m53s
httpserver-btjg2                    1/1     Running             0          7m53s
httpserver-gwf68                    1/1     Running             0          7m53s
nginx-deployment-595dff4799-4rvqh   0/1     ContainerCreating   0          27s
nginx-deployment-595dff4799-9tpgg   0/1     ContainerCreating   0          27s
nginx-deployment-595dff4799-pn287   0/1     ContainerCreating   0          27s
```

```bash
$ kubectl get replicaset --namespace default                                       日  4/28 14:48:51 2024
NAME                          DESIRED   CURRENT   READY   AGE
httpserver                    3         3         3       8m24s
nginx-deployment-595dff4799   3         3         3       58s
```

以下を実行して出力する。

```bash
$ kubectl get deployment nginx-deployment -o=jsonpath='{.spec.template.spec.container[0].image}'
nginx:1.25.3
```

```bash
$ kubectl describe deployment nginx-deployment                                     日  4/28 14:52:30 2024
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Sun, 28 Apr 2024 14:47:56 +0900
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.25.3
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  nginx-deployment-595dff4799 (0/0 replicas created)
NewReplicaSet:   nginx-deployment-789bf7b8fc (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m49s  deployment-controller  Scaled up replica set nginx-deployment-595dff4799 to 3
  Normal  ScalingReplicaSet  2m56s  deployment-controller  Scaled up replica set nginx-deployment-789bf7b8fc to 1
  Normal  ScalingReplicaSet  2m55s  deployment-controller  Scaled down replica set nginx-deployment-595dff4799 to 2 from 3
  Normal  ScalingReplicaSet  2m55s  deployment-controller  Scaled up replica set nginx-deployment-789bf7b8fc to 2 from 1
  Normal  ScalingReplicaSet  2m54s  deployment-controller  Scaled down replica set nginx-deployment-595dff4799 to 1 from 2
  Normal  ScalingReplicaSet  2m54s  deployment-controller  Scaled up replica set nginx-deployment-789bf7b8fc to 3 from 2
  Normal  ScalingReplicaSet  2m53s  deployment-controller  Scaled down replica set nginx-deployment-595dff4799 to 0 from 1
```

RollingUpdateStrategy は、Pod にも ReplicaSet にもない、Deployment のみに存在するフィールドになる。

### StrategyType

Deployment を利用して、Pod の数を管理する際に、どのような戦略を取るかを指定する。

- Recreate: 一度全ての Pod を削除してから、新しい Pod を作成する。
- RollingUpdate: 一つずつ Pod を削除して、新しい Pod を作成する。
  - maxUnavailable: 一度に削除できる Pod の最大数
  - maxSurge: 一度に作成できる Pod の最大数
