```bash
ssh root@167.99.131.124
```

# Ad-hoc

У Ansible несколько режимов работы. Самый простой – ad-hoc, когда запрос к серверу выполняется напрямую из командной строки, без создания дополнительных файлов.

```bash
# Проверяет доступность сервера по ip-адресу (ad-hoc)
# all – запрос выполняется для всех указанных машин
# 167.99.131.124 – ip-адрес моей машины
# root – пользователь для подключения по ssh
# ping – используемая команда (модуль ansible)
ansible all -i '167.99.131.124, ' -u root -m ping

ansible all -i '167.99.131.124, ' -u root -m command -a 'uptime'
ansible all -i '167.99.131.124, ' -u root -a 'uptime'

# Файл инвентаризации
#    помогает не указывать каждый раз список хостов
echo "167.99.131.124" > inventory.ini

ansible all -i inventory.ini -u root -m ping
```

# Файл инвентаризации

(hexlet.io)[https://ru.hexlet.io/courses/ansible/lessons/inventory_file/theory_unit]

В inventory.ini файле можно выделять группы `[group_name]`. В таком случае, ad-hoc вызов будет вот такой

```bash
ansible <group_name> -i inventory.ini -u root -a 'uptime'
```

Можно указать своё имя хоста в inventory.ini файле. Пример

```
# inventory.ini
do1 ansible_host=167.99.131.124

# флага --limit выполняет запросы на конкретном сервере
$ ansible all --limit do1 -i inventory.ini -m ping
```

