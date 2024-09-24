# ゴール
- wsl2上のubuntuにkindをインストールする



## kindのインストール手順解説
```
# 1. kindのバイナリをダウンロード
★　'-L' はリダイレクトされたURLをたどるオプション
★　'-o' は出力ファイル名を指定するオプション
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.24.0/kind-linux-amd64

# 2. kindに実行権限を付与
chmod +x ./kind

# 3. kindをシステムのPATHディレクトリに移動
sudo mv ./kind /usr/local/bin/kind

# 4. kindのインストール確認
kind --version


※アンインストールする場合
$ kind --version
kind version 0.24.0


```


### 参考リンク
[kind](https://kind.sigs.k8s.io/)


