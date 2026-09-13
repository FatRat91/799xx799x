作るなら、**全部暗記する単語帳じゃなく「聞いた瞬間に何の話か分かる」単語帳**が実務向き。
優先度は **🔥＝参画前に必須 / ⭐＝頻出 / △＝聞いて固まらなければOK**。添付資料でも、L1→L2→L3→L4/L7→制御→監視の順が最低ラインになってる。

Cisco部分は2026年4月更新のCatalyst 9300 / IOS XE 26.x公式も再確認済み。([Cisco][1])

# 0️⃣ 全体・レイヤー

| 単語                     | 一言で          | どこでどう使う？                            |
| ---------------------- | ------------ | ----------------------------------- |
| 🔥 L1                  | 物理           | Cable・SFP・Link Downなど、まず「刺さってる？」を見る |
| 🔥 L2                  | 同一LAN内の転送    | MAC・VLAN・Trunk・STPの話                |
| 🔥 L3                  | IPで別NWへ転送    | Gateway・Route・SVIの話                 |
| 🔥 L4                  | TCP/UDP・Port | 「443まで通る？」などService単位の切り分け          |
| ⭐ L7                   | Application層 | HTTP・DNSなど「通信の中身」側                  |
| ⭐ Frame                | L2のデータ単位     | SwitchがMACを見て転送するもの                 |
| ⭐ Packet               | L3のデータ単位     | Router/L3SWがIPを見て転送                 |
| ⭐ Source / Destination | 送信元 / 宛先     | FW・障害報告では「どこ→どこ」を必ず明確にする            |

🔥 **MAC＝L2、IP＝L3、Port＝L4**。まずこれ。

---

# 1️⃣ L1｜物理・Interface 🔌

| 単語                       | 一言で            | どこでどう使う？                   |
| ------------------------ | -------------- | -------------------------- |
| 🔥 Interface / Port      | 機器の接続口         | `Gi1/0/10`など。障害・VLAN変更の対象  |
| 🔥 Link Up / Down        | 物理接続の成立/不成立    | DownならCable・SFP・NIC・電源を疑う  |
| 🔥 administratively down | 設定でPort停止      | `shutdown`されてないか確認         |
| 🔥 err-disabled          | 保護機能でPort停止    | 原因確認→修正→承認後に復旧。即no shutしない |
| 🔥 CRC / FCS Error       | Frame破損の手掛かり   | 増加してたらCable・SFP・NIC等を疑う    |
| ⭐ Link Flap              | Up/Downを繰り返す   | 瞬断。「今Up」でもログで過去を見る         |
| ⭐ SFP                    | 光/銅線を接続するモジュール | Link Down・CRC時の物理確認候補      |
| ⭐ Duplex                 | 送受信方式          | 不一致だとError・遅延原因になる場合あり     |
| ⭐ Speed                  | 100M/1G/10G等   | 想定速度でLinkしてるかを見る           |
| △ PoE                    | LAN Cableで給電   | AP・IP Phoneが起動しない時など       |

🔥 **CRCが「ある」より「増え続けてるか」が重要。** 累積Counterだけで即故障認定しない。

---

# 2️⃣ L2｜MAC・VLAN・Switch 🔀

| 単語                    | 一言で                 | どこでどう使う？                      |
| --------------------- | ------------------- | ----------------------------- |
| 🔥 MAC Address        | L2の端末識別子            | SwitchがFrameをどこへ送るか判断         |
| 🔥 MAC Address Table  | MAC→Port対応表         | 「この端末どのSwitch Port？」を追う       |
| 🔥 ARP                | IPv4→MACを調べる        | IPしか分からない端末のMACを探す            |
| 🔥 VLAN               | L2を論理分割             | PC・Printer・Serverなどを別LANに分離   |
| 🔥 Access Port        | 基本1 VLAN用Port       | PC・Printer接続で頻出               |
| 🔥 Trunk Port         | 複数VLANを1本で運ぶ        | Switch間・AP・仮想基盤等              |
| 🔥 802.1Q             | VLAN Tag方式          | Trunkで「これはVLAN20」と識別          |
| ⭐ Allowed VLAN        | Trunkで通すVLAN一覧      | VLAN変更後「上位へ届かない」で見る           |
| ⭐ Native VLAN         | TrunkのUntagged VLAN | mismatchが障害原因になる              |
| 🔥 STP                | L2 Loop防止           | 冗長Linkを必要に応じBlock             |
| ⭐ LACP / EtherChannel | 複数Linkを束ねる          | 帯域・冗長化。CiscoではPort-Channelも頻出 |
| △ BPDU                | STP情報のFrame         | STPがSwitch同士で情報交換             |
| △ Root Bridge         | STPの基準Switch        | STP障害調査で出てくる                  |

重要な流れ👇

```text
IP
 ↓ ARP
MAC
 ↓ MAC Table
Switch Port
```

Access/Trunk/SVI等の整理も添付資料の重点。

---

# 3️⃣ L3｜IP・Gateway・Routing 🛣️

| 単語                    | 一言で                 | どこでどう使う？                   |
| --------------------- | ------------------- | -------------------------- |
| 🔥 IP Address         | L3の住所               | 通信元・宛先を指定                  |
| 🔥 Subnet             | IPの同一NW範囲           | 同一Subnetか別Subnetか判断        |
| 🔥 CIDR               | Subnet範囲の表記         | `/24` `/16`など              |
| 🔥 Default Gateway    | 別Subnetへの出口         | 別VLANに行けない時に確認             |
| 🔥 SVI                | VLANの仮想L3 Interface | L3SW上でVLANのGWになることが多い      |
| 🔥 Inter-VLAN Routing | VLAN間Routing        | VLAN10→20などをL3SW/Routerで中継 |
| 🔥 Route Table        | 宛先への道一覧             | 「この機器は宛先への道を知ってる？」         |
| 🔥 Next Hop           | 次に渡す機器              | Routeを追跡するとき使う             |
| 🔥 Default Route      | 他にRouteがない時の道       | IPv4なら`0.0.0.0/0`          |
| 🔥 最長一致               | 最も具体的なRoute優先       | `/24`と`/0`なら通常`/24`        |
| ⭐ Return Route        | 戻り道                 | 「行きは届くのに通信成立しない」で必須        |
| ⭐ OSPF                | 組織内の動的Routing       | Neighbor・Routeを自動交換        |
| ⭐ BGP                 | 主にAS間Route交換        | ISP/WAN/DX等で遭遇             |
| △ VRF                 | 1台に複数Route Table    | 管理NW等を論理分離                 |

Cisco公式でもSVIは「VLANをRouting機能につなぐInterface」で、VLAN間Routingや管理IPに使う。([Cisco][2])

🔥 **Routeがある＝通信できる、ではない。**

```text
Route
＋ FW
＋ Listener
＋ 戻りRoute
＝ 通信成立候補
```

---

# 4️⃣ L4・TCP/UDP・FW 🚪

| 単語                   | 一言で            | どこでどう使う？                    |
| -------------------- | -------------- | --------------------------- |
| 🔥 TCP               | 接続型通信          | SSH・HTTPS等。再送・順序制御あり        |
| 🔥 UDP               | Connectionless | DNS・DHCP・NTP等で頻出            |
| 🔥 Port番号            | Serviceの受付番号   | 22 SSH、443 HTTPS等           |
| ⭐ Source Port        | Client側Port    | 通常Ephemeral Portを自動利用       |
| 🔥 3-way Handshake   | TCP接続開始        | SYN→SYN/ACK→ACK             |
| 🔥 RST               | TCP Reset      | Listenerなし・明示Reject等の手掛かり   |
| 🔥 Timeout           | 返答なし           | Route・FW Drop・戻り道等、原因多数     |
| ⭐ Session            | 一連の通信状態        | Stateful FW等で追跡             |
| 🔥 Listener / Listen | Serverの待受      | FWが通ってもListenerなしなら接続不可     |
| 🔥 ACL               | 通信許可/拒否ルール     | Router/Switch等でIP/Port制御    |
| 🔥 Firewall          | 通信を検査・制御       | Src/Dst/Protocol/Port/方向を確認 |
| 🔥 Stateful          | Session状態を覚える  | 許可通信の戻りを認識                  |
| ⭐ Stateless          | Packet単位で判定    | 行き・戻りを個別に考える                |
| ⭐ MTU                | 1回で送れる最大サイズ    | 大きい通信だけ失敗する時候補              |

TCP/UDP・RST/Timeout・Ephemeral Portは添付資料でも最優先項目。

---

# 5️⃣ NAT・通信制御 👮

| 単語                   | 一言で               | どこでどう使う？                         |
| -------------------- | ----------------- | -------------------------------- |
| 🔥 NAT               | IP/Portを書き換える     | Private↔Public等。FWとは別物           |
| ⭐ SNAT               | Sourceを書き換える      | Clientの外向き通信など                   |
| ⭐ DNAT               | Destinationを書き換える | 公開Serverへの転送など                   |
| ⭐ PAT / NAPT         | Portも変換           | 1 Public IPを複数端末で共有              |
| 🔥 Allow / Permit    | 通信許可              | FW/ACLのRule                      |
| 🔥 Deny / Drop       | 通信拒否              | DropならTimeoutになることも              |
| 🔥 Rule Order        | Rule評価順           | 上位DenyにMatchすると下のAllowまで行かない製品あり |
| ⭐ Zone               | FW上のNWグループ        | Inside→Outside等、Rule条件で使う        |
| ⭐ Asymmetric Routing | 行き帰りが別経路          | Stateful FWで問題になる場合あり            |

🔥 **NAT＝住所変更。FW＝警備員。別物。**

---

# 6️⃣ DNS・DHCP 🌐

| 単語                  | 一言で              | どこでどう使う？                       |
| ------------------- | ---------------- | ------------------------------ |
| 🔥 DNS              | 名前解決             | 「IPなら○、名前なら×」で真っ先に疑う           |
| 🔥 A Record         | 名前→IPv4          | Server名のIPv4確認                 |
| ⭐ AAAA              | 名前→IPv6          | Dual Stack環境で確認                |
| ⭐ CNAME             | 別名→正式名           | Service Alias等                 |
| ⭐ PTR               | IP→名前            | Reverse Lookup                 |
| 🔥 DHCP             | IP設定を自動配布        | IP/Mask/GW/DNSをClientへ渡す       |
| ⭐ DORA              | DHCP取得の4段階       | Discover→Offer→Request→ACK     |
| 🔥 DHCP Scope       | 配布IP範囲・Option    | VLAN変更後にIP取れない時確認              |
| 🔥 DHCP Relay       | 別SubnetへDHCP中継   | DHCP Serverが別VLANなら重要          |
| ⭐ ip helper-address | CiscoのRelay設定で見る | DHCP Relay調査で遭遇                |
| ⭐ 169.254.x.x       | IPv4 Link-local  | WindowsではDHCP取得失敗の手掛かりになることが多い |
| ⭐ DNS Cache         | 名前解決結果の一時保存      | 古いIPを引いてる時候補                   |

Microsoftの2026年ガイドでも、DNSはClient/Server両側のIP設定・到達性、DHCPではCable/NIC/UDP67-68等を確認する。([Microsoft Learn][3])

---

# 7️⃣ Application寄り｜TLS・HTTP

| 単語            | 一言で                | どこでどう使う？                |
| ------------- | ------------------ | ----------------------- |
| ⭐ TLS         | HTTPS等の暗号化         | TCP443○なのにWeb×で次に見る     |
| ⭐ Certificate | Serverの証明書         | 期限・名前・CAなどでTLS失敗        |
| ⭐ SNI         | 接続先HostnameをTLSで通知 | 1IP複数HTTPS Site等        |
| 🔥 HTTP       | Web通信Protocol      | NWは正常でも4xx/5xxになる       |
| ⭐ 2xx         | HTTP成功系            | Request自体は成功            |
| ⭐ 4xx         | Client/認証等         | NW障害とは限らない              |
| ⭐ 5xx         | Server側エラー         | NetworkよりApplication側候補 |
| ⭐ Proxy       | Application通信を代理   | 社外Webに繋がらない等で確認         |

🔥 **TCP443○ ≠ HTTPS正常 ≠ Application正常。**

Microsoftも「Pingだけで全体の通信性を証明しない」として、Source→DestinationのTopologyとPort単位の確認を推奨。([Microsoft Learn][4])

---

# 8️⃣ Cisco CLI・Tera Term 🖥️

| 単語                           | 一言で             | どこでどう使う？              |
| ---------------------------- | --------------- | --------------------- |
| 🔥 Tera Term                 | Terminalソフト     | Cisco/LinuxへSSH接続     |
| 🔥 SSH                       | 暗号化管理接続         | Cisco管理では基本SSH v2     |
| 🔥 `>`                       | User EXEC       | 権限低めのCLI状態            |
| 🔥 `#`                       | Privileged EXEC | show確認など              |
| ⚠️ `(config)#`               | Config Mode     | 設定変更モード。調査だけなら入らない    |
| 🔥 running-config            | 現在の設定           | 「今どう動いてる？」            |
| 🔥 startup-config            | 保存済み設定          | 再起動後に読む設定             |
| 🔥 `show interfaces status`  | Port一覧          | Up/Down・VLAN等をざっくり    |
| 🔥 `show interfaces <IF>`    | Port詳細          | Error・CRC・Speed等      |
| 🔥 `show vlan brief`         | VLAN一覧          | VLAN存在・Access Portを見る |
| 🔥 `show interfaces trunk`   | Trunk確認         | Allowed VLANを見る       |
| 🔥 `show mac address-table`  | MAC→Port        | 接続端末の追跡               |
| 🔥 `show ip arp`             | IP→MAC          | ARP確認                 |
| 🔥 `show ip interface brief` | L3 IF一覧         | SVI/IP/up-down        |
| 🔥 `show ip route`           | Route確認         | 宛先への道を見る              |
| 🔥 `show logging`            | Cisco Log       | 障害時刻とLink変動等を照合       |
| ⭐ `show clock`               | 機器時刻            | Log時刻が正しいか確認          |
| ⭐ `show version`             | 機種/OS/Uptime    | 接続先確認・再起動確認           |

この一式が添付資料のRead-only基本セット。 Cisco IOS XE 26.xでも `show interfaces` / `show ip interface brief` は現行。([Cisco SPG][5])

---

# 9️⃣ 監視・ログ 📈

| 単語                | 一言で      | どこでどう使う？                 |
| ----------------- | -------- | ------------------------ |
| 🔥 SNMP           | 状態・数値を取得 | OpManager等が機器監視に利用       |
| 🔥 syslog         | 出来事のLog  | Link Down/UpやErrorを時系列確認 |
| 🔥 NTP            | 時刻同期     | Log時刻を合わせる。超重要           |
| 🔥 Alert          | 監視条件にHit | 「障害確定」ではなく調査開始           |
| ⭐ Event           | 起きた出来事   | 正常な状態変化も含む               |
| 🔥 Latency        | 遅延       | 「遅い」を数値化                 |
| 🔥 Packet Loss    | Packet欠損 | 通信品質・Timeout等に影響         |
| ⭐ Jitter          | 遅延の揺れ    | Voice/Videoで重要           |
| 🔥 Utilization    | 回線使用率    | 帯域逼迫を確認                  |
| ⭐ Throughput      | 実際の転送量   | Bandwidthとの違いに注意         |
| 🔥 Error          | CRC等の異常  | 物理系調査                    |
| 🔥 Discard / Drop | Packet破棄 | 輻輳・Queue・Policy等         |
| ⭐ Baseline        | 平常時の基準値  | 「今高い？」を判断する比較対象          |

🔥 **「値が高い」ではなく「平常時より増えた？」を見る。** 

---

# 🔟 運用・障害対応 🛡️

| 単語            | 一言で          | どこでどう使う？              |
| ------------- | ------------ | --------------------- |
| 🔥 Evidence   | 作業・調査証跡      | Command結果・Log・時刻を残す   |
| 🔥 Incident   | Service影響の障害 | まず安全な復旧を優先            |
| ⭐ Problem     | 根本原因の調査      | 再発防止・恒久対応             |
| 🔥 Change     | 管理された変更      | 承認・手順・検証・Rollbackがセット |
| 🔥 Rollback   | 切戻し          | 変更失敗時に元へ戻す            |
| 🔥 Backup     | 変更前状態保存      | Config変更前に取る          |
| 🔥 Escalation | 上位/担当へ連携     | 勝手に触らず証拠付きで相談         |
| ⭐ OOB         | 本番NW外の管理経路   | NW設定ミスでSSH切断した時の逃げ道   |
| ⭐ Console     | 直接CLI接続      | Network経由で入れない時       |
| 🔥 Impact     | 影響範囲         | 1人？1 VLAN？1拠点？全社？     |
| 🔥 Root Cause | 根本原因         | 推測と事実を混ぜない            |

障害時は **範囲→Path→Layer→戻り→Evidence**、変更ではBackup・Rollback・承認が軸。

---

# 1️⃣1️⃣ 頻出Portだけ暗記 🚪

|             Port | 用途           |
| ---------------: | ------------ |
|        🔥 TCP 22 | SSH          |
|    🔥 TCP/UDP 53 | DNS          |
|     🔥 UDP 67/68 | DHCP         |
|        🔥 TCP 80 | HTTP         |
|       🔥 UDP 123 | NTP          |
|    ⭐ UDP 161/162 | SNMP         |
|       🔥 TCP 443 | HTTPS        |
|        ⭐ TCP 445 | SMB          |
| ⭐ UDP 514 / TCP等 | syslog ※構成依存 |
|  ⭐ UDP 1812/1813 | RADIUS       |
|       ⭐ TCP 3389 | RDP          |

添付資料でもこの辺が現場用の主要Portとして整理されてる。

---

# ⚡ 最優先20語だけ抜くなら

これが**参画前のガチ暗記枠**。

```text
VLAN        → L2を分割
Access      → 基本1 VLAN
Trunk       → 複数VLANを運ぶ
MAC Table   → MAC→Port
ARP         → IP→MAC
STP         → L2 Loop防止

IP/Subnet   → L3の住所と範囲
Gateway     → 別NWへの出口
SVI         → VLANのL3出口
Route       → 宛先への道
Next Hop    → 次に渡す相手

TCP/UDP     → L4通信方式
Port        → Service受付番号
RST         → TCP Reset
Timeout     → 返事なし

FW/ACL      → 通信制御
NAT         → IP/Port変換

DNS         → 名前解決
DHCP        → IP設定自動配布

syslog      → 出来事のLog
```

この20個を**「一言＋どこで使うか」まで口で言えれば、最低限の会話についていく力はかなり上がる**。
その次にCiscoの`show`系を「コマンド暗記」じゃなく、**何を見たい時に使うか**で覚えるのが一番効率いい。

[1]: https://www.cisco.com/c/en/us/support/switches/catalyst-9300-series-switches/products-installation-and-configuration-guides-list.html?utm_source=chatgpt.com "Cisco Catalyst 9300 Series Switches - Configuration Guides - Cisco"
[2]: https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/26-x/configuration_guide/int_hw/b_26x_int_and_hw_9300_cg/configuring_interface_characteristics.html?utm_source=chatgpt.com "Interface and Hardware Components Configuration Guide, Cisco IOS XE 26.x.x (Catalyst 9300 Switches) - Configuring Interface Characteristics [Cisco Catalyst 9300 Series Switches] - Cisco"
[3]: https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/troubleshoot-dns-guidance?utm_source=chatgpt.com "Guidance for troubleshooting DNS - Windows Server | Microsoft Learn"
[4]: https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/troubleshoot-tcp-ip-communication-guidance?utm_source=chatgpt.com "Guidance for troubleshooting TCP/IP communication - Windows Server | Microsoft Learn"
[5]: https://spg.xgslb-v3.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/26-x/command_reference/b_26x_9300_cr/interface_and_hardware_commands.html?utm_source=chatgpt.com "Command Reference, Cisco IOS XE 26.x.x (Catalyst 9300 Switches) - Interface and Hardware Commands [Cisco Catalyst 9300 Series Switches] - Cisco"
