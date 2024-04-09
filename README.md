## Статическая и динамическая маршрутизация, OSPF

### Цель домашнего задания:
Создать домашнюю сетевую лабораторию. Научится настраивать протокол OSPF в Linux-based системах.
____________________

По умолчанию стенд развернётся с ассиметричным роутингом, для изменения на симметричный, нужно изменить переменную `symmetric_routing: false` в defaults/main.yml.\
Схема сети:\
![Скрин](https://github.com/FeeLinS9/lesson21/blob/master/img.png)\
Проверим доступность сетей с хоста router1:
```
root@router1:~# traceroute 192.168.30.1
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  192.168.30.1 (192.168.30.1)  0.717 ms  0.660 ms  0.638 ms
root@router1:~# ifconfig enp0s9 down
root@router1:~# ip a | grep enp0s9
4: enp0s9: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
root@router1:~# traceroute 192.168.30.1
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  10.0.10.2 (10.0.10.2)  0.569 ms  0.510 ms  0.474 ms
 2  192.168.30.1 (192.168.30.1)  1.588 ms  1.401 ms  1.331 ms
root@router1:~# vtysh

Hello, this is FRRouting (version 9.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router1# show ip route ospf
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O   10.0.10.0/30 [110/100] is directly connected, enp0s8, weight 1, 00:30:36
O>* 10.0.11.0/30 [110/200] via 10.0.10.2, enp0s8, weight 1, 00:00:36
  *                        via 10.0.12.2, enp0s9, weight 1, 00:00:36
O   10.0.12.0/30 [110/100] is directly connected, enp0s9, weight 1, 00:00:36
O   192.168.10.0/24 [110/100] is directly connected, enp0s10, weight 1, 00:30:36
O>* 192.168.20.0/24 [110/200] via 10.0.10.2, enp0s8, weight 1, 00:29:16
O>* 192.168.30.0/24 [110/200] via 10.0.12.2, enp0s9, weight 1, 00:00:36
```
Проверка ассиметричного роутинга:
```
root@router1:~# ping -I 192.168.10.1 192.168.20.1
PING 192.168.20.1 (192.168.20.1) from 192.168.10.1 : 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=64 time=0.983 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=64 time=0.932 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=64 time=1.24 ms
64 bytes from 192.168.20.1: icmp_seq=4 ttl=64 time=0.982 ms
64 bytes from 192.168.20.1: icmp_seq=5 ttl=64 time=1.02 ms
64 bytes from 192.168.20.1: icmp_seq=6 ttl=64 time=0.986 ms
64 bytes from 192.168.20.1: icmp_seq=7 ttl=64 time=1.00 ms
64 bytes from 192.168.20.1: icmp_seq=8 ttl=64 time=1.03 ms

root@router2:~# tcpdump -i enp0s9
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
11:40:12.698811 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 36, length 64
11:40:13.301471 IP 10.0.11.1 > ospf-all.mcast.net: OSPFv2, Hello, length 48
11:40:13.593554 IP router2 > ospf-all.mcast.net: OSPFv2, Hello, length 48
11:40:13.702317 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 37, length 64
11:40:14.703729 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 38, length 64
11:40:15.705021 ARP, Request who-has router2 tell 10.0.11.1, length 46
11:40:15.705054 ARP, Reply router2 is-at 08:00:27:a3:9f:fe (oui Unknown), length 28
11:40:15.707668 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 39, length 64
11:40:16.710393 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 40, length 64
11:40:17.714194 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 41, length 64
11:40:18.715938 IP 192.168.10.1 > router2: ICMP echo request, id 5, seq 42, length 64
```
Видим, что порт только получает трафик.
```
root@router2:~# tcpdump -i enp0s8
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
11:57:18.677635 IP router2 > 192.168.10.1: ICMP echo reply, id 7, seq 9, length 64
11:57:19.723497 IP router2 > 192.168.10.1: ICMP echo reply, id 7, seq 10, length 64
11:57:20.789751 IP router2 > 192.168.10.1: ICMP echo reply, id 7, seq 11, length 64
11:57:21.794292 IP router2 > 192.168.10.1: ICMP echo reply, id 7, seq 12, length 64
11:57:22.798239 IP router2 > 192.168.10.1: ICMP echo reply, id 7, seq 13, length 64
```
А этот порт только отправляет трафик.\
Таким образом мы видим ассиметричный роутинг.

Проверка симметричного роутинга:
```
root@router1:~# ping -I 192.168.10.1 192.168.20.1
PING 192.168.20.1 (192.168.20.1) from 192.168.10.1 : 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=63 time=1.36 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=63 time=1.45 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=63 time=1.52 ms
64 bytes from 192.168.20.1: icmp_seq=4 ttl=63 time=1.32 ms
64 bytes from 192.168.20.1: icmp_seq=5 ttl=63 time=1.41 ms
64 bytes from 192.168.20.1: icmp_seq=6 ttl=63 time=1.20 ms

root@router2:~# tcpdump -i enp0s9
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
12:10:59.641132 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 17, length 64
12:10:59.641172 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 17, length 64
12:11:00.643541 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 18, length 64
12:11:00.643588 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 18, length 64
12:11:01.203899 IP router2 > ospf-all.mcast.net: OSPFv2, Hello, length 48
12:11:01.204803 IP 10.0.11.1 > ospf-all.mcast.net: OSPFv2, Hello, length 48
12:11:01.650137 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 19, length 64
12:11:01.650183 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 19, length 64
12:11:02.655669 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 20, length 64
12:11:02.655706 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 20, length 64
12:11:03.662787 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 21, length 64
12:11:03.662825 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 21, length 64
12:11:04.665078 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 22, length 64
```
Теперь мы видим, что трафик между роутерами ходит симметрично.