# openvswitch_install

# 0. Goal

| VM | IP | Host | IP |
| --- | --- | --- | --- |
| worker1 | 192.168.33.101 | DELL precision T3620 | 192.168.11.8 |
|         | 192.168.121.xxx | DELL precision T3620 | 192.168.11.8 |
| worker2 | 192.168.33.102 | DELL precision T3620 | 192.168.11.8 |
|         | 192.168.121.xxx | DELL precision T3620 | 192.168.11.8 |
| worker4 | 192.168.33.104 | DELL Optiplex 5050 | 192.168.11.27 |
|         | 192.168.121.xxx | DELL Optiplex 5050 | 192.168.11.27 |
| worker5 | 192.168.33.105 | DELL Optiplex 5050 | 192.168.11.27 |
|         | 192.168.121.xxx | DELL Optiplex 5050 | 192.168.11.27 |


```
$ sudo apt-get update
$ sudo apt-get install -y openvswitch-switch openvswitch-common

$ sudo ovs-vsctl add-br br0
$ sudo ovs-vsctl show
a69599ba-c200-4138-963f-abf09e94655b
    Bridge br0
        Port br0
            Interface br0
                type: internal
    ovs_version: "2.13.3
    
$ ip a|grep virbr
6: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
7: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
9: virbr1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 192.168.121.1/24 brd 192.168.121.255 scope global virbr1
10: virbr1-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr1 state DOWN group default qlen 1000
11: virbr2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 192.168.33.1/24 brd 192.168.33.255 scope global virbr2
12: virbr2-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr2 state DOWN group default qlen 1000
13: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN group default qlen 1000
14: vnet1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr2 state UNKNOWN group default qlen 1000
15: vnet2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN group default qlen 1000
16: vnet3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr2 state UNKNOWN group default qlen 1000

$ sudo ovs-vsctl show
a69599ba-c200-4138-963f-abf09e94655b
    Bridge br0
        Port br0
            Interface br0
                type: internal
    ovs_version: "2.13.3"

$ sudo ovs-vsctl add-port br0 virbr1
$ sudo ovs-vsctl add-port br0 virbr2
$ sudo ovs-vsctl show
a69599ba-c200-4138-963f-abf09e94655b
    Bridge br0
        Port br0
            Interface br0
                type: internal
        Port virbr2
            Interface virbr2
        Port virbr1
            Interface virbr1
    ovs_version: "2.13.3"

$ sudo ovs-vsctl add-port br0 gre0 -- set interface gre0 type=gre options:remote_ip=192.168.11.8
$ sudo ovs-vsctl show
a69599ba-c200-4138-963f-abf09e94655b
    Bridge br0
        Port br0
            Interface br0
                type: internal
        Port virbr2
            Interface virbr2
        Port virbr1
            Interface virbr1
        Port gre0
            Interface gre0
                type: gre
                options: {remote_ip="192.168.11.8"}
    ovs_version: "2.13.3"
```

