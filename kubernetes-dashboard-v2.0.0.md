# 概要

Docker for Mac で Kubernetes を試すため、サンプルとして[kubernetes/dashboard](https://github.com/kubernetes/dashboard) (**※**)のデプロイをしてみました。

**※dashboard**: k8sで管理しているリソースの状況をグラフや表で確認したり、podの管理やログの確認などをGUI操作で利用したりすることができます。



# 必要なものを確認

- Docker for Mac
    - Kubernetesのインストールとセットアップをすることができます。
- kubectl
    - API経由でKubenetesを操作するためのクライアントツールです。


```shell
$ docker -v
Docker version 19.03.4, build 9013bf5

$ kubectl version --short --client
Client Version: v1.14.7
```

# Docker for DesktopのKubernetesを有効化

DesktopのDockerアイコンを選択してPreferences⌘をクリックします。

<img width="242" alt="スクリーンショット 2019-12-20 16.26.23.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/7711aa59-1b03-cd15-9c15-783d401607d5.png">

右にあるKubernetesを選び `Show system containers` にチェックを入れて、Applyをクリックします。

![スクリーンショット 2019-12-23 13.37.11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/bbea5307-2e6b-e92d-1b67-1a2dc9c8e007.png)


# 起動確認&切り替え

コンテキストの一覧から、docker-for-desktopが存在しているのを確認します。

```shell
$ kubectl config get-contexts
CURRENT   NAME                 CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop       docker-desktop   docker-desktop   
          docker-for-desktop   docker-desktop   docker-desktop   
```

アクセス先をdocker-for-desktopに切り替えます。

```shell
$ kubectl config use-context docker-for-desktop
Switched to context "docker-for-desktop".
```


起動した段階で動作しているpodを確認します。

```shell
$ kubectl get po --all-namespaces
NAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
docker        compose-6c67d745f6-cv5zs                 1/1     Running   0          65m
docker        compose-api-57ff65b8c7-nl4dv             1/1     Running   0          65m
kube-system   coredns-584795fc57-92qm4                 1/1     Running   0          66m
kube-system   coredns-584795fc57-nvkc9                 1/1     Running   0          66m
kube-system   etcd-docker-desktop                      1/1     Running   0          65m
kube-system   kube-apiserver-docker-desktop            1/1     Running   0          65m
kube-system   kube-controller-manager-docker-desktop   1/1     Running   0          65m
kube-system   kube-proxy-bkjgc                         1/1     Running   0          66m
kube-system   kube-scheduler-docker-desktop            1/1     Running   0          65m
```

# kubernetes dashboardをデプロイ

環境が整ったので
試しにkubernetesクラスターをGUIで管理ができる kubernetes dashboard をデプロイしてみます。
GitHubにソースコードがあるのでREADMEの通りに進めます。
https://github.com/kubernetes/dashboard/blob/master/README.md
Podを作成します。

```shell
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

Namespaceに `kubernetes-dashboard` を指定してrunningになっているのを確認します。
※Kubernetes 2.0系から、ダッシュボードのNamespaceが `kube-system` から `kubernetes-dashboard` へ移動しました。

```shell
$ kubectl get deploy,po,svc -n kubernetes-dashboard
NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/dashboard-metrics-scraper   1/1     1            1           27m
deployment.extensions/kubernetes-dashboard        1/1     1            1           27m

NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-69fcc6d9df-dglbd   1/1     Running   0          27m
pod/kubernetes-dashboard-6d75768647-d69sk        1/1     Running   0          27m

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.100.193.116   <none>        8000/TCP   27m
service/kubernetes-dashboard        ClusterIP   10.109.69.115    <none>        443/TCP    27m
```

```shell
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

以下URLからダッシュボードにアクセスしてみます。
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
<img width="719" alt="スクリーンショット 2019-12-20 16.11.18.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/55b79686-9d53-aa0f-9f43-2f9280dbb81a.png">

# トークンの取得方法

Kubernetes 1.8系から認証システムが入っており、認証を通さなければWebUIを利用することができなくなりました。
ネームスペースにkubernetes-dashboardを指定して、`kubernetes-dashboard-token-` から始まるsecretを探します。

```shell
$ kubectl -n kubernetes-dashboard get secret
NAME                               TYPE                                  DATA   AGE
default-token-d69sj                kubernetes.io/service-account-token   3      32m
kubernetes-dashboard-certs         Opaque                                0      32m
kubernetes-dashboard-csrf          Opaque                                1      32m
kubernetes-dashboard-key-holder    Opaque                                2      32m
kubernetes-dashboard-token-lcnqw   kubernetes.io/service-account-token   3      32m
```

default-token権限でログインするためにtokenを取得します。

```shell
$ kubectl -n kubernetes-dashboard describe secret kubernetes-dashboard-token-lcnqw
Name:         kubernetes-dashboard-token-lcnqw
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard
              kubernetes.io/service-account.uid: c2536c20-2541-11ea-88b0-025000000001

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciO********************************************
~~~ 略 ~~~
```

上記の
token:
以降をコピペして
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/8f1dcd40-a6ee-afe2-3494-0bfde039b0cb.png)
サインインしてみます。
<img width="1438" alt="スクリーンショット 2019-12-23 14.41.07.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/474cdde8-ff8d-e1d3-b61e-989941b06333.png">

できました👏
