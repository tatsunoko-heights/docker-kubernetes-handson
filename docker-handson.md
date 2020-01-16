# ハンズオン

## コマンドライン編

- Dockerのコマンドライン操作

### docker run

`$ docker run hello-world`

- 他にも面白いイメージをいくつかご紹介
  - `$ docker run lukaszlach/merry-christmas`
    - Ctl + Cで終了
  - `$ docker run docker/whalesay cowsay hello!`
    - `docker/whalesay`コンテナで、`cowsay hello!`コマンドを実行 の意

### docker run -it <image> bash

- コンテナでbashを実行
- `$ docker run -it ubuntu bash`

```
$ docker run --help

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
(抜粋)
  -i, --interactive                    Keep STDIN open even if not attached
  -t, --tty                            Allocate a pseudo-TTY
```

#### やってみよう

- コンテナの中で好きなプログラムをインストールしたり、ファイルを作ったりしてみましょう。


**1. コンテナ内で好きなプログラムを入れてファイルを作る**

- 配布されているDockerイメージは軽量化/セキュリティ等の観点から最小限の構成にしてあり、欲しいプログラムが入っていないことが多いです。
- 今回はPythonをインストールし、簡単なPythonプログラムを作成します。
- 思いつく人は他にも楽しいことしてみてください！

```
# apt-get update
# apt-get install python
# echo "print('Hello')" >/root/hello.py
# python /root/hello.py
Hello
```

**2. Docker commit**

- Docker commitとは?
  - Dockerfileではなく、Dockerコンテナを元にDockerイメージを作り出すコマンド

- 一度`Ctl+P, Ctl+Q`でコンテナを起動したまま抜けます
  - `# exit`や`Ctl+D`で抜けるとコンテナが終了し、インストールしたものが消えてしまうので注意！

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
ce099a01d61a        ubuntu              "bash"              9 seconds ago       Up 7 seconds                            youthful_grothendieck

$ docker commit ce099a01d61a my-test-container
($ docker commit youthful_grothendieck my-test-container でもOK)
sha256:18425e3b503ab1a6bb02a55d41d298ad750737786b3323dc058ddf64add0cfd8

$ docker image ls
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
my-test-container                     latest              18425e3b503a        3 minutes ago       127MB
```
  - my-test-containerという名前のDockerイメージが出来ました

**3. Docker commitしたDockerimageから新しいコンテナを起動**

- さっき作ったイメージの内容を保持した新しいコンテナが出来ました

```
$ docker run -it my-test-container bash
# ls /root
hello.py

$ docker run my-test-container python /root/hello.py
Hello
```

### docker container prune

- `$ docker container ls -a` でコンテナの一覧を表示
- Exited状態のコンテナが溜まっている
- 以下のコマンドで停止中のコンテナを一括で削除することが可能
  - `$ docker container prune`
- 実行後、再度コンテナの一覧を表示してみてください

### docker run --rm <image>

- `$ docker run --rm hello-world`
- コンテナの停止と同時にコンテナを消去できるオプションです。
- `$ docker container ls -a` で、hello-worldがないことを確認できます

### docker run -d -p 80:80 <webserver image>

```
$ docker run --help
  -d, --detach                         Run container in background and print container ID
  -p, --publish list                   Publish a container's port(s) to the host
```

- コンテナをバックグラウンド実行
- コンテナのポートをホストに公開
- 例: `$ docker run --rm -d -p 80:80 nginx`
  - nginxの公式イメージを利用
  - バックグラウンド実行
  - コンテナの80番ポートをホストの80番ポートに転送
    - 起動後は http://localhost:80/ でnginxに接続可能

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
9029f63fff49        nginx               "nginx -g 'daemon of…"   35 seconds ago      Up 32 seconds       0.0.0.0:80->80/tcp   naughty_taussig
```

## docker stop (containerID or name)

- バックグラウンドで起動中のコンテナを停止
- `$ docker stop 9029f63fff49`

---

## DockerFile編

- 基礎的なDockerfile

### 基本構成
- https://docs.docker.com/engine/reference/builder/

- FROM (image)
  - `FROM [--platform=<platform>] <image> [AS <name>]`
    - 利用するベースイメージを指定
- RUN
  - `RUN <command>`
  - `RUN ["executable", "param1", "param2"]`
  - ビルド時に実行する(コンテナ実行時には実行されない)コマンド。
    - `apt-get install xxx` などを指定することが多い
- CMD/ENTRYPOINT
- ADD/COPY
  - <font color="red">ちがいについて今週勉強して埋めます！！（宿題）</font>
- EXPOSE
  - `EXPOSE <port> [<port>/<protocol>...]`
  - コンテナがリッスンするポートを指定。
  - ドキュメントの役割であり、実際に接続するためには、起動時に `-p <port>:<port>` のようなオプションが必要。

### my-test-containerをDockerfile化してみよう
  - さっきubuntuコンテナをベースに作ったmy-test-containerをbuildしてみましょう 💪💪💪
  - サンプル: `/source/docker/sample01`

### nginxコンテナをDockerfile化してみよう
  - `$ docker run -p 80:80 nginx` をDockerfileで表し、build、実行してみましょう！
  - イメージは `nginx:latest` を使ってください
  - nginxの開始コマンドは `$ nginx -g daemon off;`です
  - サンプル: `/source/docker/sample02`

### 公式チュートリアルのDockerfile
- https://docs.docker.com/get-started/part2/
  - 具体的なWebアプリを乗せたコンテナのDockerfile。勉強になるのでぜひ読んでみてください！

## 最後に
- 抵抗のない方は、自分のDockerイメージをDockerhubに公開してみましょう!
  - 再配布NGのソフトウェアを置くのはNG
  - <font color="red">(宿題)ライセンス的に安全な例を示す</font>