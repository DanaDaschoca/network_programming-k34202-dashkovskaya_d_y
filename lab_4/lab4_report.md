University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34202

Author: Dashkovskaya Dana Yuryevna

Lab: Lab4

Date of create: 07.01.2025

Date of finished: 13.01.2025

# Лабораторная работа №4 "Базовая коммутация и туннелирование используя язык программирования P4"

## Цель работы
Познакомиться с языком программирования P4 и выполнить 2 задания от Open network foundation.
### Ход работы 
Сначала нужно скачать образ, так как это путь наименьшего сопротивления. 
Конечно для начала работы необходимо запустить Mininet c помощью команды ``make run``

<img width="541" alt="начинаем работу" src="https://github.com/user-attachments/assets/7f2975bb-90e4-4286-8023-457d4eefa71c" />

Теперь мы должны увидеть командную строку Mininet. Попробуем выполнить пинг между хостами в топологии с помощью команды ``pingall``

<img width="294" alt="неудачный ping all" src="https://github.com/user-attachments/assets/87adce2c-e2af-49ba-b166-c538f7bee76e" />

Как мы видимо ping прошел неудачно. У нас есть файл basic_tunnel.p4. Вот именно его мы и будем изменять. 
Основные изменения: 
- В секции parser добавляем обработку заголовков Ethernet и IPv4
- В разделе ingress реализовываем действия для пересылки пакетов.
- В разделе deparser добавляем сборку пакетов.
Скрины изменений:
1) 
<img width="499" alt="1" src="https://github.com/user-attachments/assets/58c8d730-d1d7-45c9-b445-5508826bcfb7" />

2)
<img width="407" alt="2" src="https://github.com/user-attachments/assets/a2127ca9-fcab-4fa2-a61c-c1869f21ea90" />

3)
<img width="382" alt="3" src="https://github.com/user-attachments/assets/504e4379-5ad2-4de1-aff7-b676452cd643" />

Теперь мы уверены, что ``pingall`` пройдет успешно. 

<img width="244" alt="удачный ping" src="https://github.com/user-attachments/assets/2bdfbeb9-a7d8-4754-b3ea-218fb7e71ac1" />

Приступаем к туннелированию. Опять необходимо внести изменения в файл. 
Основные изменения: 
- В разделе Parsers добавляем состояние state parse_myTunnel
- Включючаем его в оператор select
- Реализовываем логику обработки туннелированных пакетов в ingress
- Включаем обработку заголовка туннеля в deparser
Скрины изменений:

1) 
 <img width="351" alt="тунель1" src="https://github.com/user-attachments/assets/2bbfd8c6-70a0-4c65-b283-17293d7cfb98" />

2) 
<img width="359" alt="тунель2" src="https://github.com/user-attachments/assets/a713f1c3-781a-401f-bcf3-8ccef2be4fe1" />

3) 
<img width="337" alt="тунель3" src="https://github.com/user-attachments/assets/3386a3dd-ed52-4b3e-a1d3-f28cb220ae00" />

Теперь запустим Mininet и проверим. 

<img width="398" alt="запускаем" src="https://github.com/user-attachments/assets/3f25528f-ebad-4e20-88d0-8e8b4b8f9761" />

Теперь проверим как работает без туннелирования
Первый:

<img width="323" alt="без_тунель_первый" src="https://github.com/user-attachments/assets/32dbcca3-8c9a-475e-ab1f-72bd859fdfe4" />

Второй: 

<img width="275" alt="без_тунель_второй" src="https://github.com/user-attachments/assets/cbe9f048-813e-45ce-8bfe-f7911e90bad5" />

Теперь с туннелированием проверим. 

Первый: 

<img width="286" alt="с_тунель_первый" src="https://github.com/user-attachments/assets/837546eb-a00c-4cf6-8d36-879031e8c66d" />

Второй:

<img width="254" alt="с_тунель_второй" src="https://github.com/user-attachments/assets/700b7f59-cf2a-47c1-8cff-44839f18e2c5" />

Проверим работу проверки IP 

Первый: 

<img width="284" alt="айпи_первый" src="https://github.com/user-attachments/assets/59d1adb2-0678-467d-b5c6-8477d4addd7c" />

Второй:

<img width="248" alt="айпи_второй" src="https://github.com/user-attachments/assets/817f738a-8ee8-49f7-ba20-87cca0e16711" />

## Вывод: 
я познакомилась с языком программирования P4. Смотря на лабораторную работу я могу сказать, что этот язык предназначен для описания обработки сетевых пакетов неазвисимо от конкретных сетевых протоколов, P4 помогает настраивать поведение сетевых устройств под специфические потребности. Я с удовольствием ознакомилась с данным языком. 
