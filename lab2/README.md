# Лабораторная работа 2

## Ansible + Caddy

## Задачи

1. Переписать пример с созданием и удалением файла из шага 5 части 1 с ad-hoc команд на
   плейбук формат,
   а так же добавить четвертый шаг - перед удалением поменять содержимое файла на любое
   другое.
2.
    2.1 "Расширить" конфиг вебсервера Caddy любым функционалом по желанию:
       Например, добавить проксирование, или какие-нибудь заголовки (header). Вместо
       дефолтной
       страницы Caddy подставить свою, хотя бы index.html с Hello world внутри. Добавить
       это в
       качестве дополнительного шага в tasks
    2.2 сделать всю работу на удаленном сервере, а не на одном и том же localhost


## Ход работы

### Задача 1

Я отредактировал предложенный ansible playbook и добавил 3 шага: 
- создание файла
- редактирование файла
- удаление файла

Я применил 2 разных подхода к манипулированию файлами в ansible: 
- ansible.builtin.shell
- ansible.builtin.file

Вот текст получившихся тасков

```yaml
- name: Add ad-hoc files for task 1
  ansible.builtin.shell:  # используем простые sh команды
    cmd: "echo test_file_content > {{ ansible_env.HOME }}/test.txt"
  args:
    creates: "{{ ansible_env.HOME }}/test.txt"
  when: ansible_env.HOME is defined  # если существует переменная HOME

- name: change file contents before deleting it
  ansible.builtin.shell:
    cmd: "echo altered_text_for_file > {{ ansible_env.HOME }}/test.txt"
    # Опять простые sh команды, чтобы отработать
  args:
    creates: "{{ ansible_env.HOME }}/test.txt"
    # creates тут подходит лучше всего, так как файл все еще существует после редактирования
  when: ansible_env.HOME is defined

- name: Delete the test file if it exists
  # Попробовал другой подход с манипуляций файлами при помощи ansible.builtin.file
  ansible.builtin.file:
    path: "{{ ansible_env.HOME }}/test.txt"
    state: absent
  when: ansible_env.HOME is defined
```

ansible.builtin.shell позволяет очень легко выполнить определенные задачи, но в этом 
случае сложнее отслеживать состояние системы, а следовательно, можно нарушить принцип 
идемпотентности.  

ansible.builtin.file гораздо лучше подходит для этой задачи, так как он придерживается 
декларативной природы ansible. 

### Задача 2

Ссылка на получившйися сайт: [https://oopyat.duckdns.org/](https://oopyat.duckdns.org/)


Для начала мы создали файл index.html внутри папки caddy_deploy/files/www/. Далее мы указали на этот файл как то, что caddy
должен отдавать на 80 порту в Caddyfile.j2

Далее мы обновили tasks/main.yml. Мы создали папку /var/www/html и скопировали туда файл index.html. После перезагрузки плейбука 
все заработало и стандартная страница caddy сменилась на нашу кастомную страницу. 


```
- name: Ensure /var/www/html directory exists
  file:
    path: /var/www/html
    state: directory
    owner: root
    group: root
    mode: '0755'

- name: Copy index.html to the web directory
  copy:
    src: www/index.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: '0644'
```


### Результаты

Все файлы можно посмотреть в этом репозитории в папке lab-2

Далее идет вывод терминала после запуска ansible плейбука
```bash
root@oo5:~/lab-2# ansible-playbook -i inventory/hosts caddy_deploy.yml

PLAY [Install and configure Caddy webserver] **********************************************************************************

TASK [Gathering Facts] ********************************************************************************************************
[WARNING]: Platform linux on host local_server is using the discovered Python interpreter at /usr/bin/python3.10, but future
installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [local_server]

TASK [caddy_deploy : Install prerequisites] ***********************************************************************************
ok: [local_server]

TASK [caddy_deploy : Add ad-hoc files for task 1] *****************************************************************************
changed: [local_server]

TASK [caddy_deploy : change file contents before deleting it] *****************************************************************
ok: [local_server]

TASK [caddy_deploy : Delete the test file if it exists] ***********************************************************************
changed: [local_server]

TASK [caddy_deploy : Add key for Caddy repo] **********************************************************************************
ok: [local_server]

TASK [caddy_deploy : add Caddy repo] ******************************************************************************************
ok: [local_server]

TASK [caddy_deploy : add Caddy src repo] **************************************************************************************
ok: [local_server]

TASK [caddy_deploy : Install Caddy webserver] *********************************************************************************
ok: [local_server]

TASK [caddy_deploy : Create config file] **************************************************************************************
ok: [local_server]

TASK [caddy_deploy : Ensure /var/www/html directory exists] *******************************************************************
ok: [local_server]

TASK [caddy_deploy : Copy index.html to the web directory] ********************************************************************
ok: [local_server]

TASK [caddy_deploy : Reload with new config] **********************************************************************************
changed: [local_server]

PLAY RECAP ********************************************************************************************************************
local_server               : ok=13   changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Получившийся tasks/main.yml

```yml
# tasks file for caddy_deploy

- name: Install prerequisites
  apt:
    name:
    - debian-keyring
    - debian-archive-keyring
    - apt-transport-https
    - curl
    - python3
    - python3-venv
    - python3-pip
    - libpq-dev
    state: present


- name: Add ad-hoc files for task 1
  ansible.builtin.shell:
    cmd: "echo test_file_content > {{ ansible_env.HOME }}/test.txt"
  args:
    creates: "{{ ansible_env.HOME }}/test.txt"
  when: ansible_env.HOME is defined

- name: change file contents before deleting it

  ansible.builtin.shell:
    cmd: "echo altered_text_for_file > {{ ansible_env.HOME }}/test.txt"
  args:
    creates: "{{ ansible_env.HOME }}/test.txt"
  when: ansible_env.HOME is defined

- name: Delete the test file if it exists
  ansible.builtin.file:
    path: "{{ ansible_env.HOME }}/test.txt"
    state: absent
  when: ansible_env.HOME is defined

- name: Add key for Caddy repo
  apt_key:
    url: https://dl.cloudsmith.io/public/caddy/stable/gpg.key
    state: present
    keyring: /usr/share/keyrings/caddy-stable-archive-keyring.gpg

- name: add Caddy repo
  apt_repository:
    repo: "deb [signed-by=/usr/share/keyrings/caddy-stable-archive-keyring.gpg] https://dl.cloudsmith.io/public/caddy/stable/deb/debian any-version main"
    state: present
    filename: caddy-stable

- name: add Caddy src repo
  apt_repository:
    repo: "deb-src [signed-by=/usr/share/keyrings/caddy-stable-archive-keyring.gpg] https://dl.cloudsmith.io/public/caddy/stable/deb/debian any-version main"
    state: present
    filename: caddy-stable

- name: Install Caddy webserver
  apt:
    name: caddy
    update_cache: yes
    state: present

- name: Create config file
  template:
    src: templates/Caddyfile.j2  # Откуда берем
    dest: /etc/caddy/Caddyfile  # Куда кладем


# задание 2

- name: Ensure /var/www/html directory exists
  file:
    path: /var/www/html
    state: directory
    owner: root
    group: root
    mode: '0755'

- name: Copy index.html to the web directory
  copy:
    src: www/index.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: '0644'

- name: Reload with new config
  service:
    name: caddy
    state: reloaded
```

## Выводы

В данной лабораторной работе мы изучили принцип работы Ansible и Caddy. А именно изучили:
- декларативный подход в Ansible;
- использование модулей для автоматизации задач;
- безопасное подключение HTTPS с помощью Caddy.

Мы узнали, как можно повысить эффективность, а такде автоматизировать управление конфигурацией в проектах.
