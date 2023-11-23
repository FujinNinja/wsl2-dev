# WSL2検証環境

WSL2の検証環境です。   
Dockerやwikijs、wordpressなどの検証に使います。

# 環境構築
## Windowsの機能の有効化

以下の機能を有効化します。

- 仮想マシン プラットフォーム
- Linux 用 Windows サブシステム

**参考リンク**   
[WSLの有効化【埼玉大学 工学部 情報工学科 先端情報システム工学研究室】](https://www.aise.ics.saitama-u.ac.jp/~gotoh/HowToEnableWSL.html)   
[Windows 11 PC で仮想化を有効にする【Microsoft Docs】](https://support.microsoft.com/ja-jp/windows/windows-11-pc-%E3%81%A7%E4%BB%AE%E6%83%B3%E5%8C%96%E3%82%92%E6%9C%89%E5%8A%B9%E3%81%AB%E3%81%99%E3%82%8B-c5578302-6e43-4b4b-a449-8ced115f58e1)


## WSLインストール

Powershellから以下のコマンドでインストールできます。

```
wsl --install
```

インストール時、ユーザー名とパスワードが聞かれます。

```
Enter new UNIX username:
New password:
Retype new password:
```

面倒であれば以下にします。

- ユーザー名 ubuntu
- パスワード ubuntu

**参考リンク**   
[WSL を使用して Windows に Linux をインストールする方法【Microsoft Docs】](https://learn.microsoft.com/ja-jp/windows/wsl/install)   
[WSL2 のインストール，WSL2 上への Ubuntu のインストールと利用【金子邦彦研究室】](https://www.kkaneko.jp/tools/wsl/wsl2.html)

## Docker Desktopインストール

以下のリンクからインストーラーをダウンロードし、インストールします。   
[Install Docker Desktop on Windows](https://docs.docker.com/desktop/install/windows-install/)

システム要件やWSL2の有効化設定は以下に記載があります。   
[Windows に Docker Desktop をインストール](https://docs.docker.jp/docker-for-windows/install-windows-home.html)

インストールが完了したら、以下のコマンドが使用できます。

```
docker -v
```

WSL2でDocker Desktopを有効にするには
右上の歯車マーク設定 -> Resources
からubuntuのトグルスイッチをonにして「Apply & Restart」ボタンをクリックします。

## Git Clone

このライブラリをwsl上にクローンします。
windowsから見たらwsl2のパスは以下になります。
\\\wsl$\ubuntu\home\ubuntu

Git Cloneは方法がいろいろありますが、特にやり方が決まってない場合は、以下をインストールします。   
[git](https://git-scm.com/)   
[GitHub Desktop](https://desktop.github.com/)

## wsl2の操作

VisualStudioCodeから操作するのが楽です。
以下からインストールします。

[Visual Studio Code](https://azure.microsoft.com/ja-jp/products/visual-studio-code)

インストール後、拡張機能をインストールします。   
以下、入ってると良いと思われる拡張機能です。

[WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)   
[Japanese Language Pack for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-ja)   
[Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced)

## nvmのインストール

以下のコマンドをwsl2で実行します。このライブラリを使いたいがためにwslを使用してます。

```
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
```

インストール後、環境変数を通すために再起動します。
以下のコマンドでbashを終了させます。
```
exit
```

VisualStudioCodeならCtrl+@で再度立ち上がります。
これで再起動が完了です。

具体的には、wsl2の/ubuntu/homeにある.bashrcに以下の記載が追加されています。

```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

この設定がスクリプト実行時に追加されるっぽいので、再起動しないと多分うまくいかないです。

## nvmの使用
nvmはnodeのバージョンを自由に切り替えられるツールです。   
以下のコマンドで任意のバージョンをインストールできます。

### 最新版インストール
```
nvm install
```

### バージョン20インストール
```
nvm install 20
```

以下のコマンドでバージョンを切り替えられます。

### バージョン21に切り替え

```
nvm use 21
```

### バージョン20に切り替え

```
nvm use 20
```

# 動作確認
## Docker Composeの実行
Postgresqlの例で説明します。   
まずはdocker composeファイル（docker-compose.yml）の場所に移動します。   
```
cd docker
cd postgresql
```

次に以下のコマンドを実行します。
```
docker compose up -d
```

ただ、wsl2でdocker compose upコマンドをそのまま実行したところ以下のエラーが出てきました。   

```
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json?all=1&filters=%7B%22label%22%3A%7B%22com.docker.compose.config-hash%22%3Atrue%2C%22com.docker.compose.project%3Dpostgresql%22%3Atrue%7D%7D": dial unix /var/run/docker.sock: connect: permission denied
```

このエラーの回避策として以下のコマンドを実行します。

```
sudo chmod 666 /var/run/docker.sock
```

したら実行されます。   
実行したら、Docker DesktopのContainerにPostgreが追加され稼働してます。 
以下の情報でpostgreに接続できるはずです。   

ポート番号:5432   
DB名:postgres   
ユーザー名:postgres   
パスワード:passw0rd   

ホスト名だけはwsl側にipアドレスの設定が必要なので、それで確定します。

## wslのipアドレス確認

以下のコマンドでipアドレスが確認できます。   
eth0に書いてあるipアドレスがホスト名です。

```
ip a
```

現状ipアドレス固定にするいい方法が見つかりません。   
ただ、後述のwikijsで使用するconfig.ymlに関してはlocalhostで接続できます。   
それは、dockerとwikijsのサーバーが同じサーバーだからです。   

無理なのが、windowsにdbクライアントを用意したときの接続です。   
(もしかしたらできるかもしれませんが、現状はip aで確認したipアドレスじゃないと接続できません。)

## wikijsの動作確認

githubのソースコードではうまくいかないので、以下の手順を参考にします。   
https://docs.requarks.io/install/linux

このリポジトリではgithubフォルダ配下のwiki-jsフォルダにwiki-js.tar.gzをダウンロードして展開してます。   

Dockerでpostgresqlを立ち上げて、wiki-jsフォルダ配下のconfig.ymlのデータベース接続先設定をミスらなければ、立ち上がります。   

nodeはnvm use 20のものを使用しています。 

## nodejsのデバッグ
javascriptのデバッグをしたい場合は、VSCodeに以下の拡張機能をインストールします。   
[JavaScript Debugger (Nightly)](https://marketplace.visualstudio.com/items?itemName=ms-vscode.js-debug-nightly)   

この拡張機能はターミナルを新たに追加する機能です。   
このターミナル上から以下のコマンドを実行することで、デバッグできます。  

```
node server
```

デバッグする際は確認したい場所にf9でブレークポイントを貼ります。   