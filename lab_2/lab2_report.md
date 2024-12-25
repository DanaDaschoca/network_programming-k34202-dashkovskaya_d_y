University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34202

Author: Dashkovskaya Dana Yuryevna

Lab: Lab2

Date of create: 24.10.2024

Date of finished: 26.10.2024

# Лабораторная работа №2 "Развертывание дополнительного CHR, первый сценарий Ansible"

## Цель работы
С помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.

## Ход выполнения работы

1) В VirualBox содаем новую виртуальную машину Router2

2) Настраиваем через WinBox интерфейс Wireguard, как и в первой лабораторной работе. Добавляем в peers информацию о сервере

3) После выполнения настроек на втором CHR добавляем нового клиента в конфигурационный файл wg0.conf на сервере и перезапускаем Wireguard, чтобы изменения вступили в силу

![1](https://github.com/user-attachments/assets/4b738ef0-688a-4c6e-b97a-d4b428febe42)

4) Проверим, что VPN туннель поднимается

![2](https://github.com/user-attachments/assets/bd6c3e47-b044-4417-b193-2be84c9f0d06)

5) Перед работой с Ansible пробросим на роутеры публичный ключ сервера, чтобы в дальнейшем подключаться к ним по ssh без пароля

![3](https://github.com/user-attachments/assets/01f176b7-3492-46b3-b5ed-7a308e83bf37)

![4](https://github.com/user-attachments/assets/84098699-6888-4478-9f1c-6a76f772c080)

6) На сервере создадим директорию ```ansible-project```, а в ней файл [inventory.yaml](./ansible-project/inventory.yaml). В файле пропишем адреса наших роутеров, к которым будет выполняться подключение, а также следующие переменные:
   * ansible_connection=network_cli тип подключения, который используется для работы с сетевым оборудованием
   * ansible_network_os=routeros операционная система оборудования - RouterOS
   * ansible_user=admin пользователь, по которому следует выполнять подключение
   * ansible_ssh_private_key_file=~/.ssh/id_rsa путь к приватному ключу сервера, который используется при ssh подключении

```yaml

```

7) Теперь создадим плейбук, который будет выполнять следующие задачи сразу на двух CHR: настройка пароля, NTP (Network Time Protocol) и OSPF (Open Shortest Path First), сбор данных по OSPF топологии и полных конфигов устройств. Для этого создадим файл [main_playbook.yaml](./ansible-project/main_playbook.yaml) и пропишем в нем эти задачи (tasks) с использованием модуля community.routeros.command для удаленного выполнения команд на устройствах MikroTik RouterOS.

```yaml


```
8) Запускаем выполнение плейбука командой ```ansible-playbook -i invetory.yaml main_playbook.yaml```

![5](https://github.com/user-attachments/assets/ffdf639c-c4b2-46d9-8586-7429d967c34a)

9) В результате на обоих роутерах был изменен пароль, настроены NTP клиент (в качестве NTP-сервера был выбран MSK-IX) и протокол маршрутизации OSPF, а также получены данные по OSPF топологии и полные конфиги устройств:

```bash
TASK [Show OSPF topology data] **************************************************************************************************************************************************************
ok: [router1] => {
    "ospf_data.stdout_lines": [
        [
            "Flags: V - virtual; D - dynamic ",
            " 0  D instance=default area=backbone address=172.20.10.5 router-id=2.2.2.2 ",
            "      state=\"Init\" state-changes=1 timeout=36s"
        ]
    ]
}
ok: [router2] => {
    "ospf_data.stdout_lines": [
        [
            "Flags: V - virtual; D - dynamic ",
            " 0  D instance=default area=backbone address=172.20.10.2 router-id=1.1.1.1 ",
            "      state=\"Init\" state-changes=1 timeout=37s"
        ]
    ]
}
```

```bash
TASK [Show full config] *********************************************************************************************************************************************************************
ok: [router1] => {
    "router_config.stdout_lines": [
        [
            "# 2024-12-25 21:41:29 by RouterOS 7.16.2",
            "# software id = ",
            "#",
            "/interface wireguard",
            "add listen-port=13231 mtu=1420 name=wg",
            "/routing ospf instance",
            "add disabled=no name=default router-id=1.1.1.1",
            "/routing ospf area",
            "add disabled=no instance=default name=backbone",
            "/interface wireguard peers",
            "add allowed-address=10.0.0.0/24 endpoint-address=172.20.10.3 endpoint-port=\\",
            "    13231 interface=wg name=peer1 public-key=\\",
            "    \"w7cMWbtU7Io8oRxJSOwD9kqJPiLg29VYumA51zM/tQk=\"",
            "/ip address",
            "add address=10.0.0.2/24 interface=wg network=10.0.0.0",
            "add address=1.1.1.1 interface=lo network=1.1.1.1",
            "add address=2.2.2.2 interface=lo network=2.2.2.2",
            "/ip dhcp-client",
            "add interface=ether1",
            "/ip route",
            "add dst-address=10.0.0.0/24 gateway=wg",
            "/routing ospf interface-template",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "/system note",
            "set show-at-login=no",
            "/system ntp client",
            "set enabled=yes",
            "/system ntp client servers",
            "add address=194.190.168.1"
        ]
    ]
}
ok: [router2] => {
    "router_config.stdout_lines": [
        [
            "# 2024-12-25 21:41:29 by RouterOS 7.16.2",
            "# software id = ",
            "#",
            "/interface wireguard",
            "add listen-port=13231 mtu=1420 name=wg",
            "/routing ospf instance",
            "add disabled=no name=default router-id=2.2.2.2",
            "/routing ospf area",
            "add disabled=no instance=default name=backbone",
            "/interface wireguard peers",
            "add allowed-address=10.0.0.0/24 endpoint-address=172.20.10.3 endpoint-port=\\",
            "    13231 interface=wg name=peer1 public-key=\\",
            "    \"w7cMWbtU7Io8oRxJSOwD9kqJPiLg29VYumA51zM/tQk=\"",
            "/ip address",
            "add address=10.0.0.3/24 interface=wg network=10.0.0.0",
            "add address=2.2.2.2 interface=lo network=2.2.2.2",
            "/ip dhcp-client",
            "add interface=ether1",
            "/ip route",
            "add dst-address=10.0.0.0/24 gateway=wg",
            "/routing ospf interface-template",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "/system note",
            "set show-at-login=no",
            "/system ntp client",
            "set enabled=yes",
            "/system ntp client servers",
            "add address=194.190.168.1"
        ]
    ]
}
```
## Выводы
В ходе выполнения лабораторной работы была проведена настройка сразу двух устройств CHR с помощью Ansible. Для этого был написан playbook, выполнение которого позволило провести необходимую настройку устройств и собрать данные о них. 
