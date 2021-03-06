1. # Basic setting 

    1. ## Notes
        ```
        username [username] secret [password] → clear text
        username [username] password [password] → md5
        ```
    1. ## Hostname
        ```
        (config) hostname Fred
        ```
    1. ## Password for enable mode
        ```
        enable secret cs
        enable password cs
        ```
    1. ## Telnet
        ```
        (config) line vty 0 4
        (config-if) password momo
        (config-if) login
        ```
    1. ## Telnet (username)
        ```
        (config) username morishima password momo
        (config) line vty 0 4
        (config-if) login local
        ```
    1. ## SSH (username)
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
    1. ## Verification
        ```
        # show running-config
        # show ip ssh
        ```

    1. ## Ping
        - Standard ping
        - Extended ping
            - can send a specific number of packet
            - can send packet from specified interface of IP address

1. # Switch

    1. ## Basic
        1. ### Static IP address configuration
            ```
            # configure terminal
            (config) interface vlan 1
            (config-if) ip address 192.168.1.200 255.255.255.0
            (config-if) no shutdown
            (config-if) exit
            (config) ip default-gateway 192.168.1.1

            # show interfaces vlan 1
            ```
        1. ### Dynamic IP address configuration
            ```
            (config) interface vlan 1
            (config-if) ip address dhcp
            (config-if) no shutdown

            # show interfaces vlan 1
            # show dhcp lease
            ```

        1. ### Speed, duplex, description
            ```
            (config-if) speed 100 (no speed)
            (config-if) duplex full (no duplex)
    
            # show interfaces status
    
            (config-if) description Printer on 3rd floor .... (no description)
    
            # show interfaces fastEthernet 0/1
            ```

        1. ### Port security
            ```
            (config-if) switchport mode access
            (config-if) switchport port-security
            (config-if) switchport port-security mac-address [0200.1111.1111]
    
            # show port-security interfaces fastEthernet 0/1
            ```

        1. ### MAC address
            ```
            # show mac address-table
            ```
    
    1. ## VLAN
        1. ### Basic
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

        1. ### trunking, dynamic auto, dynamic desirable
            両デバイスがaccessでないときにaccessになるのはdynamic autox2のみ
            ```
            (config-if) switchport mode trunk|dynamic auto|dynamic desirable
            !! shows native vlan
            # show interfaces trunk
            # show interfaces switchport

            !! Remove vlan 10 from trunk
            (config-if) switchport trunk allowd vlan remove 10
            !! OR
            (config-if) switchport trunk allowd vlan [all vlan except 10]
            ```
    
        1. ### voice
            ```
            (config-if) switchport voice vlan 2
    
            # show interfaces switchport
            ```
    
    1. ## Spanning Tree Protocol (STP) and Rapid STP (RSTP)
        1. ### IEEE 802.1D Spanning-Tree States
            | State | Forwards Data Frames? | Learns MACs Based on Received Frames? | Transitory or Stable State? |
            | ---- | ---- | ---- | ---- |
            | Blocking | No | No | Stable |
            | Listening | No | No | Transitory |
            | Learning | No | Yes | Transitory |
            | Forwarding | Yes | Yes | Stable |
            | Disabled | No | No | Stable |
            <br>
    
        1. ### Port Roles in 802.1D STP
            | Function | Port Role |
            | ---- | ---- |
            | Nonroot switch's best path to the root | Root port |
            | Switch port designated to forward onto a collision domain | Designated port |
            | Switch port that was not chonsen as RP nor DP | NonDesignated port |
            <br>
    
        1. ### Port Roles in 802.1w RSTP
            | Function | Port Role |
            | ---- | ---- |
            | Nonroot switch's best path to the root | Root port |
            | Switch port designated to forward onto a collision domain | Designated port |
            | Replaces the root port when the root port fails | Alternate port |
            | Replaces a designated port when a designated port fails | Backup port |
            <br>
    
        1. ### Port States Compared: 802.1D STP and 802.1w RSTP
            | Function | 802.1D State | 802.1w State |
            | ---- | ---- | ---- |
            | Port is administratively disavled | Disabled | Discarding |
            | Stable state that ignores incoming data frames and is not used to forward data frames | Blocking | Discarding |
            | Interim state without MAC learning and without forwarding | Listening | Not used |
            | Interim state with MAC learning and without forwarding | Learning | Learning |
            | Stable state that allows MAC learning and forwarding of data frames | Forwaring | Forwarding |
            <br>
    
        1. ### RSTP Port Types
            | Type | Current Duplex Status | Is Spanning-Tree PortFast Configured? |
            | ---- | ---- | ---- |
            | Point-to-point | Full | No |
            | Point-to-point edge | Full | Yes |
            | Shared | Half | No |
            | Shared edge | Half | Yes |
            <br>

        1. ### STPの流れ
            1. Root Bridge (RB) の選出
                - 最小のBridge ID
                - Bridge ID = Bridge priority + MAC address
            1. Root Port (RP) の選出
                - RBまでのコストが最小のインタフェース
            1. Designated Port (DP) の選出
                - RBのすべてのポートはDP
            1. Non-Designated Port (NDP) の選出
                - 残りのポート
        
        1. ### コストの計算
            - 10Gbps : 2
            - 1Gbps : 4
            - 100Mbps : 19
            - 10Mbps : 100
    
        1. ### Enable STP
            ```
            (config) spanning-tree vlan 10
            (config) spanning-tree mode [ pvst | rapid-pvst | mst ]
            ```
    
        1. ### Disable STP
            ```
            (config) no spanning-tree vlan 10
            ```
    
        1. ### Bridge priority
            ```
            !! 4096単位
            (config) spanning-tree vlan 10 priority [priority_number]
            (config) spanning-tree vlan 10 root [ primary | secondary ]
            ```
    
        1. ### Interface cost
            - 100 for 10 Mbps
            - 19 for 100 Mbps
            - 4 for 1 Gbps
            - 2 for 10 Gbps
            ```
            (config-if) spanning-tree vlan 10 cost [cost]
            ```
    
        1. ### PortFast
            パソコンとつなぐポートはlisteningやlearningステートにせずにフォワーディングにする
            ```
            (config-if) spanning-tree portfast
            !! OR
            !! configure all ports as portfast
            (config) spanning-tree portfast default
            ```
    
        1. ### BPDU Guard
            PortFastポートがBPDU(STPのメッセ)を受信したらブロッキングする
            ```
            (config-if) spanning-tree bpduguard [ enable | disable ]
            !! OR
            (config) spanning-tree portfast bpduguard default
            ```

        1. ### Root Guard
            周囲のスイッチがルートスイッチになることを防止するための機能。ルートガードが有効になっているポートで上位のBPDUを受信すると、そのポートは root-inconsistent ステートへ移行し意図しない周囲のスイッチが、ルートスイッチにならないようにする。
            ```
            (config-if) spanning-tree guard root
            ```
        1. ### Dynamic root bridge あまり使わない
            ```
            (config) spanning-tree 10 root primary
            ```
    
        1. ### Dynamic secondary root bridge あまり使わない
            ```
            (config) spanning-tree 10 root secondary
            # show spanning-tree
            ```
    
    1. ## EtherChannel
        - 複数の物理リンクを束ねて1つの論理リンクにする
        - LACP active - active | passive
            - IEEE802.3ad
            - EtherChannelの一端は2つのスイッチに分かれてもよい
        - PAgP desirable - desirable | auto
            - シスコ独自プロトコル
            - Multicast address 01-00-0C-CC-CC-CC
        - EtherChannelを形成するマシンのどちらかは desirable | active にする必要がある

        ```
        (config) interface fa 0/1
        (config-if) channel-group 1 mode [ auto | desirable | on | active | passive ]
        (config) interface fa 0/2
        (config-if) channel-group 1 mode [ auto | desirable | on | active | passive ]
        !! mode on が多い
        !! on は PAgP でも LACP でもない

        # show spanning-tree
        # show etherchannel summary
        ```

    1. ## VLAN Trunking Protocol - (VTP)

        - Dynamic Trunking Protocol (DTP)
            - デフォルトはDynamic Auto

        1. ### Setting
            ```
            !! version 1,2 ... Extended VLAN not allowd
            !! version 3 ... Extended VLAN allowd
            (config) vtp version [ 1 | 2 | 3 ]
            (config) vtp domain domainname
            (config) vtp mode [ server | client | transparent ]
            (config) vtp password password
            ```

        1. ### VTP pruning
            不要なVLANトラフィックの伝送を動的に止める機能
            ```
            (config) (no) vtp pruning
            # show vtp status
            ```
        
        1. ### 設定ファイル
            - vlan.datに保存される

    1. ## Attacks
        1. ### Double tagging
            - VLANタグを2つつける
            - native VLAN IDを変えるとよい

    1. ## Switch stacking
        1. ### Master election
            1. The switch with the highest stack member priority value
            1. The switch that has the configuration file
            1. The switch withing the highest uptime
            1. The switch with the lowerst MAC address
        1. ### 1:N
            1:N master redundancy allows each stack member to serve as a master, providing the highest reliability for forwarding. 

1. # Router

    1. ## Basic

        1. ### Static IP address
            ```
            (config-if) ip address 172.16.1.1 255.255.0.0
            (config-if) no shutdown

            # show protocols

            (config-if) ipv6 address 2001:DB8:1111:1::1/64
            (config-if) ipv6 address 2001:DB8:1111:1::/64 eui-64

            # show ipv6 interface gigabitEthernet 0/0
            ```

        1. ### Serial DCE DTE(no clock) DB60
            - DTE -> (シリアルケーブル) -> DCE -> キャリア網 -> DCE -> DTE
            - DCEからDTEにクロック供給
            ```
            (config-if) clock rate 128000 (bps)
            (config-if) bandwidth 1544 (kbps)
            ```

        1. ### Static route
            ```
            (config) ip route 192.168.99.0 255.255.255.0 [next-hop ip addr] or [interface]
            (config) ip route 0.0.0.0 0.0.0.0 [next-hop ip addr] or [interface]
            # show ip route

            (config) ipv6 route 8163:bc23:4632:ac23::/64 Gi0/0 fe80::1234:7484:26b4:2362
            !! Link local address は Interface も必要
            # show ipv6 route
            ```

        1. ### Proxy ARP
            本来の問い合わせ先に代わってARP応答する機能

    1. ## VLAN trunking
        - ISL: Cisco’s protocol, no support native VLAN
        - 802.1Q: support native VLAN, no tag for native VLAN
        ```
        (config) interface gigabitethernet 0/0.10
        (config-subif) encapsulation dot1Q 10
        (config-subif) ip address 192.168.10.0 255.255.255.0

        # show vlan
        ```

    1. ## RIPv2
        ```
        (config) router rip
        (config-router) version 2
        (config-router) network 192.168.0.0 (must be classful)

        # show ip protocols
        # show ip rip database

        (config-router) passive-interface GigabitEthernet0/0
        !! RIPのルーティングテーブルやHelloメッセージを出さない
        !! ホストと接続しているインタフェースに使う
        ```
    1. ## DHCP server
        ```
        (config) ip dhcp excluded-address 192.168.20.1 192.168.20.100
        (config) ip dhcp pool [pool-name]
        (dhcp-config) network 192.168.20.0 255.255.255.0
        (dhcp-config) default-router 192.168.20.1

        # show ip dhcp binding
        # show ip dhcp pool [pool-name]

        !! Relay
        (config-if) ip helper-address [DHCP server’s IP address]
        (config-if) ipv6 dhcp relay destination [DHCP server’s IP address]
        ```

    1. ## DNS
        ```
        (config) ip name-server dns1-address, dns2-address, ...
        ```

    1. ## ACL
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

        1. ### Deleting
            ```
            !! Delete everything
            (config) no access-list101
                or
            (config) ip access-list [ "standard" | "extended" ] [ number | name ]
            (config-ext-nacl) no 20
            ```

        1. ### telnet, sshでログインできる機器の制御
            ```
            Cisco(config)# line vty 0 15
            Cisco(config-line)# access-class 1 in
            ```

        1. ### IPv6 ACL
            - IPv4 ACLs can be identified by number or name, while IPv6 ACLs user names only
            - 各ACLの最後にNSとNAを許可する暗黙のpermit文がある
            ```
            (config) ipv6 access-list [acl-name]
            ```

        1. ### note
            - Port filters are applied as first
            - ACL must be numbered when applied to vty

    1. ## NAT
        1. ### Static
            ```
            (config) ip nat inside source static 10.1.1.10 200.1.1.10
            (config-if) ip nat inside
            (config-if) ip nat outside
            ```

        1. ### Dynamic
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

    1. ## Device Management
        - logging console: realtime log for console, default
        - logging monitor: realtime log for terminal, default
        - terminal monitor: debug command output, not default
        - logging buffered: save log to RAM for later view
        ```
        !! LOG
        # debug ip rip
        ```

        - NTP
        ```
        (config) ntp server 172.16.1.100 (both client and server)
        (config) ntp master 2 (server)
        ```

    1. ## License
        1. ### right-to-use
            ```
            license boot module c2900 technology-package [technology-package]
            ```

        1. ### paid-for
            ```
            license install url
            ```

    1. ## OSPF
        1. ### 用語
            - Area
                - LSAを交換する範囲を示す論理グループ，もしくは同じLSDBを持つOSPFルータの論理グループ．
                - ASより小さな範囲
            - Designated Router (DR)
                - 代表ルータ
                - プライオリティが最も大きいルータ
                - 同じ場合、ルータIDが大きいルータ
            - Backup Designated Router (BDR)
            - DROTHER
            - スタブルータ
                - 末端のルータ

        1. ### OSPFの流れ
            1. Helloパケットによるネイバー関係の確立
                - Init -> 2-Way (stable) -> Full (stable)
            2. Link State Advertisement (LSA) 交換
            3. Link State Database (LSDB) 作成
            4. トポロジマップ作成
            5. Shortest Path First (SPF) アルゴリズムを用いてSPFツリーを作成
            6. 最小コストのパスがルーティングテーブルに追加される
        
        1. ### データベース交換に使用されるOSPFメッセージ
            | Message | Descrption |
            | ---- | ---- |
            | Link-State Request (LSR) | Used by a router to request specific LSAs from a neighbor |
            | Link-State Update (LSU) | Used by a router to send LSAs to a neighbor |
            | Link-State Database Descriptor (DD) | Used by a router to send a summary list of LSAs to a neighbor, as part of the process of a router deciding which LSAs it should request with LSR packets.

        1. ### Neighborの条件
            - Interfaceがup/up
            - Interfaceが同じサブネット
            - ACLがOSPFのメッセージをフィルタしていない
            - Authenticationに成功する
            - Hello timerとDead timerが一致
            - Router IDがユニーク
                - process IDは同じでなくてよい

        1. ### Single area and Multi area
            - Single area 
                - area 0 のみで構成
            - Multi area
                - ルータの負荷軽減

        1. ### OSPF ルータの種類
            | ルータ | 説明 |
            | ---- | ---- |
            | 内部ルータ | 全てのインターフェースを同じエリアに接続しているルータ |
            | バックボーンルータ | 1つ以上のインターフェースをバックボーンエリアに接続しているルータ |
            | Area Border Router (ABR) | 異なるエリアを接続しているルータ |
            | AS Boundary Router (ASBR) | 1つ以上のインターフェースが外部ASのルータと接続しているルータ |
            <br>

        1. ### Note
            - Delayはパス選択に影響しない

        1. ### Basic setting
            ```
            (config) router ospf [process-id]
            (config-router) router-id 1.1.1.1 
            (config-router) network 10.24.55.10 0.0.0.0 area [area-id]
            !! ワイルドカードでインターフェースを指定

            !! New style
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
        1. ### Passive interface
            PCと接続しているインターフェースからはHelloパケットを送らない
            ```
            (config) router ospf [process-id]
            (config-router) passive-interface GigabitEthernet 0/0 
                OR
            (config) router ospf [process-id]
            (config-router) passive-interface default 
            (config-router) no passive-interface vlan 10 
            ```

        1. ### OSPFv3 (IPv6対応)
            - OSPFv2とLSAの構造が違う
            - Neighborになるために同じサブネットである必要はない 
            - IPv6 unicast routing をenableする必要がある
            ```
            (config) ipv6 router ospf [process-id]
            !! (config) router ospf [process-id]
            (config-router) router-id 1.1.1.1
            !! 共通

            (config) interface GigabitEthernet0/0
            (config-if) ipv6 ospf [process-id] area [area-number]
            !! (config-if) ip ospf 1 area 23

            # show ipv6 protocols
            # show ipv6 ospf
            # show ipv6 ospf interface brief
            # show ipv6 ospf interface GigabitEthernet 0/0
            # show ipv6 ospf neighbor
            # show ipv6 ospf neighbor serial0/0
            # show ipv6 ospf database
            ```
        
        1. ### show command
            ```
            # show ip ospf
            # show ip ospf database
            # show ip ospf interface
            # show ip ospf interface [interface]
            # show ip ospf neighbor
            # show ip ospf neighbor [interface]
            ```

    1. ## EIGRP
        1. ### 用語
            - Successor Route (S) : プライマリルート, 最小のFD
            - Feasible Successor Route (FS) : バックアップルート, SのFDより小さなADのルート
            - Advertised (Reported) Distance (AD) : Neighbor to Dst
            - Feasible Distance (FD) : Localhost to Dst
            - Diffusing Update Algorithm (DUAL)
            - スタブルータ: 末端のルータ

        1. ### Neighborの条件
            - Interfaceがup/up
            - Interfaceが同じサブネット
            - ACLがOSPFのメッセージをフィルタしていない
            - Authenticationに成功する
            - 同じASN
            - K value (メトリックの計算に使用される)が一致
                - Hello timerとDead timerが一致する必要はない
                - Router IDがユニークである必要はない
                - process IDは同じでなくてよい

        1. ### K value
            - K1 Bandwidth
            - K2 Load 
            - K3 Delay 
            - K4 Reliability 
            - K5 MTU 

        1. ### Basic setting
            ```
            !! BGPのASとは違う
            !! Must be same as neighbor's AS number
            (config) router eigrp [AS number]
            !! Enable EIGRP
            !! ワイルドカードでインターフェースを指定
            (config-router) network 10.24.55.10 0.0.0.0

            # show running-config
            # show ip protocols
            # show ip eigrp interfaces

            # show ip eigrp neighbors 
            # show ip eigrp topology 
            !! What you see
            !! via 192.168.1.2 (FD/AD)
            ```
        1. ### Optional setting
            ```
            !! Bandwidth
            !! メトリック計算用
            (config-if) bandwidth [kbps]

            !! Load balancing
            !! ルーティングテーブルに格納する同じメトリック値のルート数
            (config-router) maximum-paths [value = 4]

            !! Unequal-cost load balancing
            !! 最小メトリックの2倍の範囲までルーティングテーブルに格納
            !! S,FSのみ
            (config-router) variance 2

            !! Auto-summary
            !! Enable
            (config-router) auto-summary 
            !! Disable
            (config-router) no auto-summary 

            !! Change Hello and hold timers
            (config-if) ip hello-interval eigrp [asn-time]
            (config-if) ip hold-time eigrp [asn-time]

            !! Debug
            # debug eigrp fsm
            ```

        1. ### EIGRP for IPv6 (EIGRPv6)
            - Neighborになるために同じサブネットである必要はない 
            - interface bandwidth コマンドは共通
            ```
            !! Create process
            !! (config) router eigrp [as-number]
            (config) ipv6 router eigrp [as-number]
            (config-router) eigrp router-id 1.1.1.1

            !! Enable EIGRP
            (config-if) ipv6 eigrp [as-number]
            ```

        1. ### show command
            ```
            # show ip eigrp database
            # show ip eigrp interfaces [AS-number]
            # show ip eigrp neighbors [AS-number]
            # show ip eigrp topology [AS-number]
            ```

    1. ## BGP

        1. ### Neighborのステート
            | State | Description |
            | ---- | ---- |
            | Idle | Refuse connections |
            | Connect | Wait for a TCP connection to be completed. |
            | Active | Try TCP handshake. Listen for and accept connection. |
            | OpenSent | An OPEN message was sent. Wait for an OPEN message. |
            | OpenConfirm | Received a reply to the OPEN message. Wait for a KEEPALIVE or NOTIFICATION message |
            | Established | UPDATE, NOTIFICATION, and KEEPALIVE message are exchanged with peers. |
            <br>
        
        1. ### Neighbor確立の条件
            - BGP version が一致

        1. ### Basic
            ```
            !! only 1 BGP routing process per router
            (config) router bgp [AS number]
            !! must configure neighbor manually
            (config-router) neighbor [ip address] remote-as [AS number]
            !! 自分のAS内のルート情報をBGPルートとしてアドバタイズ
            (config-router) network [ip address] mask [mask]

            # show ip bgp summary
            # show ip bgp
            # show tcp brief
            ```
        1. ### Option
            ```
            !! Disable neighbor
            (config-router) neighbor 10.24.55.10 shutdown
            !! Enable the exchange of information
            (config-router) neighbor [ip-address] activate
            ```

    1. ## WAN
        1. ### Basic information
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

        1. ### High-Level Data Link Control (HDLC)
            - keep-alive should be equal on both side
            - Cisco default encapsulation type on serial interfaces
            - Basic setting
            ```
            (config) interface s0/0
            (config-if) ip address 192.168.1.1 mask 255.255.255.0
            (config-if) encapsulation hdlc

            !! optional 
            !! if the serial link is a back-to-back serial link
            !! Use this command on DCE
            (config-if) clock rate 128000 (bps)
            # show controllers serial0/0

            !! optional 
            (config-if) bandwidth 1544 (kbps)
            ```

        1. ### Point-to-Point Protocol (PPP)
            - Basic setting
            ```
            (config) interface s0/0
            (config-if) ip address 192.168.1.1 mask 255.255.255.0
            (config-if) encapsulation ppp 
            ```
            1. #### CHAP
                - secure
                - md5
                - One way authentication
                - Link Control Protocol (LCP) -> CHAP
                ```
                (config) hostname R1
                !! username must match remote hostname
                !! password must match remote router's configuration
                (config) username R2 password morishima 
                (config) interface serial0/0
                (config-if) ip address 192.168.1.1 255.255.255.0
                (config-if) encapsulation ppp
                (config-if) ppp authentication chap

                # show ppp all
                ```

                - ユーザ名とパスワード
                ```
                    !! 双方向1
                    (config) hostname R1
                    (config) username R2 password morishima 
                    !! 双方向2 
                    (config) username R2 password morishima
                    (config-if) ppp chap hostname R1
                    !! 片方向
                    (config) hostname R1
                    (config-if) ppp chap password morishima
                ```
            1. #### PAP
                - clear text password
                ```
                (config) hostname R1
                !! username must match remote hostname
                !! password must match remote router's "ppp pap ... " command 
                (config) username R2 password pass2 
                (config) interface serial0/0
                (config-if) ip address 192.168.1.1 255.255.255.0
                (config-if) encapsulation ppp
                (config-if) ppp authentication pap
                (config-if) ppp pap sent-username R1 pass1
                ```

                - ユーザ名とパスワード
                ```
                    (config) username R2 password pass2 
                    (config-if) ppp pap sent-username R1 pass1 
                ```
                
        1. ### Multilink PPP (MLPPP)
            ```
            !! Create multilink interface
            (config) interface multilink [bundle-number]
            (config-if) encapsulation ppp
            (config-if) ppp multilink
            (config-if) ip address 192.168.1.10 255.255.255.0
            !! group-number must match budle-number and remote as well
            (config-if) ppp multilink group [group-number] 
            !! if use CHAP
            (config-if) ppp authentication chap

            !! Assign physical interface
            !! multilink interfaceと違うのはip addressの設定だけ
            (config) interface serial0/0
            (config-if) encapsulation ppp
            (config-if) ppp multilink
            (config-if) no ip address
            (config-if) ppp multilink group [group-number]

            # show ip route
            # show interfaces multilink 1
            # show ppp multilink
            ```

        1. ### Metro Ethernet
            - Committed Information Rate (CIR)
                - Minimum guranteed bandwidth
            - E-Line = point-to-point
            - E-Tree = hub-and-spoke
            - E-LAN = full mesh

    1. ## VPN

        1. ### IPsec
            - Cisco Adaptive Security Appliance
                - IPsec VPNを構築することができるファイアウォール

        1. ### General Routing Encapsulation (GRE) tunnel
            1. #### note
                - MTUとMaximum Segment Size (MSS)が減る
                - Must create logical interface
                - Delivery header is 24 bytes
            1. #### Basic
                ```
                (config) interface tunnel [number]
                (config-if) ip address 10.1.1.1 255.255.255.0
                (config-if) tunnel source 1.1.1.1
                !! remote router's ip address
                !! tunnel destinationがundefinedの場合，インタフェースがdownになる
                (config-if) tunnel destination 2.2.2.2

                # show ip interface brief
                # show interfaces tunnel0
                ```
            1. #### ACL that permit GRE
                ```
                (config) ip access-list extended inbound-from-Internet
                (config-ext-nacl) permit tcp
                (config-ext-nacl) permit udp
                (config-ext-nacl) permit gre any any 
                (config-ext-nacl) interface s0/0/0
                (config-if) ip access-group inbound-from-Internet in 
                ```

            1. #### IPsec over GRE tunnel verification
                ```
                # show crypto ipsec sa
                # show crypto isakmp sa
                (config) debug crypto isakmp
                ```

        1. ### Dynamic Multipoint VPN (DMVPN)
            - ハブアンドスポーク
                - スポーク拠点間の通信のためにハブ拠点経由するので遅延が発生
            - フルメッシュ
                - コンフィグ量が多くなり管理上の手間が発生
            - DMVPNのソリューション
                - オンデマンドにスポーク間VPNを構築
                - Next Hop Resolution Protocol (NHRP) でスポークはハブに自身のIPアドレスを登録
                - NHRPでハブは他のスポークにそのIPアドレスを通知

        1. ### PPP over Ethernet (PPPoE)
            - Basic
            ```
            (config) interface dialer 2
            (config-if) ip address negotiated
            (config-if) mtu 1492
            (config-if) encapsulation ppp
            !! hostname given from ISP
            !! hostname コマンドより優先される
            (config-if) ppp chap hostname Fred
            !! password given from ISP
            !! username ... password ... コマンドの次に確認される
            !! クライアント側が相手を認証する必要がないような場合に使用される
            (config-if) ppp chap password Barney 
            (config-if) dialer pool 1

            (config) interface GigabitEthernet 0/0
            (config-if) no ip address 
            (config-if) pppoe enable
            (config-if) pppoe-client dial-pool-number 1

            # show pppoe session
            # show interface dialer 2
            # show interface virtual-access 2 configuration

            !! Debug
            (config) debug ppp authentication
            (config) debug ppp negotiation
            (config) debug pppoe event
            ```

        1. ### Teredo
            - IPv6 host-enabled tunneling technique that uses IPv4 UDP
            - Can pass through existing IPv4 NAT gateways

    1. ## QoS
        - What manages
            - Bandwidth
            - Delay
            - Jitter
            - Loss

        1. ### QoSのモデル
            | モデル | 説明 |
            | ---- | ---- |
            | Integrated Services (IntServ) | アプリケーションの通信フローごとに帯域を予約する方式 |
            | Differentiated Services (DIffServ) | トラフィックを分類、マーキング（優先度付け）、キューイング（トラフィックのキューへの振り分け）、スケジューリング（キューの優先度に応じたパケットの送出）する |
            | Best Effort | FIFO |
            <br>

        1. ### QoSの手順
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
                - Others
                    - QoS Group
                    - Discard class
            3. Queueing キューにパケットを格納すること
            4. Scheduling どのキューにあるパケットから送出していくのか
                - FIFO
                - Class-based Waited Fair Queueing (CBWFQ)
                - Priority Queueing (PQ)

        1. ### Shaping
            - 定義した通信速度を超過した場合、それ以上のパケットはバッファに格納しておいて通信速度の超過が落ち着いてきてからトラフィックを送信していく方式
            - outbound にのみ設定可能
            - Configured in bits per second

        1. ### Policing
            - 定義した通信速度を超過した場合、それ以上のパケットを破棄する。必ず破棄するわけではなく，バーストは許すときがある
            - inbound outbound 両方設定可能
            - IP precedence を書き換えることが可能
            - Configured in bytes

        1. ### Others
            - CAR: policies traffic based on its bandwidth allocation
            - Best effort: service level that provides basic connectivity without differentiation
            - Soft QoS: service level that provides preferred handling
            - Hard QoS: service level that provides reserved network resources
            - PBR: uses route maps to match traffic criteria
            - NBAR: identification tool ideal for handling web applications 

    1. ## Inter-VLAN routing

        1. ### Router
            1. #### Router On A Stick (ROAS)

                1つの物理インタフェースに複数の論理インタフェースを作成し，それぞれにVLANを割り当てる(トランクリンク)

            ```
            !! 1. Create sub interface
                !! subinterface-number = 1
                (config) interface FastEthernet0/0.1
            !! 2. カプセル化タイプとVLAN IDの指定
                !! VLAN ID = 10
                (config-subif) encapsulation dot1q 10 
            !! 3. Configure IP address
                (config-subif) ip address 192.168.10.254 255.255.255.0
            !! 論理インタフェースをn個作成する場合，以上をn回行う
            ```

        1. ### Switch
            1. #### Switch Virtual Interface (SVI) 
                - VLANごとに論理インタフェースを作成する
                - 1つの物理インタフェースで複数のVLANトラフィックを転送(トランスポート)

                ```
                !! IPルーティングの有効化
                (config) ip routing
                !! SVIの作成
                (config) interface vlan 10
                (config-if) ip address 192.168.10.254 255.255.255.0
                (config-if) no shutdown

                # show vlan
                # show interfaces status
                # show interface brief
                # show ip route
                ```

            1. #### Routed port
                - 1つの物理インタフェースで1つのVLANトラフィックのみ転送
                - L3 EtherChannel を作成するのに使用する
                ```
                !! IPルーティングの有効化
                (config) ip routing
                !! ルーテッドポートの作成 L3ポートになる
                (config) interface gigabitEthernet 0/1
                (config-if) no switchport 
                (config-if) ip address 192.168.10.254 255.255.255.0
                (config-if) no shutdown

                # show ip interface brief
                # show ip route
                ```

            1. #### L3 EtherChannelの設定
            ```
            (config)# ip routing
            !! ルーテッドポートの作成
            !! 複数の物理インタフェースで設定する
            (config)# interface interface-id
            (config-if) no switchport
            !! group-number = 10
            (config-if) channel-group 10 mode [ auto | desirable | on | active | passive ]

            !! port-channel = 10
            (config) interface port-channel 10 
            (config-if) no switchport
            (config-if) ip address address mask
            (config-if) no shutdown
            ```

    1. ## First Hop Redundancy Protocol (FHRP)

        デフォルトゲートウェイを冗長化させる技術

        1. ### Three FHRP Options
            | Acronym | Full Name | Origin | Redundancy Approach | Load Balancing |
            | ---- | ---- | ---- | ---- | ---- |
            | HSRP | Hot Standby Router Protocol | Cisco | Active/standby | Per subnet |
            | VRRP | Virtual Router Redundancy Protocol | RFC 5798 | Active/standby | Per subnet |
            | GLBP | Gateway Load Balancing Protocol | Ciso | Active/active | Per host |
            <br>

        1. ### HSRPv1 Versus HSRPv2
            | Feature | Version 1 | Version 2 |
            | ---- | ---- | ---- |
            | IPv6 support | No | Yes |
            | Smallest unit for Hello timer | Second | Millisecond |
            | Range of group numbers | 0-255 | 0-4095 |
            | MAC address used | 0000.0C07.ACxx | 0000.0C9F.Fxxx |
            | IPv4 multicast address used | 224.0.0.2 | 224.0.0.102 |
            | Does protocol use a unique identifier for each router ? | No | Yes |
            <br>

            - Active router will be chosen by
                - Highest IP address
                - Configured priority

            ```
            !! R1
            (config-if) standby 1 ip 10.1.1.1
            !! higher is better
            (config-if) standby 1 priority 110 
            !! take over active when current avtive router's priority is lower than own priority
            (config-if) standby 1 preempt 
            !! R2
            (config-if) standby 1 ip 10.1.1.1
            (config-if) standby 1 priority 100 

            # show standby brief
            # show standby
            !! shows standby version
            ```

    1. ## IPv6

        1. ### Prefix
            |      |      |
            | ---- | ---- |
            | FE80 | Link-local |
            | FD | Unique-local (プライベートIPv4) |
            | 2,3 | Global unicast (グローバルIPv4) |
            | FF | Multicast |
            | FF02::2 | all-router multicast group |
            | FF02::5 | OSPFv3 all SPF routers |
            | FF02::6 | OSPFv3 all DR routers |
            | FF02::9 | All Routing Information Protocol (RIP) routers on a link |
            | FF02::A | EIGRP routers | 
            <br>

        1. ### 特徴
            - plug-and-play
            - no broadcasts
            - autoconfigration

        1. ### Enable routing IPv6 packets
            ```
            (config) ipv6 unicast-routing
            ```

        1. ### IPv6 addressing
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

    1. ## Simple Network Management Protocol (SNMP) 

        ルータ、スイッチ、サーバなどTCP/IPネットワークに接続された通信機器に対し、ネットワーク経由で監視、制御するためのアプリケーション層プロトコル

        1. ### 用語
            - SNMP Manager
                - 管理する側
                - Windows server, Linux server
                - UDP port 162
            - SNMP Agent
                - 管理される側
                - Router, Switch, Server
                - UDP port 161

            - SNMP trap
                - SNMP AgentからSNMP Managerに能動的に送信する通知のこと
                ```
                (config) snmp-server enable traps
                ```

            - Management Information Base (MIB)
                - SNMPエージェントが持っている機器情報の集合体のこと
                - OID (オブジェクトID)
                - Remote Monitoring MIB (RMON)

            - SNMP Community
                - SNMPで管理するネットワークシステムの範囲のこと
                - MIBへのアクセス権限
                    - Read Only (RO)
                    - Read Write (RW)
                    - Read-write-all
                        - コミュニティを含めてMIB情報の読み書きが可能

        1. ### バージョンの比較 
            | SNMP version | RFC | What |
            | ---- | ---- | ---- |
            | SNMPv1 | RFC 1157 | SNMPコミュニティによる平文認証。SNMPトラップにおける再送確認なし。|
            | SNMPv2c | RFC 1901 | SNMPコミュニティによる平文認証。SNMPトラップにおける再送確認あり。 |
            | SNMPv3 | RFC 2273-2275 | ユーザ単位の暗号化されたパスワード認証。SNMPトラップにおける再送確認あり。 |

        1. ### SNMPv2c
            - SNMP AgentはSNMP Community の文字列でSNMP Managerを認証
            ```
            !! Enable SNMP agent
            (config) snmp-server community [community-string] RO [ipv6 acl-name] [acl-name]
            !! Option
            !! e.g., (config) snmp-server location Atlanta
            (config) snmp-server location [text-describing-location]
            !! 問題が起きた時の連絡先
            (config) snmp-server contact [contact-name]

            !! statusが分かる
            # show snmp
            !! 設定が分かる
            # show snmp community
            ```

        1. ### SNMPv3
            ```
            (config) snmp-server group [group-name] v3 [noauth|auth|priv] write [view-name]
            !! userをgroupに参加させる
            (config) snmp-server user morishima [group-name] v3 auth md5 [password] priv AES [key-len] [key-value]
            !! トラップの設定
            !! hostはSNMPマネージャ
            (config) snmp-server host [host] [informs|traps(default)] version 3 [noauth|auth|priv] md5 [password] 

            !! noauth: Neither Auth nor Priv
            !! auth: Auth but no Priv
            !! priv: Auth and Priv

            !! auth md5 [password] は snmp-server groupでauthかprivを選択したときに必要
            !! priv AES [key-len] [key-value] は snmp-server groupでprivを選択したときに必要
            ```

    1. ## IP Service Level Agreement (IP SLA)
        - 機器間のRTTのthresholdを定め，SNMPトラップを送信することなどができる
        - can use to dynamically identify a connectivity problem between a Cisco device and a designated endpoint

    1. ## Switched Port Analyzer (SPAN)

        スイッチのミラーリング機能

        1. ### 注意点
            - SPAN dst portは１つのSPAN sessionにつき１つ
            - SPAN src portは１つのSPAN sessionにつき複数設定可能 
            - srcはinterfaceかVLANのどちらかのみ
            - SPAN dst portとSPAN src portは別々でなければならない
            - SPAN dst portはMAC learningしない

        1. ### Local SPAN の影響
            - It doubles the load on the forwarding engine
            - It prevents span destination from using port security
            - It double internal switch traffic

            ```
            (config) monitor session 1 source interface G1/0/11 - 12 rx
            !! rx がない場合，両方向
            (config) monitor session 1 destination interface G1/0/21
            ```

1. # Others

    1. ## Cloud Computing
        | | Internet | Internet VPN | MPLS VPN | Ethernet WAN | Intercloud Exchange |
        | ---- | ---- | ---- | ---- | ---- | ---- |
        | Secure | No | Yes | Yes | Yes | Yes |
        | QoS | No | No | Yes | Yes | Yes |
        | Requires capacity planning | Yes | Yes | Yes | Yes | Yes |
        | Easier migration to new provider | Yes | Yes | No | No | Yes |
        | Can begin using punlic cloud quickly | Yes | Yes | No | No | No |

        - MPLS for WAN
            - Supports CoS
            - Provides VPN

        - Circumstances your organization require a wide bandwidth Internet connection
            - Cloud computing
            - peer-to-peer file sharing
            - IaaS

    1. ## Software Defined Network (SDN) and Network Programmability

        1. ### 用語
            - Management Plane
                - Telnet, SSH, SNMP
            - Control Plane
                - データ転送における経路の制御や計算を行う機能
            - Data Plane
                - フレームの転送を行う機能

        1. ### SDN Architecture

            - Application

                |  
                | Northbound API (Northbound Interface)  
                | e.g. REST  
                |

            - Control

                |  
                | Southbound API (Southbound Interface)  
                | e.g. OpenFlow   
                |

            - Infrastracture

        1. ### Cisco Application Centric Infrastructure (Cisco ACI)
            Ciscoが提供するSDN製品
            1. #### Application Policy Infrastructure Controller (APIC)
                - Cisco ACIにおけるデータセンター向けのSDNコントローラ
            1. #### APIC Enterprise Module (APIC-EM)
                - LAN/WAN向けのSDNコントローラ 
                - ライセンス不要
                - 設定にはSrc address と Dest address が必要
                1. ##### Path Trace App
                    - 送信デバイスから宛先デバイスに転送されるまでのパスを確認
                    - 対応プロトコル
                        - BGP
                        - Dynamic Multipoint VPN (DMVPN)
                        - EIGRP
                        - IS-IS
                        - Equal Cost Multi Path / Trace Route (ECMP/TR)
                        - Equal Cost Multi Path(ECMP)
                        - HSRP 
                1. ##### Path Trace ACL
                    - どのACLによりパケットが破棄されるのかをGUIで視覚的に確認
                    - Only matching access control entries (ACEs) are reported.
                    - If you leave out the protocol, source port, or destination port when defining a path trace, the results include ACE matches for all possible values for these fields.
                    - If no matching ACEs exists in the ACL, the flow is reported to be implicitly denied
                1. ##### Roll-based Access Control (RBAC)
                    - アクセス制御手法の一つ
                    - 特定の操作を実行するための権限が、特定のロールに割り当てられる

    1. ## L2 Security
        1. ### IEEE802.1X
            - 有線LANや無線LANにおけるユーザ認証の規格
            - 3つのモード
                - Monitor mode
                - Low impact mode
                - High security mode
            - 3つの役割
                - Supplicant
                    - Client的な
                    - Authentication server, Authenticator とは EAPで通信
                - Authentication server
                    - username と password を管理する
                - Authenticator
                    - Supplicant と　Authentication server の仲立ち
        1. ### AAA 3つのセキュリティ機能
            - Authentication
                - 認証
            - Authorization
                - 認証に成功したユーザに対して提供する権限
            - Accounting
        1. ### AAAの実装
            | | RADIUS | TACACS+ |
            | ---- | ---- | ---- |
            | Transportation & Ports | UDP port 1812, 1813<br> Support Extensible Authentication Protocol (EAP) | TCP port 49 |
            | Encryption | Only passwords | Entire packet |
            | Standards | Open standard | Cisco proprietary |
            | Operation | Authentication and Authorization are combined in one function | AAA are separated |
            | Logging | No cammand logging | Full command logging |
        1. ### Responses from TACACS+
            - ACCEPT
            - REJECT
            - ERROR
            - CONTINUE
                - The user is prompted for additional authentication information.
        1. ### DHCP Snooping on switch 嗅ぐ->盗み見る
            ファイアウォールのようなセキュリティ機構
            1. #### 用語
                - Trusted: Switch, Router, DHCP Server
                - Untrusted: Client device such as PC
            1. #### What DHCP Snooping do
                - Validates DHCP messages received from untrusted sources and filters out invalid messages.
                - Rate-limits DHCP traffic from trusted and untrusted sources.
                - Builds and maintains the DHCP snooping binding database, which contains information about untrusted hosts with leased IP addresses.
                - Utilizes the DHCP snooping binding database to validate subsequent requests from untrusted hosts.

# 5. Note

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
    - Distance-vector 伝言ゲーム Bellman-Ford
        - RIPv1
            - Classful
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
            - Dijkstra
            - Classful
        - IS-IS
            - Classful
    - Hybrid
        - EIGRP
            - Diffusing Update Algorithm (DUAL)
            - Classless
- EGP (AS間で使用されるプロトコル)
    - Path-vector
        - BGP
            - IBGP peer (same AS)
            - EBGP peer (other AS)

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