---
marp: true
title: Docker 101
theme: default
paginate: true
style: |
  section.center {
    text-align: center;
  }
---

# Docker活用入門

Dockerの基礎を座学で学ぶと共に、基本的な操作をハンズオンで実践しましょう。

![bg right 50%](assets/images/vertical-logo-monochromatic.webp)

---

# 研修用 Azure Virtual Desktop (AVD) への接続

## AVD接続クライアント

### (1) Microsoft Store クライアント（ストアアプリ）

- https://apps.microsoft.com/store/detail/9WZDNCRFJ3PS
- ワークスペース接続先：
  https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery

### (2) Web クライアント（ブラウザ）

- https://client.wvd.microsoft.com/arm/webclient/index.html

---

## AVD接続情報

- 受講者番号：3桁数字、001～
- ユーザID：画面共有にて伝達
- パスワード：画面共有にて伝達

---

## AVD初期セットアップ

*注意）以前のアカウント情報が残っている場合は受講者番号が合っているか要確認*

1. AVDのクライアントと前頁のAVD接続情報を使用してログイン（数分かかる）
2. ログインできたら、Windowsのスタートメニューから「研修環境初期セットアップ」を起動（数分かかる）
3. 研修環境のWindowsを再起動
4. 本資料を研修環境のブラウザで開く（接続元と研修環境でクリップボードは共有できるのでURLはコピペ）

---

# 学習の流れ

1. Windowsローカル開発環境上にDockerを使える環境をセットアップする
2. Dockerの教材（外部資料）を見ながら基礎知識の習得と実践を行う
3. 典型的なDocker活用ケースの実践

※AVD初期セットアップの待ち時間の関係で、2.の座学を先に開始し、状況を見ながら1.を実施します

---

# 2. Dockerの教材（外部資料）を見ながら基礎知識の習得と実践を行う【教材A#1～28】

![bg left contain 50%](assets/images/vertical-logo-monochromatic.webp)

---

# Dockerの教材（外部資料）

GitHubで公開されている[Docker勉強会(ハンズオンセッション) @TakumaKurosawa](https://github.com/Takumaron/DockerHandsOn)を使わせてもらいます（教材A）

![width:500px](https://www.canva.com/design/DAED6dNQnj0/dtsCXpG-yqhdZT_sRvrsTw/screen)

---

# (教材A#28) 現場での使われ方補足①

クラウドネイティブ技術の台頭により、基盤技術となるコンテナ（Docker）やKubernetesの利用が増えている

refs. [クラウドネイティブとは？アプリケーション開発のアーキテクチャやコンテナについて](https://www.itmanage.co.jp/column/cloud-native/)

---

# (教材A#28) 現場での使われ方補足②

YouTube: [Container and Kubernetes 101](https://youtu.be/gFozhTXOx18)

![width:600px](https://i.ytimg.com/vi_webp/gFozhTXOx18/maxresdefault.webp)

---

# (教材A#28) 現場での使われ方補足③

金融のような分野でもクラウドネイティブ技術（Kubernetesなど）が採用されている

- [ジェーシービー（JCB）がKubernetesクラスターを導入](https://www.hpe.com/jp/ja/customer-case-studies/services-synergy-jcb.html)
- [Google Cloudに勘定系システム--みんなの銀行が利用状況など公開](https://japan.zdnet.com/article/35176674/)

---

# 1. Windowsローカル開発環境上にDockerを使える環境をセットアップする

![bg left contain 50%](assets/images/vertical-logo-monochromatic.webp)

---

# ローカル開発環境におけるDockerの選択肢

Docker DesktopというWindows向けアプリが提供されているが、2021年9月に有償化されたため、Windows Subsystem for Linux (WSL) 上でDocker Community Editionを実行するやり方が広く使われている。

※WSLについては[Linux活用入門](https://t-tsutsumi-scc.github.io/kensyu-slides/linux-101.html#8)でも紹介

---

# Docker環境のセットアップ（Windows）

1. スタートメニュー→`Ubuntu 22.04 LTS`を起動
   1. `Please create a default UNIX user account`というメッセージが表示されるまで待つ
   2. WSL上のUbuntuに作成するユーザ名とパスワードを聞かれるので、どちらも`user`と入力する（パスワードは2回入力）
   3. bashのプロンプト（`user@kensyu-avd-<連番>:~$`）が表示されたら`exit`または`Ctrl+d`を入力してウィンドウを閉じる
2. スタートメニュー→`ターミナル`を起動→PowerShellのプロンプトで以下のコマンドを実行→`変換が完了しました。`というメッセージが表示されるまで待つ
   ```powershell
   wsl --set-version Ubuntu-22.04 2
   ```
3. ターミナルアプリの新規タブで`Ubuntu 22.04 LTS`を開く（タブ部分の`∨`をクリックして`Ubuntu 22.04 LTS`を選択または`Ctrl+Shift+6`）

---

# Docker環境のセットアップ（WSL: Ubuntu）

## WSL環境の設定

```bash
# sudoをパスワードなしでできるように変更（パスワードを聞かれるので'user'と入力）
sudo sed -Ei 's/^(%sudo\s.*) ALL$/\1 NOPASSWD:ALL/' /etc/sudoers

# OSのタイムゾーンを日本のもの（UTC+9）に変更
sudo ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

# OSのパッケージインストール高速化のため、Azureのリポジトリを使用するように変更
sudo sed -i 's|/archive.ubuntu.com/|/azure.archive.ubuntu.com/|' /etc/apt/sources.list
```

---

## Dockerのインストール

Docker公式の[インストール手順](https://docs.docker.com/engine/install/ubuntu/)より抜粋

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
     | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" \
     | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

---

## ローカル開発環境用にDockerの初期設定を実施

```bash
# nftables（現時点でDocker非対応）の代わりにiptablesを使用するように変更
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy

# 現在のユーザでDockerの各種操作ができるように権限を付与（dockerグループに追加）
sudo usermod -aG docker $USER
exit
```

※exitコマンド後にWSLが終了するので再度ターミナルアプリでUbuntuを開きなおす

---

## Dockerの起動と確認

1. Dockerサービスを起動
   ```bash
   sudo service docker start
   ```
2. 以下のコマンドを実行し、Dockerのシステム情報を表示
   ※Dockerサービスが起動していない場合は`ERROR: Cannot connect to the Docker daemon...`が表示される
   ```bash
   docker info
   ```

---

## Dockerを便利に使うツール

WSL上でDockerを動かす場合、Docker Desktopにあるような便利なGUIツールが使用できないため、lazydockerのようなTUIツールを入れておくと便利です。

![width:700px](https://github.com/jesseduffield/lazydocker/raw/master/docs/resources/demo3.gif)

---

## lazydockerのインストール

1. lazydockerのダウンロード＆インストール
   ```bash
   curl -fsSL https://github.com/jesseduffield/lazydocker/releases/download/v0.18.1/lazydocker_0.18.1_Linux_x86_64.tar.gz \
        | tar -xzO lazydocker \
        | sudo tee /usr/local/bin/lazydocker > /dev/null \
        && sudo chmod +x /usr/local/bin/lazydocker
   ```

2. lazydockerを起動（ESCかqキーで終了）
   ```
   lazydocker
   ```

---

# 2. Dockerの教材（外部資料）を見ながら基礎知識の習得と実践を行う【教材A#29～41】

![bg left contain 50%](assets/images/vertical-logo-monochromatic.webp)

---

# (教材A#31) Docker概要：リポジトリの補足①

よく使用されるレジストリ

- クラウドサービス
  - Docker Hub
  - GitHub Container Registry (GHCR)
  - Red Hat Quay
  - Amazon Elastic Container Registry (ECR)、Azure Container Registry (ACR)、Google Container Registry (GCR)

- オンプレミス（社内利用）
  - GitLab Container Registry
  - Nexus Repository Manager

---

# (教材A#34) Docker概要：イメージの指定方法の補足

`<レジストリのホスト名>/<リポジトリ名>:<タグ>`

- レジストリのホスト名：`ghcr.io`など
  ※省略時はDocker Hub（`registry-1.docker.io`）が使用される
- リポジトリ名：任意のリポジトリ名で一般的には`ユーザアカウント名/リポジトリ名`のような形式
  ※Docker HubかつDocker公式イメージ（Docker Inc.が維持管理しているイメージ）の場合はユーザアカウント名の部分がない
- タグ：イメージを識別する任意のタグ名で一般的にはバージョン番号を使用
  ※省略時は暗黙的に`latest`が使用される

---

# (教材A#40) はじめてのDockerイメージの補足

1. ターミナルアプリで`Ubuntu 22.04 LTS`を開く
2. 教材のGitリポジトリをclone
   ```bash
   git clone https://github.com/Takumaron/DockerHandsOn.git
   cd 'DockerHandsOn/1_はじめてのDockerイメージ'
   ```
3. [README.md](https://github.com/Takumaron/DockerHandsOn/blob/master/1_%E3%81%AF%E3%81%98%E3%82%81%E3%81%A6%E3%81%AEDocker%E3%82%A4%E3%83%A1%E3%83%BC%E3%82%B8/README.md)を見ながら各種コマンドを実行してみる

---

# (教材A#41) DockerHubを使ってみようの補足

教材の手順はDocker Hubのアカウントが必要なため以下に変更

1. ターミナルアプリの別タブ/別ウィンドウでUbuntuを開き、lazydockerを実行（後ろの手順を実行する際に見ながら進める）

2. Docker Hubから色々なイメージをpullしてみる
   ```bash
   # Docker公式ubuntuイメージ（タグ：jammy）をpull
   docker pull ubuntu:jammy
   
   # Docker公式debianイメージ（タグ：bullseye-slim）をpull
   docker pull debian:bullseye-slim
   
   # Docker公式alpineイメージ（タグ：bullseye-slim）をpull
   docker pull alpine
   
   # pullしたイメージの一覧を表示（サイズを比較してみよう）
   docker images
   ```

---

3. Docker公式debianイメージを起動してみる
   ```bash
   docker run --rm -it debian:bullseye-slim
   
   # OS情報の確認（WSLのUbuntuではなくDebianになっていることが分かる）
   cat /etc/os-release
   
   # プロセス一覧の確認（コマンドが存在しないエラーとなる）
   ps aux
   
   # その他好きなコマンドを実行してexit
   exit
   ```

---

4. Docker公式alpineイメージを起動してみる
   ```bash
   docker run --rm -it alpine
   
   # OS情報の確認（WSLのUbuntuではなくAlpineになっていることが分かる）
   cat /etc/os-release
   
   # プロセス一覧の確認（シェルとpsコマンドのプロセスのみが表示される）
   ps aux
   
   # その他好きなコマンドを実行してexit
   exit
   ```

---

# 別の教材（教材B）を使用して教材Aに不足している基礎を押さえる

- [【図解】Dockerの全体像を理解する -前編-](https://qiita.com/etaroid/items/b1024c7d200a75b992fc)
- [【図解】Dockerの全体像を理解する -中編-](https://qiita.com/etaroid/items/88ec3a0e2d80d7cdf87a)（～⑤ Dockerネットワーク）

---

# 2. Dockerの教材（外部資料）を見ながら基礎知識の習得と実践を行う【教材A#42～】

![bg left contain 50%](assets/images/vertical-logo-monochromatic.webp)

---

# (教材A#47) はじめてのDocker-composeの補足

1. ターミナルアプリで`Ubuntu 22.04 LTS`を開く
2. 教材のGitリポジトリをcloneしたディレクトリに移動
   ```bash
   cd '~/DockerHandsOn/3_はじめてのDocker-compose'
   ```
3. [README.md](https://github.com/Takumaron/DockerHandsOn/blob/master/3_%E3%81%AF%E3%81%98%E3%82%81%E3%81%A6%E3%81%AEDocker-compose/README.md)を見ながら各種コマンドを実行してみる

---

# 別の教材（教材B）を使用して教材Aに不足している基礎を押さえる

- [【図解】Dockerの全体像を理解する -中編-](https://qiita.com/etaroid/items/88ec3a0e2d80d7cdf87a#-docker-compose)（⑥ Docker Compose）

---

# 3. 典型的なDocker活用ケースの実践

![bg left contain 50%](assets/images/vertical-logo-monochromatic.webp)

---

# Redmineを立ち上げよう

[Docker公式のRedmineイメージ](https://hub.docker.com/_/redmine)とDocker Composeを使用してローカル環境にプロジェクト管理ツール「Redmine」を立ち上げよう

- WindowsのWebブラウザから`http://localhost/`でRedmineにアクセスできるように構築し、チケットの作成が行えることを確認する
- Redmineのデータ（ファイル、DB）はDocker Volumeを使用して永続化すること（対象データは上記Docker HubのWebページの`Where to Store Data`を参照）

---

# Spring Boot（Java）で作成されたアプリケーションを実行しよう

ローカル環境でJava製のアプリ[Spring Boot Traditional Example](https://github.com/t-tsutsumi-scc/spring-boot-traditional-example)を実行しよう

- README.mdを参考にアプリケーションを立ち上げて動作確認を実施する
- Gitリポジトリ内の`Dockerfile`をもとにビルドしたDockerイメージが[GitHub Container Registry](https://github.com/t-tsutsumi-scc/spring-boot-traditional-example/pkgs/container/spring-boot-traditional-example)にデプロイされている
- Gitリポジトリ内に参考資料として`docker-compose.yml`がコミットされている
- コンテナでの実行となるため、README.md内の手順の「実行環境構築手順」は実施不要。アプリ初回起動手順の「(2) マスタデータを投入」から実施すればよい
- DBへのデータ投入は任意の手段で実行してよい（GUIツールとしてDBeaverがインストール済）
