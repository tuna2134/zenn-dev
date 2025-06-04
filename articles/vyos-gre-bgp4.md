---
title: "VyOSでBGPをプライベートAS番号で張る話 1"
emoji: "💨"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["vyos", "bgp"]
published: false
publication_name: neody
---

VyOS使ってBGPごっこをする話です。

## 環境
- VyOS - v2025.05.13-0019-rolling
- `10.47.0.0/31` - 自組織側の境界IP
- `10.47.0.1/31` - 相手組織側の境界IP
- `2001:db8:2::1` - 自組織側のトンネル終端IP
- `2001:db8:2::2` - 相手組織側のトンネル終端IP

## BGPとは？
Border Gateway Protocolの略です。前回の記事で解説したのでそちらをご覧ください。

https://zenn.dev/neody/articles/f501261728997c

## GREトンネルをとりあえず張る
プロバイダーのIPをBGPの境界IPで使えないので、GREトンネルを張ってうまいことやります。
```
set interfaces tunnel tun0 address 10.47.0.0/31
set interfaces tunnel tun0 ipv6 adjust-mss 1416
set interfaces tunnel tun0 encapsulation 'ip6gre'
set interfaces tunnel tun0 source-address '2001:db8:2::1'
set interfaces tunnel tun0 remote '2001:db8:2::2'
```

## 疎通確認
```
ping 10.47.0.1 -c 4
```
これでちゃんとパケットが通る確認してください。
通っているのであれば問題ありません。

## 次回予告
BGPを張って、動くところまでやる予定です。