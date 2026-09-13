# 8/28

ネクストホップ：ネットワークでデータを次の宛先に送るために経由する「お隣さんのルーター」

→**やってきたデータのたらい回し先。**

```powershell
PC
 │ Access VLAN10
SW1
 │
 │ trunk
 │ VLAN10,20,30
 │
SW2
```

L1：リンクランプ・ケーブルとか

L2：MAC、VLAN、Trunk、STP

L3：IP、ゲートウェイ、ルート、Ping

L4：TCP/UDP、ポート、TCPハンドシェイク

L7：DNS、TLS、HTTP、認証、アプリケーション

⭐️実際はOSI７階層モデルでネットワークの切り分けや会話、TCP４階層は「インターネット全体」

# 理解しておかなきゃダメ

通信できない？

→IP正しい？

→VLAN/L2が通る？

→経路はある？

→TCP/DNS/ポートは？

→ログ・監視で証拠を集める

パケットがどこで死んだ？を順番にレイヤー別で探していく。

レイヤー：OSI7階層　上から7️⃣アプリ→プレゼン→セッション→トランスポート→データリンク→2️⃣ネットワーク→物理層

⭐️TCP/IP：パケット（小包）を渡す　4️⃣アプリ→3️⃣トランスポート→2️⃣インターネット→1️⃣ネットワークインターフェース

IP：住所

CIDR：同じ町内

```bash
⭐️同じサブネット内
自分：192.168.10.20/24
相手：192.168.10.50
	→おなじサブネットにいる（192.168.10.20/24の範囲内）なので同一サブネット、ご近所。

⭐️違うサブネット
自分：192.168.10.20/24
相手：192.168.20.50
　→192.168."10"と"20"のサブネットなので違うサブネット。
		→違う場合はPC→ゲートウェイ→ルーター→別サブネットへ繋げる。
		
/24はアドレス数256（全部使えるわけではない）
/25は128、/28は16アドレス…といったように倍々の関係。/26は64だけどそもそも/24以降はあんまり。

例：192.168.10.20/24

ネットワーク
192.168.10.0
      ↓
192.168.10.1 ～ 254
      ↓
192.168.10.255
```

---

## プライベートIPは以下の通り（クラスA大規模～C小規模）で決められている。

**クラスAの範囲**：`10.0.0.0` ～ `10.255.255.255` （10.0.0.0/8）

**クラスBの範囲**：`172.16.0.0` ～ `172.31.255.255` （172.16.0.0/12）

**クラスCの範囲**：`192.168.0.0` ～ `192.168.255.255` （192.168.0.0/16）

---

# VLAN/trunk/STP

<aside>
💡

同じスイッチの中に、別々の世界を作る仕組み

</aside>

```bash
物理的に同じスイッチでも論理的に分離できる。
最優先はアクセスとトランク。
--------------------------------------------
アクセス：端末用、１路線
PC
 │ VLAN10
SW
--------------------------------------------
トランク：スイッチとスイッチの間、複数路線をまとめて運ぶ
           　  VLAN10
SW1 =========  VLAN20 ========= SW2
           　  VLAN30
--------------------------------------------
VLAN10：社員
VLAN20：サーバー
VLAN30：ゲストWi-Fi
```

STP：ループ防止装置

⭐️VLAN：見えない壁で複数のネットワークに分割

⭐️アクセスポート：PCを接続する１つのVLAN専用ポート

⭐️トランクポート：複数のVLANを１本のケーブル（イメージ）でまとめて運ぶポート

→荷物に「VLAN10」や「VLAN20」といったような荷札をつけるイメージ

⭐️STP：スイッチ間に複数の道があるとループが発生する、予備ルートを論理的に止めておく（緊急時に解放するなど）

```bash
・スイッチの冗長化でループができてしまう
SW1 ─── SW2
 │       │
 └──SW3──┘
 
 
 ・STPが止める役割
     ┌───────┐
SW1 ─┤       ├─ SW2
     └── X ──┘
         ↑
      STPが止める
      
⭐️最初に見るのはまずL2。ルーティングを疑う前にまずL2
端末→L2→L3スイッチ or ルーター→FW→WAN/インターネット→LB→サーバー
```

| 領域 | 何を見る？ | 重要単語 | 関与度 |
| --- | --- | --- | --- |
| 🔌 **L1 物理** | そもそも線が生きているか | LANケーブル、光、SFP、リンク、速度、Duplex、PoE | ★★★★★ |
| 🟦 **L2 LAN内** | 同じネットワーク内の配送 | MAC、VLAN、Access、Trunk、STP、LACP | ★★★★★ |
| 🟩 **L3 ネットワーク間** | 別ネットワークへの道案内 | IP、CIDR、Gateway、Route、OSPF、VRF | ★★★★★ |
| 🟨 **L4 通信制御** | TCP/UDPとポート | TCP、UDP、443、Session、ACL、NAT | ★★★★☆ |
| 🟥 **L7 サービス** | 名前解決やアプリ通信 | DNS、DHCP、NTP、HTTP、Proxy、LB | ★★★★☆ |
| 📡 **無線** | Wi-Fi接続 | AP、WLC、SSID、Channel、RSSI、802.1X | ★★★☆☆ |
| 👀 **監視・管理** | 異常の発見と証拠集め | SNMP、Syslog、NMS、SSH、SPAN、Wireshark | ★★★★★ |

---

# ルータ、ゲートウェイ、OSPF

→「パケット用の地図」

ルータは経路表を見て宛先→ルートテーブル→ネクストホップ→次のルータと判断

10.20.0.0/16
↓
192.168.1.1へ送る

Connected：自分に直接つながっているネットワーク

Static（スタティック、静的）：人間が手動で決めている経路

Default：どの経路にも当てはまらない場合、の行き先（ALL0、とりあえずこのルータに流す）

## ⭐️最長一致

→宛先が「10.20.5.100」の場合、一番下の10.20.5.0/24が選ばれる。「最も具体的」だから。

10.0.0.0/8
10.20.0.0/16
10.20.5.0/24

OSPF：ルータ同士で経路情報を自動交換する

→⭐️OSPFについてはあとで理解し直す

# TCP・UDP・DNS・ポート

IP：住所

ポート：部屋番号

10.10.1.50:443　→　443ポート（HTTPS）

TCPは接続をつくる、3ウェイハンドシェイク。

1️⃣クライアント→宛先　SYN（送っていい？） SYN=1、ACK＝0

→これで2️⃣が返らない場合はFW、ACL、経路、サーバー停止など。

2️⃣宛先→クライアント　SYN & ACK（いいよ、こっちも送っていい？）SYN=1、ACK=1

3️⃣クライアント→宛先　ACK（いいよ）SYN=0、ACK=1

→つまり　SYN：送ってええか？　ACK：ええよ　で事前のネゴみたいな感じ。

22 SSH

53 DNS

67/68 DHCP

80 HTTP

123 NTP

161 SNMP

443 HTTPS

DNSはご存知、aaa.com⇔IPアドレスに変換する仕組み。

→IP指定で繋がるが、hostname失敗の場合は「DNSを疑う」

```powershell
PowershellでDNS確認
Resolve-DnsName server.example.com
```

LinuxでもやったけどPingは「ICMP」で確認用のプロトコル。

ping通るけどHTTPSが死んでる（TCP/443）は普通に多い。

```bash
Linuxの場合の疎通確認、段階的な確認

ping 10.0.0.10
nc -vz 10.0.0.10 443
curl -vk https://10.0.0.10
```

```bash
Pingは通る、Webは開かない場合は以下の順番で切り分ける。

DNS
↓
TCP/443
↓
FW
↓
Server LISTEN
↓
TLS / Webアプリ
```

---

---

---

# 監視・ログ・障害の切り分け

- 壊さない
- いじらない
- 証拠を残す

生きてるか

```bash
ping
interface up/down
```

リソースどう？

```bash
CPU
Memory
帯域使用率
Error
Discard
Latency
Packet Loss
```

何が起きてる？→syslog

誰が誰と通信してる？

```bash
NetFlow
IPFIX
Packet Capture
```

大量にアラートが出ても焦らない。コアSWが一個死んだだけかも。

```bash
Core SW DOWN
     ↓
SW01 DOWN
SW02 DOWN
SW03 DOWN
Server01 DOWN
Server02 DOWN
・・・
```

## 見る順番は下を常に意識する。

① 一番最初のアラート
↓
② 共通する上位機器
↓
③ ネットワーク構成図
↓
④ 同時刻syslog
↓
⑤ 直前変更

落とさない、再起動しない、消さない

```bash
障害発生
 ↓
時刻確認
 ↓
影響範囲
 ↓
show / ログ取得
 ↓
仮説
 ↓
承認
 ↓
変更
 ↓
正常性確認
```

## ⭐️下のチートチャートを参考に

通信できない
│
├─① IP設定合ってる？
│   IP / Mask / Gateway
│
├─② L2通ってる？
│   Link / VLAN / MAC / ARP / STP
│
├─③ 行き方ある？
│   Route / Next Hop / OSPF
│
├─④ サービスまで届く？
│   DNS / TCP / UDP / Port / FW
│
└─⑤ 証拠は？
Monitoring / syslog / Packet Capture

# 実際どんな感じ？とかイメージ

- おそらくWin端末＋テラターム＋Powershellを使う…かも。
- コマンド文法はNW機器による。
- Linuxサーバー・Windowsサーバーに入るケースもあるかも。
    - SSH接続、踏み台は経由する

![image.png](image.png)

おそらくは↑の図のような形。

踏み台・管理NWは恐らく確定で経由する。（うっすらした記憶）

Telnetは古い、SSH v2が主流。

Windows端末からCISCO：シスコのコマンド。Linuxサーバーに入る：Linuxのコマンド。

→シスコ限定で覚えるのではなく各メーカーのコマンドが存在することを強く意識する。

```bash
例：Ciscoの場合のコマンド

# 時刻・ログ
show clock
show logging

# L1：ポート・エラー
show interfaces status
show interfaces GigabitEthernet1/0/24

# L2：VLAN・trunk・STP・MAC
show vlan brief
show interfaces GigabitEthernet1/0/24 switchport
show interfaces trunk
show spanning-tree interface GigabitEthernet1/0/24
show mac address-table interface GigabitEthernet1/0/24

# L3：ARP・経路・OSPF
show ip arp
show ip route 10.20.30.40
show ip ospf neighbor

```

## ⭐️PowershellはWindows端末から見た通信状態を調べる

```powershell
# IP・Gateway・DNS
Get-NetIPConfiguration

# DNS
Resolve-DnsName app.example.com

# TCP/443
Test-NetConnection app.example.com -Port 443

# 経路
tracert app.example.com

# ICMP　PingはあくまでPing、またPingだけを弾いている場合もある。
ping app.example.com
```

※恐らくTeraTermだけど、Windows公式のOpenSSHクライアントもある。

## ⭐️Linuxサーバー（DNS、DHCPとか）に入る可能性もある

```powershell
ip -br addr #自分のIP
ip route #経路
ss -lntp #ポート
systemctl status <サービス名> #サービスのステータス
journalctl -u <サービス名> --since "2026-08-28 10:00" #システムログ、期間指定a
```

---

# シナリオ例：Webシステムにアクセスできない

## 1️⃣事象　何が起きた？

## 2️⃣範囲　どこまで？どれが？

## 3️⃣差分　正常な状態との違い

→この３つで切り分けて、その後仮説→技術的な確認→原因特定→修正→問題ないかを確認

L1（物理）のケーブルや接続などから、IP（L3）に上がっていくなど。

物理的に問題ない→ルーターにPing→WebサーバーにPing、これは成功するがICMPでの成功、なだけで実際TCP HTTPS443での通信ではない。それを確認するためには nslookupなど。正しいIPに名前解決できてるかどうか。コレもOKと仮定する、ここで対象のドメインのポート443が空いてるかを確認する。

## 今回はL3（物理L1、IP取得、ゲートウェイ、Web鯖へのPingなどL3）まではOKだが、L4（TCP、UDPやNATあたり）に問題があることがわかる。

```powershell
Test-NetConnection app.example.com -Port 443
```

Pingがそもそも通らない→tracerouteなどを使い、経路がどこまで生きてるかを確認する。

→ルーティングテーブルを見る、ネクストホップ＝たらい回し先などを確認する

今回のシナリオ例では、同じフロアだが席替えで別スイッチに繋いでおり、VLANが10から20に変わっていた。

別スイッチによりCIDR範囲が変わり、192.168.10.xから192.168.20.xxxに変わっており、

そのIPの範囲では社内Webシステムにはアクセスできない設定（TCP443がDeny）になっていた。

![image.png](image%201.png)

![image.png](image%202.png)

---

# RADIUS.TACACS+、Ansible

例えばPowershellのTest-NetConnection app.example.com -Port 443は、開いてるかだけを確認。

WireSharkは誰が送った、返した、どこで止まった、とかより詳細に追うことが出来る。

HTTPSの場合でも、IPアドレス、TCPポート、SYN/ACK/RST（RSTは存在しないポートへのアクセスや、通信拒否などでサーバーからRST、またはRSTとACKがセットで帰されて、接続が即時切断される）

RADIUSは審査員、ネットワーク認証を一元管理する

個々のアカウントでの認証の手間を省き、社内の認証基盤（ADやEntraID）と連携する。

TACACS+はNW管理機器の権限や操作を管理する。

Ansibleはサーバー内部の設定と、スイッチ・ルータ・FWの設定管理も行える。

stateファイルは持たないらしい？

terraformはプロビジョニングと状態管理（宣言）

Ansibleは「構成管理とリソース作成後のオーケストレーション

GitのPlaybook→レビュー・承認→Ansible実行サーバー→各SWなどに配信？

NW機器にはエージェントを基本的に入れない。

terraformとansibleで設定の取り合いをしない（同じリソースを管理しない、分担する）

| 観点 | Terraform | Ansible |
| --- | --- | --- |
| 一言 | 何を存在させるか | どう設定するか |
| 得意 | リソース作成・削除・ライフサイクル | 設定変更・配布・確認・手順実行 |
| 状態管理 | `tfstate`を持つ | Terraform相当の状態ファイルは基本持たない |
| 接続 | Provider API | SSH・NETCONF・HTTPS API等 |
| AWS例 | VPC、Subnet、EC2、RDS、SG | OS、ユーザー、パッケージ、設定ファイル |
| NW例 | クラウドVPC、仮想FWリソース | VLAN、ACL、NTP、SNMP、OSPF、BGP |

各単語の整理

<aside>
💡

Ansibleが設定する
↓
VLAN       = ネットワークを部屋分け
NTP        = 時計を合わせる
syslog     = ログを集める
SNMP       = 状態を監視する
OSPF/BGP   = 経路を自動交換する

</aside>

| 設定 | レイヤー | 一言 | 使用場面 |
| --- | --- | --- | --- |
| VLAN | L2 | 部屋分け🚪 | 社員・サーバー・ゲストを分離 |
| NTP | 管理 | 時計合わせ⏰ | 全機器のログ時刻を統一 |
| syslog | 管理 | 日記送信📝 | 障害・設定変更ログを集中保存 |
| SNMP | 管理 | 健康診断📊 | CPU、帯域、ポート状態を監視 |
| OSPF | L3 | 社内カーナビ🗺️ | 組織内部の経路を自動交換 |
| BGP | L3 | 国同士の経路交渉🤝 | ISP・拠点・クラウド等との経路交換 |

---

# WireSharkとCiscoコマンド　記入予定

# 雑多に

netstatはPowershellの後発コマンドレットのコマンドがある。

dockerのコマンド置き換え（docker ps →docker container ls）

netstat　→　Get-NetTCPConnection

ipconfig　→　Get-NetIPConfiguration

社員が障害を訴えている場合は以下のコマンド例、netstatやipconfigも使う（自分のを見る）

```powershell
障害が起きている本人のPCで確認
同じSSID・VLAN・拠点の正常PCと比較
自分のPCでも確認
AP・スイッチ・FW・サーバー側を確認
```

VLAN：L2レイヤーでLANを論理的に分ける

CIDR：L3レイヤーでIPアドレスの範囲を示す

ARP：同じLAN内、IPからMACアドレスを調べる

/24は256個から.0のネットワークアドレス、.255のブロードキャストを除いて254

■同じサブネット

192.168.10.20
↓ ARP
「192.168.10.50のMACは？」
↓
MACを使って直接送る

■違うサブネット

192.168.10.20
↓ ARP
「Gateway 192.168.10.1のMACは？」
↓
Gatewayへ渡す
↓
Gatewayが経路表を見て192.168.20.50へ転送
