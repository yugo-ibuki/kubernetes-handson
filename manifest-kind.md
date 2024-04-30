# マニフェストの kind の種類

- **Pod**: Kubernetesの最小単位であり、1つ以上のコンテナを含むことができます。
- **ReplicaSet**: 指定された数のPodレプリカを維持するために使用されます。
- **Deployment**: ReplicaSetを管理し、Podの更新やロールバックを処理します。
- **StatefulSet**: 状態を持つPodを管理するために使用されます。各Podは、一意の識別子と安定したストレージを持ちます。
- **DaemonSet**: 各ノードに1つずつPodを配置するために使用されます。
- **Job**: 1つ以上のPodを作成し、指定されたタスクが完了するまで実行します。
- **CronJob**: 定期的にJobを実行するためのスケジュールを定義します。
- **Service**: Podのグループに対して安定したネットワークエンドポイントを提供します。
- **Ingress**: クラスター外からのHTTPおよびHTTPSのルートを管理します。
- **ConfigMap**: アプリケーションの設定情報を保存するために使用されます。
- **Secret**: パスワード、トークン、キーなどの機密情報を保存するために使用されます。
- **PersistentVolume**: クラスターで使用されるストレージのプロビジョニングを表します。
- **PersistentVolumeClaim**: PersistentVolumeを要求し、Podでマウントできるようにします。
- **Namespace**: クラスターを仮想的に分割し、リソースを論理的に分離するために使用されます。
- **ServiceAccount**: Podが他のKubernetesサービスに対して認証するために使用します。
- **RoleとClusterRole**: Kubernetesの認可（RBAC）に使用され、ユーザーまたはサービスアカウントのアクセス権を定義します。
- **RoleBindingとClusterRoleBinding**: RoleまたはClusterRoleをユーザーまたはサービスアカウントに関連付けます。
- **HorizontalPodAutoscaler**: Podのレプリカ数を自動的に調整し、CPUやメモリの使用量に基づいてスケーリングを行います。
