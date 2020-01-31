# Docker & Kubernetesハンズオン目次

## 事前準備

### Dockerのインストール

[こちらのページ](https://docs.docker.com/install/)を参考にしてください

よく使用されるOSに関してのみ以下に少し詳細を書いています

#### Docker Desktop on Mac/Windows

- [こちらのページ](https://docs.docker.com/docker-for-mac/install/)を参考にしてください
- Docker Hubのアカウントを作成する必要があります！
- `Download from Docker Hub`をクリックするとDocker Hubへのログインを促されるのでログインしてください
  - ログインしたらMacもしくはWindows用のインストーラをダウンロードできます

#### Ubuntu

- [こちらのページ](https://docs.docker.com/install/linux/docker-ce/ubuntu/)を参考にしてください
  - 上記ページの内容でインストールはできるのですが、dockerコマンドを利用するのに毎回`sudo`が必要になります
  - `sudo`をつけずにdockerコマンドを利用するには[こちらのページ](https://docs.docker.com/install/linux/linux-postinstall/)を参考にしてください

### Kubernetesのインストール

#### Docker Desktop on Mac/Windows

- メニュー＞Preference > Kubernetesタブ > Enable Kubernetes

#### kind

- https://kind.sigs.k8s.io
- 参考インストール日本語ブログ: https://blog.cybozu.io/entry/2019/07/03/170000
- Goが必要
- マルチノードクラスタを作成できる

#### minikube

- https://kubernetes.io/ja/docs/setup/learning-environment/minikube/
- 日本語ドキュメント

### インストールが難しい方

ブラウザでDocker/Kubernetesを利用することができます。下記にアクセスしてください。

ただし動作が非常に遅いので、可能な限りインストールしておくことをおすすめします。

- Play with Docker
  - Dockerのみ
  - 時間制限4時間
  - https://labs.play-with-docker.com
- kataconda
  - https://www.katacoda.com/courses/kubernetes/launch-single-node-cluster

## 資料
座学で使用する資料は以下です
適宜参照してください

- Docker
  - https://speakerdeck.com/pypypyo14/tatufalsekohaitu-dockerhandson
- Kubernetes
  - https://speakerdeck.com/aoi1/kuberneteshandson
