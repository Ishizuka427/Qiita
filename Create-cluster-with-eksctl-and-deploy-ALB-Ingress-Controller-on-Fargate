# EKS とは

AWS が提供しているマネージド型の Kubernetes サービスです。
この記事では、具体的な構成をもとに実際に触ってみることで理解を深めることを目標とします。

EKS で作成した　Kubernetesクラスターとは別に
Docker Desktop などで作成した Kubernetesクラスターを触りつつ進めると分かりやすかったです。
(何か Error があった際にも切り分けや原因究明が容易になります)
👆クラスターの切り替えは Docker アイコンからワンクリックで可能なので Docker Desktop がお薦めです。

# クラスターの作成方法

AWS EKS (Elastic Kubernetes Service) を利用して Kubernetes クラスターのセットアップ、デプロイまで行います。

開始方法は2通りあります。

- EKS コンソールからクラスター作成
- eksctl でクラスター作成

今回は VPC やセキュリティグループなど必要なリソースも併せて自動設定できる `eksctl` を用いてセットアップを行いたいと思います。
参考: [Getting Started with eksctl - Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)

また、内部で稼働しているコンテナ (EC2) マネジメントサービスである Fargate を利用するため、こちらの公式 Blog も参考にします。
参考: [Amazon EKS on AWS Fargate Now Generally Available | AWS News Blog](https://aws.amazon.com/jp/blogs/aws/amazon-eks-on-aws-fargate-now-generally-available/)

# 環境

Kubernetes 1.14
AWS EKS
AWS Fargate
Docker for mac
Mojavi 10.14.6

# バージョンについて

⚠`Kubernetes 1.14系`は脆弱性が報告されているバージョンです！
(わたしの検証時は未報告でした)
別バージョンでの検証をオススメします。
[CVE-2019-11247: API server allows access to custom resources via wrong scope · Issue #80983 · kubernetes/kubernetes](https://github.com/kubernetes/kubernetes/issues/80983)

**Vulnerable versions:**
- Kubernetes 1.7.x-1.12.x
- Kubernetes 1.13.0-1.13.8
- Kubernetes 1.14.0-1.14.4
- Kubernetes 1.15.0-1.15.1

**Fixed versions:**
- Fixed in v1.13.9
- Fixed in v1.14.5
- Fixed in v1.15.2
- Fixed in master

# 参照

[Amazon EKS での AWS Fargate の開始方法 - Amazon EKS](https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/fargate-getting-started.html
)


# EKS の下準備

## AWS CLI インストール

```console
$ brew install awscli --upgrade --user
```

`hostname doesn't match ` というエラーが出た場合は？
=> 確認: Python のバージョンは 2.7.9 以上か確認を

## IAMユーザー

1. 必要に応じて検証用のユーザーを作成
2. 使用する IAM ユーザーにてアクセスキーを作成する
3. AWS CLI に AWS 認証情報を設定する

### 必要に応じて検証用のユーザーを作成

IAMユーザーに権限がうまく振れていないとクラスター作成で失敗します。
検証段階では **小さく検証 ＆ 権限を大きめに振る** で試してみて、少しずつ権限を狭めていくと、効率よく進められるのでオススメです。

### 使用する IAM ユーザーにてアクセスキーを作成する

IAM のコンソール画面から、アクセスキーを作成します。

<img width="1139" alt="スクリーンショット 2020-02-26 10.52.16.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/b4feac8d-ab91-0b27-8c29-b357314d35ea.png">

`AWS Access Key ID`と`AWS Secret Access Key`が発行されるので保存する。

### AWS CLI に AWS 認証情報を設定する

`$ aws configure`を実行すると
AWS CLI によって`アクセスキー`、`シークレットアクセスキー`、`AWSリージョン`、`出力形式`、以上4つの情報の入力が求められる。これらは自動的に `default` という名前のプロファイル設定群に保存される。(既に設定してある場合は上書きされるので注意！)

- `~/.aws` 以下の config と credentials ファイルで確認可能。
- 手動修正するなどして、複数プロファイル管理も可能。
- 参考: [名前付きプロファイル - AWS Command Line Interface](
https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-profiles.html)

```console
$ aws configure
AWS Access Key ID [None]: 🆔＊＊＊＊＊
AWS Secret Access Key [None]: 🔑＊＊＊＊＊
Default region name [None]: ap-northeast-1
Default output format [None]: json
```

複数プロファイル設定などの詳しい方法は以下の記事にまとめてあります。
[AWS CLI設定のアレコレ](https://qiita.com/suwa3/items/ba730274b7228b7a1b71)

## eksctl インストール

AWS がサードパーティー的に保有している brew 形式のリポジトリをインストールします。

```console
$ brew tap weaveworks/tap
```

eksctl をインストールします。

```console
$ brew install weaveworks/tap/eksctl
```

インストールできたか確認。

```console
$ eksctl version
```

## クラスターを作成します。 

コマンド実行したら約15分待つ。

```console
$ eksctl create cluster --name demo-cluster --region ap-northeast-1 --fargate
```

内部で Cloud Formation が実行されているため、時間がかかります。
コーヒーでも飲みながら待ちます☕

## クラスターの確認

メニューバーにある Docker アイコンから切り替えることができます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/fe01ad53-3475-57f0-aaca-7e41c8ca5060.png)

## ノード一覧を表示

ノード一覧を表示させたところ
fargate が動いていました。

```console
$ kubectl get nodes
NAME                                                         STATUS   ROLES    AGE   VERSION
fargate-ip-***-***-***-**.ap-northeast-1.compute.internal    Ready    <none>   9d    v1.14.8-eks
fargate-ip-***-***-***-***.ap-northeast-1.compute.internal   Ready    <none>   9d    v1.14.8-eks
```

kube-system が動いていることも確認。

```console
$ kubectl get po --all-namespaces
NAMESPACE     NAME                      READY   STATUS    RESTARTS   AGE
kube-system   coredns-d784dc748-pkfqv   1/1     Running   0          9d
kube-system   coredns-d784dc748-t9pq5   1/1     Running   0          9d
```

# EKS 用のマニフェストファイル作成

マニフェストファイルは用途によってオブジェクトが何種類かあります。
シングルファイルにまとめても、マルチファイルとして別々にファイルを作成しても OK です。

- Deployment
- Service
- Ingress
- Secret
- ServiceAccount

などなど。。
今回は deployment.yml と service.yml を作成して検証を行います。


## 作業場所を作成

```console
$ mkdir kube
$ cd kube/
```

## 単一の nginx Deployment を作成

下記の内容で deployment.yml を作成 / 編集します。
nginx-deployment.yml を作成

```console
$ vi nginx-deployment.yml
```

```yml:nginx-deployment.yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.5
        ports:
        - containerPort: 80
```


```console
$ kubectl apply -f nginx-deployment.yml
deployment.extensions/nginx created
```

Pod 一覧を表示させます。

```console
$ kubectl get pod
NAME                    READY   STATUS    RESTARTS   AGE
nginx-954765466-zmj5p   0/1     Pending   0          9s
```

全ネームスペースの、Pod一覧を表示させます。

```console
// kubernets 1.13まで
$ kubectl get pods --all-namespaces
// kubernetes 1.14以降
$ kubectl get pod -A
```

全ネームスペースの、サービス一覧を表示させます。

```console
$ kubectl get svc --all-namespaces
NAMESPACE     NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
default       kubernetes   ClusterIP   10.100.0.1    <none>        443/TCP         9d
kube-system   kube-dns     ClusterIP   10.100.0.10   <none>        53/UDP,53/TCP   9d
```

## Service 作成

以下の内容で service.yml を　作成 / 編集します。
nginx-service.yml という yaml ファイルを作成。

```console
$ vi nginx-service.yml
```

```yml:nginx-service.yml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
  externalIPs:
     - 10.100.0.1
```

service を作成します。

```
$ kubectl apply -f nginx-service.yml
service/nginx created
```

追加されているかの確認。

```console
$ kubectl get svc --all-namespaces
NAMESPACE     NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
default       kubernetes   ClusterIP   10.100.0.1      <none>        443/TCP         9d
default       nginx        ClusterIP   10.100.77.190   10.100.0.1    80/TCP          10s
kube-system   kube-dns     ClusterIP   10.100.0.10     <none>        53/UDP,53/TCP   9d
```

## deployment.yml で設定した Pod を色々な角度で見てみる

### Podの詳細を表示

結構な情報量。。

```console
$ kubectl describe pod nginx
```

👆実行すると
下記のような詳細が表示されます。

```yml
Name:           nginx-954765466-zmj5p
Namespace:      default
Priority:       0
Node:           docker-desktop/192.168.65.3
Start Time:     Fri, 31 Jan 2020 20:33:22 +0900
Labels:         app=nginx
                pod-template-hash=5754944d6c
Annotations:    <none>
Status:         Running
IP:             10.1.0.141
Controlled By:  ReplicaSet/nginx-deployment-5754944d6c
Containers:
  nginx:
    Container ID:   docker://0ae17409e095d3a13c4bca879fc7095f92d0effd221b0053cf024acb809b358d
    Image:          nginx:1.7.9
    Image ID:       docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 23 Feb 2020 19:30:16 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-slqvm (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-slqvm:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-slqvm
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>

Name:           nginx-deployment-5754944d6c-9d6lf
Namespace:      default
Priority:       0
Node:           docker-desktop/192.168.65.3
Start Time:     Fri, 31 Jan 2020 20:33:22 +0900
Labels:         app=nginx
                pod-template-hash=5754944d6c
Annotations:    <none>
Status:         Running
IP:             10.1.0.140
Controlled By:  ReplicaSet/nginx-deployment-5754944d6c
Containers:
  nginx:
    Container ID:   docker://ac1c96d75b4e32ce7ab8eecd4e0ede68077e55bbf03c1a655d3a870b421d711c
    Image:          nginx:1.7.9
    Image ID:       docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 23 Feb 2020 19:30:16 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-slqvm (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-slqvm:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-slqvm
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>
```

### Pod のログを参照する

以下のコマンドで Pod のログを表示することが出来ます。

```console
$ kubectl logs <pod-name>
```

api-server のログを表示させてみました。

```console
$ kubectl logs kube-apiserver-docker-desktop -n kube-system
Flag --insecure-port has been deprecated, This flag will be removed in a future version.
I0223 10:30:03.769470       1 server.go:560] external host was not specified, using 192.168.65.3
I0223 10:30:03.772335       1 server.go:147] Version: v1.15.5
I0223 10:30:05.417959       1 plugins.go:158] Loaded 10 mutating admission controller(s) successfully in the following order: NamespaceLifecycle,LimitRanger,ServiceAccount,NodeRestriction,TaintNodesByCondition,Priority,DefaultTolerationSeconds,DefaultStorageClass,StorageObjectInUseProtection,MutatingAdmissionWebhook.
(〜略〜)
```

### 外からPodにアクセスしてみる (port foward編)

service.yml を書かなくても
起動できているかの確認であれば、 port foward で確認することができます。

Pod の一覧を表示。
こちらはレプリカ数 1 で起動しているので、ひとつだけ表示されます。

```console
$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
nginx-954765466-zmj5p   1/1     Running   0          5h59m
```

port foward でローカルから繋げられるようにします。

```console
$ kubectl port-forward nginx-954765466-zmj5p 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/cf47599c-6f6c-1b2b-7d2b-5a599e51abcb.png)


問題なく表示できました👏

#### 掃除方法

```console
$ kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           26d
```

deployment の削除。

```console
$ kubectl delete deployment nginx-deployment
deployment.extensions "nginx-deployment" deleted
```

# 疑問:thinking:: `kubectl apply` と `kubectl create` の違い

どちらもマニフェストファイルを指定して構築するさいに使うコマンド

- `kubectl apply -f  <マニフェストファイル>` 
- `kubectl create -f <マニフェストファイル>`

apply は URL 指定で、 create はカレントにある yml ファイルを指定するときに使っている。
具体的な違いを調べてみます。

[Kubernetes: kubectl apply の動作](https://qiita.com/tkusumi/items/0bf5417c865ef716b221)
>kubectl apply はオブジェクトが存在しなければ作成し、存在すれば差分を反映してくれる便利なコマンドです。(略)apply 以外の create, replace などのサブコマンドは、現在のオブジェクトの状態を意識して操作をする必要があります。

なるほど〜〜

サブコマンド | オブジェクトが存在しない | オブジェクトが存在する
--- | --- | ---
apply   | :white_check_mark: 新規作成 ( POST ) | :white_check_mark: 差分反映 ( PATCH, DELETE )
create  | :white_check_mark: 新規作成 ( POST ) | :warning:  エラー
replace | :warning: エラー   | :white_check_mark: 上書き ( PUT )
patch   | :warning: エラー   | :white_check_mark: 差分反映 ( PATCH )
delete  | :warning: エラー   | :white_check_mark: 削除 ( DELETE )

(上記URLから引用)

## 結論⭐: kubectl applyが優秀

# docker-desktop🐳マニフェストファイルを構築

メニューバーにある Docker アイコンから docker-desktop に切り替えます。

<img width="634" alt="スクリーンショット 2020-02-27 18.07.34.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/3b44fa48-6422-cee2-d963-d96db762e40c.png">

```
$ ls
sample-deployment.yml	sample-service.yml
```

## deployment.yml 

```
$ vi sample-deployment.yml 
```

```yml:sample-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  labels:
    app: sample-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - image: aoi1/tofu-sample-app:1.0
        name: sample-app
        ports:
        - containerPort: 3000
          name: sample-app
```

## service.yml 

```console
$ vi sample-service.yml 
```

```yml:sample-service.yml
apiVersion: v1
kind: Service
metadata:
  name: sample-app
spec:
  type: NodePort
  selector:
    app: sample-app
  ports:
    - name: "http-port"
      protocol: "TCP"
      port: 80
      targetPort: 3000
```

※`type:NodePort`でクラスタ外からアクセスが可能

## Service が適用されているか見てみる

`$ kubectl apply` で作成していきます。

```console
$ kubectl apply -f sample-deployment.yml
deployment.apps/sample-app created

$ kubectl apply -f sample-service.yml
service/sample-app created
```

Service が作成されているか確認します。

```console
$ kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        26d
sample-app   NodePort    10.109.160.47   <none>        80:32763/TCP   73s
```

ブラウザで Service の Port にアクセスしてみます。

http://127.0.0.1:32763

わーい、できました 👏

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/6ed3a657-b3ed-3432-dfc9-7378e8b72448.png)


Deployment だけでは、 port foward してあげないとブラウザからの確認ができませんでした。
Service を作成することで IP アドレスが固定され、ブラウザから確認することができます。

[Kubernetesの Service についてまとめてみた](https://qiita.com/kouares/items/94a073baed9dffe86ea0)


>**Service の役割**
PodはNodeに散らばっているため、それぞれのPodと通信しようとしたら、愚直にIPアドレスを指定して通信することになります。
また、K8sの特性上、NodeのIPアドレスは一定になりません。
突然Podと通信できなくなる可能性があります。
ということは、複数のPod,NodeのIPアドレスをクライアントが意識せずに単一のエンドポイントで通信する方法が必要になります。
上記の事象に対する解決策としてPod,Nodeの存在を抽象化し、Podとの通信に単一のエンドポイントを提供するのがServiceの主な役割になります。

# EKS 用マニフェストファイルを再構築

EKS クラスターに切り替えます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/c772c4f3-f5ec-8ddc-8d32-d0762709ad05.png)

EKS クラスターで docker-desktop 同様に yaml ファイルを作成して
ブラウザで Service の Port にアクセスしてみましたが、表示されませんでした。
ぐぬぬ

### なぜ表示されないのか

[Serviceの公開 (Serviceのタイプ)](https://kubernetes.io/ja/docs/concepts/services-networking/service/#publishing-services-service-types)

>ユーザーのアプリケーションのいくつかの部分において(例えば、frontendsなど)、ユーザーのクラスターの外部にあるIPアドレス上でServiceを公開したい場合があります。 (〜略〜)
LoadBalancer: クラウドプロバイダーのロードバランサーを使用して、Serviceを外部に公開します。

[Amazon EKS クラスターで実行されている Kubernetes Services を公開するにはどうすればよいですか ?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/eks-kubernetes-services-cluster/)

>注意 : Amazon EKS は、Load Balancer を通じて Amazon Elastic Compute Cloud (Amazon EC2) インスタンスワーカーノードで実行されるポッドに対して Network Load Balancer と Classic Load Balancer をサポートしています。Amazon EKS は、AWS Fargate で実行されるポッドの Network Load Balancer および Classic Load Balancer をサポートしていません。

EC2 上のワーカーノードにおいてはロードバランサーのサポートあるけれど
Fargate の場合はないらしいです。

>Fargate イングレスの場合、Amazon EKS で ALB Ingress Controller (最小バージョン 1.1.4) を使用するのがベストプラクティスです。

**Fargate を使う際には、ALB Ingress Controller を使いましょう**
とのことです！

ということで `LoadBalancer` を指定して、 Fargate 上で Service の公開ができなかったというのは、 **むしろ期待値である** ということが判明したので検証を続けます。
外部から表示させてみたいので、ベストプラクティスである ALB Ingress Controller の使用をしてみたいと思います。

# ALB Ingress Controller の検証

ここを参考に進めます。
[Amazon EKS の ALB Ingress Controller](https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/alb-ingress.html)

>AWS ALB Ingress Controller for Kubernetesは、kubernetes.io/ingress.class: alb 注釈を付けて Ingress リソースがクラスターで作成されるたびに、Application Load Balancer (ALB) および必要な AWS サポートリソースの作成をトリガーするコントローラーです。

GitHub にソース上がっていました。
https://github.com/kubernetes-sigs/aws-alb-ingress-controller

ガイドコーナーです。 Route53 を使うときは、このファイルを参考にします。
[Spec - AWS ALB Ingress Controller](https://kubernetes-sigs.github.io/aws-alb-ingress-controller/guide/ingress/spec/)

一度、単一 Service 構成で Ingress を検証してみます。
[Ingress - Kubernetes](https://kubernetes.io/ja/docs/concepts/services-networking/ingress/#%E5%8D%98%E4%B8%80service%E3%81%AEingress
)

```
$ ls
sample-deployment.yml	sample-ingress.yml	sample-service.yml
```

## deployment.yml 

```
$ vi sample-deployment.yml 
```

```yml:sample-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  labels:
    app: sample-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - image: aoi1/tofu-sample-app:1.0
        name: sample-app
        ports:
        - containerPort: 3000
          name: sample-app
```

## service.yml 

```
$ vi sample-service.yml
```

```yml:sample-service.yml
apiVersion: v1
kind: Service
metadata:
  name: sample-app
spec:
  type: NodePort
  selector:
    app: sample-app
  ports:
    - name: "http-port"
      protocol: "TCP"
      port: 80
      targetPort: 3000
```

## ingress.yml

```
$ vi sample-ingress.yml
```

```yml:sample-ingress.yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: sample-app
  annotations:
    kubernetes.io/ingress.class: alb
spec:
  backend:
    serviceName: sample-app
    servicePort: 80
```

`ingress` を作成。

```console
$ kubectl apply -f sample-ingress.yml
ingress.networking.k8s.io/sample-app created
```

```console
$ kubectl get ingress 
NAME         HOSTS   ADDRESS   PORTS   AGE
sample-app   *                 80      107s
```

## 準備

### サブネットと ALB の紐付け

使用する VPC のサブネットにタグを付けて、そのサブネットを使用できることを
ALB Ingress Controller に伝える必要があります。

※ ekctl を使用してクラスターをデプロイした場合は、タグがすでに適用されているため不要。

### IAM OIDC プロバイダーを作成し、クラスターに関連付け

```console
$ eksctl utils associate-iam-oidc-provider \
    --region ap-northeast-1 \
    --cluster demo-cluster \
    --approve
```

IAM OIDC プロバイダーってなんだろう
- [Kubernetes サービスアカウントに対するきめ細やかな IAM ロール割り当ての紹介 | Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/introducing-fine-grained-iam-roles-service-accounts/)
- [[アップデート] EKSでIAMロールを使ったPod単位のアクセス制御が可能になりました！ ｜ Developers.IO](https://dev.classmethod.jp/cloud/aws/eks-supports-iam-roles-for-service-accounts/)

なるほどなるほど~

```console
$ eksctl utils associate-iam-oidc-provider --region ap-northeast-1 --cluster demo-cluster --approve
[ℹ]  eksctl version 0.13.0
[ℹ]  using region ap-northeast-1
[ℹ]  will create IAM Open ID Connect provider for cluster "demo-cluster" in "ap-northeast-1"
[✔]  created IAM Open ID Connect provider for cluster "demo-cluster" in "ap-northeast-1"
```

IAM コンソールで確認。
できました👏

<img width="850" alt="スクリーンショット 2020-03-04 14.09.27.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/2acd5b6e-3eb6-01df-199b-799b9055c3a4.png">


## IAM ポリシーの作成

ユーザーに代わって AWS API を呼び出すことを
ALB Ingress Controller Pod に許可するための
ALBIngressControllerIAMPolicy という IAM ポリシーを作成。

```console
$ aws iam create-policy \
    --policy-name ALBIngressControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json
```

実行してみました。

```console
$ aws iam create-policy --policy-name ALBIngressControllerIAMPolicy --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json
{
    "Policy": {
        "PolicyName": "ALBIngressControllerIAMPolicy", 
        "PermissionsBoundaryUsageCount": 0, 
        "CreateDate": "2020-03-04T05:24:33Z", 
        "AttachmentCount": 0, 
        "IsAttachable": true, 
        "PolicyId": "ANPAWRYCWNNWLBILX6M2P", 
        "DefaultVersionId": "v1", 
        "Path": "/", 
        "Arn": "arn:aws:iam::*********:policy/ALBIngressControllerIAMPolicy", 
        "UpdateDate": "2020-03-04T05:24:33Z"
    }
}
```

- alb-ingress-controller という名前の Kubernetes サービスアカウントを kube-system 名前空間に作成。
- クラスターロールと ALB Ingress Controller で使用するクラスターロールバインディングを作成。

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml
clusterrole.rbac.authorization.k8s.io/alb-ingress-controller created
clusterrolebinding.rbac.authorization.k8s.io/alb-ingress-controller created
serviceaccount/alb-ingress-controller created
```

ALB Ingress Controller の IAM ロールを作成し
このロールを前のステップで作成したサービスアカウントにアタッチする。

以下のコマンドは、 eksctl で作成したクラスターでのみ使用できる。

```console
$ eksctl create iamserviceaccount \
    --region ap-northeast-1 \
    --name alb-ingress-controller \
    --namespace kube-system \
    --cluster demo-cluster \
    --attach-policy-arn arn:aws:iam::***********:policy/ALBIngressControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
```

実行した。アタッチできました。

```console
$ eksctl create iamserviceaccount --region ap-northeast-1 --name alb-ingress-controller --namespace kube-system --cluster demo-cluster --attach-policy-arn arn:aws:iam::***********:policy/ALBIngressControllerIAMPolicy --override-existing-serviceaccounts --approve
[ℹ]  eksctl version 0.13.0
[ℹ]  using region ap-northeast-1
[ℹ]  1 iamserviceaccount (kube-system/alb-ingress-controller) was included (based on the include/exclude rules)
[!]  metadata of serviceaccounts that exist in Kubernetes will be updated, as --override-existing-serviceaccounts was set
[ℹ]  1 task: { 2 sequential sub-tasks: { create IAM role for serviceaccount "kube-system/alb-ingress-controller", create serviceaccount "kube-system/alb-ingress-controller" } }
[ℹ]  building iamserviceaccount stack "eksctl-demo-cluster-addon-iamserviceaccount-kube-system-alb-ingress-controller"
[ℹ]  deploying stack "eksctl-demo-cluster-addon-iamserviceaccount-kube-system-alb-ingress-controller"
[ℹ]  serviceaccount "kube-system/alb-ingress-controller" already exists
[ℹ]  updated serviceaccount "kube-system/alb-ingress-controller"

```

## ALB Ingress Controller をデプロイ

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/alb-ingress-controller.yaml
deployment.apps/alb-ingress-controller created
```

ALB Ingress Controller のデプロイマニフェストを編集。
--ingress-class=alb 行の後にクラスター名の行を追加する。
Fargate で ALB Ingress Controller を実行している場合は、 VPC ID の行と、クラスターの AWS リージョン名も追加する必要がある。
一部抜粋

```console
$ kubectl edit deployment.apps/alb-ingress-controller -n kube-system

    spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=demo-cluster
        - --aws-vpc-id=vpc-***********
        - --aws-region=ap-northeast-1
```

全部

```console
$ kubectl edit deployment.apps/alb-ingress-controller -n kube-system
```

```yml
  selfLink: /apis/apps/v1/namespaces/kube-system/deployments/alb-ingress-controller
  uid: ***********
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/name: alb-ingress-controller
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app.kubernetes.io/name: alb-ingress-controller
    spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=demo-cluster
        - --aws-vpc-id=vpc-***********
        - --aws-region=ap-northeast-1
        image: docker.io/amazon/aws-alb-ingress-controller:v1.1.4
        imagePullPolicy: IfNotPresent
        name: alb-ingress-controller
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: alb-ingress-controller
      serviceAccountName: alb-ingress-controller
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2020-03-04T07:57:41Z"
    lastUpdateTime: "2020-03-04T07:57:41Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2020-03-04T07:51:13Z"
    lastUpdateTime: "2020-03-04T07:57:41Z"
    message: ReplicaSet "alb-ingress-controller-***********" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 2
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
```


ALB Ingress Controller が実行されていることを確認。
良さそう。

```console
$ kubectl get pods -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
alb-ingress-controller-***********   1/1     Running   0          97s
coredns-d784dc748-fmkzx                   1/1     Running   0          23h
coredns-d784dc748-r28sm                   1/1     Running   0          23h
```

## サンプルアプリケーションをデプロイ

サンプルアプリケーションの名前空間を含む Fargate プロファイルを作成する。

```console
$ eksctl create fargateprofile --cluster demo-cluster --region ap-northeast-1 --name alb-sample-app --namespace 2048-game
[ℹ]  creating Fargate profile "alb-sample-app" on EKS cluster "demo-cluster"
[ℹ]  created Fargate profile "alb-sample-app" on EKS cluster "demo-cluster"
```

できました🙌

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/4216e328-5687-e675-89e0-a3d84fe4960c.png)


マニフェストファイルをダウンロードして適用し
- Kubernetes の名前空間
- Deployment
- Serviceを作成

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-namespace.yaml
namespace/2048-game created

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-deployment.yaml
deployment.apps/2048-deployment created

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-service.yaml
service/service-2048 created
```

入力マニフェストファイルをダウンロード

```console
$ curl -o 2048-ingress.yaml https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-ingress.yaml
```

`2048-ingress.yaml` ファイルを編集。
既存の `alb.ingress.kubernetes.io/scheme: internet-facing` 行の下に
行 `alb.ingress.kubernetes.io/target-type: ip` を追加する。

```console
$ vi 2048-ingress.yaml
```

```yml:2048-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: "2048-ingress"
  namespace: "2048-game"
  annotations:
    kubernetes.io/ingress.class: alb 
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
  labels:
    app: 2048-ingress
spec:
  rules:
    - http:
        paths:
          - path: /*
            backend:
              serviceName: "service-2048"
              servicePort: 80
```

入力したマニフェストファイルを適用

```console
$ kubectl apply -f 2048-ingress.yaml
ingress.extensions/2048-ingress created
```

作成に数分かかるのでコーヒーを飲みながら待ちます。
Ingress リソースが作成されたことを確認する。

```console
$ kubectl get ingress/2048-ingress -n 2048-game
NAME           HOSTS   ADDRESS                                                                       PORTS   AGE
2048-ingress   *       cc1babba-2048game-2048ingr-*************.ap-northeast-1.elb.amazonaws.com   80      5m50s
```

指定された ADDRESS にアクセスする。
http://cc1babba-2048game-2048ingr-6fa0-*********.ap-northeast-1.elb.amazonaws.com/

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/34aa2468-f131-86ab-8ec0-af095272ff07.png)

できました👏

