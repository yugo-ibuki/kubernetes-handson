## ターミナル操作のtips

### 自動補完の設定

```shell
kubectl <TAB>
```

リソース名の補完

```shell
kubectl get p<TAB>
```

## 省略できる

```shell
kubectl api-resources
```

| 名前                             | ショートネーム                        | APIバージョン                  | 名前空間  | 種類                  |
|----------------------------------|--------------------------------|-----------------------------|------------|----------------------|
| bindings                         |                                | v1                          | true       | Binding              |
| componentstatuses                | cs                             | v1                          | false      | ComponentStatus      |
| configmaps                       | cm                             | v1                          | true       | ConfigMap            |
| endpoints                        | ep                             | v1                          | true       | Endpoints            |
| events                           | ev                             | v1                          | true       | Event                |
| limitranges                      | limits                         | v1                          | true       | LimitRange           |
| namespaces                       | ns                             | v1                          | false      | Namespace            |
| nodes                            | no                             | v1                          | false      | Node                 |
| persistentvolumeclaims           | pvc                            | v1                          | true       | PersistentVolumeClaim|
| persistentvolumes                | pv                             | v1                          | false      | PersistentVolume     |
| pods                             | po                             | v1                          | true       | Pod                  |
| podtemplates                     |                                | v1                          | true       | PodTemplate          |
| replicationcontrollers           | rc                             | v1                          | true       | ReplicationController|
| resourcequotas                    | quota                          | v1                          | true       | ResourceQuota        |
| secrets                          |                                | v1                          | true       | Secret               |
| serviceaccounts                   | sa                             | v1                          | true       | ServiceAccount       |
| services                          | svc                            | v1                          | true       | Service              |
| mutatingwebhookconfigurations     |                                | admissionregistration.k8s.io/v1   | false  | MutatingWebhookConfiguration|
| validatingwebhookconfigurations   |                                | admissionregistration.k8s.io/v1 | false  | ValidatingWebhookConfiguration|
| customresourcedefinitions         | crd,crds                       | apiextensions.k8s.io/v1      | false      | CustomResourceDefinition|
| apiservices                       |                                | apiregistration.k8s.io/v1       | false      | APIService            |
| controllerrevisions               |                                | apps/v1                         | true       | ControllerRevision   |
| daemonsets                        | ds                             | apps/v1                       | true       | DaemonSet            |
| deployments                       | deploy                         | apps/v1                       | true       | Deployment            |
| replicasets                       | rs                             | apps/v1                       | true       | ReplicaSet            |
| statefulsets                      | sts                            | apps/v1                       | true       | StatefulSet           |
| selfsubjectreviews                |                                | authentication.k8s.io/v1        | false      | SelfSubjectReview   |
| tokenreviews                      |                                | authentication.k8s.io/v1        | false      | TokenReview          |
| localsubjectaccessreviews         |            | authorization.k8s.io/v1        | true       | LocalSubjectAccessReview|
| selfsubjectaccessreviews          |            | authorization.k8s.io/v1        | false      | SelfSubjectAccessReview|
| selfsubjectrulesreviews           |            | authorization.k8s.io/v1        | false      | SelfSubjectRulesReview|
| subjectaccessreviews              |            | authorization.k8s.io/v1        | false      | SubjectAccessReview   |
| horizontalpodautoscalers          | hpa                            | autoscaling/v2               | true       | HorizontalPodAutoscaler|
| cronjobs                          | cj                             | batch/v1                      | true       | CronJob               |
| jobs                              |            | batch/v1                       | true       | Job                    |
| certificatesigningrequests        | csr                            | certificates.k8s.io/v1        | false      | CertificateSigningRequest|
| leases                            |            | coordination.k8s.io/v1         | true      | Lease                  |
| endpointslices                    |            | discovery.k8s.io/v1            | true       | EndpointSlice          |
| events                            | ev                             | events.k8s.io/v1              | true       | Event                   |
| flowschemas                       |            | flowcontrol.apiserver.k8s.io/v1 | false    | FlowSchema             |
| prioritylevelconfigurations       |            | flowcontrol.apiserver.k8s.io/v1 | false     | PriorityLevelConfiguration|
| ingressclasses                    |            | networking.k8s.io/v1           | false    | IngressClass            |
| ingresses                         | ing                            | networking.k8s.io/v1            | true     | Ingress                 |
| networkpolicies                   | netpol                         | networking.k8s.io/v1           | true       | NetworkPolicy         |
| runtimeclasses                    |            | node.k8s.io/v1                 | false     | RuntimeClass           |
| poddisruptionbudgets              | pdb                            | policy/v1                         | true     | PodDisruptionBudget  |
| clusterrolebindings               |            | rbac.authorization.k8s.io/v1   | false       | ClusterRoleBinding   |
| clusterroles                      |            | rbac.authorization.k8s.io/v1   | false       | ClusterRole           |
| rolebindings                      |            | rbac.authorization.k8s.io/v1   | true         | RoleBinding           |
| roles                             |            | rbac.authorization.k8s.io/v1   | true         | Role                    |
| priorityclasses                   | pc                             | scheduling.k8s.io/v1          | false         | PriorityClass        |
| csidrivers                        |            | storage.k8s.io/v1              | false        | CSIDriver               |
| csinodes                          |            | storage.k8s.io/v1              | false         | CSINode                  |
| csistoragecapacities              |            | storage.k8s.io/v1              | true       | CSIStorageCapacity    |
| storageclasses                    | sc                             | storage.k8s.io/v1                | false        | StorageClass          |
| volumeattachments                 |            | storage.k8s.io/v1              | false         | VolumeAttachment     |

## 便利なツール

### terminal

- stern
  - Pod のログを出力するツール
  - pod の名前が Deployment-XXX になっていても、すべての Pod のログを出力する
  - 削除や、追加されても自動で出力してくれる
- k9s
  - コマンドを使わずにターミナル上で分かりやすい UI を提供するツール
  - ログの確認や、リソースの確認ができる
- starship
  - シェルプロンプトをカスタマイズするツール
  - Kubernetes の Namespace やコンテキストを表示することができる

### プラグイン

- kubectx / kubens
  - GitHub プロジェクト
  - kubectx はコンテキスト（クラスタ接続情報）をスイッチするために利用する
  - kubens は Namespace をスイッチするために利用する
  - 複数クラスタを利用する場合は、次の事に留意する必要がある
    - 何もしなければデフォルトコンテキスト（ローカルクラスタ）が使用される。
    - デフォルトコンテキスト以外（ステージングクラスタ）を使用したい場合、kubectl --context を利用するか、kubectl config use-context を利用してデフォルトコンテキストを切り替える必要がある