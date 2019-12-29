# Basic setting 
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

# Switch

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

## Spanning Tree Protocol - (STP)

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
    - 4096単位
```
    (config) spanning-tree vlan 10 priority [priority]
```

- PortFast パソコンとつなぐポートはフォワーディング
```
    (config) spanning-tree portfast default OR
    (config-if) spanning-tree portfast
```

- BPDU Guard PortFastポートがBPDU- (STPのメッセ)を受信したらブロッキングする
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
```
    (config) interface fa 0/1
    (config-if) channel-group 1 mode on
    (config) interface fa 0/2
    (config-if) channel-group 1 mode on

    # show spanning-tree
    # show etherchannel summary
```

## VLAN Trunking Protocol - (VTP)

- Version
```
    (config) vtp version [ 1 | 2 | 3 ]
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
```
    (config) (no) vtp pruning

    # show vtp status
```

# Router

## Basic

- Enable routing IPv6 packets
```
    (config) ipv6 unicast-routing
```

- Static IP address
```
    (config-if) ip address 172.16.1.1 255.255.0.0
    (config-if) no shutdown

    # show protocols

    (config-if) ipv6 address 2001:DB8:1111:1::1/64
    (config-if) ipv6 address 2001:DB8:1111:1::/64 eui-64

    # show ipv6 interface gigabitEthernet 0/0
```

- IPv6 DHCP and SLAAC
```
    (config) ipv6 address dhcp
    (config) ipv6 address autoconfig
```

- Serial DCE DTE(no clock) DB60
    - DTE -> (シリアルケーブル) -> DCE -> キャリアの網 -> DCE -> DTE
    - DCEからDTEにクロック供給
```
    (config-if) clock rate128000
    (config-if) bandwidth 1544
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
- Standard: 1-99, 1300-1999
- Extended: 100-199, 2000-2699
```
    (config) access-list 1 permit 192.168.10.0 0.0.0.255
    (config) access-list 1 permit 192.168.20.0 0.0.0.255
    (config) interface gigabitEthernet 0/0
    (config-if) ip access-group 1 in | Apply
        or
    (config-if) ip access-list extended Name (複数行設定)

    # show access-lists
    # show ip access-lists - (only IPv4)
    # show ip interface gigabitEthernet 0/0
```
- Deleting
```
    (config) no access-list101 | Delete everything
        or
    (config) ip access-list extended 101
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
- Basic setting
```
    (config) router ospf [process-id]
    (config-router) router-id 1.1.1.1 
    (config-router) network 10.24.55.10 0.0.0.0 area [area-id]
    ! ワイルドカードでインターフェースを指定

    # show ip protocols
    # show ip ospf
    # show ip ospf interface brief
    # show ip ospf interface GigabitEthernet 0/0

    # show ip ospf neighbor
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
- Basic setting
```
    (config) router eigrp [AS number]
    ! BGPのASとは違う
    ! Must be same as neighbor's AS number
    (config-router) network 10.24.55.10 0.0.0.0
    ! ワイルドカードでインターフェースを指定

    # show running-config
    # show ip eigrp interfaces
    # show ip protocols

    # show ip eigrp neighbors 
    # show ip eigrp topology 
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
    (config-router) variance 2

    ! Auto-summary
    ! Enable
    (config-router) auto-summary 
    ! Disable
    (config-router) no auto-summary 

    ! Debug
    # debug eigrp fsm
```

# Note

## IPv6
- Neighbor Discovery Protocol - (NDP)
    - Neighbor Solicitation - (NS) : ARP request
    - Neighbor Advertisement - (NA) : ARP return
    - Router Solicitation - (RS) : Router info request, no MAC addr
    - Router Advertisement - (RA) : Router info return, no MAC addr
- Prefix
    - link-local: FE80
    - unique-local: FD
    - global unicast: 2,3
    - multicast: FF
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
            - Designated Router (DR) 代表ルータ
                - プライオリティが最も大きいルータ
                - 同じ場合、ルータIDが大きいルータ
            - Backup Designated Router (BDR)
            - DROTHER
        - IS-IS
            - Classful
    - Hybrid
        - EIGRP
            - Classless
            - Diffusing Update Algorithm (DUAL)
            - Successor (S) : プライマリルート, 最小のFD
            - Feasible Successor (FS) : バックアップルート, SのFDより小さなADのルート
            - Advertised Distance (AD) : Neighbor to Dst
            - Feasible Distance (FD) : Localhost to Dst
- EGP (AS間で使用されるプロトコル)
    - Distance-vector
        - BGP

## STP

- EtherChannel
    - PAgP desirable - desirable | auto
    - LACP active - active | passive
- PortFast
    - パソコンとつなぐポートはフォワーディング
- Bridge Protocol Data unit - (BPDU)
    - BPDU Guard
        - PortFastポートがBPDUを受信したらブロッキングする
- Rapid STP - (RSTP)
    - pvst
    - rapid-pvst
    - mst

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