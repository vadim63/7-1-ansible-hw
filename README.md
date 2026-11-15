# Домашнее задание к занятию "Ansible. Часть 2" - Качьянов Вадим

### Задание 1

Было написано 3 плейбука, результаты выполнения на рисунках:



**Плейбук 1** 
```yaml
---
- name: Download and unpacking Kafka
  hosts: web

  tasks:
    - name: Download Kafka
      ansible.builtin.get_url:
        url: https://downloads.apache.org/kafka/4.1.2/kafka-4.1.2-src.tgz
        dest: /tmp/kafka-4.1.2-src.tgz
        mode: "0644"

    - name: Create dir
      ansible.builtin.file:
        path: /home/vadim/kafka
        state: directory
        mode: "0755"

    - name: Unpacking archive
      ansible.builtin.unarchive:
        src: /tmp/kafka-4.1.2-src.tgz
        dest: /home/vadim/kafka
        remote_src: true
        creates: /home/vadim/kafka/kafka-4.1.2-src
```

**Плейбук 2** 
```yaml
---
- name: Download and start daemon "tuned"
  hosts: web
  become: true

  tasks:
    - name: Install "tuned"
      ansible.builtin.apt:
        name: tuned
        state: present
        update_cache: true

    - name: Start daemon
      ansible.builtin.systemd:
        name: tuned
        state: started
        enabled: true

```

**Плейбук 3** 
```yaml
---
- name: Change message of the day
  hosts: web
  become: true

  vars:
    motd_txt: "Welcome!"

  tasks:
    - name: Change config
      ansible.builtin.copy:
        dest: /etc/motd
        content: "{{ motd_txt }}\n"

```

Скриншоты:
1. ![Playbook1](img/play1.png)
2. ![Playbook2](img/play2.png)
3. ![Playbook3](img/play3.png)



### Задание 2

В данном задании был модифицрован 3 плейбук


```yaml
---
- name: Change message of the day
  hosts: web
  become: true

  vars:
    motd_txt: |
      Hostname: {{ ansible_facts['hostname'] }},
      IP address: {{ ansible_facts['default_ipv4']['address'] }}.
      Have a nice day, sysadmin!

  tasks:
    - name: Change config
      ansible.builtin.copy:
        dest: /etc/motd
        content: "{{ motd_txt }}\n"

```

Скриншот:
1. ![Playbook3.1](img/play3.1.png)


### Задание 3

В данном задании была написана роль

```yaml
---
# tasks file for roles/apache
- name: Download Apache
  ansible.builtin.apt:
    name: apache2
    state: present
    update_cache: true
  
- name: Configure index.html
  ansible.builtin.template:
    src: index.html.j2
    dest: /var/www/html/index.html
    mode: "0644"
  notify: Reload Apache

- name: Start and enable Apache
  ansible.builtin.service:
    name: apache2
    state: started
    enabled: true

- name: Open port 80
  community.general.ufw:
    rule: allow
    port: "{{ apache_port }}"
    proto: tcp
  
- name: Get request
  ansible.builtin.uri:
    url: "http://{{ ansible_host }}"
    status_code: 200
  delegate_to: localhost
  become: false
```


![playbook role](img/role.png)`
