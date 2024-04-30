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


