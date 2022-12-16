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