### Урок 3. Используем Ansible на практике (base)

Установка приложения MySQL + WordPress + FPM + nginx с помощью плейбука Ansible.
\* (по желанию, но рекомендую) Доработать проект чтобы получить сразу работающее приложение Wordpress.



Настраиваем каталог проекта. В репозитории git - ветка lesson03, папка Project.
Ранее была выполнена настройка хоста geekbrains (Debian 12) на авторизацию по SSH с помощью сертификата и повышение прав пользователя Server Administrator (sa) с помощью утилиты sudo без пароля. 

На ansible хосте:

```bash
ssh-keygen -t rsa -C "sa@ansible.local"
ssh-copy-id sa@geekbrains.local
```

 На geekbrains хосте: 

```bash
cat > /etc/sudoers.d/sa_without_pwd << _EOF_
sa ALL=(ALL) NOPASSWD: ALL
_EOF_
```

1. Обогащаем фаил hosts.yml (и прорабатываем информационные сообщения из предыдущего ДЗ - указываем путь к интерпритатору)

   ```yaml
   all:
     children:
       webservers:
         hosts:
           geekbrains:
             ansible_host: 10.78.1.14
         vars:
           ansible_user: sa
           ansible_python_interpreter: /usr/bin/python
           php_webserver_deamon: "nginx"
   ```

   Проверяем: 
   ![image-20230618135211648](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230618135211648.png)  

2. Правим фаил-конфигурацию проекта:

   ![image-20230618145506550](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230618145506550.png) 

   Остальные параметры оставляем по-умолчанию.
   Проверяем корректность внесенных параметров ролью из лекции:
   ![image-20230618122301292](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230618122301292.png) 

3. Инициализируем роли из Ansible Galaxy:

   ```bash
   ansible-galaxy install geerlingguy.nginx geerlingguy.mysql geerlingguy.php
   ```

4. Создаем роль Wordpress.
   **~/ansible/roles/wordpress/wordpress.yml**:

   ```yaml
   ---
   - hosts: all
   
     vars_files:
       - vars/wordpress.yml
     roles:
       - geerlingguy.nginx
       - geerlingguy.php
       - geerlingguy.mysql
       - wordpress
   ```

   **~/ansible/roles/wordpress/tasks/main.yml**:

   ```yaml
   ---
   # tasks file for wordpress
   
   # WordPress Configuration
   - name: create /var/www/{{ server_name }} directory for unarchiving
     file:
       path: /var/www/{{ server_name }}
       state: directory
   
   - name: Download and unpack latest WordPress
     unarchive:
       src: https://wordpress.org/latest.tar.gz
       dest: "/var/www/{{ server_name }}"
       remote_src: yes
       creates: "/var/www/{{ server_name }}/wordpress"
   
   - name: Set ownership
     file:
       path: "/var/www/{{ server_name }}"
       state: directory
       recurse: yes
       owner: www-data
       group: www-data
   
   - name: Set permissions for directories
     shell: "/usr/bin/find /var/www/{{ server_name }}/wordpress/ -type d -exec chmod 750 {} \\;"
   
   - name: Set permissions for files
     shell: "/usr/bin/find /var/www/{{ server_name }}/wordpress/ -type f -exec chmod 640 {} \\;"
   
   - name: Set up wp-config
     template:
       src: "wp-config.php.j2"
       dest: "/var/www/{{ server_name }}/wordpress/wp-config.php"
   ```

   **~/ansible/roles/wordpress/vars/wordpress.yml**:

   ```yaml
   ---
   server_name: "geekbrains.local" # тут ваш домен
   mysql_db: "wordpress"
   mysql_user: "wordpress"
   mysql_password: "WordpreSS0!"
   
   nginx_vhosts:
     - listen: "80"
       server_name: "{{ server_name }} www.{{ server_name }}"
       return: "301 https://{{ server_name }}$request_uri"
       filename: "{{ server_name }}.80.conf"
   
     - listen: "443 ssl http2"
       server_name: "{{ server_name }}"
       root: "/var/www/{{ server_name}}/wordpress"
       index: "index.php index.html index.htm"
       error_page: "500 502 503 504 /50x.html"
       access_log: "/var/log/nginx/{{ server_name }}.access.log main"
       error_log: ""
       state: "present"
       template: "{{ nginx_vhost_template }}"
       filename: "{{ server_name }}.conf"
       extra_parameters: |
         location ~ \.php$ {
             fastcgi_split_path_info ^(.+\.php)(/.+)$;
             fastcgi_pass unix:/var/run/php/php-fpm.sock;
             fastcgi_index index.php;
             fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
             include fastcgi_params;
         }
         ssl_certificate /etc/letsencrypt/live/{{ server_name }}/fullchain.pem;
         ssl_certificate_key /etc/letsencrypt/live/{{ server_name }}/privkey.pem;
         ssl_protocols TLSv1.1 TLSv1.2;
         ssl_ciphers HIGH:!aNULL:!MD5;
   
   
   php_memory_limit: "128M"
   php_max_execution_time: "90"
   php_upload_max_filesize: "256M"
   php_webserver_daemon: "nginx"
   php_packages:
     - php
     - php-cli
     - php-common
     - php-gd
     - php-mbstring
     - php-pdo
     - php-xml
     - php-curl
     - php-gd
     - php-mbstring
     - php-xml
     - php-xmlrpc
     - php-soap
     - php-intl
     - php-zip
     - php-mysql
   
   mysql_databases:
     - name: "{{ mysql_user }}"
       host: "%"
       password: "{{ mysql_password }}"
       priv: "wordpress.*:ALL"
   ```

   **~/ansible/roles/wordpress/templates/wp-config.php.j2**:

   ```php+HTML
   <?php
   /** The name of the database for WordPress */
   define( 'DB_NAME', '{{ mysql_db }}' );
   /** MySQL database username */
   define( 'DB_USER', '{{ mysql_user }}' );
   /** MySQL database password */
   define( 'DB_PASSWORD', '{{ mysql_password }}' );
   /** MySQL hostname */
   define( 'DB_HOST', 'localhost' );
   /** Database Charset to use in creating database tables. */
   define( 'DB_CHARSET', 'utf8' );
   /** The Database Collate type. Don't change this if in doubt. */
   define( 'DB_COLLATE', '' );
   /** Filesystem access **/
   define('FS_METHOD', 'direct');
   
   define( 'AUTH_KEY', '{{ lookup('password', '/dev/null chars=ascii_letters length=64') }}' );
   define( 'SECURE_AUTH_KEY', '{{ lookup('password', '/dev/null chars=ascii_letters length=64') }}' );
   define( 'LOGGED_IN_KEY', '{{ lookup('password', '/dev/null chars=ascii_letters length=64') }}' );
   define( 'NONCE_KEY', '{{ lookup('password', '/dev/null chars=ascii_letters length=64') }}' );
   define( 'AUTH_SALT', '{{ lookup('password', '/dev/null chars=ascii_letters length=64') }}' );
   define( 'SECURE_AUTH_SALT', '{{ lookup('password', '/dev/null chars=ascii_letters length=64') }}' );
   define( 'LOGGED_IN_SALT', '{{ lookup('password', '/dev/null chars=ascii_letters length=64') }}' );
   define( 'NONCE_SALT', '{{ lookup('password', '/dev/null chars=ascii_letters length=64') }}' );
   
   $table_prefix = 'wp_';
   
   define( 'WP_DEBUG', false );
   
   if ( ! defined( 'ABSPATH' ) ) {
   define( 'ABSPATH', dirname( __FILE__ ) . '/' );
   }
   
   /** Sets up WordPress vars and included files. */
   require_once( ABSPATH . 'wp-settings.php' );
   ```

5. Запускаем деплой:

   ```bash
   cd ~/ansible && ansible-playbook -i inventory/hosts.yml roles/wordpress/wordpress.yml -b
   ```

   Результат 

   ![image-20230618164707885](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230618164707885.png) 