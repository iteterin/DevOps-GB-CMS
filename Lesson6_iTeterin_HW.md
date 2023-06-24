## Системы управления конфигурациями

### Урок 6. Практика на проекте

Добавить в проект роль Ansible установки сервера БД.



Создаем роль для деплоя MySql:

```bash
cd ~/ansible/roles && ansible-galaxy init deploy_mysql
```

Создаем файл тасков **~/ansible/roles/deploy_mysql/tasks/main.yml**:

```yaml
---
# vars file for deploy_mysql
#
mysql_root_password: "root4ever"
```

Шифруем (vault):

```bash
cd ~/ansible && ansible-vault encrypt roles/deploy_mysql/vars/main.yml
```

Создаем файл с переменными **~/ansible/roles/deploy_mysql/vars/mysql_vars.yml**:

```yaml
mysql_packages:
      - mariadb-client
      - mariadb-server
      - python3-mysqldb
mysql_python_package_debian: python3-mysqldb
mysql_enabled_on_startup: true
mysql_port: "3306"
mysql_bind_address: '0.0.0.0'
mysql_databases:
  - name: wp_geek
mysql_users:
  - name: wp_geek_user
    host: "%"
    password: geek_pwd
    priv: "wp_geek.*:ALL"
```

Создаем плейбук для развертывания MySQL и базы wp_geek  **~/ansible/roles/deploy_mysql/deploy_mysql.yml**: 

```yaml
---
- hosts: mysqldb 
  become: yes
  vars_files:
    - vars/main.yml
    - vars/mysql_vars.yml
  roles:
    - role: geerlingguy.mysql
```

Запускаем плейбук: 

```bash
cd ~/ansible && ansible-playbook roles/deploy_vm_ovirt/deploy_vm_ovirt.yml  --vault-password-file ./vault.pass
```

Задание успешно выполнено: 

 

```bash
ansible-playbook -i inventory/hosts.yml roles/deploy_mysql/deploy_mysql.yml --vault-password-file ./vault.pass

PLAY [mysqldb] ********************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [mysqldb]

~ Skip output ~

RUNNING HANDLER [geerlingguy.mysql : restart mysql] ************************************************************************************************************************************************************
[WARNING]: Ignoring "sleep" as it is not used in "systemd"
changed: [mysqldb]

PLAY RECAP ********************************************************************************************************************************
mysqldb                    : ok=37   changed=9    unreachable=0    failed=0    skipped=19   rescued=0    ignored=0   
```

![image-20230624111703034](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230624111703034.png) 



Проверим работу: 

 ![image-20230624110742746](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230624110742746.png) 

И с РМ DBA, хост доступен по ЛВС, коннект к созданной БД под созданным пользователем есть: 

![image-20230624112240306](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230624112240306.png)



Роль работает. Но нам же надо развернуть ее автоматически.. 

Создаем **~/ansible/playbooks/main.yml**:

```yaml
---
- name: deploy_vm_ovirt
  import_playbook: /home/sa/ansible/roles/deploy_vm_ovirt/deploy_vm_ovirt.yml

- name: deploy_mysql
  import_playbook: /home/sa/ansible/roles/deploy_mysql/deploy_mysql.yml
```

Проверяем:

```bash
cd ~/ansible && ansible-playbook playbooks/mail.yaml  --vault-password-file ./vault.pass
```

Результат:

![image-20230624114902488](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230624114902488.png) 
