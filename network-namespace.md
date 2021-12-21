# Network Namespace

### List network namespaces

command:
```
$ ip netns
```
output:
```
$ ip netns
ns-02
ns-01
```
or

command:
```
$ ls /var/run/netns/
```


output:
```
$ ls /var/run/netns/
ns-01  ns-02
```

---

### Add network namespaces

command:

```
$ ip netns add ns-01
$ ip netns add ns-02
```

---

### Exec command in network namespace

command:
```
$ ip netns exec ns-01 ip link
```

or

```
$ ip -n ns-01 link
```

where:
- **ip link** - command to be executed inside network namespace *ns-01*

---

### Connect two network namespaces


*The **veth** devices are virtual Ethernet devices.*

***veth** devices are always created in interconnected pairs.*

*Place one end of a **veth** pair in one network namespace and the other end in another network namespace, thus allowing communication between network namespaces.*

command:
```
$ ip link add veth-for-ns-01 type veth peer name veth-for-ns-02
```

inspect:
```
$ ip link | grep -A1 veth-for-ns-01

18: veth-for-ns-02@veth-for-ns-01: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 0e:77:22:e7:68:6e brd ff:ff:ff:ff:ff:ff
19: veth-for-ns-01@veth-for-ns-02: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 06:ad:97:47:a5:17 brd ff:ff:ff:ff:ff:ff
```

Place one end of **veth** pair in ns-01 and the otherr end in ns-02

command:
```
$ ip link set veth-for-ns-01 netns ns-01
$ ip link set veth-for-ns-02 netns ns-02
```

inspect:
```
$ ip link | grep -A1 veth-for-ns-01
```

At this point both items (18,19) won't be present in the list.

Execute command inside both namespaces to ensure that **veth** ends are added to corresponded network namespaces.

command:
```
$ ip -n ns-01 link
```

output:
```
$ ip -n ns-01 link

1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
19: veth-for-ns-01@if18: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 06:ad:97:47:a5:17 brd ff:ff:ff:ff:ff:ff link-netnsid 1
```

---

### Assign IP address to Network Namespace's veth

command:
```
$ ip -n ns-01 addr add 192.168.10.1/24 dev veth-for-ns-01
$ ip -n ns-02 addr add 192.168.10.2/24 dev veth-for-ns-02
```

inspect:
```
$ ip -n ns-01 addr

1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
19: veth-for-ns-01@if18: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 06:ad:97:47:a5:17 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 192.168.10.1/24 scope global veth-for-ns-01
       valid_lft forever preferred_lft forever
```

---

### Bringing veth UP

command:
```
$ ip -n ns-01 link set veth-for-ns-01 up
$ ip -n ns-02 link set veth-for-ns-02 up
```

inspect:
```
vagrant@k8s-master:~$ sudo ip -n ns-02 link

1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
18: veth-for-ns-02@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 0e:77:22:e7:68:6e brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

---

### Check connection beetween Network Namespaces

inspect:
```
$ ip netns exec ns-01 ping -c1 192.168.10.2

PING 192.168.10.2 (192.168.10.2) 56(84) bytes of data.
64 bytes from 192.168.10.2: icmp_seq=1 ttl=64 time=0.051 ms

--- 192.168.10.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.051/0.051/0.051/0.000 ms
```

inspect:
```
$ ip netns exec ns-02 ping -c1 192.168.10.1

PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=64 time=0.047 ms

--- 192.168.10.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.047/0.047/0.047/0.000 ms
```
