# *Kubernetesのデプロイ手順*

## *Golangコンテナ編*

### *Docker imageの作成*

1. Docker imageの作成

`docker build ./ -t k8s-anyconfig-backend`

2. docker imageが作成されたか以下で確認する。

`docker images | grep k8s-anyconfig-backend`

3. 上記で作成したimageを元にhelloという名前のcontainerを作成する。

`docker run --name hello -d -p 8000:8080 k8s-anyconfig-backend`

4. 動作確認

```
$ curl http://localhost:8000/hello
{"message":"Hello World!"}
```

### *Clusterの作成*

1. Docker for Desktopにて、Kubernetesの機能を有効化する。

2. 下記コマンドでクラスタの状態を確認する。

`kubectl cluster-info`

### *Deployment（Pod）の作成*

Deployment は Kubernetes で管理するアプリケーションの単位です。

1. アプリケーションの作成

`kubectl apply -f deployment.yaml`

2. Deployment の一覧や詳細を表示する。

`kubectl get deployments`

`kubectl describe deployment/k8s-anyconfig-backend`

3. Deployment が作成されると、アプリケーションを実行するノードが自動的に選択(スケジュール)され、実行されます。この実行の単位を Pod と呼びます。

`kubectl get pods`

※　可用性や性能を考慮してひとつの Deployment を複数の Pod で実行することもできます。Pod は適切なノードに配布され、実行されます。ノードがダウンすると Kubernetes が検知し、代わりに別のノードで Pod を実行させます。

### *Serviceの作成*

アプリケーション(Deployment)を外部に公開するには Service を作成します。Service の公開方法には下記などがあります。

* ClusterIP - クラスター内部のIPでServiceを公開します。クラスター内部からのみアクセス可能です。
* NodePort - NATを使用してServiceを公開します。クラスター外部からのアクセスが可能となります。
* LoadBalancer - ロードバランサでServiceを公開します。Minikubeではサポートされていません。
* ExternalName - FQDN と Kube-DNS を用いてServiceを公開します。

1. Serviceの公開前の状態を確認する。

```
$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   23d
```

2. Serviceを公開する。

`kubectl expose deployment/k8s-anyconfig-backend --type=NodePort --port 8080`

3. Serviceの公開後の状態を確認する。

```
$ kubectl get service
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
k8s-anyconfig-backend   NodePort    10.101.106.152   <none>        8080:30614/TCP   3m28s
kubernetes              ClusterIP   10.96.0.1        <none>        443/TCP          23d
```

4. 動作確認

```
$ curl http://localhost:30614/hello
{"message":"Hello World!"}
```
</br>

- 【参考】kubectlコマンド
kubectrl は下記のコマンドをサポートしています。

```
# ヘルプ
kubectl --help		# kubectlのヘルプを表示
kubectl command --help	# 指定したコマンドのヘルプを表示

# 表示
kubectl cluster-info	# クラスター情報を表示
kubectl get pods	# Pod一覧を表示
kubectl get nodes	# ノード一覧を表示
kubectl get deployments	# Deployment一覧を表示
kubectl get services	# サービス一覧を表示

# 基本コマンド(初級)
kubectl create		# Deploymentなどのリソースを作成
kubectl expose		# サービスを公開
kubectl run		# 指定したイメージをクラスターで実行
kubectl set		# 指定した機能をオブジェクトに設定

# 基本コマンド(中級)
kubectl explain		# リソースに関するドキュメントを表示
kubectl get		# リソースを表示
kubectl edit		# リソースを編集
kubectl delete		# リソースを削除

# デプロイコマンド
kubectl rolloout	# ローリングアップデート
kubectl scale		# スケールアップ/スケールダウン
kubectl autoscale	# オートスケール

# クラスター管理
kubectl certificate	# 証明書を管理
kubectl cluster-info	# クラスター情報を表示
kubectl top		# リソースの利用状況(上位)を表示
kubectl cordon		# ノードをスケジュール不可状態にする
kubectl uncordon	# ノードをスケジュール可能状態にする
kubectl drain		# ノードをメンテナンスモードにする
kubectl taint		# ノードにスケジュール制御のためのTaintを設定/設定解除

# トラブルシュートとデバッグ
kubectl describe	# リソースの詳細を表示
kubectl logs		# Podのログを表示
kubectl attach		# コンテナにアタッチ
kubectl exec		# コンテナでコマンドを実行
kubectl port-forward	# ポートフォワード
kubectl proxy		# Kubernetes API Serverへのプロキシを起動
kubectl cp		# コンテナとのファイルコピー
kubectl auth		# 認可検査
kubectl debug		# デバッギングセッションを作成

# アドバンスドコマンド
kubectl diff		# 現バージョンと新バージョンの差異を表示
kubectl apply		# コンフィグレーション情報をリソースに設定
kubectl patch		# リソースの一部を変更
kubectl replace		# リソースを置換
kubectl wait		# リソースの状態待ち
kubectl kustomize	# カスタマイズターゲットを生成

# セッティングコマンド
kubectl label		# リソースにラベルを設定
kubectl annotate	# リソースのアノテーションを設定
kubectl completion	# シェル補完用コードを生成

# その他コマンド
kubectl alpha		# アルファバージョンのコマンドを表示
kubectl api-resources	# リソース種別の一覧を表示
kubectl api-versions	# APIバージョンを表示
kubectl config		# コンフィグ管理
kubectl plugin		# プラグインを管理
kubectl version		# kubectlのバージョンを表示
```
