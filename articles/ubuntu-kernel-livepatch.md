---
title: "livepatchとは"
emoji: "🔨"
type: "tech"
topics: ["ubuntu"]
published: true
---

# Livepatchを導入してみた

## Livepatchとは

Livepatchはその名の通り、Ubuntu(Linux系のos)で`apt-get upgrade`された時に、kernelパッチを更新するためにーサーバーを再起動せずに適用することはできます。

Ubuntu Pro向けなのですが、実は一人につき三台まで無料でできるので、使ってみてください。

利用している企業だと、GMOペパボなどが挙げられます。

## 導入方法

※Ubuntu oneのアカウントを持っていることが前提条件です。

## 1. Ubuntu advantageのサイトに向かう

[サイト](https://ubuntu.com/security/livepatch)でGet Ubuntu Advantageと書かれたボタンをクリックします。

## 2. 登録ページに向かう。

Free for personal useの下にあるregisterと書かれたボタンをクリックします。

## 3. 登録もしくはログインする

Please type your email:の下にに自分のメールアドレスを入れてください。

Ubuntuアカウントを持っていない方はI don’t have an Ubuntu One accountを選択してください。

そのあとPasswordと書かれているところにパスワードを書いてください。

## 4. トークンを取得

Tokenと書かれているところをコピーしてください。

## 5. 設定

シェル画面にて、このコマンドを実行してください。

```sh
sudo ua attach <あなたのトークン>
```

## 最後に

Livepatch適用すると再起動が必要なくなるので便利だと思います。ただしパッチを適用するため常に最新のLinuxカーネルとして保たれているわけではありません。

なので、幅を開けて必ず再起動はしましょう。

是非試してみてください。
