# openvswitch_install

# 0. Goal

| VM | IP | Host | IP |
| --- | --- | --- | --- |
| master  | 192.168.33.100 | DELL precision T3620 (CentOS8)| 192.168.11.8 |
|         | 192.168.121.xxx (Internal IP in k8s cluster)| DELL precision T3620 (CentOS8)| 192.168.11.8 |
| worker1 | 192.168.33.101 | DELL precision T3620 (CentOS8)| 192.168.11.8 |
|         | 192.168.121.xxx (Internal IP in k8s cluster)| DELL precision T3620 (CentOS8)| 192.168.11.8 |
| worker2 | 192.168.33.102 | DELL precision T3620 (CentOS8)| 192.168.11.8 |
|         | 192.168.121.xxx (Internal IP in k8s cluster)| DELL precision T3620 (CentOS8)| 192.168.11.8 |
| worker3 | 192.168.33.102 | DELL precision T3620 (CentOS8)| 192.168.11.8 |
|         | 192.168.121.xxx (Internal IP in k8s cluster)| DELL precision T3620 (CentOS8)| 192.168.11.8 |
| worker4 | 192.168.33.104 | DELL Optiplex 5050 (Ubuntu)| 192.168.11.27 |
|         | 192.168.121.xxx (Internal IP in k8s cluster)| DELL Optiplex 5050 (Ubuntu)| 192.168.11.27 |
| worker5 | 192.168.33.105 | DELL Optiplex 5050 (Ubuntu)| 192.168.11.27 |
|         | 192.168.121.xxx (Internal IP in k8s cluster)| DELL Optiplex 5050 (Ubuntu)| 192.168.11.27 |


# 1. CentOS8
```
$ sudo dnf install -y epel-release
$ sudo dnf install -y centos-release-openstack-train
$ sudo dnf install openvswitch
$ sudo systemctl enable --now openvswitch
$ sudo ovs-vsctl show
5c4bc60e-e5e6-450a-9a2a-53abd4cb3eb0
    ovs_version: "2.12.0"

$ ifconfig |grep virbr -A 1
virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
--
virbr1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.121.1  netmask 255.255.255.0  broadcast 192.168.121.255
--
virbr2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.33.1  netmask 255.255.255.0  broadcast 192.168.33.255
--
virbr3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.133.1  netmask 255.255.255.0  broadcast 192.168.133.255

$ sudo ovs-vsctl add-br br0
$ sudo ovs-vsctl show
5c4bc60e-e5e6-450a-9a2a-53abd4cb3eb0
    Bridge "br0"
        Port "br0"
            Interface "br0"
                type: internal
    ovs_version: "2.12.0"

$ sudo ovs-vsctl show
5c4bc60e-e5e6-450a-9a2a-53abd4cb3eb0
    Bridge "br0"
        Port "virbr2"
            Interface "virbr2"
        Port "br0"
            Interface "br0"
                type: internal
        Port "virbr1"
            Interface "virbr1"
    ovs_version: "2.12.0"

$ sudo ovs-vsctl show
5c4bc60e-e5e6-450a-9a2a-53abd4cb3eb0
    Bridge "br0"
        Port "virbr2"
            Interface "virbr2"
        Port "br0"
            Interface "br0"
                type: internal
        Port "gre0"
            Interface "gre0"
                type: gre
                options: {remote_ip="192.168.11.27"}
        Port "virbr1"
            Interface "virbr1"
    ovs_version: "2.12.0"
```

# 2. Ubuntu
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
    
$ ip a |grep virbr
10: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
11: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
13: virbr1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master ovs-system state UP group default qlen 1000
    inet 192.168.121.1/24 brd 192.168.121.255 scope global virbr1
14: virbr1-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr1 state DOWN group default qlen 1000
15: virbr2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master ovs-system state UP group default qlen 1000
    inet 192.168.33.1/24 brd 192.168.33.255 scope global virbr2
16: virbr2-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr2 state DOWN group default qlen 1000
17: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN group default qlen 1000
18: vnet1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr2 state UNKNOWN group default qlen 1000
19: virbr3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 192.168.133.1/24 brd 192.168.133.255 scope global virbr3
20: virbr3-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr3 state DOWN group default qlen 1000
21: vnet2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN group default qlen 1000
22: vnet3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr2 state UNKNOWN group default qlen 1000
23: vnet4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN group default qlen 1000
24: vnet5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr2 state UNKNOWN group default qlen 1000
25: vnet6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr3 state UNKNOWN group default qlen 1000

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

Next time, you might stop openvswitch-switch service before vagrant up.
```
$ sudo systemctl disable --now openvswitch-switch
$ vagrant up --provider=libvirt
$ sudo systemctl enable --now openvswitch-switch
```

# 3. Ping between master and worker4,5
```
vagrant@master:~$ ping 192.168.33.104
PING 192.168.33.104 (192.168.33.104) 56(84) bytes of data.
64 bytes from 192.168.33.104: icmp_seq=1 ttl=64 time=3.52 ms
64 bytes from 192.168.33.104: icmp_seq=2 ttl=64 time=1.11 ms
64 bytes from 192.168.33.104: icmp_seq=3 ttl=64 time=1.06 ms
64 bytes from 192.168.33.104: icmp_seq=4 ttl=64 time=1.02 ms
^C
--- 192.168.33.104 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3002ms
rtt min/avg/max/mdev = 1.016/1.678/3.524/1.065 ms
vagrant@master:~$ ping 192.168.33.105
PING 192.168.33.105 (192.168.33.105) 56(84) bytes of data.
64 bytes from 192.168.33.105: icmp_seq=1 ttl=64 time=3.26 ms
64 bytes from 192.168.33.105: icmp_seq=2 ttl=64 time=0.948 ms
64 bytes from 192.168.33.105: icmp_seq=3 ttl=64 time=1.15 ms
64 bytes from 192.168.33.105: icmp_seq=4 ttl=64 time=1.07 ms
^C
--- 192.168.33.105 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3008ms
rtt min/avg/max/mdev = 0.948/1.606/3.264/0.959 ms
```

