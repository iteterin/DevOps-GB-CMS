## Системы управления конфигурациями

### Урок 8. Практика на проекте

Добавить в проект роль балансировщика и завершить работу над проектом. Проект должен разворачиваться полностью автоматически, в результате получаем работающую систему.
(по желанию) Добавить в проект мониторинг.

Добавляем роль балансировщика и создаем файлы:

```bash
ansible-galaxy init nginx_proxy
```

/home/sa/ansible/roles/nginx_proxy/files/nginx.conf

```bash
server {
    listen 80 ;
}
```

/home/sa/ansible/roles/nginx_proxy/handlers/main.yml

```yaml
---
# handlers file for nginx_proxy
- name: restart nginx
  service: name=nginx state=restarted
```

/home/sa/ansible/roles/nginx_proxy/tasks/main.yml

```yaml
---
# tasks file for nginx_proxy

- name: install nginx
  yum: name=nginx state=latest
  when: ansible_os_family == "Debian" or ansible_os_family == "Ubuntu"

- name: install conf
  template: src=proxy.conf.j2 dest=/etc/nginx/conf.d/vhost1.conf
  tags: conf
  notify: restart nginx

- name: install nginx.conf
  copy: src=nginx.conf  dest=/etc/nginx/sites-available/default
- name: start nginx
  service: name=nginx state=started
```

/home/sa/ansible/roles/nginx_proxy/templates/proxy.conf.j2

```yaml
upstream websrv {
    server 10.78.1.46:80;
    server 10.78.1.47:80;
}
 
server {
    listen 80 default_server;
    server_name "{{ server_name }}";
    location / {
        proxy_pass http://websrv/;
        proxy_set_header Host $host;
        proxy_set_header X-Forward-For $remote_addr;
    }
}

```

/home/sa/ansible/roles/nginx_proxy/nginx_proxy.yml:

```yaml
---
- hosts: haproxy
  become: true

  roles:
  - role: nginx_proxy
    vars:
      - server_name: "app.local"
```

Запускаем роль: 

```bash
ansible-playbook roles/nginx_proxy/nginx_proxy.yml
```

Проверяем: 

Добавим на каждый сервер по файлу index.php в корень с разным содержимым. 
Для App01:

![image-20230705094219348](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230705094219348.png) 

App02:
![image-20230705094240386](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230705094240386.png) 

Проверяем: 
![image-20230705094256822](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230705094256822.png) 

![image-20230705094312982](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230705094312982.png) 

RR работает! 

Проверяем ранее раздеплоеный сайт: 

![image-20230705093013198](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230705093013198.png) 

Завершаем установку мастером и все работает: 

![image-20230705093140904](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230705093140904.png) 



Осталось только добавить деплой прокси в проект: 

```yaml
---
- name: deploy_vm_ovirt
  import_playbook: /home/sa/ansible/roles/deploy_vm_ovirt/deploy_vm_ovirt.yml

- name: deploy_mysql
  import_playbook: /home/sa/ansible/roles/deploy_mysql/deploy_mysql.yml

- name: deploy_webserver
  import_playbook: /home/sa/ansible/roles/deploy_webserver/deploy_webserver.yml

- name: deploy_nginx_proxy
  import_playbook: /home/sa/ansible/roles/nginx_proxy/nginx_proxy.yml

- name: deploy_wordpress
  import_playbook: /home/sa/ansible/roles/wordpress/wordpress.yml
```

