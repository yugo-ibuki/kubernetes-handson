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

#### 注意点

maxSurge 100% は、元あった Pod の数と同じ数だけ Pod を作成する。 

そのため、更新が早く、かつ安全な方法であるが、必要なリソースが倍になるため、使用する際はリソースキャパシティに注意する。

## ReplicaSet のエラー解消

maxUnavailable の値が 25% の場合、3つ立ち上がってると 0.75 になり、切り捨てられて 0 になる。

そのため、Pod の再作成ができない。

maxSurge は切り上げなので、1 になる。

そのため、1個 Pod を立ち上げることができる。

その後、Pod の数が4つになるので、1個 Pod を削除することができる。

## Service

### Service の種類

| Serviceタイプ        | 説明                                                                                                                               |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------|
| ClusterIP (デフォルト) | クラスタ内部の IP アドレスで Service を公開します。この Type で指定された IP アドレスはクラスタ内部からしか疎通できない。Ingress というリソースを利用することで外部公開が可能になる。                       |
| NodePort          | すべての Node の IP アドレスで指定したポート番号(NodePort)を公開する。                                                                                    |
| LoadBalancer      | 外部ロードバランサを用いて外部 IP アドレスを公開します。ロードバランサは別で用意する必要がある。                                                                               |
| ExternalName      | Service を externalName フィールドの内容にマッピングする(例えば、ホスト名が api.example.com)。このマッピングにより、クラスタの DNS サーバがその外部ホスト名の値を持つ CNAME レコードを返すように設定される。 |

接続先の Pod が削除されてしまった時に、Service があれば、自動的に新しい Pod に接続することができるルーティングの役割を持つ。

### ClusterIP

手順

1. Deployment で Pod を作成する。
2. Service の Port を Deployment に設定する。
3. Service を適用する。

以下コマンドで Service を通して外部からクラスターにアクセスできる。

```bash
$ kubectl run curl --image curlimages/curl --rm --stdin --tty --restart=Never --command -- curl <Cluster IP>:8080
```

### NodePort

全 Node に対して Port をひもづけるので、 port-forward しなくてもアクセスできる。

毎回 port-forward するのは面倒なので、NodePort を使うと、外部からアクセスできるが

本番環境では、Node が故障などで利用できなくなると使えなくなるので、ClusterIP や LoadBalancer を使うのが良い。

### Service の DNS

Kubernetes では、Service 用の DNS レコードを自動で作成してくれるため、FQDN を覚えておくと良い。

```bash
<Service名>.<Namespace名>.svc.cluster.local
```

Deployment & Service を作成し、以下のコマンドを叩くと、内部の DNS を利用してアクセスできることが確認できる。

```bash
$ kubectl --namespace default run curl --image curlimages/curl --rm --stdin --tty --restart=Never --command -- curl hello-server-service.default.svc.cluster.local:8080
Hello, world!pod "curl" deleted
```

## Service のトラブルシューティング

1. pod の状況を見る
2. deployment の状況を見る
3. Service の状況を見る

問題なかった場合は、以下の手順で見ていく。

アプリケーションに近いところから見ていくのが良い。

1. Pod 内からアプリケーションの接続確認を行う
2. クラスタ内かつ別 Pod から接続確認を行う
3. クラスタ内かつ別 Pod から Service 経由で接続確認を行う

1に問題があれば、Pod 内の問題。

2に問題があれば、Pod のネットワーク周りの問題

3に問題があれば、Service の設定の問題

全部違ったら、クラスタ内とクラスタ外に接続する設定周りの問題。

1. Pod 内からアプリケーションの接続確認を行う

```bash
$ kubectl get pod --namespace default
```

pod の確認を行った後

```bash
$ kubectl --namespace default debug --stdin --tty <Pod名> --image curlimages/curl --target=hello-server -- sh
```

Pod名参考: hello-server-6cc6b44795-469mw

すると、Session に入ることができる。

### クラスタ内かつ別 Pod から接続確認を行う

次のコマンドで Pod 一覧を参照し、Pod の IP を取得する

```bash
$ kubectl get pods -o custom-columns=NAME:.metadata.name,IP:.status.podIP
```

次のコマンドで、別の Pod から接続確認を行う

```bash
$ kubectl --namespace default run curl --image curlimages/curl --rm --stdin --tty --restart=Never --command -- curl <PodIP>:<Port>
```

もし、curl の Pod が消えてなかったら

```bash
$ kubectl delete pod curl
```

削除することでできる。

稀に、削除できずに残ってしまうことがあるようだ。

### クラスタ内かつ別 Pod から Service 経由で接続確認を行う

Service の情報を取得する。

```bash
$ kubectl get svc -o custom-columns=NAME:.metadata.name,IP:.spec.clusterIP
NAME                    IP
hello-server-external   10.96.3.161
kubernetes              10.96.0.1
```

Service に対して curl を実行する。

```bash
$ kubectl --namespace default run curl --image curlimages/curl --rm --stdin --tty --restart=Never --command -- curl <Service名>:<Port>
```

サービスに原因があることが分かったので、Service の設定を見直す。

```bash
$ kubectl describe service hello-server-external --namespace default
Name:                     hello-server-external
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=hello-serve
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.3.161
IPs:                      10.96.3.161
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30599/TCP
Endpoints:                <none>
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

これで適用前の状態と比較する。

```bash
$ kubectl diff --filename chapter-06/service-nodeport.yaml
```

## Pod の外部から情報を読み込む ConfigMap

環境変数などをコンテナ外から値を設定したい時に利用するリソース。

ConfigMap の利用する方法

1. コンテナ内のコマンドの引数として読み込む
2. コンテナの環境変数として読み込む
3. ボリュームを利用してアプリケーションのファイルとして読み込む

configMap の変更による環境変数は、アプリケーションの再起動をしないとアプリケーションに反映できない。

そのため、「ボリューム」を利用して、コンテナに設定ファイルを読み込ませる、ことでアプリケーションの再作成なしで configMap の内容を再読み込みすることができる。

configMapKeyRef というフィールドを利用することで、configMap の値を環境変数として利用することができる。

name と key を指定する。

## ボリュームを利用してアプリケーションのファイルとして読み込む

containers の volumeMounts に volume を指定する。

volumes には、configMap を指定する。

## トラブルシューティング

Status が、`CreateContainerConfigError` となっている場合は、コンテナの設定に問題がある。

```bash
$ kubectl describe pod --namespace default
```

エラーを確認し、HOST が定義されていないことを確認。

HOST を設定し直し、apply する。

この時、deployment が maxSurge で 0 になっていると、Pod が立ち上がらない。

今回は rollingUpdate の maxSurge が 0 になっていたため、kubernetes での更新ができなかった。

## Secret

データベースなどのパスワードを実装にハードコーディングしたくないケースや、本番環境と切り分けてる場合など

アプリケーションの外から設定したい場合がある。

こういった設定に、ConfigMap を参照できる人が全員秘密情報にアクセスできるのはセキュリティ上良くない。

そういった時、base64 などでエンコードした上で、Secret というリソースを使うことでアクセス権を分けることができる。

### Secret の読み込む方法

2つある。

1. コンテナの環境変数として読み込む
2. ボリュームを利用してコンテナに設定ファイルを読み込む

#### コンテナの環境変数として読み込む

Secret のデータを作成する。

```bash
$ echo -n 'admin' | base64
$ echo -n 'admin123' | base64
```

作成したデータを利用してマニフェストを作成する。

その後、secret のリソースを作成する。

```bash
$ kubectl apply -f chapter-06/secret/nginx-sample.yaml --namespace default
pod/nginx-sample created
secret/nginx-secret created
```

以下を実行し、コンテナの中に入る。

```bash
$ kubectl exec -it nginx-sample -- /bin/sh
```

```bash
echo $USERNAME
echo $PASSWORD
```

登録されていることが確認できる。

#### ボリュームを利用してコンテナに設定ファイルを読み込む

```bash
kubectl exec --stdin --tty nginx-sample -- /bin/sh       145ms  火  4/30 16:03:40 2024
# cat /etc/config/server.key
eM9ku3ecCpUL9zPoIIuG2ptZZC5Cu4ZCQXRymlHajYvZyffpM6
```

## 1回限りの Job の実行

Job は1回限り実行したい Pod を利用する。

job を見てみる。

```bash
kubectl get job --namespace default
NAME           COMPLETIONS   DURATION   AGE
date-checker   1/1           16s        21s
```

pod を見てみる。

```bash
kubectl get pod --namespace default
NAME                 READY   STATUS      RESTARTS   AGE
date-checker-zg6mj   0/1     Completed   0          40s
```

ready が 0/1 になっている。

実行してみる。

```bash
$ kubectl logs date-checker-zg6mj --namespace default
```

```bash
kubectl describe job date-checker --namespace default
Name:             date-checker
Namespace:        default
Selector:         batch.kubernetes.io/controller-uid=5fcd8a28-b1f1-47a8-9b84-394f28658a72
Labels:           batch.kubernetes.io/controller-uid=5fcd8a28-b1f1-47a8-9b84-394f28658a72
                  batch.kubernetes.io/job-name=date-checker
                  controller-uid=5fcd8a28-b1f1-47a8-9b84-394f28658a72
                  job-name=date-checker
Annotations:      <none>
Parallelism:      1
Completions:      1
Completion Mode:  NonIndexed
Start Time:       Tue, 30 Apr 2024 16:19:12 +0900
Completed At:     Tue, 30 Apr 2024 16:19:28 +0900
Duration:         16s
Pods Statuses:    0 Active (0 Ready) / 1 Succeeded / 0 Failed # <- 1 Succeeded となっている
Pod Template:
  Labels:  batch.kubernetes.io/controller-uid=5fcd8a28-b1f1-47a8-9b84-394f28658a72
           batch.kubernetes.io/job-name=date-checker
           controller-uid=5fcd8a28-b1f1-47a8-9b84-394f28658a72
           job-name=date-checker
  Containers:
   date:
    Image:      ubuntu:22.04
    Port:       <none>
    Host Port:  <none>
    Command:
      date
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From            Message
  ----    ------            ----   ----            -------
  Normal  SuccessfulCreate  4m3s   job-controller  Created pod: date-checker-zg6mj
  Normal  Completed         3m48s  job-controller  Job completed
  ```

## CronJob

定期的に Job を実行するためのリソース。

spec に `schedule` というフィールドを追加する。

'2/1 * * * *' という形式で指定する。

```bash
$ kubectl get cronjob --namespace default
```

```bash
$ kubectl get job --namespace default
NAME            COMPLETIONS   DURATION   AGE
date-28574386   1/1           3s         51s
```

```bash
kubectl get pod --namespace default                              火  4/30 16:48:04 2024
NAME                  READY   STATUS      RESTARTS   AGE
date-28574386-bn46s   0/1     Completed   0          2m10s
date-28574388-q69g8   0/1     Completed   0          10s
```

2分ごとに実行されているのが分かる。
