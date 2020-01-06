# 1. Basic setting 
- Notes
```
    username [username] secret [password] → clear text
    username [username] password [password] → md5
```
- Hostname
```
    (config) hostname Fred
```
- Password for enable mode
```
    enable secret cs
    enable password cs
```
- Telnet
```
    (config) line vty 0 4
    (config-if) password momo
    (config-if) login
```
- Telnet (username)
```
    (config) username morishima password momo
    (config) line vty 0 4
    (config-if) login local
```
- SSH (username)
```
    (config) hostname SW1
    (config) ip domain-name example.com
    (config) crypto key generate rsa
    (config) ip ssh version 2
    (config) username morishima password momo
    (config) line vty 0 4
    (config) transport input ssh | Allow only ssh
    (config-if) login local
```
- Verification
```
    # show running-config
    # show ip ssh
```

# 2. Switch

## Basic

- Static IP address configuration
```
    # configure terminal
    (config) interface vlan 1
    (config-if) ip address 192.168.1.200 255.255.255.0
    (config-if) no shutdown
    (config-if) exit
    (config) ip default-gateway 192.168.1.1

    # show interfaces vlan 1
```

- Dynamic IP address configuration
```
    (config) interface vlan 1
    (config-if) ip address dhcp
    (config-if) no shutdown

    # show interfaces vlan 1
    # show dhcp lease
```

- Speed, duplex, description
```
    (config-if) speed 100 (no speed)
    (config-if) duplex full (no duplex)

    # show interfaces status

    (config-if) description Printer on 3rd floor .... (no description)

    # show interfaces fastEthernet 0/1
```

- Port security
```
    (config-if) switchport mode access
    (config-if) switchport port-security
    (config-if) switchport port-security mac-address [0200.1111.1111]

    # show port-security interfaces fastEthernet 0/1
```

- MAC address
```
    # show mac address-table
```

## VLAN

- basic
```
    (config) vlan 2
    (config-vlan) name morishima-vlan
    (config-vlan) exit
    (config) interface range fastethernet 0/13 - 14
    (config-if-range) switchport access vlan 2
    (config-if-range) switchport mode access

    # show vlan
    # show vlan brief
```
- trunking, dynamic auto, dynamic desirable
    - 両デバイスがaccessでないときにaccessになるのはdynamic autox2のみ
```
    (config-if) switchport mode trunk|dynamic auto|dynamic desirable
        # show interfaces trunk
        # show interfaces switchport
```

- voice
```
    (config-if) switchport voice vlan 2

    # show interfaces switchport
```

## Spanning Tree Protocol (STP) and Rapid STP (RSTP)

- IEEE 802.1D Spanning-Tree States

| State | Forwards Data Frames? | Learns MACs Based on Received Frames? | Transitory or Stable State? |
| ---- | ---- | ---- | ---- |
| Blocking | No | No | Stable |
| Listening | No | No | Transitory |
| Learning | No | Yes | Transitory |
| Forwarding | Yes | Yes | Stable |
| Disabled | No | No | Stable |

- Port Roles in 802.1D STP

| Function | Port Role |
| ---- | ---- |
| Nonroot switch's best path to the root | Root port |
| Switch port designated to forward onto a collision domain | Designated port |
| Switch port that was not chonsen as RP nor DP | NonDesignated port |

- Port Roles in 802.1w RSTP

| Function | Port Role |
| ---- | ---- |
| Nonroot switch's best path to the root | Root port |
| Switch port designated to forward onto a collision domain | Designated port |
| Replaces the root port when the root port fails | Alternate port |
| Replaces a designated port when a designated port fails | Backup port |

- Port States Compared: 802.1D STP and 802.1w RSTP

| Function | 802.1D State | 802.1w State |
| ---- | ---- | ---- |
| Port is administratively disavled | Disabled | Discarding |
| Stable state that ignores incoming data frames and is not used to forward data frames | Blocking | Discarding |
| Interim state without MAC learning and without forwarding | Listening | Not used |
| Interim state with MAC learning and without forwarding | Learning | Learning |
| Stable state that allows MAC learning and forwarding of data frames | Forwaring | Forwarding |

- RSTP Port Types

| Type | Current Duplex Status | Is Spanning-Tree PortFast Configured? |
| ---- | ---- | ---- |
| Point-to-point | Full | No |
| Point-to-point edge | Full | Yes |
| Shared | Half | No |
| Shared edge | Half | Yes |

-  Enable STP
```
    (config) spanning-tree vlan 10
    (config) spanning-tree mode [ pvst | rapid-pvst | mst ]
```

- Disable STP
```
    (config) no spanning-tree vlan 10
```

- Bridge priority
```
! 4096単位
    (config) spanning-tree vlan 10 priority [priority_number]
    (config) spanning-tree vlan 10 root [ primary | secondary ]
```

- Interface cost
    - 100 for 10 Mbps
    - 19 for 100 Mbps
    - 4 for 1 Gbps
    - 2 for 10 Gbps
```
    (config-if) spanning-tree vlan 10 cost [cost]
```

- PortFast
    - パソコンとつなぐポートはフォワーディング
```
    (config) spanning-tree portfast default OR
    (config-if) spanning-tree portfast
```

- BPDU Guard
    - PortFastポートがBPDU(STPのメッセ)を受信したらブロッキングする
```
    (config-if) spanning-tree bpduguard [ enable | disable ]
```

- Dynamic root bridge あまり使わない
```
    (config) spanning-tree 10 root primary
```

- Dynamic secondary root bridge あまり使わない
```
    (config) spanning-tree 10 root secondary

    # show spanning-tree
```

## EtherChannel
- 複数の物理リンクを束ねて1つの論理リンクにする
- PAgP desirable - desirable | auto
    - シスコ独自プロトコル
- LACP active - active | passive
    - IEEE802.3ad
- EtherChannelを形成するマシンのどちらかは desirable | active にする必要がある

```
    (config) interface fa 0/1
    (config-if) channel-group 1 mode [ auto | desirable | on | active | passive ]
    (config) interface fa 0/2
    (config-if) channel-group 1 mode [ auto | desirable | on | active | passive ]
! mode on が多い

    # show spanning-tree
    # show etherchannel summary
```

## VLAN Trunking Protocol - (VTP)

- Version
```
    (config) vtp version [ 1 | 2 | 3 ]
! version 1,2 ... Extended VLAN not allowd
! version 3 ... Extended VLAN allowd
```

- Domain
```
    (config) vtp domain domainname
```

- Mode
```
    (config) vtp mode [ server | client | transparent ]
```

- Password
```
    (config) vtp password password
```

- VTP pruning
    - 不要なVLANトラフィックの伝送を動的に止める機能
```
    (config) (no) vtp pruning

    # show vtp status
```

# 3. Router

## Basic

- Static IP address
```
    (config-if) ip address 172.16.1.1 255.255.0.0
    (config-if) no shutdown

    # show protocols

    (config-if) ipv6 address 2001:DB8:1111:1::1/64
    (config-if) ipv6 address 2001:DB8:1111:1::/64 eui-64

    # show ipv6 interface gigabitEthernet 0/0
```

#- IPv6 DHCP and SLAAC
#```
#    (config) ipv6 address dhcp
#    (config) ipv6 address autoconfig
#```

- Serial DCE DTE(no clock) DB60
    - DTE -> (シリアルケーブル) -> DCE -> キャリア網 -> DCE -> DTE
    - DCEからDTEにクロック供給
```
    (config-if) clock rate 128000 (bps)
    (config-if) bandwidth 1544 (kbps)
```

- Static route
```
    - (config) ip route 192.168.99.0 255.255.255.0 [next-hop ip addr] or [interface]
    - (config) ip route 0.0.0.0 0.0.0.0 [next-hop ip addr] or [interface]

    # show ip route

    - (config) ipv6 route 8163:bc23:4632:ac23::/64 Gi0/0 fe80::1234:7484:26b4:2362
! Link local address は Interface も必要

    # show ipv6 route
```

## VLAN trunking

- ISL: Cisco’s protocol, no support native VLAN
- 802.1Q: support native VLAN, no tag for native VLAN
```
    (config) interface gigabitethernet 0/0.10
    (config-subif) encapsulation dot1Q 10
    (config-subif) ip address 192.168.10.0 255.255.255.0

    # show vlan
```

## RIPv2
```
    (config) router rip
    (config-router) version 2
    (config-router) network 192.168.0.0 (must be classful)

    # show ip protocols
    # show ip rip database

    (config-router) passive-interface GigabitEthernet0/0
! RIPのルーティングテーブルやHelloメッセージを出さない
! ホストと接続しているインタフェースに使う
```
## DHCP server
```
    (config) ip dhcp excluded-address 192.168.20.1 192.168.20.100
    (config) ip dhcp pool [pool-name]
    (dhcp-config) network 192.168.20.0 255.255.255.0
    (dhcp-config) default-router 192.168.20.1

    # show ip dhcp binding
    # show ip dhcp pool [pool-name]

! Relay
    (config-if) ip helper-address [DHCP server’s IP address]
    (config-if) ipv6 dhcp relay destination [DHCP server’s IP address]
```

## DNS
```
    (config) ip name-server dns1-address, dns2-address, ...
```

## ACL
- Standard
    - 1-99, 1300-1999
    - Match source IP
- Extended
    - 100-199, 2000-2699

```
    (config) access-list 1 permit 192.168.10.0 0.0.0.255
    (config) access-list 1 permit 192.168.20.0 0.0.0.255
    (config) interface gigabitEthernet 0/0
    (config-if) ip access-group 1 in | Apply
        or
    (config-if) ip access-list extended [name] (複数行設定)

    # show access-lists
    # show ip access-lists (only IPv4)
    # show ip interface gigabitEthernet 0/0
```
- Deleting
```
    (config) no access-list101 | Delete everything
        or
    (config) ip access-list [ "standard" | "extended" ] [ number | name ]
    (config-ext-nacl) no 20
```
- telnet, sshでログインできる機器の制御
```
    Cisco(config)# line vty 0 15
    Cisco(config-line)# access-class 1 in
```

## NAT

- Static
```
    (config) ip nat inside source static 10.1.1.10 200.1.1.10
    (config-if) ip nat inside
    (config-if) ip nat outside
```

- Dynamic
```
    (config) ip nat pool fred 200.1.1.1 200.1.1.2 netmask 255.255.255.252
    (config) ip nat inside source list 1 pool fred
    (config) ip nat inside source list 1 pool fred overload (PAT)
    (config) access-list 1 permit 10.1.1.1
    (config) access-list 1 permit 10.1.1.2
    (config-if) ip nat inside
    (config-if) ip nat outside

    # show ip nat translations
    # show ip nat statistics
```

## Device Management

- logging console: realtime log for console, default
- logging monitor: realtime log for terminal, default
- terminal monitor: debug command output, not default
- logging buffered: save log to RAM for later view
```
! LOG
    # debug ip rip
```

- NTP
```
    (config) ntp server 172.16.1.100 (both client and server)
    (config) ntp master 2 (server)
```

## License

- right-to-use
```
    license boot module c2900 technology-package [technology-package]
```

- paid-for
```
    license install url
```

## OSPF
1. Helloパケットによるネイバー関係の確立

    Init -> 2-Way -> Full

2. Link State Advertisement (LSA) 交換
3. Link State Database (LSDB) 作成
4. トポロジマップ作成
5. Shortest Path First (SPF) アルゴリズムを用いてSPFツリーを作成
6. 最小コストのパスがルーティングテーブルに追加される

- Area

    LSAを交換する範囲を示す論理グループ，もしくは同じLSDBを持つOSPF
　ルータの論理グループ

- Single area and Multi area
    - Single area 

        area 0 のみで構成

    - Multi area

        ルータの負荷軽減

- OSPF ルータの種類

| ルータ | 説明 |
| ---- | ---- |
| 内部ルータ | 全てのインターフェースを同じエリアに接続しているルータ |
| バックボーンルータ | 1つ以上のインターフェースをバックボーンエリアに接続しているルータ |
| Area Border Router (ABR) | 異なるエリアを接続しているルータ |
| AS Boundary Router (ASBR) | 1つ以上のインターフェースが外部ASのルータと接続しているルータ |

- Designated Router (DR) and Backup Designated Router (BDR)
    - DR 代表ルータ
        - プライオリティが最も大きいルータ
        - 同じ場合、ルータIDが大きいルータ
    - BDR
    - DROTHER

- Note
    - Delayはパス選択に影響しない

- Basic setting
```
    (config) router ospf [process-id]
    (config-router) router-id 1.1.1.1 
    (config-router) network 10.24.55.10 0.0.0.0 area [area-id]
! ワイルドカードでインターフェースを指定

! New style
    (config) interface GigabitEthernet0/0
    (config-if) ip address 10.1.23.2 255.255.255.0
    (config-if) ip ospf 1 area 23

    # show ip protocols
    # show ip ospf
    # show ip ospf interface brief
    # show ip ospf interface GigabitEthernet 0/0

    # show ip ospf neighbor
    # show ip ospf neighbor serial0/0
    # show ip ospf database
```
- Passive interface
    - PCと接続しているインターフェースからはHelloパケットを送らない
```
    (config) router ospf [process-id]
    (config-router) passive-interface GigabitEthernet 0/0 
! OR
    (config) router ospf [process-id]
    (config-router) passive-interface default 
    (config-router) no passive-interface vlan 10 
```

## EIGRP
- Diffusing Update Algorithm (DUAL)

- 用語
    - Successor Route (S) : プライマリルート, 最小のFD
    - Feasible Successor Route (FS) : バックアップルート, SのFDより小さなADのルート
    - Advertised (Reported) Distance (AD) : Neighbor to Dst
    - Feasible Distance (FD) : Localhost to Dst

- Basic setting
```
    (config) router eigrp [AS number]
! BGPのASとは違う
! Must be same as neighbor's AS number
    (config-router) network 10.24.55.10 0.0.0.0
! ワイルドカードでインターフェースを指定

    # show running-config
    # show ip protocols
    # show ip eigrp interfaces

    # show ip eigrp neighbors 
    # show ip eigrp topology 
        ! What you see
        via 192.168.1.2 (FD/AD)
```
- Optional
```
! Bandwidth
! メトリック計算用
    (config-if) bandwidth [kbps]

! Load balancing
! ルーティングテーブルに格納する同じメトリック値のルート数
    (config-router) maximum-paths [value = 4]

! Unequal-cost load balancing
! 最小メトリックの2倍の範囲までルーティングテーブルに格納
! S,FSのみ
    (config-router) variance 2

! Auto-summary
! Enable
    (config-router) auto-summary 
! Disable
    (config-router) no auto-summary 

! Debug
    # debug eigrp fsm
```

## BGP
- Basic
```
! only 1 BGP routing process per router
    (config) router bgp [AS number]
! must configure neighbor manually
    (config-router) neighbor [ip address] remote-as [AS number]
! 自分のAS内のルート情報をBGPルートとしてアドバタイズ
    (config-router) network [ip address] mask [mask]

    # show ip bgp summary
    # show ip bgp
    # show tcp brief
```
- Option
```
! Disable neighbor
    (config-router) neighbor 10.24.55.10 shutdown
```

## WAN

- DTE -> DCE -> キャリア網 -> DCE -> DTE

| WAN device | 説明 |
| ---- | ---- |
| DTE | Data Terminal Equipment. ルータやパソコン |
| DCE | Data Circuit-Terminating Equipment. DTEから送られる信号をDCEが接続している網に適した信号に変換して送信。WANの網から送られてくる信号をDTEに適した信号に変換し送信。WANがシリアル回線の場合、DCEはクロック信号を送信する。DCEはモデム(アナログ)、CSU/DSU(デジタル)、ONUが該当。|

- HDLC
- PPP
    - 認証
        - PAP
            - Clear text
        - CHAP
            - Use hash
            - 両方のルータでパスワードを設定しておく
    - 圧縮
    - マルチリンク
    - エラー制御

- Customer Edge (CE)
- Provider Edge (PE)

### High-Level Data Link Control (HDLC)
- Basic
```
    (config) interface s0/0
    (config-if) ip address 192.168.1.1 mask 255.255.255.0
    (config-if) encapsulation hdlc

! optional 
! if the serial link is a back-to-back serial link
! Use this command on DCE
    (config-if) clock rate 128000 (bps)
    # show controllers serial0/0

! optional 
    (config-if) bandwidth 1544 (kbps)
```

### Point-to-Point Protocol (PPP)
- Basic
```
    (config) interface s0/0
    (config-if) ip address 192.168.1.1 mask 255.255.255.0
    (config-if) encapsulation ppp 
```
- CHAP
    - secure
```
    (config) hostname R1
! username must match remote hostname
! password must match remote router's configuration
    (config) username R2 password morishima 
    (config) interface serial0/0
    (config-if) ip address 192.168.1.1 255.255.255.0
    (config-if) encapsulation ppp
    (config-if) ppp authentication chap

    # show ppp all
```
- PAP
    - clear text password
```
    (config) hostname R1
! username must match remote hostname
! password must match remote router's "ppp pap ... " command 
    (config) username R2 password pass2 
    (config) interface serial0/0
    (config-if) ip address 192.168.1.1 255.255.255.0
    (config-if) encapsulation ppp
    (config-if) ppp authentication pap
    (config-if) ppp pap sent-username R1 pass1
```
- Multilink PPP (MLPPP)
```
! Create multilink interface
    (config) interface multilink [bundle-number]
    (config-if) encapsulation ppp
    (config-if) ppp multilink
    (config-if) ip address 192.168.1.10 255.255.255.0
! group-number must match budle-number and remote as well
    (config-if) ppp multilink group [group-number] 
! if use CHAP
    (config-if) ppp authentication chap

! Assign physical interface
! multilink interfaceと違うのはip addressの設定だけ
    (config) interface serial0/0
    (config-if) encapsulation ppp
    (config-if) ppp multilink
    (config-if) no ip address
    (config-if) ppp multilink group [group-number]

    # show ip route
    # show interfaces multilink 1
    # show ppp multilink
```

## VPN

### General Routing Encapsulation (GRE) tunnel
- Basic
```
　  (config) interface tunnel [number]
　  (config-if) ip address 10.1.1.1 255.255.255.0
　  (config-if) tunnel source 1.1.1.1
! remote router's ip address
　  (config-if) tunnel destination 2.2.2.2

    # show ip interface brief
    # show interfaces tunnel0
```
- ACL that permit GRE
```
    (config) ip access-list extended inbound-from-Internet
    (config-ext-nacl) permit tcp
    (config-ext-nacl) permit udp
    (config-ext-nacl) permit gre any any 
    (config-ext-nacl) interface s0/0/0
    (config-if) ip access-group inbound-from-Internet in 
```

### Dynamic Multipoint VPN (DMVPN)
- ハブアンドスポーク
    - スポーク拠点間の通信のためにハブ拠点経由するので遅延が発生
- フルメッシュ
    - コンフィグ量が多くなり管理上の手間が発生
- DMVPNのソリューション
    - オンデマンドにスポーク間VPNを構築
    - Next Hop Resolution Protocol (NHRP) でスポークはハブに自身のIPアドレスを登録
    - NHRPでハブは他のスポークにそのIPアドレスを通知

### PPP over Ethernet (PPPoE)
- Basic
```
    (config) interface dialer 2
    (config-if) ip address negotiated
    (config-if) mtu 1492
    (config-if) encapsulation ppp
! hostname given from ISP
    (config-if) ppp chap hostname Fred
! password given from ISP
    (config-if) ppp chap password Barney 
    (config-if) dialer pool 1

    (config) interface GigabitEthernet 0/0
    (config-if) no ip address 
    (config-if) pppoe enable
    (config-if) pppoe-client dial-pool-number 1

    # show pppoe session
    # show interface dialer 2
    # show interface virtual-access 2 configuration
```

## QoS
- What manages
    - Bandwidth
    - Delay
    - Jitter
    - Loss

- QoSのモデル

| モデル | 説明 |
| ---- | ---- |
| Integrated Services (IntServ) | アプリケーションの通信フローごとに帯域を予約する方式 |
| Differentiated Services (DIffServ) | トラフィックを分類、マーキング（優先度付け）、キューイング（トラフィックのキューへの振り分け）、スケジューリング（キューの優先度に応じたパケットの送出）する |
| Best Effort | FIFO |

1. Classification
2. Marking
    - L2 Marking
        - End to End でない
        - CoS: 802.1Qタグ内の3bitフィールド
    - L3 Marking
        - IP Precedence (IPP)
            - IPv4ヘッダ内のType of Service (ToS) フィールド先頭3bit
        - Defferentiated Services Code Point (DSCP)
            - Per Hop Behavior (PHB) DSCP値により処理を決めること
            - IPv4ヘッダ内のType of Service (ToS) フィールド先頭6bit
3. Queueing キューにパケットを格納すること
4. Scheduling どのキューにあるパケットから送出していくのか

- Shaping

    定義した通信速度を超過した場合、それ以上のパケットはバッファに格納しておいて通信速度の超過が落ち着いてきてからトラフィックを送信していく方式


- Policing

    定義した通信速度を超過した場合、それ以上のパケットを破棄する。必ず破棄するわけではなく，バーストは許すときがある

## Inter-VLAN routing

- Router
    - Router On A Stick (ROAS)
    
        1つの物理インタフェースに複数の論理インタフェースを作成し，それぞれにVLANを割り当てる(トランクリンク)

```
! 1. Create sub interface
    ! subinterface-number = 1
    (config) interface FastEthernet0/0.1
! 2. カプセル化タイプとVLAN IDの指定
    ! VLAN ID = 10
    (config-subif) encapsulation dot1q 10 
! 3. Configure IP address
    (config-subif) ip address 192.168.10.254 255.255.255.0
! 論理インタフェースをn個作成する場合，以上をn回行う
```

- Switch
    - Switch Virtual Interface (SVI) 
        - VLANごとに論理インタフェースを作成する
        - 1つの物理インタフェースで複数のVLANトラフィックを転送(トランスポート)

```
! IPルーティングの有効化
    (config) ip routing
! SVIの作成
    (config) interface vlan 10
    (config-if) ip address 192.168.10.254 255.255.255.0
    (config-if) no shutdown

    # show vlan
    # show interfaces status
    # show interface brief
    # show ip route
```

- Switch
    - Routed port
        - 1つの物理インタフェースで1つのVLANトラフィックのみ転送
        - L3 EtherChannel を作成するのに使用する
```
! IPルーティングの有効化
    (config) ip routing
! ルーテッドポートの作成 L3ポートになる
    (config) interface gigabitEthernet 0/1
    (config-if) no switchport 
    (config-if) ip address 192.168.10.254 255.255.255.0
    (config-if) no shutdown

    # show ip interface brief
    # show ip route
```

- L3 EtherChannelの設定
```
　  (config)# ip routing
! ルーテッドポートの作成
! 複数の物理インタフェースで設定する
　  (config)# interface interface-id
　  (config-if) no switchport
    ! group-number = 10
　  (config-if) channel-group 10 mode [ auto | desirable | on | active | passive ]

    ! port-channel = 10
　  (config) interface port-channel 10 
　  (config-if) no switchport
　  (config-if) ip address address mask
　  (config-if) no shutdown
```

## First Hop Redundancy Protocol (FHRP)

    デフォルトゲートウェイを冗長化させる技術

| Acronym | Full Name | Origin | Redundancy Approach | Load Balancing |
| ---- | ---- | ---- | ---- | ---- |
| HSRP | Hot Standby Router Protocol | Cisco | Active/standby | Per subnet |
| VRRP | Virtual Router Redundancy Protocol | RFC 5798 | Active/standby | Per subnet |
| GLBP | Gateway Load Balancing Protocol | Ciso | Active/active | Per host |

```
    ! R1
    (config-if) standby 1 ip 10.1.1.1
    ! higher is better
    (config-if) standby 1 priority 110 
    ! take over active when current avtive router's priority is lower than own priority
    (config-if) standby 1 preempt 
    ! R2
    (config-if) standby 1 ip 10.1.1.1
    (config-if) standby 1 priority 100 

    # show standby brief
    # show standby
        ! shows standby version
```

## IPv6

- Prefix
    - Link-local: FE80
    - Unique-local: FD (like Private in IPv4)
    - Global unicast: 2,3 (like Public in IPv4)
    - multicast: FF

- Enable routing IPv6 packets
```
    (config) ipv6 unicast-routing
```

- IPv6 addressing
    1. Stateless Address Auto Configuration (SLAAC)
        - Prefix is autoconfiguration
        - Interface ID is EUI-64
        - ホストはルータにRSを送信し，ルータはRAをホストに送信する
        - DHCPv6サーバが要らない
    1. Stateless DHCPv6
        - SLAACで求めたIPv6アドレスでは足りない情報をDHCPv6サーバから取得
        - DNSサーバの情報を入手
    1. Statefull DHCPv6
        - DHCPv6 provide prefix and interface ID
        - DNSサーバの情報を入手
        - IPv4と同じ動作

# 4. Note

## IPv6
- Neighbor Discovery Protocol - (NDP)
    - Neighbor Solicitation - (NS) : ARP request
    - Neighbor Advertisement - (NA) : ARP return
    - Router Solicitation - (RS) : Router info request, no MAC addr
    - Router Advertisement - (RA) : Router info return, no MAC addr
- SLAAC
    - Autoconfigration

- CDP 
```
    show cdp neighbors - (no ip address shown) 
    show cdp neighbors detail
    show cdp entry [name]
```

- Boot
    1. POST
    2. bootstrap ROM → RAM
    3. bootstrap loads the OS
    4. startup-config NVRAM → RAM
        - If boot field = 0, use the ROMMON OS
        - If boot field = 1, load the first IOS in flash
        - If boot field = 2-F,
            - try “boot system” command in NVRAM to load the first IOS in flash
        - else, load ROMMON mode
    - Configuration register 0x2100: ROMMON, last hex is 0

## Routing protocol

| ルート情報 | AD値 | 略字 |
| ---- | ---- | ---- |
| 直接接続ルート | 0 | C |
| スタティックルート | 1 | S |
| EIGRP - サマリールート | 5 | | 
| BGP - 外部 | 20 | |
| EIGRP - 内部 | 90 | D |
| IGRP | 100 | |
| OSPF | 110 | O |
| IS-IS | 115 | |
| RIP | 120 | R |
| ODR | 160 | |
| EIGRP - 外部 | 170 | EX |
| BGP - 内部 | 200 | |
| 不明 - ( Routing Table にのらない ) | 255 | |

- IGP (AS内で使用されるプロトコル)
    - Distance-vector 伝言ゲーム
        - RIPv1
            - Classful
            - Bellman-Ford
            - broadcase update
            - no VLSM
        - RIPv2
            - Classless
            - multicast update
            - VLSM
            - hop-count
            - split horizon
            - route poisoning
            - update messages 224.0.0.9
            - auto-summary (advertize classful subnet)
        - IGRP
            - Classful
    - Link-state 全てのルータが同じ情報を持つ
        - OSPF
            - Classful
        - IS-IS
            - Classful
    - Hybrid
        - EIGRP
            - Classless
- EGP (AS間で使用されるプロトコル)
    - Distance-vector
        - BGP
            - IBGP peer (same AS)
            - EBGP peer (other AS)

## STP

- PortFast
    - パソコンとつなぐポートはフォワーディング
- Bridge Protocol Data unit - (BPDU)
    - BPDU Guard
        - PortFastポートがBPDUを受信したらブロッキングする
- Rapid STP - (RSTP)
    - pvst
    - rapid-pvst
    - mst

## L2 Security
- IEEE802.1X
    - 有線LANや無線LANにおけるユーザ認証の規格
- AAA 3つのセキュリティ機能
    - Authentication
        - 認証
    - Authorization
        - 認証に成功したユーザに対して提供する権限
    - Accounting
- AAAの実装
    - RADIUS
        - IETF standard
        - UDP 1812, 1813
        - パスワード情報のみ暗号化
    - TACACS+
        - Cisco original
        - TCP 49
        - パケット全体を暗号化
        - ユーザごとに異なるCLI群の認証
- DHCP Snooping on switch 嗅ぐ->盗み見る
    - Trusted: Switch, Router, DHCP Server
    - Untrusted: Client device such as PC

## Misc

- Three tier
    - core, distribute, access
- Firewall
    - Stateful Inspection
- HDLC
    - type field
- Protocol and port number
    - POP3: TCP, 110
    - DNS: TCP and UDP, 53
    - FTP: TCP, 20 21
    - TFTP: UDP, 69
    - SMTP: TCP, 25
- RFC 1918: プライベートIPの割り当ての規約

- crossover cable
    - 1-3
    - 2-6

- rollover cable
    - 1-8
    - 2-7
    - 3-6
    - 4-5

- 802.3z: fiber optic

- PC, router 1-2 trans, 3-6 rcv
- switch 3-6 trans, 1-2 rcv 

- MAC address
    - Organizationally Unique Identifier (OUI) 3byte

- DSL -  telephone cable
    - DSL と Cable internet
        - asymmetric speed
    - Leased line 
        - symmetric speed

- TCP terminating
    - -> ACK FIN
    - ACK <-
    - ACK FIN <-
    - -> ACK

- 9600, None, 8, 1
    - 9600bps, No parity, 8 data bits, 1 stop bit

- User authentication on Cisco
    - RADIUS, TACACS+