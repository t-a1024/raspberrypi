# ラズパイの動かし方
Raspberry PI Imager
Raspberry PIデバイス、OS、ストレージを以下の画像のとうりに選択する。
![初期設定](スクリーンショット%202024-12-26%2023.29.32.png)
次へ->設定を編集する
一般->Wi-Fiを設定する
サービス->SSHを有効化する->公開鍵認証のみを許可する(どっちでもいいかも)
オプション->特になし
保存->はい->はい(警告は無視でいい)

書き込みが始まる(結構時間かかる)

書き込みが終わったら、マイクロSDを取り出し、ラズパイに取り付け、起動する。(ケーブル刺したら自動で起動する)

ターミナルを起動し、以下のコマンドを実行
```bash
$ ping raspberrypi.local
```
結果
```
64 bytes from <IPアドレス>: icmp_seq=0 ttl=64 time=224.162 ms
```
ラズパイのIPアドレスがわかったら、以下のコマンドでssh接続
```bash
$ ssh pi@<IPアドレス>
```
のようなものがたくさん出るはず

Node-REDの起動
Node.jsとnpmをインストール
```bash
$ sudo apt update
$ sudo apt install nodejs npm
```
Node-REDのインストール
```bash
$ sudo npm install -g --unsafe-perm node-red
```
node-redの起動
```bash
$ node-red-pi
```
ブラウザで、下のページにアクセス
<IPアドレス>:1880
node-redのフロー1が見れるはず

右上のハンバーガーバー -> パレットの管理(option+shift+p)
->ノードの追加->node-red-node-pi-gpioを検索、追加

controle+Cで一旦停止

サービスファイルの作成
```bash
$ sudo nano /lib/systemd/system/nodered.service
```

以下の内容をコピー&ペースト
```makefile
[Unit]
Description=Node-RED graphical event wiring tool
After=network.target

[Service]
ExecStart=/usr/bin/env node-red-pi --max-old-space-size=256
Restart=on-failure
User=pi
Group=pi
Environment="NODE_OPTIONS=--max-old-space-size=256"
WorkingDirectory=/home/pi

[Install]
WantedBy=multi-user.target
```
Ctrl + X を押す
Y を押して保存
Enter を押して終了

サービスの再読み込みと有効化
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable nodered.service
$ sudo systemctl start nodered.service
```

これで、自動でnode-redが起動するようになったはず

gttsのインストール(エラーが発生するため、システム全体にインストールする)
```bash
pip3 install gtts --break-system-packages
```
mpg123のインストール
```bash
sudo apt install -y mpg123
```

```bash
nano speak.py
```
以下のコードをコピー&ペースト

```python
import sys
from gtts import gTTS
import os

# コマンドライン引数からテキストを取得
text = sys.argv[1]

# gTTSで音声ファイルを生成
tts = gTTS(text=text, lang='ja')
tts.save("/home/pi/output.mp3")

# 音声再生
os.system("mpg123 -a hw:1,0 output.mp3 /home/pi/output.mp3")
```

flow.jsonの中身をコピー

https://github.com/t-a1024/raspberrypi/blob/main/flow.json

ブラウザで、下のページにアクセス
<IPアドレス>:1880
⌘iで、flow.jsonの中身をペースト
デプロイ

これで、センサー同士がくっついている状態から離れたタイミングに天気予報がなるはずです。
動作確認ができたら接続を解除して大丈夫です。
```bash
$ exit
```
