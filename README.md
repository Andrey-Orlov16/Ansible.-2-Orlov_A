# Домашнее задание к занятию "`Название занятия`" - `Фамилия и имя студента`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. В личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1
Выполните действия, приложите файлы с плейбуками и вывод выполнения.

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.
Плейбуки должны:

Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать официальный сайт и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.
Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.
Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.

---
- name: Download and extract Apache Kafka archive
  hosts: all
  become: true
  vars:
    kafka_version: "3.8.1"
    kafka_archive_name: "kafka_2.13-{{ kafka_version }}.tgz"
    kafka_download_url: "https://archive.apache.org/dist/kafka/{{ kafka_version }}/{{ kafka_archive_name }}"
    kafka_base_dir: "/opt/kafka"
    kafka_extract_dir: "{{ kafka_base_dir }}/current"

  tasks:
    - name: Ensure base directory exists
      file:
        path: "{{ kafka_base_dir }}"
        state: directory
        mode: "0755"

    - name: Create extraction directory
      file:
        path: "{{ kafka_extract_dir }}"
        state: directory
        mode: "0755"

    - name: Download Kafka archive
      get_url:
        url: "{{ kafka_download_url }}"
        dest: "{{ kafka_base_dir }}/{{ kafka_archive_name }}"
        mode: "0644"
        checksum: sha256  # опционально: можно добавить checksum_url, если есть

    - name: Extract archive
      unarchive:
        src: "{{ kafka_base_dir }}/{{ kafka_archive_name }}"
        dest: "{{ kafka_extract_dir }}"
        remote_src: yes
        creates: "{{ kafka_extract_dir }}/bin/kafka-server-start.sh"
---
- name: Install and enable tuned service
  hosts: all
  become: true

  tasks:
    - name: Install tuned package
      package:
        name: tuned
        state: present

    - name: Start tuned service
      systemd:
        name: tuned
        state: started
        daemon_reload: yes

    - name: Enable tuned on boot
      systemd:
        name: tuned
        enabled: yes
        daemon_reload: yes
---
- name: Update MOTD with a custom message via variable
  hosts: all
  become: true
  vars:
    motd_message: "Добро пожаловать в систему! Это тестовое приветствие из Ansible."

  tasks:
    - name: Write MOTD file
      copy:
        content: "{{ motd_message }}\n"
        dest: /etc/motd
        owner: root
        group: root
        mode: "0644"


### Задание 2

Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору.

---
- name: Update MOTD with host IP, hostname and greeting
  hosts: all
  become: true

  tasks:
    - name: Generate MOTD content with host info and greeting
      set_fact:
        motd_message: |
          Добро пожаловать!
          Hostname: {{ ansible_hostname }}
          IP-адрес: {{ ansible_default_ipv4.address | default('N/A') }}

          Хорошего дня, системный администратор! 😊

    - name: Write MOTD file
      copy:
        content: "{{ motd_message }}"
        dest: /etc/motd
        owner: root
        group: root
        mode: "0644"



---

### Задание 3
Выполните действия, приложите архив с ролью и вывод выполнения.

Ознакомьтесь со статьёй «Ansible - это вам не bash», сделайте соответствующие выводы и не используйте модули shell или command при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

Установить веб-сервер Apache на управляемые хосты.
Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес. Используйте Ansible facts и jinja2-template. Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
Сделать проверку доступности веб-сайта (ответ 200, модуль uri).

---
- name: Deploy Apache with system info page
  hosts: all
  become: true
  roles:
    - apache_info
---
# Установка Apache (пакетное имя зависит от ОС; здесь универсальный подход)
- name: Install Apache web server
  package:
    name: "{{ apache_package_name }}"
    state: present
  register: pkg_result

- name: Ensure Apache is enabled and started
  systemd:
    name: httpd          # для RHEL/CentOS/Fedora
    # name: apache2       # для Debian/Ubuntu (см. vars ниже)
    enabled: yes
    state: started
  notify: Restart Apache if config changed
  when: pkg_result.changed or not pkg_result | skipped

# Открытие порта 80 (firewalld для RHEL-семейства)
- name: Open HTTP port in firewalld
  firewalld:
    port: 80/tcp
    permanent: true
    state: enabled
    immediate: true
  when: ansible_os_family == "RedHat"

# Открытие порта 80 (ufw для Debian/Ubuntu)
- name: Allow HTTP in UFW
  ufw:
    rule: allow
    port: "80"
    proto: tcp
  when: ansible_os_family == "Debian"

# Генерация index.html через шаблон
- name: Generate index.html with system facts
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: "0644"
  notify: Restart Apache if config changed

# Проверка доступности сайта (HTTP 200)
- name: Verify web server returns HTTP 200
  uri:
    url: "http://{{ ansible_default_ipv4.address }}/"
    method: GET
    timeout: 10
    status_code: 200
  register: site_check
  failed_when: site_check.status != 200


