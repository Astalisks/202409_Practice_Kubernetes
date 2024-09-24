# ゴール
- wsl2上のubuntuにkubectlをインストールする



## kubectlのインストール手順解説
```
# 1. パッケージリストの更新と必要なパッケージのインストール
# 最新のソフトウェアリストを取得し、SSL証明書やcurlをインストールします。
sudo apt update
sudo apt install -y ca-certificates curl

# 2. snapdのインストール（もしインストールされていない場合）
# snapパッケージを扱うためのツールsnapdがインストールされていない場合に備えて、インストールします。
sudo apt install -y snapd

# 3. snapdサービスの有効化と開始
# snapdを有効化し、起動します。
sudo systemctl enable --now snapd

# 4. snapコマンドの最新バージョンを取得するための更新
# 最新のsnapパッケージを取得できるようにsnapdをアップデートします。
sudo snap install core
sudo snap refresh core

# 5. snapを使用してkubectlをインストール
# Snapパッケージマネージャーを使ってkubectlをクラシックモードでインストールします。
sudo snap install kubectl --classic

# 6. kubectlのインストール確認
# インストールが成功したかどうか、kubectlのバージョンを確認します。
kubectl version --client

※以下で動作を確認済み
$ kubectl version --client
Client Version: v1.31.1
Kustomize Version: v5.4.2
```


### 参考リンク
[Kubernetes](https://kubernetes.io/ja/docs/home/)