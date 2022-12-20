# VPN
## Плейбук для проверки разницы между режимами работы OpenVpn.
---

***- Для выполнения первого задания был разработан плйбук - vpn.yml. Для запуска сенда понадобится ввести команды vagrant up && ansible-playbook.yml;***  
***- Файл defaults/main.yml содержит переменную по переключению режимов работы туннеля - dev_type;***  
***- Запустим стенд, на server и client запустим perf3 и псомотрим на скорость работы***  
<br> 

```console
[  4] local 10.10.10.2 port 36644 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec  38.8 MBytes  65.1 Mbits/sec    2    272 KBytes       
[  4]   5.00-10.00  sec  50.5 MBytes  84.7 Mbits/sec    0    382 KBytes       
[  4]  10.00-15.00  sec  54.3 MBytes  91.0 Mbits/sec    0    622 KBytes       
[  4]  15.00-20.00  sec  55.7 MBytes  93.5 Mbits/sec   10    419 KBytes       
[  4]  20.00-25.00  sec  54.5 MBytes  91.4 Mbits/sec  151    427 KBytes       
[  4]  25.00-30.00  sec  51.3 MBytes  86.1 Mbits/sec  205    341 KBytes       
[  4]  30.00-35.00  sec  48.1 MBytes  80.7 Mbits/sec    0    432 KBytes       
[  4]  35.00-40.00  sec  42.1 MBytes  70.7 Mbits/sec  163    488 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-40.00  sec   395 MBytes  82.9 Mbits/sec  531             sender
[  4]   0.00-40.00  sec   394 MBytes  82.7 Mbits/sec                  receiver
```

**Вывод : Скорость рвботы туннеля около 83 Mbits/sec**
---
---
<br>

***- Запишем в переменную dev_type: tap, запустим плейбук ansible-playbook vpn.yml и проверим скорость работы в режиме tap.***  
<br>
```console
[vagrant@client ~]$ iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  4] local 10.10.10.2 port 36640 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec  40.4 MBytes  67.7 Mbits/sec   37    671 KBytes       
[  4]   5.00-10.00  sec  40.9 MBytes  68.5 Mbits/sec    0    787 KBytes       
[  4]  10.00-15.00  sec  38.4 MBytes  64.4 Mbits/sec  233    751 KBytes       
[  4]  15.00-20.00  sec  38.3 MBytes  64.2 Mbits/sec    0    787 KBytes       
[  4]  20.00-25.01  sec  37.2 MBytes  62.4 Mbits/sec  207    773 KBytes       
[  4]  25.01-30.00  sec  37.2 MBytes  62.5 Mbits/sec    0    791 KBytes       
[  4]  30.00-35.00  sec  37.3 MBytes  62.5 Mbits/sec  622    544 KBytes       
[  4]  35.00-40.00  sec  39.8 MBytes  66.8 Mbits/sec  170    495 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-40.00  sec   309 MBytes  64.9 Mbits/sec  1269             sender
[  4]   0.00-40.00  sec   307 MBytes  64.4 Mbits/sec                  receiver
```
**Вывод : Скорость рвботы туннеля около 65 Mbits/sec**
---
<br>

---
## Задание 2. RAS на базе OpenVPN
<br>

***- Задание выполнено с использованием ec2 instance в aws. Там был настроен openvpn server, созданы ключи и сертификаты с помощью easy-rsa, создан следющий конфигурационный файл***  
```console
proto udp
dev tun
ca /home/ec2-user/pki/ca.crt
cert /home/ec2-user/pki/issued/server.crt
key /home/ec2-user/pki/private/server.key
dh /home/ec2-user/pki/dh.pem
server 10.10.10.0 255.255.255.0
route 172.31.0.0 255.255.240.0
push "route 172.31.0.0 255.255.240.0"
ifconfig-pool-persist ipp.txt
client-to-client
client-config-dir /etc/openvpn/client
keepalive 10 120
comp-lzo
persist-key
persist-tun
management localhost 7505
status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3
```
Здесь видим, что до клиента пушим роут в приватную сетку vps - 172.31.0.0 255.255.240.0
<br>
<br>

***- В роли коента выступил macbook на который были скопированы ca.crt, client.crt, client.key, выпущенные и подписанные на сервере. Клиентская машина имеет следующий конфиг:***

```console
anatolijcurikov@Anatolijs-MacBook-Pro aws_client % cat aws_client.conf 
dev tun
proto udp
remote 34.221.184.182 1194
client
resolv-retry infinite
ca ./ca.crt
cert ./client.crt
key ./client.key
route 192.168.1.0 255.255.255.0
persist-key
persist-tun
comp-lzo
verb 3
```
***- Подключимся с клиента на сервер используя конфиг клиента и посмотрим имеем ли мы доступ до внутренней сетки aws instance 172.31.0.0/20 host 172.31.7.90

```console
sudo /opt/homebrew/opt/openvpn/sbin/openvpn --config aws_client.conf
2022-12-20 12:59:56 /sbin/ifconfig utun7 10.10.10.6 10.10.10.5 mtu 1500 netmask 255.255.255.255 up
2022-12-20 12:59:56 /sbin/route add -net 192.168.1.0 10.10.10.5 255.255.255.0
route: writing to routing socket: File exists
add net 192.168.1.0: gateway 10.10.10.5: File exists
2022-12-20 12:59:56 /sbin/route add -net 172.31.0.0 10.10.10.5 255.255.240.0
add net 172.31.0.0: gateway 10.10.10.5
2022-12-20 12:59:56 /sbin/route add -net 10.10.10.0 10.10.10.5 255.255.255.0
add net 10.10.10.0: gateway 10.10.10.5
2022-12-20 12:59:56 Initialization Sequence Completed

anatolijcurikov@Anatolijs-MacBook-Pro aws_client % ping 172.31.7.90
PING 172.31.7.90 (172.31.7.90): 56 data bytes
64 bytes from 172.31.7.90: icmp_seq=0 ttl=255 time=200.088 ms
64 bytes from 172.31.7.90: icmp_seq=1 ttl=255 time=215.736 ms
64 bytes from 172.31.7.90: icmp_seq=2 ttl=255 time=191.640 ms

anatolijcurikov@Anatolijs-MacBook-Pro aws_client % netstat -nr           
Routing tables

Internet:
Destination        Gateway            Flags           Netif Expire
default            192.168.1.1        UGScg             en0       
default            link#27            UCSIg            ppp0       
10.10.10/24        10.10.10.5         UGSc            utun7       
10.10.10.5         10.10.10.6         UH              utun7       
62.117.112.36      192.168.1.1        UGHS              en0       
127                127.0.0.1          UCS               lo0       
127.0.0.1          127.0.0.1          UH                lo0       
169.254            link#15            UCS               en0      !
172.31/20          10.10.10.5         UGSc            utun7   
```
**Вывод: с хоста пингуем внутренний интерфейс сервера, в таблице маршрутезации видим маршрут до сети 172.31/20 через tun7. Задание выполнено.**
---