# 使用ツール

- kind

# コマンド

## Cluster 作成 

```shall
kind create cluster --image=kindest/node:v1.29.0
```

## Cluster 削除

```shall
kind delete cluster
```

## Cluster Info

```shall
kubectl cluster-info --context kind-kind 
```

以下で、デフォルトのコンテキストを指定することもできる

```shell
kubectl config use-context
```
