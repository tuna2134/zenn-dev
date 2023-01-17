---
title: "livepatchとは"
emoji: "🔨"
type: "tech"
topics: ["ubuntu"]
published: true
publication_name: "rext"
---

# Livepatchを導入してみた

## Livepatchとは

Livepatchはその名の通り、Ubuntu(Linux系のos)でkernelパッチがリリースされ、

ダウンロードされた時に、パッチがサーバーを再起動せずに適用することはできます。

## 導入方法

※Ubuntu oneのアカウントを持っていることが前提条件です。

### 1. Ubuntu advantageのサイトに向かう

[サイト](https://ubuntu.com/security/livepatch)でGet Ubuntu Advantageと書かれたボタンをクリックします。

### 2. 登録ページに向かう。

Free for personal useの下にあるregisterと書かれたボタンをクリックします。

### 3. 登録もしくはログインする

Please type your email:の下にに自分のメールアドレスを入れてください。

Ubuntuアカウントを持っていない方はI don’t have an Ubuntu One accountを選択してください。

そのあとPasswordと書かれているところにパスワードを書いてください。

### 4. トークンを取得

Tokenと書かれているところをコピーしてください。

### 5. 設定

シェル画面にて、このコマンドを実行してください。

```sh
sudo ua attach <あなたのトークン>
```

## 最後に

Livepatch適用すると再起動が必要なくなるので便利だと思います。

是非試してみてください。
