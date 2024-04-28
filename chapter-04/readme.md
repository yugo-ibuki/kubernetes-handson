# 最小構成リソース: Pod

Pod というリソースがある。

# クラスタ上にリソースを作る

```shell
kubectl apply --filename chapter-04/myapp.yaml --namespace default
```

すると以下のログが出て、作成できた模様。

```bash
pod/myapp created
```

## Pod の存在の確認

```shell
kubectl get pod --namespace default
```

以下のように表示される。

```bash
NAME    READY   STATUS    RESTARTS   AGE
myapp   1/1     Running   0          49s
```

作成できているのが確認できた。

## kubectl run ではない理由

コマンド自体は存在している。

しかし、 apply の方が推奨されている。

- マニフェストがあった方が変更の差分が参照できる
- kubectl run は Pod の冗長化などの高度な設定には使えない

run は一時的なデバッグに使われることが多い。

# IPやNode情報の取得

```shell
kubectl get pod --output wide --namespace default
```

yaml ファイルに出力結果を出す

```shell
kubectl get pod myapp --output yaml --namespace default > pod.yaml
```

`diff pod.yaml chapter-04/myapp.yaml` で差分を確認できる。

すると、大量の差分が出る。

これは、マニフェストにあるものはあくまで必須の内容で、宣言的に指定したい内容のみを記載しているだけ。

そのため、自動的にいろんな情報をリソースに付与するため、--output yaml には、たくさんの情報が含まれる。

そこで、欲しい情報だけを取得するのに、以下のコマンドを使う。

```shell
kubectl get pod <Pod名> --output jsonpath='{.spec.containers[].image}' --namespace default
```

## ログレベルを変更する

あまり変更することはない。

```shell
kubectl get pod <Pod名> --v=<ログレベル>
```

## リソースの詳細の取得

Events の内容は、トラブルシューティング時に役に立つ。

ただし、Events は一定時間で消えてしまう。

```shell
kubectl describe pod myapp --namespace default
```

## コンテナのログを取得する

これで表示されるのは、標準出力のログになる。

```shell
kubectl logs myapp --namespace default
```

## Deployment にひもづく pod のログを参照

```shell
kubectl logs deploy<Deployment名>
```

## ラベルを指定して参照する pod を絞り込む

Deployment 以外の理由で Pod の参照を絞り込みたい時、

同じラベルを利用していればラベルを指定して Pod を参照できる。

```shell
kubectl get pod --selector(-1) <labelのキー名>=<labelの値>
```

## デバッグ用のサイドカーコンテナを立ち上げる

権限が必要な可能性もある。

企業の場合は、自分の権限で立ち上げることができないかもしれない。

```shell
kubectl debug --stdin --tty <デバッグ対象Pod名> --image=<デバッグ用のコンテナのimage> --target=<デバッグ対象コンテナ名>
```

### 実際にデバッグ用のコンテナを立ち上げる

```shell
kubectl debug --stdin --tty myapp --image=curlimages/curl:8.4.0 --target=hello-server --namespace default -- sh
```

実行結果

```bash
 kubectl debug --stdin --tty myapp --image=curlimages/curl:8.4.0 --target=hello-server --namespace default -- sh
Targeting container "hello-server". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-c6b9r.
If you don't see a command prompt, try pressing enter.
~ $ curl localhost:8080
Hello, world!~ $
```

## コンテナを即座に実行する

```shell
kubectl run <Pod名> --image=<イメージ名>
```

これまでは、クラスタ内からアクセスするために、デバッグ用 Pod を起動する必要があった。

```shell
kubectl --namespace default run busybox --image=busybox:1.36.1 --stdin --tty --restart=Never --rm --command -- nslookup google.com
```
これは、busybox という Pod を起動し、nslookup コマンドを実行したら終了

各オプションの説明のテーブル

| オプション | 説明                                    |
| --- |---------------------------------------|
| --stdin | 標準入力を受け付ける                            |
| --tty | 擬似端末を割り当てる                            |
| --restart=Never | 終了したら再起動しない。デフォルトでは常に再起動する。           |
| --rm | 終了したら削除する                             |
| --command | -- の後に渡される拡張引数の一つ目が引数ではなくコマンドとして使われる。 |

## コンテナへログイン

```shell
kubectl exec --stdin --tty <Pod名> -- <コマンド名>
```

使用用途としては、アプリケーションがインターネット上からアクセスできなくなったときに、クラスタ内から上記のように IP アドレスでアクセスできるか確認することで、問題を切り分けることができる。

### 手順

1. ログイン用の Pod を起動する (kubectl run <pod> --command -- /bin/sh -c "while true: do sleep 1; done")
2. Pod ができているか確認する (kubectl get pod)
3. IP アドレスを取得する (kubectl get pod <pod> --output wide)
4. ログインする (kubectl exec --stdin --tty <pod> -- /bin/sh)
5. ログイン後、curl でアクセスする (curl <IPアドレス>:<ポート番号>)

## Port-forward でアクセスする

Service というリソースを利用することで、クラスタ外からのアクセスを可能にする。

kubectl でもお手軽にアクセスすることが可能で、以下のコマンドを利用する。

```shell
kubectl port-forward <Pod名> <転送先ポート番号>:<転送元ポート番号>
```

```shell
kubectl port-forward myapp 5555:8080 --namespace default
```

## マニフェストをその場で編集する (kubectl edit)

この方法を推奨されない

```shell
kubectl edit <リソース名>
```

## リソースを削除する (kubectl delete)

本番環境では、通常「Deployment」というリソースを使って、Pod を冗長化している。

ある特定の Pod だけハングしてしまった、と言った時に Pod を削除します。

Deployment を使用していれば、自動的に削除した Pod が再度作成されるようになるので、Pod を削除しても問題ないケースが多い。

```shell
kubectl delete <リソース名>
```

### 再起動したい場合 (kubectl rollout restart)

Deployment を利用した Pod をすべて順番に再起動したい場合は、kubectl delete ではなく、kubectl rollout restart を利用する。

```shell
kubectl rollout restart deployment <Deployment名>
```

他のコマンドは、チートシートを見るのが良い。

https://kubernetes.io/docs/reference/kubectl/quick-reference/
