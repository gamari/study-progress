# Kubernetes 基礎まとめ

このノートは Kubernetes の基礎を学ぶための要約です。クラスタの構成要素、主要なオブジェクト、操作コマンド、よく使うマニフェストの例、運用でよく出会う概念を簡潔にまとめています。
## 1. Kubernetes の目的
- コンテナ化されたアプリケーションのデプロイ、スケーリング、管理を自動化するオープンソースシステム。
- 抽象化により「アプリケーション」中心の操作が可能になる（ノードや VM を直接扱う必要が少ない）。
## 2. 基本用語と概念
- Pod: Kubernetes における最小の実行単位。1 つまたは複数のコンテナを含む。基本は単一のアプリケーションコンテナとそのサイドカーで構成される。
- Node: Pod を実行するワーカーノード（ノードは VM や物理マシン）。
- Cluster: 複数の Node と Control Plane を含む集合。
- Control Plane: クラスタ管理側。主に API サーバ（kube-apiserver）、etcd、スケジューラ（kube-scheduler）、コントローラマネージャ（kube-controller-manager）などからなる。
- Namespace: リソースの論理的な分割。複数のユーザやチームでクラスタを共有するときに便利。
- Labels / Selectors: リソースを識別・グルーピングするために使うキー/バリュー。
- ReplicaSet / Deployment: Pod のレプリカを管理・スケーリングする仕組み。Deployment はローリングアップデートやロールバックなどの高レベル操作をサポートする。
- Service: Pod の集合に対する永続的なネットワークアクセスポイント。ClusterIP, NodePort, LoadBalancer, ExternalName などのタイプがある。
- Ingress: HTTP/HTTPS のルーティングを提供。Ingress Controller と組み合わせて動作する。
- ConfigMap / Secret: 設定や機密値を Pod に注入するための手段。Secret は base64 エンコードされた小さな秘密情報を扱う。
- PersistentVolume (PV) / PersistentVolumeClaim (PVC): ストレージの抽象化と要求。PV は管理者が提供し、PVC はユーザ側の要求。

## 3. Kubernetes のアーキテクチャ（ざっくり）
- API Server: Kubernetes のフロントエンド。kubectl などからのリクエストを受け付ける。
- etcd: クラスタの状態を保持する分散 K-V ストア（重要）。
- Controller Manager: クラスタの状態を所望の状態に保つためのコントローラ群を含む。
- Scheduler: Pod を Node に割り当てる（リソース、NodeSelector、affinity/anti-affinity などを考慮）。
- kubelet: Node 上で動作し、Pod を起動して状態を報告する。
- kube-proxy: Service のネットワークルーティングを実現する。
## 4. 主要な kubectl コマンド（覚えるべきもの）
- kubectl get pods
- kubectl get svc
- kubectl get deployments
- kubectl create -f <file>.yaml
- kubectl apply -f <file>.yaml
- kubectl delete -f <file>.yaml
- kubectl describe <resource> <name>
- kubectl logs <pod> [-c <container>]
- kubectl exec -it <pod> -- /bin/sh
- kubectl port-forward <pod> <localPort>:<podPort>
- kubectl get events -n <namespace>

## 5. サンプルマニフェスト
以下は典型的な Deployment と Service の例。

Deployment: nginx アプリケーション
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        # 例: Liveness / Readiness probe
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        readinessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
```

Service (ClusterIP):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

ConfigMap の例
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  LOG_LEVEL: "info"
```

Secret の例（base64 された値を利用）
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

PVC の例（動的プロビジョニングを使うシンプルな例）
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

## 6. 在り来たりなパターンと実践的な注意点
- Rolling Update と戦略（Deployment の updateStrategy）。
- Pod Disruption Budget による高可用性の維持。
- リソース設定（requests, limits）を必ず設定してスケジューリングと QoS を管理。
- Liveness / Readiness Probes を設定して Pod の正常性を保つ。Readiness はサービスのトラフィック受け入れ可否に影響。
- RBAC（Role-Based Access Control）: ユーザー/サービスアカウントに最小限の権限を付与。
- Namespaces と ResourceQuota を使った分離と課金/リソース管理。
## 7. ネットワークと Service Discovery
- Pod 間通信は IP ベース（各 Pod は独自の IP）。
- Service は DNS 名で解決できる（例: <service>.<namespace>.svc.cluster.local）。
- CNI（Container Network Interface）プラグイン: Calico, Flannel, Weave, Cilium など。ネットワークポリシーは CNI に依存することが多い。
## 8. ストレージ
- PV/PVC と StorageClass によってストレージを動的にプロビジョニングできる。
- Block (raw volume) と File 系のストレージサポートがある。
- StatefulSet で永続性が求められるアプリ（DB、ステートフルなサービス）を管理する。
## 9. セキュリティ
- RBAC を有効にし、不要なクラスター権限を与えない。
- Network Policy を使って Pod 間通信を制限する。
- Secret は etcd に保存されるため、暗号化（encryption at rest）を検討する。
- イメージの署名・スキャニング（例: Trivy）を導入し、脆弱性管理を行う。
## 10. モニタリングとロギング
- Prometheus + Grafana でメトリクス収集と可視化。
- EFK/ELK スタック（Elasticsearch, Fluentd/Fluent Bit, Kibana）や Loki + Grafana でログ集約。
- kube-state-metrics や node-exporter なども重要。
## 11. クラスタの作成とクイック開始
- ローカル開発: Kind, Minikube, k3s
- クラウドプロバイダ: GKE, EKS, AKS などのマネージド Kubernetes。
- kustomize や Helm でアプリケーションのパッケージングと構成管理を行う。
## 12. トラブルシューティングのヒント
- Pod が Pending の場合: Node のリソース不足、欠落する PVC、または NodeSelector/affinity の制約をチェック。
- Pod が CrashLoopBackOff の場合: logs と describe を確認して原因を特定する。
- ネットワークの問題: NetworkPolicy、CNI の Pod ログ、kube-proxy の状態を確認。

## Ingress の簡単な例
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: example.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```
## 13. よく使うコマンドまとめ
- kubectl get all -n <namespace>
- kubectl apply -f <manifest.yaml>
- kubectl describe pod <pod> -n <namespace>
- kubectl logs -f <pod> -n <namespace>
- kubectl rollout status deployment/<deployment>
- kubectl port-forward svc/<service> 8080:80

Cheat sheet (よく使う追加コマンド)
- kubectl get nodes
- kubectl top pod|node        # requires metrics-server
- kubectl get configmap,secret
- kubectl explain <resource>  # 説明を表示
- kubectl diff -f <file>.yaml # 変更差分を確認
- kubectl apply -k ./dir     # kustomize

## 14. 参考（読むと良い公式ドキュメント／書籍）
- Kubernetes 公式ドキュメント: https://kubernetes.io/ja/docs/
- The Kubernetes Book（例）
- CNCF のトレーニングや各クラウドプロバイダのハンズオン

---
メモ: ここは基礎のまとめです。学習を進めるときは、実際にクラスタを作って hands-on で練習することをおすすめします。

## Try it — クイックスタート例
- 1) ローカルクラスタを起動: kind または minikube
  - kind: `kind create cluster --name local`
  - minikube: `minikube start`
- 2) マニフェストを保存して適用
```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```
- 3) Service をローカルで確認
```bash
kubectl get svc nginx-service
kubectl port-forward svc/nginx-service 8080:80
```
- 4) Pod のログ・状態確認
```bash
kubectl get pods -o wide
kubectl logs -f <pod-name>
kubectl describe pod <pod-name>
```
