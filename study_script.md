```bash
ssh root@do_test_1
```

# Ad-hoc

У Ansible несколько режимов работы. Самый простой – ad-hoc, когда запрос к серверу выполняется напрямую из командной строки, без создания дополнительных файлов.

```bash
# Проверяет доступность сервера по ip-адресу (ad-hoc)
# all – запрос выполняется для всех указанных машин
# do_test_1 – ip-адрес моей машины
# root – пользователь для подключения по ssh
# ping – используемая команда (модуль ansible)
ansible all -i 'do_test_1, ' -u root -m ping

ansible all -i 'do_test_1, ' -u root -m command -a 'uptime'
ansible all -i 'do_test_1, ' -u root -a 'uptime'

# Файл инвентаризации
#    помогает не указывать каждый раз список хостов
echo "do_test_1" > inventory.ini

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
do1 ansible_host=do_test_1

# флага --limit выполняет запросы на конкретном сервере
$ ansible all --limit do1 -i inventory.ini -m ping
```

# Плейбуки

Одним из основных понятий в Ansible является плейбук. Это просто yaml-файл, в котором мы указываем, какие задачи и на каких серверах будут выполняться.

```yaml
# На какой группе серверов
- hosts: webservers

  tasks:
    - name: install redis server
      # apt-get update && apt-get install redis-server
      ansible.builtin.apt: # имя модуля Ansible
        name: redis-server
        state: present
        update_cache: yes

    - name: remove redis server
      # apt-get remove redis-server
      ansible.builtin.apt:
        name: redis-server
        state: absent

```

```bash
ansible-playbook playbooks/test_redis.yml -i inventory.ini -u root
```

# Тэги

С ростом количества задач плейбуки становятся достаточно большими. И при отладке сценария это может вызывать неудобства. Мы можем пометить задачи тегами и запускать их, когда это необходимо.

```bash
ansible-playbook --check playbooks/git_install.yml -i inventory.ini  -u root -t update
ansible-playbook --check playbooks/git_install.yml -i inventory.ini  -u root -t git
ansible-playbook playbooks/git_install.yml -i inventory.ini  -u root
```

# Обработчики (Handlers)

При выполнении задач в плейбуках периодически возникает необходимость перезапускать какой-либо сервис. Хорошая практика, - создать хэндлер (перезапуск сервиса) отдельно и связать его с таском.

Обработчики отрабатывают, когда завершается весь сценарий!

```yaml
- hosts: webservers
  tasks:
    ...

  handlers:
    - name: restart nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
      become: yes
```

# Переменные

...позволяют описывать конфигурационные параметры в одном месте и переиспользовать их многократно.

```yaml
- hosts: webservers
  vars:
    root_dir: /var/tmp/www
  tasks:
    - name: update nginix config
      # template прогоняет файл через Jinja 2
      ansible.builtin.template:
        src: templates/nginx.conf.j2
        dest: "{{root_dir}}/nginx.conf"

    - name: update index.html
      ansible.builtin.copy:
        src: files/index.html
        # переменная из vars
        dest: "{{root_dir}}/index.html"
```

```bash
ansible-playbook playbooks/nginx_install.yml -i inventory.ini -u root
```

# Роли

Ansible выделяет повторяющиеся вещи в роли. Эти роли выкладываются в общий (каталог)[https://galaxy.ansible.com/home], через который любой человек может найти себе готовое решение по установке и настройке чего-либо.

Роль представляет собой набор задач, обработчик, переменных, файлов и других артефактов, которые распространяются и подключаются как единое целое к плейбуку. Обычно, роли выполняют достаточно высокоуровневые задачи, например установку баз данных, веб-серверов и тому подобное. Иногда они автоматизируют работу с каким-то низкоуровневым сервисом, который не встроен в сам Ansible.

```bash
# сгенерировать новую Роль
ansible-galaxy init deploy_apache_web

# запустить роль
ansible-playbook -i inventory.ini -u root roles/deploy-example.**yml**
```

# rsync

Позволяет синхронизировать директории, в том числе и удалённо.

```bash
# однократно синхронизировать локально
rsync -r test/ test_1

# однократно синхронизировать с удалённой машиной
rsync -zaP test/ --rsh='ssh -p 22' root@do_test_1:/tmp/test_2
```