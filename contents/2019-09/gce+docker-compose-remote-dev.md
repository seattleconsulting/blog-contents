# GCE + docker-composeでリモート開発環境を構築する

これを書いているのは2019年の9月ですが、今年はあまり暑くなくて過ごしやすい夏でしたね。

とはいえ残暑の方が厳しい可能性もあるので、できればローカルのMacbookなどでDockerを立ち上げることで余計な熱源は減らしたいですよね。
そこでGCE(Goodle Compute Engine)にインスタンスを立てて、VSCodeで繋いで、ローカルMacbookでの開発と遜色ない環境を構築する方法を紹介します。
（Windowsも基本は同じ手順でイケると思いますが、手元にないので）

Macbookが熱くならない以外にも、以下のメリットがあります。

* ブラウザとターミナルとエディタ、ネットワークさえあれば開発できる
* 強いマシンを買う必要がない=エコ
* ネットワークが速い（dockerイメージのpullやnode modulesのインストールなどで体感できる）

## 目次

以下の手順で設定します。

* 事前準備
* GCEインスタンス作成＆起動
* GCEインスタンス上で、個別プロジェクトのdocker-compose実行
* Visual Studio Code Insiders & Extensionsで開発

## 事前準備

GCPアカウントの作成・設定は、この辺りを参考に適宜お願いします。
<https://cloud.google.com/iam/docs/creating-managing-service-accounts?hl=ja>

アカウント作成後、適当なGCPプロジェクトを作成または選択してください。

## GCEでインスタンス作成＆起動

この後の手順は、基本的にはこのチュートリアルの通り。
<https://cloud.google.com/community/tutorials/docker-compose-on-container-optimized-os>

作成したプロジェクトでVMインスタンスを作成します。

--削除ここから--
今回はdocker入りコンテナを使うために、
ブートディスクを `Container-Optimized OS 76-12239.60.0 stable` に変更してください。
（自前でdocker, docker-compose入れる前提であれば、他のディストリビューションでも問題ないです）

Container-Optimized OS の機能と利点はこちら。
<https://cloud.google.com/container-optimized-os/docs/concepts/features-and-benefits?hl=ja>
--削除ここまで--

ブートディスクはDebian9のまま、
リージョン、インスタンスサイズはお好みで。最初のビルド時はn1-standardくらいが良いと思います。
あと、複数プロジェクトを同インスタンス内で使いたい場合は、storageが10GBだと足りなくなる可能性が高いので増やした方がよいやも。

## GCEインスタンス上で、個別プロジェクトのdocker-compose実行

--削除ここから--
次にdocker / docker-composeの設定です。
基本はこのページの通り。
<https://cloud.google.com/community/tutorials/docker-compose-on-container-optimized-os>

docker-composeを導入するために、以下を実行していきます。
dockerはすでにインストールされているので、docker-composeのエイリアスを作ります。
なぜかはわかりませんが、docker-composeをインストールするのではなく、dockerコマンドでやりくりするようです。

1. VMインスタンス一覧画面のSSHボタンでブラウザターミナルを開く
2. `docker run docker/compose:1.24.0 version`

3. ```docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v "$PWD:$PWD" \
    -w="$PWD" \
    docker/compose:1.24.0 up```

4. ```echo alias docker-compose="'"'docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v "$PWD:$PWD" \
    -w="$PWD" \
    docker/compose:1.24.0'"'" >> ~/.bashrc```

5. `source ~/.bashrc`
--削除ここまで--

### git, docker. docker-composeの設定

docker, docker-composeのインストールはこの辺を参考に。
<https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-debian-9>

* VMインスタンス一覧でSSHボタンクリックでブラウザターミナル起動

git

* sudo apt install git

docker

1. `sudo apt update`
2. `sudo apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common`
3. `curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -`
4. `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"`
5. `sudo apt update`
6. `apt-cache policy docker-ce`
7. `sudo apt install docker-ce`
8. `sudo systemctl status docker`
9. `docker --version`
10. `sudo usermod -aG docker ${USER}` # sudo なしでdockerコマンド実行できるようにgroupに追加
11. SSHログインし直す

docker-compose

1. ```sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose```
2. `sudo chmod +x /usr/local/bin/docker-compose`
3. `docker-compose --version`

### 個別プロジェクトのソースコードをGitHubからcloneした後、docker-compose upを実行

前準備として、このVMインスタンスからGitHubに接続できるように設定します。

1. ブラウザターミナルで `ssh-keygen` 実行。ファイル名はデフォルトのid_rsaでOK。パスフレーズを入れて忘れないようにする
2. GitHubのSettings > SSH and PGP Keysページに上記で作った公開鍵（~/.ssh/id_rsa.pubの中身）を登録

これでVMインスタンス上でcloneできるはずです。

1. `mkdir /home/<username>/projects` （ディレクトリ名は何でも可）
2. `cd projects`
3. `git clone <githubなどのclone先URL>`
4. `cd <cloneしたプロジェクト>`
5. `docker-compose up -d`

ここまでで、ポート80でリクエストを受けられるようなdocker-compose環境であれば、VMインスタンス一覧画面の外部IPリンクをクリックすればブラウザで実行結果を確認できます。
80以外のポートを開けたい場合は、以下を追加で実施してください。
SPAの開発環境であれば、2つくらいは開けたくなると思います。

1. GCPのVPCネットワーク > ファイヤーウォールルールをクリック
2. ファイヤーウォールを作成をクリック
3. 名前とターゲットタグに `http-8080` 、プロトコルとポートに許可したいポートを指定（例：tcp:8080,3000,3100）
4. VMインスタンスの編集画面で、ネットワークタグに `http-8080` を追加

これで `http://<外部IP>:<ポート>/` にブラウザで接続できると思います。
SPAなどの場合、通信先IPの指定も忘れずに。

## Visual Studio Code Insiders & Extensionsで開発

Visual Studio Code Insidersをインストールします。
2019年8月現在、Visual Studio Code（以下VSCode）正式版ではRemote SSHなどが動作しないので
Insiders版をローカルPCにインストールします。
<https://code.visualstudio.com/docs/?dv=osx&build=insiders>

### Remote Developmentをインストール

以下のExtension全部入りのExtensionであるRemote Developmentをインストールします。
VSCode Insiders版のExtensionタブで「Remote Development」で検索してください。

* Remote Containers
* Remote SSH
* Remote WSL

### VMインスタンス側の接続設定

* VMインスタンス編集画面で、SSH認証鍵欄にローカルMacの公開鍵を追加して保存
（またはVMインスタンス内のauthorized_keys に公開鍵を追加）

### ローカル側の接続設定

```yml
# ~/.ssh/config
Host <ホスト名>
  Hostname <立ち上げたVMインスタンスの外部IP>
  User <GCPユーザー名>
```

1. 上記のようにローカルMacの~/.ssh/config を編集
2. VSCode のRemote SSHタブ上に、上記で追加した<ホスト名>が出てくるのでダブルクリック
3. 接続完了

接続できない場合は、以下を疑ってみてください。

* ~/.ssh/config のユーザー名がGCPユーザー名ではない、または外部IPが違う
* ssh初回接続をVSCodeで実施してしまうと、ローカルMacのknown_hostsに追加するところで止まってしまうので、ターミナルなどで一度ssh接続してみる

## 余談

本当は、GCEのContainer-Optimized OS(COS)を使ってリモート環境を構築し始めたんですが、
VSCodeのRemote Serverインストールでコケたのでやめました。
Container-Optimized OS の機能と利点はこちら。
<https://cloud.google.com/container-optimized-os/docs/concepts/features-and-benefits?hl=ja>

また、完全にブラウザだけで開発できるようにCloud Shellのコードエディタで開発できるようにしようかと思いましたが、VMインスタンスからCloud Shell上にソースコードを同期する必要があるため、面倒なのでやめました。
素直にVSCodeでソースを書きましょう。

では、良いリモート開発ライフを。
