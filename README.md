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
Шаг 1. Соберите схему:

Перетащите на рабочее поле 3 коммутатора Cisco 2960-24TT.

Перетащите 3 компьютера (PC-PT).

Соедините коммутаторы между собой: каждый с каждым используйте по 2 кабеля (тип Copper Straight-Through). Это даст 6 физических линков.

Подключите каждый ПК к своему коммутатору (например, PC0 → Switch1, PC1 → Switch2, третий ПК → Switch3) одним кабелем.

Убедитесь, что подключения используют порты GigabitEthernet (например, Gig0/1 – Gig0/4).

Шаг 2. Настройте IP-адреса на ПК:

Кликните на каждый ПК → вкладка Desktop → IP Configuration.

Назначьте адреса из одной подсети (маска 255.255.255.0):

PC0: 192.168.1.1

PC1: 192.168.1.2

PC2: 192.168.1.3

Шлюз не обязателен (поскольку это одна подсеть), но можно указать 192.168.1.254 на всех.

Шаг 3. Примените команды на коммутаторах (копируйте и вставляйте):

Перейдите в CLI каждого коммутатора и выполните конфигурацию из моего предыдущего ответа. Вот краткая сводка:

Switch1 (группы 1 и 2):


enable \
configure terminal \
interface range gig0/1-2 \
channel-group 1 mode active \
exit \
interface range gig0/3-4 \
channel-group 2 mode active \
exit \
interface port-channel 1 \
switchport mode trunk \
exit \
interface port-channel 2 \
switchport mode trunk \
end \
write \


Switch2 (группы 1 и 3):


enable \
configure terminal \
interface range gig0/1-2 \
channel-group 1 mode active \
exit \
interface range gig0/3-4 \
channel-group 3 mode active \
exit \
interface port-channel 1 \
switchport mode trunk \
exit \
interface port-channel 3 \
switchport mode trunk \
end \
write \


Switch3 (группы 2 и 3):


enable \
configure terminal \
interface range gig0/1-2 \
channel-group 2 mode active \
exit \
interface range gig0/3-4 \
channel-group 3 mode active \
exit \
interface port-channel 2 \
switchport mode trunk \
exit \
interface port-channel 3 \
switchport mode trunk \
end \
write 

Шаг 4. Тестирование:

Проверьте пинг между ПК (например, от PC0 к PC1). Он должен быть успешным.

Выполните команды show etherchannel summary и show interfaces status на любом коммутаторе, чтобы увидеть объединённые каналы.
