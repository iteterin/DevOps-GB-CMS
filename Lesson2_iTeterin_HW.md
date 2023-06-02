## Системы управления конфигурациями

### Урок 2. Ansible — как готовить

Установить и настроить Ansible. Результатом работы должен быть вывод Ansible -m setup.

```bash
ansible localhost -m setup -a 'filter=ansible_local'
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
[WARNING]: Platform linux on host 127.0.0.1 is using the discovered Python interpreter at /usr/bin/python, but future
installation of another Python interpreter could change this. See
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
127.0.0.1 | SUCCESS => {
    "ansible_facts": {
        "ansible_local": {},
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
```

![image-20230602121942262](C:\Users\itete\AppData\Roaming\Typora\typora-user-images\image-20230602121942262.png) 
