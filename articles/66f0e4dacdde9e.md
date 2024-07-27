---
title: "Wireguardとk3s"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kubernetes", "wireguard"]
published: true
publication_name: "neody"
---

どうもインフラを専門にやっているtuna2134です。

ところで、みなさーん、k8sやってますかー？

僕は当然やっています。

さて今回はwireguardを使っておうちにあるサーバをドイツのマスターノードに参加させてみようと思います。

けしからんぐらい遠いですけど、やります。
![keshiakran](/images/keshikaran.webp)

## 構成
- Contabo (ドイツ、マスターノード)
- 普通のVM (日本、ワーカーノード)

OSは基本的にdebian使います。

## まずWireguardをインストール
これはすべてのノードで実行してください。
```
apt-get install wireguard-tools
```

## wireguard設定
まずキーを生成します。

- server.keyはサーバのプライベートキーです
- server.pubはサーバのパブリックキーです

### サーバーのキーを生成
```
wg genkey | tee server.key | wg pubkey > server.pub
```

### クライアントキーの設定
```
wg genkey | tee client.key | wg pubkey > client.pub
```

### サーバ側のwireguardの設定
`/etc/wireguard/wg0.conf`を編集して以下のように編集してください。
```
[Interface]
PrivateKey=$(Server private key)
Address=172.16.42.1
ListenPort=51820

[Peer]
PublicKey=$(Client public key)
AllowedIPs=172.16.42.2
PersistentKeepAlive=30
```

wireguardを起動
```
sudo systemctl start wg-quick@wg0
sudo systemctl enable wg-quick@wg0
```

### クライアント側のwireguardの設定
`/etc/wireguard/wg0.conf`を編集して以下のように編集してください。
```
[Interface]
PrivateKey=$(Client private key)
Address=172.16.42.2

[Peer]
PublicKey=$(Server public key)
AllowedIPs=172.16.42.1/24
Endpoint=$(server ip):51820
PersistentKeepAlive=25
```

wireguardを起動
```
sudo systemctl start wg-quick@wg0
sudo systemctl enable wg-quick@wg0
```

## k3sの設定

### マスターノードを作成
```
curl -sfL https://get.k3s.io | sh -
```

その後、`/var/lib/rancher/k3s/server/node-token`に書かれている内容を控えてください。

### ワーカーノードの参加
```
curl -sfL https://get.k3s.io | K3S_URL=https://172.16.42.1:6443 K3S_TOKEN=$(控えたもの) sh -
```

これで構築完了しました。お疲れ様です。

## 最後に
Kubernetesの軽量版k3sでk8sクラスターを構築しました。
ドイツという場所が遠いので、かなりレスポンスなどが遅いです、、、
なので、できるだけ近いところ同士でつなぎましょう。