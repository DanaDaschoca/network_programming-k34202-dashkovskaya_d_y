University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34202

Author: Dashkovskaya Dana Yuryevna

Lab: Lab1

Date of create: 1.10.2024

Date of finished: 15.10.2024

# Лабораторная работа №1 "Установка CHR и Ansible, настройка VPN"
## Описание
Данная работа предусматривает обучение развертыванию виртуальных машин (VM) и системы контроля конфигураций Ansible а также организации собственных VPN серверов.

## Цель работы
Целью данной работы является развертывание виртуальной машины на базе платформы Microsoft Azure с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox

## Ход работы
1) В VirtualBox была создана виртуальная машина Router1, на которой был установлен CHR (Cloud Hosted Router) от MikroTik.

![1](https://github.com/user-attachments/assets/cd5e2071-69e6-44eb-8715-21ec282abf25)

2) С помощью WinBox подключаемся к роутеру и настраиваем интерфейс Wireguard.
   
![2](https://github.com/user-attachments/assets/03cdeb02-2429-4c9f-89d6-addcf178a7b5)

3) Созданному интерфейсу wg назначаем адрес 10.0.0.2/24
   
![3](https://github.com/user-attachments/assets/5a8523ea-c451-4711-993b-9280e2d49833)

4) Далее создадим еще одну виртуальную машину. Для сервера был выбран образ Ubuntu 22.04

![4](https://github.com/user-attachments/assets/3d4ef645-4e5f-4861-9fde-9bc384b95745)

5) После установки Ubuntu, были установлены Python и Ansible, а также Wireguard

![5](https://github.com/user-attachments/assets/b6c846a6-3b05-4f15-81d6-9488ba35f7f9)

6) Для работы с Wireguard генерируем публичный и приватный ключи
   
```bash
wg genkey | tee prkey | wg pubkey | tee pubkey
```

7) Далее в той же директории ```/etc/wireguard``` создаем конфигурационный файл wg0.conf, где необходимо определить интерфейс и клиента

![6](https://github.com/user-attachments/assets/0befda59-dea4-4e99-9842-b8f84d11bb3d)

8) После этого запускаем службу Wireguard

![7](https://github.com/user-attachments/assets/81983f60-7a1a-431a-b152-3d7d7b98f571)

9) Возвращаемся на роутер и добвляем в Wireguard информацию о нашем сервере
   
![8](https://github.com/user-attachments/assets/0c69babb-659b-4a37-ab42-7658fb36b9e8)

10) Теперь можно проверить связность:

Клиент >> Сервер

![9](https://github.com/user-attachments/assets/d0588a93-d4e7-48fe-83e2-d9260aec67ff)

Сервер >> Клиент

![10](https://github.com/user-attachments/assets/718437d9-b960-4932-9fc5-d162a3c963ae)

## Вывод: 
В ходе лабораторной работы была настроена связь между сервером и клиентом. Также был установлен python и Ansible. Настройка связи была сделана с помощью сертификатов, которые необходимо было клиенту передать, также была произведена настйрока сервера и клиента через конфигурационные файлы. Сложность лабораторной работы заключалась в опечатках, которые были допущены из-за больших конфигурационных файлов, эти опечатки сложно было найти, что занимало много времени. 
