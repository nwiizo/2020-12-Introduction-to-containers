# Pre Setup
下記は自分の検証環境で入っているversionなので特にこだわりなく入れてもらって大丈夫だと思います。

* git
* docker 18.09.7
* kubectl 1.16.3
* Golang 1.14.1
* Kind v0.8.1


# Docker の検証
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
$ docker run -d -p 8000:80 --name nginx-latest docker.io/nginx:latest
$ wget 127.0.0.1:8000
```
## golang の[main.go](./main.go) を[DockerFile](./Dockerfile)でのデプロイ
こちら、[main.go](./main.go) になります。
```
package main

import (
  "fmt"
  "net/http"
  "os"
)

func main(){
  http.HandleFunc("/", handler)
  http.ListenAndServe(":8080", nil)
}

func handler(w http.ResponseWriter, r *http.Request){
  msg := os.Getenv("ENENENVVV")
  fmt.Fprintf(w, msg)
}
```
こちら、[DockerFile](./Dockerfile)になります。
```
FROM golang:latest AS builder

WORKDIR /work
COPY main.go .
RUN go build -o web .

FROM alpine

WORKDIR /exec
COPY --from=builder /work/web .
EXPOSE 8080
CMD ["./web"]
```
準備ができましたらbuildの実行をお願いします
``` sh
docker build -t webweb:v1 .

```
Load

kind load docker-image webweb:v1


# install [myapp](https://kind.sigs.k8s.io/docs/user/quick-start/#loading-an-image-into-your-cluster)

KindにはローカルのDockerイメージを読み込む機能が存在している。この章では自分で構築したアプリをコンテナ化して実際にデプ
