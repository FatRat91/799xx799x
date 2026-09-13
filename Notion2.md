# 9/13　一夜漬け

よくある業務

VLAN変更：サーバーやプリンタの所属するネットワークの引っ越し

FW変更：必要な通信の通行許可。

※あくまで一種の例として、PC～外に出るための典型例

![image.png](image.png)

■VLAN変更

有線プリンターの場合は接続先のスイッチが所属するVLANを変更する。

必要に応じてプリンターのIPも変更する。→それぞれ別の作業で、固定IPは自動では変わらない。

①依頼の確認（どのプリンターを、どのVLANへ、いつ引っ越すか）

②接続先ポートの特定（スイッチAの10番ポートなど）

③引っ越し先の準備（VLAN20が存在するか、Trunk＝通信にラ・ベル（荷札）をつけて、複数のVLAN通信を一気にSW間で運ぶ。

→VLAN間の通信ではない。別VLANの場合はL3SWやルーターを経由してIPで通信する必要がある。

L2の通信はMACアドレスで行う。（フレーム）L3の場合はデフォげに問い合わせ、IPで行う（パケット）。

④実際に作業　コマンド例は以下。あくまで手順書に従う。switchport access vlan 20でVLANの変更。

※当然だがこの作業の前にバックアップや承認などを経由する。

```powershell
configure terminal
interface GigabitEthernet1/0/10
switchport access vlan 20
end
```

⑤実際に問題ないかを確認

⑥設定の保存、台帳などの更新

---

# まず覚えておくシスコのコマンド

| コマンド | 何を知りたいとき？ |
| --- | --- |
| `show running-config` | **今、どう設定されている？** |
| `show running-config interface <IF>` | **このポートだけの設定は？** |
| `show startup-config` | 起動時に使う保存済み設定は？ |
| `show logging` | **いつ、何が起きた？** |
| `show interfaces status` | どのポートが接続中？ VLAN・速度は？ |
| `show interfaces <IF>` | このポートの状態・通信量・エラーは？ |
| `show interfaces <IF> switchport` | Access／Trunkの設定・動作状態、VLANは？ |
| `show vlan brief` | VLANは存在する？ Accessポートの所属は？ |
| `show interfaces trunk` | Trunkで対象VLANが許可され、転送できる状態？ |

---

L2はMAC、L3はIP、L7はアプリ。

L1物理　ケーブルとか電気とか

L2データリンク　MAC、フレーム、VLAN　STP（経路冗長化などのループ帽子、緊急時に使う）

L3ネットワーク　IP、パケット、ルート

L4トランスポート　ポート、セッション

L7アプリケーション

TCP・IPの場合はNWインターフェース（下）→インターネット→トランスポート→アプリ（上）

OSI７階層は「NW切り分け」、TCP/IPは「インターネット全体」の話として両方使う。

VLANはサブネットのようなもので、ARP（L2でMACアドレスを使って通信する）プロトコルを使う。

IPv4⇔MACアドレスの変換のような形。

L2SWの通常の通信はVLANとMACテーブルを使う形。

フレームとパケットは入れ子で、包んでいくような形（パケット＝小包）

フレームの中にパケットを入れる。必要な荷札を必要なところで使っていく、最終的に届けるイメージ。

トランクは🔴VLAN 10・🔵VLAN 20の荷物を、色分けしたまま運ぶ🚛イメージ。

役割はVLANの識別と運搬、異なるVLANの通信はL3の役割。

Ping成功はあくまでICMPプロトコルの応答があっただけ。生きてるかどうかは原因を特定する必要がある。

---

- 同じサブネットならGWを通らない。
- ARPはIpv4⇔MACの変換ではなく、そのIPアドレスに対応するMACアドレスを調べる仕組み。
    - ドラクエでいうところの同じ街（サブネット）：相手の家に直接
    - 違う街（サブネット）：まず城門＝GWを出る必要がある

→⭐️同一サブネットなら相手のMAC、別サブネットならGWのMACをARPする

- ARPテーブル：IP　⇔　MACの対応を見る　「MACは何か教えて？」
- MACアドレステーブル：MAC+VLAN　⇔　スイッチポート　「そのMACアドレス、どのポート？」
- ルーティングテーブル：宛先NW　⇔　次の道

```powershell
ARP
192.168.10.50
      ↓
AA:BB:CC:DD:EE:FF

MAC Table
AA:BB:CC:DD:EE:FF
      ↓
Gi1/0/15
```

- ping 10.1.1.50が通っても、PowershellでTest-NetConnection 10.1.1.50 -Port 443が通らないことはあり得る。
    - TCP/443が通信できても、証明書・認証エラーやアプリ障害などは普通にある。
    - IP・DNSでの生存切り分け　DNS生きてるかでResolve-DnsName [server.example.com](http://server.example.com/)　など。
- ポートは物理ポート（スイッチの差込口）とTCP/UDPのサーバーの受付番号（SSH22とかHTTPS443とか）
- プラベIPの代表例

```powershell
10.0.0.0/8　クラスA　でかい
172.16.0.0/12
192.168.0.0/16　クラスC　小さい
```

- 169.254.x.xのDHCPエラーを疑うケース

⭐️この場合はDHCPから正常にIPが払い出されていない場合に、IPv4 Link-Local Address（169.254.0.0/16）になる。

意図的に使う場合もあるが、DHCP取得失敗の手がかりになることも多い。

DHCPはプラベIP専用ではないが、基本的には企業のLANではプラベIPを配るケースが圧倒的に多い。

```powershell
PC
 ↓ DHCP Discover
DHCP Serverから返事なし
 ↓
169.254.x.x を自動設定

---------------------

ipconfig

IPv4 Address
169.254.34.12

---------------------
```

- VLAN：L2を分ける　部屋
- サブネット：L3・IPを分ける　住所の区画
    - 以下のような合わせるケースが多いが、別物。
        
        VLAN10 → 192.168.10.0/24
        VLAN20 → 192.168.20.0/24
        
- STPはL2 Loopを防ぐために意図的に経路を止める。
    - この経路は予備だから止めとこ　→　Blocking / Discarding
    - その状態（Blocking / Discarding）」が正常な場合がある。
    - STP：ループ防止の交通整理係。
- ポートの状態

```powershell
administratively down
→ 人間の設定でOFF 🛑

down / notconnect
→ 物理接続などを疑う 🔌
```

- DV証明書：そのドメインを管理していることだけ確認
    - ACMはこれ
- OV証明書：ドメイン＋会社が実在するか（契約・監査や顧客要件）
- EV証明書：会社の情報をさらに厳格に（政府や大企業など）

---

- SVI：VLANにつける「仮想L3インターフェース」
    - これがPCから見たデフォゲになる事が多い。
    - VLAN：L2の部屋（サブネットはL3の住居区画）
        - SVI：その部屋にあるL3への出口
    
    ```powershell
    PC VLAN10
       ↓
    L2
       ↓
    SVI VLAN10 192.168.10.1
       ↓
    L3 Routing
       ↓
    SVI VLAN20 192.168.20.1
       ↓
    Server VLAN20
    ```
    
- IP持ってるSwitchはL3SWではない。L2SWでも管理用IPを持てる。
    - PCからSSHでSwitch 192.168.99.10　に接続するみたいな。
    - IPがある≠ルーティングしている
- ルートが会っても通信ができない、道がある ≠ 通れる ≠ 相手が受付してる

<aside>
💡

このパターンのいずれかに当てはまる場合が多い。

① 行きRoute
② FW / ACL許可
③ ServerがPortをListen
④ 戻りRoute
⑤ 戻り側のFW / ACL

</aside>

- ルーティングにステートフル・ステートレスは一旦考えない。
    - PC→サーバーの道があっても、　サーバー→PCの道がないと返事が帰れないってだけ。
    - ステートフル・ステートレスはFW/ACL側の話になる。
    
    ```powershell
    Stateful FW
    → Sessionを覚える
    → 戻り通信を認識できる
    
    Stateless ACL
    → 行きと戻りを個別に考える
    ```
    
    - 結局ステートフルでも戻りのルート（道）がないと帰れない。
- RSTは「その接続やめろ」、「そのポートでは受付してない」の明示的な拒否、リセット。
    - 返事が来ない、死んでいるのはTimeout（TCP通信）

<aside>
💡

待受ポートが443だからといってクライアントが443で通信するわけではない。

⭐️一時的なエフェメラルポート（動的ポート）を使う。⭐️

443：ホテルの受付　53124：客（クライアント）に渡される整理券の番号

Client
10.0.0.10:53124
↓
Server
10.0.1.20:443

逆に返事としては以下のようになる。

Server 443
↓
Client 53124

</aside>

- ステートフルはセッションへの返事。既に許可されたセッションへの応答を許可するだけ。
    - もちろん戻りの道も必要になる。
- NAT：IPを書き換える。
    - SNAT：送信元を書き換える（プラベ→グローバル）外へ出るときに。
    - DNAT：宛先を書き換える（グローバル→プラベ）外部公開サーバーなどで。
    - ⭐️PAT：IPと「ポート」を使って一個のパブリックIPを皆で仲良く使う。
- FW：通信の許可・拒否する

<aside>
💡

Ping以外で使う確認（Powershell）

■DNS確認

Resolve-DnsName [server.example.com](http://server.example.com/)

■TCP/443

Test-NetConnection server -Port 443

■経路
tracert server

■HTTPSまで見る

curl.exe -v [https://server/](https://server/)

Ping         → ICMP
Resolve-Dns  → DNS
Test-Net...  → TCP Port
curl         → HTTP/TLS

</aside>

- VLAN変えたらIPどうなる？→固定IPなら通信死ぬ、自動で変わらない。
    - DHCP端末なら取得できるかも、けどプリンタとかをDHCPにすることってあるのか…？
- FWのルール順序は、上（小さい番号、優先）から見ていって、マッチしたら判定終了。下は見ない。
    - オーダード　ルールベース？
    - AWSのNACLと同じ。
- 一気に20台同時にダウンした！→20台とも壊れた、ではない。

```powershell
20台
↓
同じL2SW？
↓
同じ上位SW？
↓
同じFW？
↓
同じ回線？
```

- アラート≠インシデント確定ではない。影響範囲の確認。

<aside>
💡

変更後に何を確認する？

① Interface状態
② VLAN / Trunk
③ Gateway / Route
④ FW
⑤ End-to-End疎通
⑥ 実際のApplication
⑦ Error / Drop / Log
⑧ Monitoring

</aside>

running-config
＝ 今、機器が使ってる設定 🏃

startup-config
＝ 再起動時に読み込む保存設定 💾

-