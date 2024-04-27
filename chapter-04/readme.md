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