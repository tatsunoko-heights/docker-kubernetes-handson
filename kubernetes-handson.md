# ハンズオン

このハンズオンは以下のバージョンでの動作を確認しています。
```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.8", GitCommit:"211047e9a1922595eaa3a1127ed365e9299a6c23", GitTreeState:"clean", BuildDate:"2019-10-15T12:11:03Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.8", GitCommit:"211047e9a1922595eaa3a1127ed365e9299a6c23", GitTreeState:"clean", BuildDate:"2019-10-15T12:02:12Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}

Mac OS High Sierra
docker desktop version 2.1.0.5
```

また、他のバージョンによる組み合わせで不具合が出たので、同様の事象がある方は以下を参考にしてください
https://gist.github.com/pypypyo14/812210ea1acfdeb718ed5d4ba7e223d2

## 準備

### kubectlがインストールできていることを確認する

```
$ kubectl version
```

### clusterを作成する

```
# kind
$ kind create cluster
# minikube
$ minikube start
# dockerの場合UIでON
```

clusterを作成することができれば以下で確認

```
$ kubectl get componentstatuses
```

出力結果は以下のような結果が見えるはず

```
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

## kubectl編

### Kubernetesのリソースを確認する

ここではKubernetesのリソースを確認しながらkubectlのコマンドに慣れていきます。

#### Nodeの情報を取得する

```
$ kubectl get nodes
NAME             STATUS   ROLES    AGE   VERSION
docker-desktop   Ready    master   35d   v1.14.8
```

アーキテクチャーで説明したように通常KubernetesはMaster NodeとWorker Nodeでわかれますが、
この例ではdocker-desktopを利用しているためNodeは一つになります。

より詳細にNodeの情報を取得するためには以下のコマンドを打ちます。

```
$ kubectl describe node docker-desktop
```

以下のような結果が得られるはずです。

```
Name:               docker-desktop
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=docker-desktop
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
                    CreationTimestamp:  Sun, 08 Dec 2019 22:26:45 +0900
(以下略)
```

#### Kubernetes のコンポーネントを確認する

```
$ kubernetes get pod -A
NAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
docker        compose-6c67d745f6-sxfmn                 1/1     Running   2          35d
docker        compose-api-57ff65b8c7-krtb8             1/1     Running   2          35d
kube-system   coredns-6dcc67dcbc-mx8rj                 1/1     Running   2          35d
kube-system   coredns-6dcc67dcbc-wq4t5                 1/1     Running   2          35d
kube-system   etcd-docker-desktop                      1/1     Running   2          35d
kube-system   kube-apiserver-docker-desktop            1/1     Running   2          35d
kube-system   kube-controller-manager-docker-desktop   1/1     Running   2          35d
kube-system   kube-proxy-s46x7                         1/1     Running   2          35d
kube-system   kube-scheduler-docker-desktop            1/1     Running   2          35d
```

各コンポーネントについてはアーキテクチャーのスライドを参照にしてください！
最初のうちは一つ一つ正確に意味を理解できていなくても大丈夫ですよ。

理解できていなくても動作させることができるという手軽さもKubernetesのとっつきやすさでもあります！

### Podを操作する

ここでは代表的なリソースであるPodを一例に、よく使うkubectlのコマンドを利用してみます。

#### Deploymentの作成

```
# kubectl apply -f <manifest file>
$ kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

- `-f`でファイルを指定する
- `kubectl create`でリソースを作成することもできますが、`Infrastructure as Code`の理念としてファイルを使用するほうが望ましいです
- ファイルの中身は[Run a Stateless Appliction Using a Deployment](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)を参照

#### Podをリストする

```
# kubectl get <リソース名>
$ kubectl get pods
```

- Deploymentを作成したらからといって即座にPodができるわけではありません
- Podができあるところをみたい場合以下のコマンドでPodが作成される様子が見えるのでみてみてください

```
$ kubectl get pods -w
```

Ctrl + cで抜けます。

#### Podの詳細を表示する

```
# kubectl describe <リソース名> <オブジェクト名>
$ kubectl describe pod nginx-deployment
```

- podの詳細を見ることができます
- 最後の`Events`はエラーが起きたときなど確認することがよくあります

#### Pod内でbashを操作する

```
# kubectl exec -it <pod-name> bash
# pod名はkubectl get podsでリストして取得してください
$ kubectl exec -it nginx-deployment-XXXX bash
```

- docker同様Pod内でbashを操作することが可能
- 色々なコマンドを打ってみましょう！使いたいコマンドが入っていないことも多いです。例えばvimとか...!!!

#### Podのlogを参照する

```
# kubectl logs <pod-name>
$ kubectl logs kube-apiserver-docker-desktop -n kube-system
```

- pod-nameはdocker-desktopではない場合、この名前ではないと思います
  - `kubectl get pods -A`でapi-serverのPod名を調べてみましょう！
- `-n` でnamespaceを指定しています

#### 外からPodにアクセスする

```
$ kubectl port-forward <pod-name> 8080:80
```

- `localhost:8080`にアクセスする
- `Welcome to nginx!`と表示されて入れば成功！

#### Podの削除

```
# kubectl delete pod <podname>
```

- `kubectl get pods`で消えていく様子が伺える
- DeploymentでPodの数を指定しているので、Podを消してもPodは復活する！(Reconcile)

## アプリをデプロイし、画面におとうふくんの絵を表示させよう！

ここからは考えながら手を動かしてみましょう！

`Deploymentの作成`で利用した[yamlファイル](https://k8s.io/examples/application/deployment.yaml)を利用し、
要件を満たすアプリがデプロイできるよう修正していきましょう。

わからなければ`source/kubernetes/deployment.yaml`に動作するyamlがあるので、まずはそれをそのまま利用しても良いです。

### アプリをデプロイする

以下の要件を満たすアプリをデプロイしてみましょう！！

- podの数は2
- Deploymentを使用する
- containerPortは3000
- image名はaoi1/tofu-sample-app:1.0
- コンテナ名などは自由

できたら以下が出来上がっているか確認しましょう!

```
$ kubectl get deployments
# 作成したdeploymentsがREADY 2/2 となっている
$ kubectl get pods
# 作成したpodが2個、STATUSがREADYになっている
```

アプリが正常に動作していることは次のステップで確認します

### アプリが正常に起動していることを確認する

port-forwardを利用してアプリが起動していることを確認しましょう！

起動して入れば画面にイラストが表示されます

- deploymentに対してport-forwardする場合は`kubectl port-forward deployment/<deployment name> XXXX:XXXX`で実行可能です
- targetPortは3000番

### (応用編) Serviceを利用してクラスタの外からアプリの動作を確認する

Serviceリソースを作成してクラスタの外からアプリにアクセスできるようにしましょう。

`source/kubernetes/service.yaml`をそのまま利用してください。

```
apiVersion: v1
kind: Service
metadata:
  name: tofu-sample-app
spec:
  type: NodePort
  selector:
    app: tofu-sample-app
  ports:
    - name: "http-port"
      protocol: "TCP"
      port: 80
      targetPort: 3000
```

- `type:NodePort`でクラスタ外からアクセス可能
- `kubectl apply -f <servicename>.yaml`でServiceを作成する

以下のコマンドでServiceが作成できていることを確認しましょう

```
$ kubectl get service
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP        41d
tofu-sample-app   NodePort    10.105.21.227   <none>        80:32697/TCP   3s
```

作成できていればブラウザでServiceのPortに記載されているポートにアクセスしましょう！

```
localhost:<port number(上記例だと32697)>
```

イラストが表示されていれば成功です！

# クリーンナップ

作ったものは消してしまいましょう！！

残しておきたい場合はこの章を行う必要はありませんが、余計なPCリソースを食うので気をつけてくださいね。

## ファイルを使用してリソースを消す方法

以下のコマンドでリソース作成に使用したファイルを使用してリソースを消すことができます

Podの削除で一度出てきていますね

```
$ kubectl delete -f <ファイル名>
```

## コマンドでリソースを指定して消す方法

以下の方法でリソースを指定して消すこともできます

```
$ kubectl delete <リソース> <リソース名>
# 例
$ kubectl delete deployment deploymentname
$ kubectl delete service servicename
```

- Podの削除のところにも書きましたが、Podだけ消してもDeploymentの指定によってPodは復活します！
