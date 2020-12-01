# Pre Setup
下記は自分の検証環境で入っているversionなので特にこだわりなく入れてもらって大丈夫だと思います。

* git
* docker 19.03.7
* kubectl 1.18.0
* Golang 1.15.2
* Kind v0.9.0


# Docker のハンズオン
[Docerの各種コマンド](https://docs.docker.com/engine/reference/commandline/docker/)は一度読んでおくとよいかもしれません。
``` sh
$ docker -v
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
```

## Nginxの公開
ブラウザで確認後 Docker の削除をお願いします。
``` sh
$ docker pull docker.io/nginx
$ docker run -d -p 8080:80 --name nginx-latest docker.io/nginx:latest
$ curl 127.0.0.1:8080
```
## golang の[main.go](./main.go) を[DockerFile](./Dockerfile)でのデプロイ
* こちら、[main.go](./main.go) になります。
* こちら、[DockerFile](./Dockerfile)になります。
* 準備ができましたらwebweb というイメージ名でv1 のタグをつけて buildの実行をお願いします。
``` sh
docker build -t webweb:v1 .
docker run -p 8080:8080 --name 2020-12-Introduction-to-containers webweb:v1
curl 127.0.0.1:8080
```

## 課題
* main.go という名前が気に入らなかったので [v2](./v2) にserver.go というファイルを生成しました。 Dockerfile の変更を行いwebweb というイメージ名でv2 のタグをつけて buildの実行をお願いします。

# Kubernetesのハンズオン
## [Kind](https://kind.sigs.k8s.io/) によるclusterの作成
``` sh
$ kind create cluster --name introduction-to-containers
# kubeconfig の書き込み
$ kind get kubeconfig --name introduction-to-containers > kubeconfig.yaml
# kubeconfing の確認
$ kubectl version  --kubeconfig=kubeconfig.yaml
```
## Nginx リソースの作成
``` sh
# deployment の作成
$ kubectl create deployment nginx-preview --image nginx:latest --kubeconfig=kubeconfig.yaml
deployment.apps/nginx-preview created
# deplyment の確認
$ kubectl get deployment nginx-preview --kubeconfig=kubeconfig.yaml
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
nginx-preview   1/1     1            1           69s
# accessの為のサービス作成
$ kubectl expose --type ClusterIP --port 80 deployment nginx-preview
service/nginx-preview exposed
# アクセスの為に`kubectl` にプロキシをしてもらいます
$ kubectl port-forward svc/nginx-preview 8080:80
# 確認
$ curl 127.0.0.1:8080
```
検証ではよく [Telepresence](https://www.telepresence.io/discussion/overview) を用いますが今回は利用するツールを最小化する為に排除

## webweb:v1 を[install](https://kind.sigs.k8s.io/docs/user/quick-start/#loading-an-image-into-your-cluster) して実行してみる
``` sh 
# Image の Load を行います
$ kind load docker-image webweb:v1  --name introduction-to-containers
# deployment の作成
$ kubectl create deployment webweb-preview --image webweb:v1 --kubeconfig=kubeconfig.yaml
# svc の作成
$ kubectl expose --type ClusterIP --port 8080 deployment webweb-preview --kubeconfig=kubeconfig.yaml
# アクセスの為に`kubectl` にプロキシをしてもらいます
$ kubectl kubectl port-forward svc/webweb-preview 8080:8080 --kubeconfig=kubeconfig.yaml
# 確認
$ curl 127.0.0.1:8080
```

## Manifest によるデプロイ
``` sh
# deployment の作成
$ kubectl apply -f deployment.yaml --kubeconfig=kubeconfig.yaml
# svc の作成
$ kubectl apply -f service.yaml --kubeconfig=kubeconfig.yaml
# アクセスの為に`kubectl` にプロキシをしてもらいます
$ kubectl kubectl port-forward svc/webweb-preview02 8080:8080 --kubeconfig=kubeconfig.yaml
# 確認
$ curl 127.0.0.1:8080
```

## 課題
* [webweb:v2](./v2) をclusterで実行してaccessしてみてください
