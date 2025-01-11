University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34202

Author: Dashkovskaya Dana Yuryevna

Lab: Lab3

Date of create: 11.01.2025

Date of finished: 11.01.2025

# Лабораторная работа №3 "Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"

## Цель работы
С помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.

## Ход выполнения работы

### Установка NetBox
Чтобы не заниматься поднятием NetBox вручную, устанавливаем на виртуальную машину Docker и скачиваем репозиторий netbox-docker: ```git clone -b release https://github.com/netbox-community/netbox-docker.git```. Переходим в полученную директорию и создаем файл docker-compose.override.yml со следующим содержимым:
```yaml
services:
  netbox:
    ports:
      - 8000:8080
```
Таким образом, netbox будет доступен на порту 8000. Запускаем ```docker compose up``` и получаем все необходимые контейнеры для работы netbox

![1](https://github.com/user-attachments/assets/27b4a22d-b284-4d9b-85fe-836ea3d20ebf)

Пробрасываем порт 8000 на хост машину в VirtualBox и открываем netbox в браузере

![2](https://github.com/user-attachments/assets/2b810b4c-c8d6-491a-ba29-044855bac4bb)

Но прежде чем авторизоваться, нужно создать пользователя. Возвращаемся на виртуальную машину и выполняем ```docker compose exec netbox /opt/netbox/netbox/manage.py createsuperuser```

![3](https://github.com/user-attachments/assets/58cf6a48-7a57-47ec-bb41-7386337136e7)

### Заполнение информации об устройствах
В netbox добавляем сайт, роль и тип устройства, производителя, интерфейсы и IP-адреса, и наконец сами устройства: CHR1 и CHR2

![4](https://github.com/user-attachments/assets/dfa63678-56b5-405e-8b0a-e349e0b0364f)

### Сохранение данных из NetBox в отдельный файл
Переходим к работе с Ansible и создаем файл inventory.yaml со следующим содержимым:
```yaml
all:
  hosts:
    localhost:
      ansible_connection: local
```
А также плейбук fetch_info.yaml, который будет собирать информацию, доступную в netbox, и записывать полученные данные в файл формата json
```yaml
---
- name: Gather devices info
  hosts: localhost
  gather_facts: false
  vars:
    netbox_url: "http://10.0.2.15:8000"
    netbox_token: "c2319e8b9a666cc67fa072c6465ebf917679c778"
    output_file: "netbox_devices.json"
  tasks:
    - name: NetBox Connect
      uri:
        url: "{{ netbox_url }}/api/dcim/devices/"
        headers:
          Authorization: "Token {{ netbox_token }}"
        validate_certs: false
        return_content: true
      register: netbox_response

    - name: Export Data
      copy:
        content: "{{ netbox_response.json | to_nice_json }}"
        dest: "{{ output_file }}"
```

*Для подключения к netbox был создан токен, который используется для авторизации

Выполнение плейбука:

![5](https://github.com/user-attachments/assets/5edc8b81-4e3b-404e-ace7-47883bc3b056)

### Изменение имени устройства и добавление IP-адреса на основе данных из netbox
Для работы с роутером дополним файл inventory.yaml:
```yml
all:
  hosts:
    localhost:
      ansible_connection: local
    chr_host:
      ansible_host: 172.20.10.2
      ansible_user: admin
      ansible_connection: ansible.netcommon.network_cli
      ansible_network_os: community.routeros.routeros
      ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

Создадим новый плейбук router_setting.yaml, в котором используем написанную ранее часть плейбука fetch_info.yaml для сбора информации из netbox, а также напишем сценарий ```Set name and ip address```, в котором будут выполняться задачи по изменению имени устройства (по умолчанию - MikroTik, в netbox - CHR1) и добавлению ip-адреса (адрес устройства в netbox был изменен на 192.168.0.1) 
```yaml
---
- name: Fetch Info from NetBox
  hosts: localhost
  gather_facts: false
  vars:
   netbox_url: "http://10.0.2.15:8000"
   netbox_token: "c2319e8b9a666cc67fa072c6465ebf917679c778"
  tasks:
    - name: Fetch info
      uri:
        url: "{{ netbox_url }}/api/dcim/devices"
        headers:
          Authorization: "Token {{ netbox_token }}"
        method: GET
        return_content: yes
        validate_certs: false
      register: device_data
    - name: Get name and ip address
      set_fact:
        device_name: "{{ device_data.json.results[0].name }}"
        netbox_ip_address: "{{ device_data.json.results[0].primary_ip.address }}"

- name: Set name and ip address
  hosts: chr_host
  gather_facts: false
  tasks:
    - name: Change name
      community.routeros.command:
        commands:
          - /system identity set name={{ hostvars['localhost'].device_name }}
    - name: Add ip address
      community.routeros.command:
        commands:
          - /ip address add address={{ hostvars['localhost'].netbox_ip_address }} interface=ether1 disabled=no
```

Выполнение плейбука:

![6](https://github.com/user-attachments/assets/e15ce2c8-158e-441c-8f25-69f365bd87c9)

Отследить изменения, вносимые выполнением плейбука, можно через логи в Winbox:

![7](https://github.com/user-attachments/assets/7dee65c8-3499-483e-bafc-7b641874bd0c)

И проверим в консоли самого устройства:

![8](https://github.com/user-attachments/assets/f6933e2d-5893-4e5e-a346-e690729a54c2)

Видим, что все прошло успешно, имя устройства было изменено на CHR1, на интерфейсе ether1 появился еще один ip-адрес 192.168.0.1

### Внесение серийного номера в netbox
Файл inventory.yaml не меняем, в нем уже содержится все необходимое. Создаем еще один плейбук serial_number.yaml:
```yaml
---
- name: Working on CHR
  hosts: chr_host
  gather_facts: false
  tasks:
    - name: Get serial number
      community.routeros.command:
        commands:
          - /system license print
      register: serial_output
    - name: Parsing
      set_fact:
        serial_number: "{{ serial_output.stdout_lines[0][0] | regex_search('system-id: (\\S+)','\\1') }}"
    - name: Debug
      debug:
        msg: "Получили номер {{ serial_number }}"

- name: Working in NetBox
  hosts: localhost
  gather_facts: false
  vars:
   netbox_url: "http://10.0.2.15:8000"
   netbox_token: "c2319e8b9a666cc67fa072c6465ebf917679c778"
  tasks:
    - name: Add serial number to NetBox
      uri:
        url: "{{ netbox_url }}/api/dcim/devices/1/"
        method: PATCH
        headers:
          Authorization: "Token {{ netbox_token }}"
          Content-Type: "application/json"
        body:
          serial: "{{ hostvars['chr_host'].serial_number[0] | string }}"
        body_format: json
        validate_certs: no
      register: update_response
```

В нем есть сценарий ``` Working on CHR``` для работы на роутере (получаем данные о серийном номере устройства, парсим вывод, чтобы убрать лишнее, и на всякий случай выводим в консоль полученное) и сценарий ```Working in NetBox``` для работы с netbox (используем метод PATCH для добавления полученной информации об устройстве). Выполнение:

![9](https://github.com/user-attachments/assets/43a96636-0c8c-4a73-bac8-3ba8027d43b5)

Смотрим на появившийся серийный номер устройства в netbox:

![10](https://github.com/user-attachments/assets/46de35f2-74d6-4feb-b686-09f104240457)

### Результат
В ходе выполнение лабораторной работы был получен файл с данными об устройствах из NetBox ([devices_info.json](./result/devices_info.json)), а также написаны сценарии для настройки CHR на основе данных из NetBox ([router_setting.yaml](./result/router_setting.yaml)) и внесения серийного номера в NetBox ([serial_number.yaml](./result/serial_number.yaml))














