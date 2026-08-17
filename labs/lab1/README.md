# Лабораторная работа. Базовая настройка коммутатора 

## Задачи:

1. [Проверка конфигурации коммутатора по умолчанию](#проверка-конфигурации-коммутатора-по-умолчанию)
2. [Создание сети и настройка основных параметров устройства](#создание-сети-и-проверка-настроек-коммутатора-по-умолчанию)
    - Настройте базовые параметры коммутатора.
    - Настройте IP-адрес для ПК.
3. [Проверка сетевых подключений](#проверка-сетевых-подключений)

## Проверка конфигурации коммутатора по умолчанию

   - **Изучите текущий файл running configuration.**
      - *Сколько интерфейсов FastEthernet имеется на коммутаторе 2960?*
        На используемом образе vIOS L2 интерфейсы не предусмотрены.

      - *Сколько интерфейсов Gigabit Ethernet имеется на коммутаторе 2960?*
     8 интерфейсов GigabitEthernet (согласно выводу команды `show version`).

   - **Изучите файл загрузочной конфигурации (startup configuration), который содержится в энергонезависимом ОЗУ (NVRAM).?**
     Образ чистый, загрузочная конфигурация ещё не создана.

   - **Изучите характеристики SVI для VLAN 1. Изучите IP-свойства интерфейса SVI сети VLAN 1.**
     Интерфейс не создан, IP-адрес не назначен.

   - **Подсоедините кабель Ethernet компьютера PC-A к порту 6 на коммутаторе и изучите IP-свойства интерфейса SVI сети VLAN 1. Дождитесь согласования параметров скорости и дуплекса между коммутатором и ПК.**
    Настройку интерфейса для управления я оставлю на последующие логические шаги лабораторной работы. Для соединения был выбран Gi1/1. 
    Линк поднялся, служебные пакеты дошли до порта PC-A, автоматически согласованы параметры физического уровня: скорость передачи и режим дуплекса. На PC-A еще не выставлены сетевые настройки - пакетов на вход не поступало.
     ```bash
     GigabitEthernet1/1 is up, line protocol is up (connected)
     Auto-duplex, Auto-speed
     0 packets input, 0 bytes
     303 packets output, 24657 bytes
     ```
   - **Изучите сведения о версии ОС Cisco IOS на коммутаторе**
    *Под управлением какой версии ОС Cisco IOS работает коммутатор?
    Как называется файл образа системы?*
     ```bash
     vios_l2 Software (vios_l2-ADVENTERPRISEK9-M), Version 15.2(4.0.55)E
     ```
   - **Изучите свойства по умолчанию интерфейса, который используется компьютером PC-A.**
    *Интерфейс включен или выключен?
     Что нужно сделать, чтобы включить интерфейс?*
     ```bash
     GigabitEthernet1/1 is up, line protocol is up (connected)
     ```
     Для включения интерфейса необходимо в EXEC режиме ввести:
     ```bash
     Switch# configure terminal
     Switch(config)# interface <нужный интерфейс>
     Switch(config-if)# no shutdown
     Switch(config-if)# end
     ```
   - **Изучите флеш-память**
    Указаных команд в файле не оказалось на моем образе, были использованны аналоги
     ```bash
     Switch#dir flash0:
     Directory of flash0:/

       1  drw-           0  Jan 30 2013 00:00:00 +00:00  boot
     264  drw-           0  Oct 14 2013 00:00:00 +00:00  config
     266  -rw-         375  Jul 28 2015 00:00:00 +00:00  config.grub
     267  -rw-   107412732  Jul 28 2015 00:00:00 +00:00  vios_l2-adventerprisek9-m
     268  -rw-      524288  Aug 16 2026 19:35:28 +00:00  nvram

     2142715904 bytes total (2030174208 bytes free)
     Switch#
     Switch#
     Switch#show running-config | include hostname
     hostname Switch
     ```
     - Имя файла образа IOS: vios_l2-adventerprisek9-m
     - Присутствуют каталоги boot и config, а также файл nvram, в котором хранится загрузочная конфигурация.


## Создание сети и проверка настроек коммутатора по умолчанию
### Настройка базовых параметров коммутатора

В режиме глобальной конфигурации были применены базовые параметры конфигурации. Смена имени хоста, подключение пароля и т.п.

1. Назначьте IP-адрес интерфейсу SVI на коммутаторе.    Благодаря этому вы получите возможность удаленного управления коммутатором.
    ```bash
    S1#configure terminal
    Enter configuration commands, one per line.  End with CNTL/Z.
    S1(config)#interface vlan 1
    S1(config-if)#ip address 192.168.1.10 255.255.255.0
    S1(config-if)#no shutdown
    S1(config-if)#end
    S1#
    *Aug 16 22:41:43.659: %SYS-5-CONFIG_I: Configured from console by console
    *Aug 16 22:41:44.222: %LINK-3-UPDOWN: Interface Vlan1, changed state to up
    *Aug 16 22:41:45.226: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
    ```
2. Сделать ограничение доступа через порт консоли с помощью пароля и включить проверку. 
    ```bash
    S1#show running-config | section line vty
    line vty 0 4
    password 7 094F471A1A0A
    login
    S1#
    ```
3. Настроить IP-адрес на PC-A.
     ```bash
     VPCS> ip 192.168.1.2/24
     Checking for duplicate address...
     VPCS : 192.168.1.2 255.255.255.0
     ```

## Проверка сетевых подключений
 1. Проверка конфигурации и тестирование соединения, путем отправки эхо-запроса.
 ```bash
 #PC-A -> S1

 VPCS> ping 192.168.1.10

 192.168.1.10 icmp_seq=1 timeout
 84 bytes from 192.168.1.10 icmp_seq=2 ttl=255 time=3.980 ms
 84 bytes from 192.168.1.10 icmp_seq=3 ttl=255 time=4.270 ms
 84 bytes from 192.168.1.10 icmp_seq=4 ttl=255 time=3.623 ms
 84 bytes from 192.168.1.10 icmp_seq=5 ttl=255 time=3.488 ms
 ```

 ```bash
 #S1 -> PC-A

 S1#ping 192.168.1.2
 Type escape sequence to abort.
 Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
 !!!!!
 Success rate is 100 percent (5/5), round-trip min/avg/max = 2/3/6 ms
 ```
2. Проверка удаленного доступа