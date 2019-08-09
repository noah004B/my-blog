---
title: "GKE上に検証用のPosgreSQLコンテナのデプロイする"
date: 2019-08-09
published: true
tags: ['postgres', 'kubernetes', 'gke']
series: false
canonical_url: false
description: "ローカルのPostgreSQLクライアントから接続できるPostgreSQLの検証環境がほしかったので、GKE上にコンテナとしてデプロイしてみました。手順を残しておきます。"
---

ローカルのPostgreSQLクライアントから接続できるPostgreSQLの検証環境がほしかったので、GKE上にコンテナとしてデプロイしてみました。手順を残しておきます。

ServiceはNodePortとして公開するため、クライアントからは `<NodeIP>:<NodePort>` でアクセスします。

## 前提

ローカルマシンでgcloud, kubectlコマンドが使えること

## 外部からアクセスできるPosgreSQLコンテナのデプロイの流れ

1. Kubernetesのマニフェストファイルを作成する
2. kubectl コマンドでクラスタにデプロイ
3. PostgreSQLの接続情報を確認
4. GCPのファイアウォールに許可設定を追加

## 手順

### Kubernetesのマニフェストファイルを作る

 `postgres.yaml` に以下を書き込んで保存します。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
  labels:
    app: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:9.4
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_PASSWORD
          value: "P@ssw0rd"
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  type: NodePort
  ports:
    - protocol: TCP
      port: 5432
      targetPort: 5432
```

### クラスタにデプロイ

```
$ kubectl apply -f postgres.yaml
```

### 接続先の確認

NodeのIPアドレスを確認します。EXTERNAL-IPのカラムのIPです。

```
$ kubectl get nodes --output wide
NAME                                            STATUS   ROLES    AGE   VERSION         INTERNAL-IP   EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-your-first-cluster-1-pool-1-xxxxx-xxx   Ready    <none>   9h    v1.13.7-gke.8   10.128.0.2     35.186.158.46   Container-Optimized OS from Google   4.14.127+        docker://18.9.3
```

NodePortのポート番号を確認します。下記の例の場合30974です。

```
$ kubectl get service postgres-service -o yaml
（省略）
spec:
  clusterIP: 10.0.12.209
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 30974
    port: 5432
    protocol: TCP
    targetPort: 5432
```

上記のマニフェストの内容を含めると、Posgresの接続情報は以下のようになります。

```
Host: 35.186.158.46
Port: 30974
Database: postgres
Username: postgres
Password: P@ssw0rd
```

### GCPのファイアウォールに許可設定を追加

一時的な検証環境とはいえもう少しセキュアにする方法を検討したいところですが、今回はNodePortを許可します。

```
$ gcloud compute firewall-rules create test-node-port --allow tcp:30974
```

もとに戻す場合は以下のコマンドを実行します。

```
$ gcloud compute firewall-rules delete test-node-port
```



---

## 参考情報

Exposing applications using services（GKEの公式ドキュメント）
https://cloud.google.com/kubernetes-engine/docs/how-to/exposing-apps