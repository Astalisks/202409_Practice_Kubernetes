# ゴール
- 04_NodeExporterとRBACを用いてPrometheusをセットアップする
  - helmを入れ使い方を理解する
  - RBACについてRBACロールをつくる


## kubernetes（kind, kubectl）の動作確認
```
$ sudo snap install helm --classic
helm 3.16.1 from Snapcrafters✪ installed
$ helm version
version.BuildInfo{Version:"v3.16.1", GitCommit:"5a5449dc42be07001fd5771d56429132984ab3ab", GitTreeState:"clean", GoVersion:"go1.22.7"}

$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories

$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "prometheus-community" chart repository
Update Complete. ⎈Happy Helming!⎈





```

## kubernetesの通信について
- kubernetesの通信は、クラスタ内のPod間、クラスタ外のPod間、クラスタ外のクラスタ内の通信がある
  - クラスタ内のPod間の通信は、PodのIPアドレスを使って通信する
  - ブラウザに入力してもアクセスできない
    - 例：10.96.80.101, 10.244.1.2:80など
  - クラスタ外のPod間の通信は、NodePortを使って通信する
    - 例：localhost:30001など
  - クラスタ外のクラスタ内の通信は、Ingressを使って通信する（今回は未使用）
    - 外部から複数のサービスに対して、一つのエントリーポイント（例: example.com）を設定したい場合。
    - HTTPSの設定やドメインごとのルーティングを行いたい場合。


### 参考リンク
[helm（公式）](https://helm.sh/ja/)
[helm（github）](https://github.com/helm/helm)


