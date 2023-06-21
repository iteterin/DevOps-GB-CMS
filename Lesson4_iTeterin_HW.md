## Системы управления конфигурациями

### Урок 4. Ansible advanced

Модифицировать роль с использованием vault и tags.

Создано хранилище:

```bash
ansible-vault create ~/ansible/roles/wordpress/vars/wordpress_vault.yml
```

![image-20230620212322683](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230620212322683.png) 

Запущен деплой:

![image-20230620212352758](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230620212352758.png) 

![image-20230620212413604](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230620212413604.png) 

Роль раздеплоена успешно с использованием хранилища. 

```yaml
---
- hosts: all

  vars_files:
    - vars/wordpress.yml
    - vars/wordpress_vault.yml
  roles:
    - { role: geerlingguy.nginx, tags: [nginx] }
    - { role: geerlingguy.php, tags: [php] }
    - { role: geerlingguy.mysql, tags: [mysql] }
    - { role: wordpress, tags: [wordpress, app] }
```

Проверим: 

```bash
echo "geek" > vault.pass
cd ~/ansible && ansible-playbook -i inventory/hosts.yml roles/wordpress/wordpress.yml -b -K -k --vault-password-file ./vault.pass --list-tags
```

![image-20230620213006506](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230620213006506.png) 

Проверим задание из примера: 

![image-20230620213102856](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230620213102856.png)

 ![image-20230620213116200](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230620213116200.png) 