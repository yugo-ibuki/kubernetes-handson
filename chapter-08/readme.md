## 復習トラブルシューティング

手順

1. `kubectl get pod --namespace default` で Pod の状態を確認します。
2. `kubectl describe pod <Pod名> --namespace default` で Pod の詳細を確認します。
3. health check のエラーを確認します。
4. ポートが 8082 で開かれていることを確認します。
5. `kubectl logs <Pod名> --namespace default` でログを確認します。
6. kubectl get deployment hello-server --output yaml --namespace default` でマニフェストを確認します。
7. ポートが 8081 になっていることを確認する。
8. ` cp chapter-08/hello-server-update.yaml chapter-08/hello-server-update-fix.yaml` でファイルをコピーします。
9. `diff chapter-08/hello-server-update.yaml chapter-08/hello-server-update-fix.yaml` で差分を確認します。
10. `kubectl apply -f chapter-08/hello-server-update-fix.yaml --namespace default` で適用します。
11. 再度、`kubectl get pod --namespace default` で Pod の状態を確認します。
12. まだ、Ready にならない Pod があるのを確認します。
13. 再度、`kubectl describe pod <Pod名> --namespace default` で詳細を確認します。
14. endpoint 名がずれているのを直します。
15. curl で確認します。
16. empty reply from server が返ってくるのを確認します。
17. running になっているので、コンテナを立ち上げて、コンテナ内部から接続確認してみます。
18. `kubectl debug --stdin --tty hello-server-758cd6dbd7-t527q --image=curlimages/curl --container=debug-container -- sh` でコンテナに接続します。
19. 