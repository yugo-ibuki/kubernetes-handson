## Control Plane を破壊する

Cluster を作成する。

```bash
$  kind create cluster -n multinode-nodeport --config ./kind/multinode-config.yaml --image=kindest/node:v1.29.0
```

確認

```bash
$ kubectl get node multinode-nodeport-worker -o jsonpath='{.status.addresses[?(@.type=="InternalIP")].address}'
```

docker 確認

```bash
$ docker ps
```

```bash
$ kubectl stop <Control Plane の ID>
```

Node は動いていることを確認

```bash
$ curl localhost:30599
```

しかし、Pod は動かない

```bash
$ kubectl get pod --namespace default                                                                                                金  5/ 3 21:12:54 2024
The connection to the server 127.0.0.1:61470 was refused - did you specify the right host or port?
```

これは、Control Plane が停止したため、kube-apiserver が動かなくなったため。

コンテナの数は増減しないが、少なくともサービスの稼働が即座に損なわれるということはない。

Control Plane を起動し直すと、今まで通り kubectl を使えるようになる。

```bash
$ docker start <Control Plane の ID>
```
