## Задание 1

Компания ОАО «XXX – век» планирует увеличить пропускную способность канала связи. К вам поступила заявка: настроить статическую агрегацию для сети. \
Необходимо настроить и проверить доступность компьютеров, проверить настройки через пространство команд show.

![1](https://user-images.githubusercontent.com/73060384/150137949-45bfd56c-a35c-4042-9377-5764cb09594d.png)

Шаг 1. Соберите топологию в Cisco Packet Tracer \
Устройство	Модель	Порт подключения \
PC0	PC-PT	FastEthernet0 → Switch0 Fa0/3 \
Switch0	2960-24TT	Fa0/1 → Switch3 Fa0/1 (кросс-кабель) \
Switch0	2960-24TT	Fa0/2 → Switch3 Fa0/2 (кросс-кабель) \
Switch3	2960-24TT	Fa0/3 → PC1 FastEthernet0

Шаг 2. Настройте IP на компьютерах \
Кликните на каждый ПК → вкладка Desktop → IP Configuration:

ПК	IP-адрес	Маска	Шлюз \
PC0	192.168.1.10	255.255.255.0	192.168.1.1 \
PC1	192.168.1.20	255.255.255.0	192.168.1.1

Шаг 3. Скопируйте готовую конфигурацию для Switch0 \
Откройте Switch0 → вкладка CLI → нажимайте Enter, пока не появится Switch>, затем вставьте этот блок целиком (копируйте всё от enable до write):


enable \
configure terminal \
hostname Switch0

interface range fa0/1-2 \
 shutdown \
 switchport mode trunk \
 channel-group 1 mode on \
 no shutdown

interface port-channel 1 \
 switchport mode trunk \
 no shutdown

interface fa0/3 \
 switchport mode access \
 switchport access vlan 1 \
 no shutdown

interface vlan 1 \
 ip address 192.168.1.1 255.255.255.0 \
 no shutdown

ip routing \
exit \
write memory

Шаг 4. Скопируйте готовую конфигурацию для Switch3 \
Откройте Switch3 → вкладка CLI → вставьте этот блок:

enable \
configure terminal \
hostname Switch3

interface range fa0/1-2 \
 shutdown \
 switchport mode trunk \
 channel-group 1 mode on \
 no shutdown

interface port-channel 1 \
 switchport mode trunk \
 no shutdown

interface fa0/3 \
 switchport mode access \
 switchport access vlan 1 \
 no shutdown

interface vlan 1 \
 ip address 192.168.1.2 255.255.255.0 \
 no shutdown

ip routing \
exit \
write memory

Шаг 5. Проверьте работу \
Теперь на любом ПК откройте Command Prompt (Desktop → Command Prompt) и выполните:

ping 192.168.1.20 \
(если пингуете с PC0) или ping 192.168.1.10 (с PC1).

Результат: Должны прийти успешные ответы.

Шаг 6. Проверьте настройки командами show \
На Switch0 и Switch3 выполните:

show etherchannel summary \
Должны увидеть:

Group  Port-channel  Protocol   Ports \
------+-------------+----------+------------------------------------------- \
1      Po1(SU)       -          Fa0/1(P)   Fa0/2(P)


show interfaces trunk \
Должны увидеть Port-channel1 в режиме trunk.

Шаг 7. Проверьте отказоустойчивость \
Отключите один из кабелей между коммутаторами (Fa0/1 или Fa0/2) и снова выполните пинг. \
Связь не должна пропасть — трафик пойдёт по второму кабелю.

## Задание 2

Компания ОАО «XXX – век» после модернизации и расширения сети задумалась над резервированием существующего агрегированного канала. \
К вам поступила заявка: настроить динамическую агрегацию каналов для сети, так как есть риск разрыва соединения на основных интерфейсах.

Необходимо:

настроить сетевое оборудование и проверить доступность компьютеров, \
проверить настройки через пространство команд show, \
проверить работу после отключения основных интерфейсов.

![2](https://user-images.githubusercontent.com/85602495/152174571-f344c6ec-ec34-4683-8f8b-51dbe57d6b45.png)

## Ответ:

Конфигурация для кольцевой топологии (с 2 линками между соседями) \
Switch0 \


Связь со Switch1 (группа 1) и со Switch3 (группа 4):


enable \
configure terminal \
! Связь со Switch1 \
interface range gigabitethernet 0/1-2 \
channel-group 1 mode active \
exit \
interface port-channel 1 \
switchport mode trunk \
exit \
! Связь со Switch3 \
interface range gigabitethernet 0/3-4 \
channel-group 4 mode active \
exit \
interface port-channel 4 \
switchport mode trunk \
end \
write memory \
Switch1 \


Связь со Switch0 (группа 1) и со Switch2 (группа 2): \


enable \
configure terminal \
! Связь со Switch0 \
interface range gigabitethernet 0/1-2 \
channel-group 1 mode active \
exit \
interface port-channel 1 \
switchport mode trunk \
exit \
! Связь со Switch2 \
interface range gigabitethernet 0/3-4 \
channel-group 2 mode active \
exit \
interface port-channel 2 \
switchport mode trunk \
end \
write memory \
Switch2 


Связь со Switch1 (группа 2) и со Switch3 (группа 3):


enable \
configure terminal \
! Связь со Switch1 \
interface range gigabitethernet 0/1-2 \
channel-group 2 mode active \
exit \
interface port-channel 2 \
switchport mode trunk \
exit \
! Связь со Switch3 \
interface range gigabitethernet 0/3-4 \
channel-group 3 mode active \
exit \
interface port-channel 3 \
switchport mode trunk \
end \
write memory \
Switch3 


Связь со Switch2 (группа 3) и со Switch0 (группа 4):


enable \
configure terminal \
! Связь со Switch2 \
interface range gigabitethernet 0/1-2 \
channel-group 3 mode active \
exit \
interface port-channel 3 \
switchport mode trunk \
exit \
! Связь со Switch0 \
interface range gigabitethernet 0/3-4 \
channel-group 4 mode active \
exit \
interface port-channel 4 \
switchport mode trunk \
end \
write memory

 IP-адреса ПК
 
Устройство	IP-адрес	Маска подсети \
PC0	192.168.1.1	255.255.255.0 \
PC1	192.168.1.2	255.255.255.0 \

 
 Проверка и тестирование

 
1. Проверка EtherChannel:
   
show etherchannel summary \
Ожидаемый вывод:

text \
Group  Port-channel  Protocol  Ports \
------+-------------+---------+--------------- \
1      Po1(SU)      LACP      Gi0/1(P) Gi0/2(P) \
2      Po2(SU)      LACP      Gi0/3(P) Gi0/4(P) \
3      Po3(SU)      LACP      Gi0/1(P) Gi0/2(P) \
4      Po4(SU)      LACP      Gi0/3(P) Gi0/4(P)

2. Проверка связности:
   
С PC0 выполните ping 192.168.1.2 — должен быть успешным.

4. Тест отказоустойчивости:
   
Отключите один порт в группе (например, на Switch0):


configure terminal
interface gigabitethernet 0/1
shutdown
end
Снова выполните ping — связь должна сохраниться через второй порт в группе.

4. Проверка STP (Spanning Tree Protocol):
В кольцевой топологии STP автоматически заблокирует один из линков, чтобы избежать петель:

show spanning-tree

## Задание 3

Компания ОАО «XXX – век» разместила в своей сети сетевые сервисы. Для увеличения полосы пропускания, повышения надежности и реализации высокой доступности серверов и расположенных на них сервисах Вам поставили задачу доработать топологию сети с учетом возможностей оборудования и настроить балансировку на основе ip адресов источника и получателя.

Необходимо:

настроить и проверить доступность компьютеров, \
проверить настройки через пространство команд show. \
Важно! В эмуляторе Cisco Packet Tracer есть особенность: часть настроек сетевого оборудования в этой лабораторной работе не сохраняется, когда вы сохраняете ее в .pkt файл. \
Решение: На всех сетевых устройствах после окончания настройки нужно выполнить команду "write" или "copy running-config startap-config".

![3](https://user-images.githubusercontent.com/77622076/229756419-f559b248-6336-4976-858b-19443ef4c32f.jpg)
