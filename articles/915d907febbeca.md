---
title: "おうちk8sでグローバルIP"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["bgp", "kubernetes", "frrouting", "wireguard"]
published: false
publication_name: neody
---
## 1. はじめに
現在、私はAS152873を運用しており、好き勝手にIPアドレスを使える状態です。
しかし、ふと、原点に戻り、AS運用していない人でも、Kubernetes + MetalLBで
Service LBにIPを割り振る方法がないか、考えてみて、思いついたので共有します。

## 2. 環境
お手軽にやりたかったので、k3sを利用しました。

## 3.1. まずVPSの契約
現在、VPSにはIPアドレスをお代わりすることができる機能があります。
今回、その機能を使って実装する予定です。

Vultrの`Settings`でIPを追加してください。(追加したIPのことを`<Additional IP>`と記します。)

私は、Vultrを使いましたが、おそらくLinodeでも動くと思います。

OSはdebianを使いました。

## 3.2 パッケージのインストール
トンネルにWireGuardを使い、BGPデーモンでFRRouting使います。以下のでダウンロードしてください。
```sh
curl -s https://deb.frrouting.org/frr/keys.gpg | sudo tee /usr/share/keyrings/frrouting.gpg > /dev/null
FRRVER="frr-stable"
echo deb '[signed-by=/usr/share/keyrings/frrouting.gpg]' https://deb.frrouting.org/frr \
     $(lsb_release -s -c) $FRRVER | sudo tee -a /etc/apt/sources.list.d/frr.list

sudo apt update && sudo apt install frr frr-pythontools wireguard-tools
```

参考：https://deb.frrouting.org/

## 3.3 WireGuard
WireGuardのconfigを書きます。clientとserverの秘密鍵および公開鍵を作る必要があります。以下のコマンドを参考して作成してください。
```
[Interface]
PrivateKey=<サーバ側のWireGuardのPrivate Key>
MTU=1460
Address=100.64.1.1/30
ListenPort=51820
Table=off # 重要：無効化しないと、AllowedIPsに書かれているIPが経路として学習される。offにすること。

[Peer]
PublicKey=<クライアント側のWireGuardのPublic Key>
AllowedIPs=100.64.1.2/32, <Additional IP>/32
PersistentKeepAlive=30
```

## 3.4 FRRの設定
/etc/frr/daemonsで`bgpd=no`となっているので、`yes`にしてください。

また、BGPの設定は以下の通りです。
```
# おかわりIPのみを受け取るように設定
ip prefix-list ALLOW-IN seq 5 permit <Additional IP>/32 
!
route-map ALLOW-IN permit 10
 match ip address prefix-list ALLOW-IN
exit
!
route-map ALLOW-IN deny 100
exit
!
route-map DENY-ALL-OUT deny 10
exit
!
router bgp 65000
 neighbor 100.64.1.2 remote-as 65001
 !
 address-family ipv4 unicast
  neighbor 100.64.1.2 next-hop-self
  neighbor 100.64.1.2 soft-reconfiguration inbound
  neighbor 100.64.1.2 route-map ALLOW-IN in
  neighbor 100.64.1.2 route-map DENY-ALL-OUT out
 exit-address-family
exit
!
```

## 3.5 カーネルパラメーターの調整
`/etc/sysctl.d/10-bgp.conf`に以下の内容を書いてください。

```
net.ipv4.ip_forward=1
```

その後、

```
sysctl -p
```

を実行してください

## 4.1 環境構築
MetalLBで構築するために、service LoadBalancerを無効化する必要があります。
構築時に以下のコマンドを入力してください。

```
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable=servicelb" sh -
```

また、その後、WireGuardをインストールしてください。

```sh
apt-get update
apt-get install wireguard-tools
```

そして、MetalLBをインストールしてください。

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.16.2/config/manifests/metallb-frr-k8s.yaml
```

## 4.2 WireGuardの設定
以下の内容でWireGuardを設定してください。
```
[Interface]
PrivateKey=<クライアント側のWireGuardの秘密鍵>
Table=off # 重要：余計な学習を防ぐため。
MTU=1460
Address=100.64.1.2/30

[Peer]
Endpoint=<Vultr側のIP>:51820
AllowedIPs=0.0.0.0/0
PersistentKeepAlive=30
PublicKey=<サーバ側のWireGuardの公開鍵>
```

## 4.3 VRFの作成
VRFでmainとは別でMetalLB用のVRFを作成する必要があります。

```sh
ip link add vrf-blue type vrf table 1001
ip link set up dev vrf-blue
ip route add default via 100.64.1.1 vrf vrf-blue # 戻りの通信用
```

## 4.4 ポリシーベースルーティングの設定
かなり、肝のところです。4.3の部分でVRF分離している関係で、vrf mainに帰りのトラヒックが出てしまいます。

その対策で以下のコマンドを入力してください。

```
ip rule add pref 1100 from <Additional IP>/32 lookup 1001
```

## 4.5 MetalLBの設定
以下の内容で、MetalLBを書いてください。

BGPピアの設定：
```yaml
apiVersion: metallb.io/v1beta2
kind: BGPPeer
metadata:
  name: peer
  namespace: metallb-system
spec:
  myASN: 65001
  peerASN: 65000
  peerAddress: 100.64.1.1
  vrf: vrf-blue
```

BGP広告の設定：
```yaml
apiVersion: metallb.io/v1beta1
kind: BGPAdvertisement
metadata:
  name: advertise
  namespace: metallb-system
```

MetalLBのIPのプール：
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - <Additional IP>/32
```

その後、いつも通りの方法でmanifestを反映してください。

## 5. 検証
VPS側で以下のコマンドを実行してください。

```
sudo vtysh -c "sh ip bgp"
```

そうすると、以下の出力結果が出るので、そこに自分の追加されているIPがあるか確認してください。もしもあれば、それで終わりです。

```
BGP table version is 5, local router ID is 100.64.0.1, vrf id 0
Default local pref 100, local AS 65000
Status codes:  s suppressed, d damped, h history, u unsorted, * valid, > best, = multipath,
               i internal, r RIB-failure, S Stale, R Removed
Nexthop codes: @NNN nexthop's vrf id, < announce-nh-self
Origin codes:  i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  <Additional IP>/32
                    100.64.1.2               0             0 65001 i

Displayed 1 routes and 1 total paths
```

## 6. 最後に
ここまで読んでくださり、ありがとうございました。

ISPとBGPを張って、IPを持ち込まずに、VultrなどのVPS事業者からIPアドレスを持っていくという偉業を達成できたあなたに拍手を！！！

実は、私が個人的におうちk8sをやった際に、AS152873のIPを使うのは避けて、VPSのを使いたいと思っていたので、今回検証がてらやりました。