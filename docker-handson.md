# ハンズオン

このハンズオンは以下のバージョンでの動作を確認しています。

```
docker desktop version 2.1.0.5

$ docker version
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.12.12
 Git commit:        633a0ea
 Built:             Wed Nov 13 07:22:34 2019
 OS/Arch:           darwin/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.5
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.12
  Git commit:       633a0ea
  Built:            Wed Nov 13 07:29:19 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

## コマンドライン編

Dockerのコマンドライン操作に慣れていきましょう！

必要に応じ[公式リファレンス](https://docs.docker.com/engine/reference/commandline/docker/)を見ながら進めてみてください。

### docker run

```
$ docker run hello-world
```

hello-worldコンテナは先程スライドの中で動かしましたね！

他にも面白いイメージを2つご紹介します。

- `$ docker run lukaszlach/merry-christmas`
  - Ctl + Cで終了します。

- `$ docker run docker/whalesay cowsay Hello!`
  - `docker/whalesay`コンテナで、`cowsay hello!`コマンドを実行 の意


### docker run -it <CONTAINER_IMAGE> bash

```
$ docker run -it ubuntu bash
```

コンテナ内でbashを実行することができるコマンドです。


```
$ docker run --help

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
(抜粋)
  -i, --interactive                    Keep STDIN open even if not attached
  -t, --tty                            Allocate a pseudo-TTY
```

#### コンテナを自由に操作する

コンテナの中で好きなプログラムをインストールしたり、ファイルを作ったりしてみましょう。

**1. コンテナ内に自作のプログラムを配置**

今回は簡単なPythonプログラムを作成します。

通常ubuntuサーバー（コンテナではない）にはデフォルトでPythonがインストールされていますが、コンテナには入っていないのでPythonのインストールから行います。

（※ Dockerイメージは軽量化/セキュリティ等の観点から最小限の構成にしてあり、使いたいプログラムが入っていないことが多いです）

例のとおりでなくとも、思いつく人は他にもなんでも楽しいことしてみてください！

```
# apt-get update
# apt-get install python
# echo "print('Hello')" >/root/hello.py
# python /root/hello.py
Hello
```

ここまで出来たら、`Ctl+P, Ctl+Q`でコンテナを起動したままコンソールから抜けます。

`# exit`や`Ctl+D`で抜けるとコンテナが終了し、今設定したものが消えてしまうので注意！

**2. Dockerイメージを作る (Docker commit)**

Docker commit コマンドとは？

→ Dockerfileではなく、Dockerコンテナを元にDockerイメージを作り出すコマンドのことです。

```
# Ctl+P, Ctl+Qで抜けた後、docker psで先程のubuntuコンテナを確認
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
ce099a01d61a        ubuntu              "bash"              9 seconds ago       Up 7 seconds                            youthful_grothendieck

# docker <CONTAINER_ID/CONTAINER_NAME> <REPOSITORY_NAME>
$ docker commit ce099a01d61a my-test-container
# $ docker commit youthful_grothendieck my-test-container でもOK

sha256:18425e3b503ab1a6bb02a55d41d298ad750737786b3323dc058ddf64add0cfd8

$ docker images
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
my-test-container                     latest              18425e3b503a        3 minutes ago       127MB
```

my-test-containerという名前のオリジナルDockerイメージが出来ました！


**3. Docker commitしたDockerイメージから新しいコンテナを起動**

まず、commitした元のコンテナを終了しておきましょう。

```
# docker stop <CONTAINER_ID/CONTAINER_NAME>
$ docker stop ce099a01d61a
# $ docker stop youthful_grothendieck でもOK
```

それでは、先程作成したオリジナルDockerイメージからコンテナを作成してみましょう！

```
$ docker run -it my-test-container bash
# ls /root
hello.py
# exit

$ docker run my-test-container python /root/hello.py
Hello
```

オリジナルDockerイメージを元に、自作のプログラムを持ったコンテナが新しく立ち上がりました🎊

### docker container prune(docker v1.13~ 新コマンド)

`$ docker ps -a` でコンテナの一覧を表示すると、Exited状態のコンテナが溜まっていることが確認できると思います。

以下のコマンドで終了したコンテナを一括で削除することが可能です！

```
$ docker container prune
```

```
$ docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
b59ef95d640d19e46790724e1bf04cbf58018bd8bfb4e1a040705eb8088f61b4
(略)
```

削除された CONTAINER IDの一覧が出力されます。

実行前後の `$ docker ps -a` の結果を比較してみてください。

### docker run --rm <image>

` --rm` はコンテナの停止と同時にコンテナを消去できるオプションです。

```
# prune直後はコンテナ一覧はなにもない
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

$ docker run --rm hello-world
(出力略)

# --rm オプションによりコンテナの停止と同時にコンテナが削除されるため、hello-worldのコンテナが表示されない
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

### docker run -d -p 80:80 <webserver image>

コンテナをバックグラウンド実行し、コンテナの80番ポートで待ち受けているWebアプリをホストに公開します。

以下は、Nginxの公式イメージをバックグラウンド実行するコマンドです。

コンテナ起動中、 http://localhost:80/ でNginxに接続可能になります！

```
$ docker run --rm -d -p 80:80 nginx
```

```
$ docker run --help
  -d, --detach                         Run container in background and print container ID
  -p, --publish list                   Publish a container's port(s) to the host
```



```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
9029f63fff49        nginx               "nginx -g 'daemon of…"   35 seconds ago      Up 32 seconds       0.0.0.0:80->80/tcp   naughty_taussig
```

## docker stop (containerID or containerName)

バックグラウンドで起動中のコンテナを停止するコマンドです。

```
# Nginxのコンテナ(ID: 9029f63fff49)を停止
$ docker stop 9029f63fff49
```

---

## DockerFile編

ここからは、基礎的なDockerfileを書いてみましょう🙆‍♀️

### 基本構成

[公式リファレンス](https://docs.docker.com/engine/reference/builder/)の情報量が多いので、ここではよく使う命令を抜粋！

- FROM (image)
  - `FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]`
    - Dockerfileで唯一必須となる命令です！
    - 利用するベースイメージを指定します。
    - DockerHubから、イメージごとに存在するtag名を確認できます
      - このハンズオンでは困ったら最新を表す`latest`を使ってみましょう(本番運用はまた別)
      - `FROM ubuntu:latest` のように書きます。
- RUN
  - `RUN <command>`
  - `RUN ["executable", "param1", "param2"]`
    - ビルド時に実行する(コンテナ実行時には実行されない)コマンド。
      - `apt-get install xxx` など、コンテナを起動する前に実行しておきたいコマンドを指定することが多いです。
- CMD/ENTRYPOINT
  - `CMD ["executable","param1","param2"]`
  - `ENTRYPOINT ["executable", "param1", "param2"]`
    - いずれもコンテナ起動時に実行するコマンドを指定するもの。
    - `$ docker run ~` でコマンドを上書きするときの挙動に違いがあります
      - 今日のハンズオンでは迷ったら`CMD`を使っていただければと思います！
- ADD/COPY
  - `ADD [--chown=<user>:<group>] <src>... <dest>`
  - `COPY [--chown=<user>:<group>] <src>... <dest>`
    - コンテナ内にファイルを配置するコマンド
    - COPYと違い、ADDは以下のことができる
        - リモートからのファイル追加
        - 圧縮ファイルの自動解凍
    - 今日のハンズオンでは迷ったら `COPY`にしましょう！
- EXPOSE
  - `EXPOSE <port> [<port>/<protocol>...]`
  - コンテナがリッスンするポートを指定。
  - あくまでドキュメントの役割であり、実際に接続するためには、起動時に `-p <port>:<port>` のようなオプションが必要です。

### コンテナビルドの仕方

以降でDockerfileを書きますが、

その前にDockerfileのビルドの仕方と、コンテナとしての動かし方を書いておきます。

Dockerfileを書き終わったらこの手順を参照してビルドしてください！

```
# ディレクトリ内に1つDockerfileを作成します(bashの場合)
$ ls
Dockerfile

# Dockerfileを書き終わったら、ターミナルで以下コマンド実行。
# Dockerイメージが作成されます。
# $ docker build . -t <IMAGE_NAME>:<TAG>
$ docker build . -t mysamplecontainer:v1

# 作成したDockerイメージを元にコンテナを実行(必要に応じてオプションを付けてみてください)
$ docker run mysamplecontainer:v1
```

**1. my-test-containerをDockerfile化してみよう**
  - 先程ubuntuコンテナをベースに作ったmy-test-containerをbuildしてみましょう 💪💪💪
  - `$ docker run <自作IMAGE_NAME>:<TAG>` で自作スクリプトが動くよう、最後に`CMD`命令を使ってみてください！
  - サンプル: `/source/docker/sample01`

**2. NginxコンテナをDockerfile化してみよう**
  - `$ docker run -d -p 80:80 nginx` で `http://localhost:80/` に表示されるHTMLを書き換えてみましょう！
  - イメージは `nginx:latest` を使ってください。
  - 自分で`index.html`を作成し、`/usr/share/nginx/html`の下にコピーすればOKです！
  - サンプル: `/source/docker/sample02`

### (3) 公式チュートリアルのDockerfile
- https://docs.docker.com/get-started/part2/
  - 具体的なWebアプリを乗せたコンテナのDockerfile。
  - 勉強になるのでぜひ読んで動かしてみてください！

## クリーンアップ

コンテナが存在しっぱなし、不要イメージが溜まりっぱなしだと、不要なリソースを食います。

最後にお掃除しましょう！

(特にイメージは気づくと結構ローカルストレージを圧迫します😵)

### 起動中のすべてのコンテナの終了

起動中のすべてのコンテナを一括で終了するコマンドです。

```
# docker stop <CONTAINER ID>
$ docker stop $(docker ps -q)
```

### コンテナの削除

終了状態のコンテナをすべて削除します。

```
$ docker container prune
```

### イメージの一括削除

Docker イメージを削除します。

(消したくないイメージがある場合は注意してください)

```
$ docker image prune -a
```

- 注意!
  - Docker for Mac(Win)は、一度膨らんだ領域はイメージを削除するだけでは小さくなりません。
  - PCの動作が重くなってきたら、定期的にDocker for Mac(Win)のGUIから `preferense > Reset > reset disk image` するとよいです。


## おまけの宿題
- 抵抗のない方は、`my-test-container`をDockerhubに公開してみましょう!
  - 再配布NGのソフトウェアを置くのはNGですのでお気をつけてください！
