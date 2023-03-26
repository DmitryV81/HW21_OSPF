Домашняя работа № 21. OSPF

Задание:

1. Развернуть 3 виртуальные машины

2. Объединить их разными vlan

- настроить OSPF между машинами на базе FRR;

- изобразить ассиметричный роутинг;

- сделать один из линков "дорогим", но что бы при этом роутинг был симметричным.

Ход работы.

Используемый для домашнего задания стенд изображен на рисунке:

![Scheme of test stand](https://github.com/DmitryV81/HW21_OSPF/blob/main/pictures/scheme.png)


Для переключения между симметричным и ассиметричным роутингом используется переменная ansible symmetric_routing.

По умолчанию включен симметричный роутинг, переменной присвоено значение true

"Поднимаем" стенд:
```
vagrant up
```
Заходим на router1 пингуем сеть 192.168.20.1:
```
root@router1:~# ping -I 192.168.10.1 192.168.20.1
PING 192.168.20.1 (192.168.20.1) from 192.168.10.1 : 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=64 time=0.677 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=64 time=0.735 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=64 time=0.715 ms
64 bytes from 192.168.20.1: icmp_seq=4 ttl=64 time=0.709 ms
```
Заходим на router2 и запускаем tcpdump, который будет собирать траффик с интерфейса enp0s8:
```
root@router2:~# tcpdump -i enp0s8
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
15:59:40.248202 IP 192.168.10.1 > router2: ICMP echo request, id 1, seq 31, length 64
15:59:40.248240 IP router2 > 192.168.10.1: ICMP echo reply, id 1, seq 31, length 64
```
Из вывода tcpdump видим, что пакеты принимаются и отправляются с интерфейса enp0s8

Включаем ассиметричный роутинг, переменной symmetric_routing в файле ansible/defaults/main.yaml присваиваем значение false, таким образом мы увеличиваем стоимость маршрута от router1 до router2 через интерфейс enp0s8, установив ему значение 1000.

В консоли выполняем комманду vagrant provision, для применения новых настроек к виртуальным машинам.

На router1 повторно запускаем ping до сети 192.168.20.1:
```
root@router1:~# ping -I 192.168.10.1 192.168.20.1
PING 192.168.20.1 (192.168.20.1) from 192.168.10.1 : 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=64 time=1.07 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=64 time=1.01 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=64 time=1.02 ms
64 bytes from 192.168.20.1: icmp_seq=4 ttl=64 time=1.04 ms
```
На router2 запускаем tcpdump с интерфейса enp0s8:
```
root@router2:~# tcpdump -i enp0s8
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
16:11:38.162298 IP router2 > 192.168.10.1: ICMP echo reply, id 2, seq 30, length 64
16:11:39.163828 IP router2 > 192.168.10.1: ICMP echo reply, id 2, seq 31, length 64
16:11:39.429647 IP router2 > ospf-all.mcast.net: OSPFv2, Hello, length 48
16:11:39.441746 IP 10.0.10.1 > ospf-all.mcast.net: OSPFv2, Hello, length 48
16:11:40.166411 IP router2 > 192.168.10.1: ICMP echo reply, id 2, seq 32, length 64
16:11:41.167932 IP router2 > 192.168.10.1: ICMP echo reply, id 2, seq 33, length 64
16:11:42.170029 IP router2 > 192.168.10.1: ICMP echo reply, id 2, seq 34, length 64
16:11:43.171473 IP router2 > 192.168.10.1: ICMP echo reply, id 2, seq 35, length 64
```
И, с интерфейса enp0s9:
```
root@router2:~# tcpdump -i enp0s9
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
16:11:49.182353 IP 192.168.10.1 > router2: ICMP echo request, id 2, seq 41, length 64
16:11:49.429725 IP router2 > ospf-all.mcast.net: OSPFv2, Hello, length 48
16:11:49.430540 IP 10.0.11.1 > ospf-all.mcast.net: OSPFv2, Hello, length 48
16:11:50.184047 IP 192.168.10.1 > router2: ICMP echo request, id 2, seq 42, length 64
16:11:51.185474 IP 192.168.10.1 > router2: ICMP echo request, id 2, seq 43, length 64
16:11:52.186915 IP 192.168.10.1 > router2: ICMP echo request, id 2, seq 44, length 64
```
Как видно из вывода tcpdump входящий траффик получаем с интерфейса enp0s9, а ответ на ping идёт с интерфейса enp0s8
