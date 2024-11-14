# Установка Docker через роль Ansible
[Кастомная Docker роль](https://github.com/is-devops-4sem/ansible-docker-role/tree/master)

## Ход работы

### 1. Инициализация структуры роли Docker
Для создания структуры роли "Docker" использовалась команда `ansible-galaxy init Docker`. Эта команда автоматически создает шаблон файлов и каталогов для роли, что помогает упорядочить задачи и переменные.

### 2. Перенос задач установки Docker в отдельную роль
Задачи, связанные с установкой Docker, были вынесены из предыдущего плейбука и перенесены в новую роль "Docker". Теперь они находятся в файле `tasks/main.yml`.

`tasks/main.yml`:

```yaml
---
- name: Add Docker GPG key
  apt_key:
    url: "{{ docker_gpg_key_url }}"
    state: present

- name: Add Docker repository
  apt_repository:
    repo: "deb {{ docker_repository_url }} {{ docker_repo_distribution }} {{ docker_package_state }}"
    state: present

- name: Install Docker
  apt:
    name: "{{ docker_package }}"
    state: latest
    update_cache: true

- name: Add current user to the docker group
  user:
    name: "{{ docker_group_user }}"
    groups: docker
    append: yes
```

### 3. Параметризация роли через переменные
Для роли "Docker" были добавлены переменные, которые позволяют гибко настраивать установку Docker. Пример переменных:

`defaults/main.yml`:

```yaml
docker_gpg_key_url: "https://download.docker.com/linux/ubuntu/gpg"
docker_repository_url: "https://download.docker.com/linux/ubuntu"
docker_repo_distribution: "focal"
docker_package_state: "stable"
docker_package: "docker-ce"
docker_group_user: "{{ ansible_user }}"
```

### 4. Создание организации для репозиториев с ролями в Github и загрузка роли Docker
Для хранения ролей Ansible была создана организация `is-devops-4sem` на Github. Роль "Docker" была загружена в репозиторий `is-devops-4sem/ansible-docker-role`.

`ssh-keygen -t rsa -b 4096 -C "your-email@example.com"` - геренация ssh-ключа для доступа к Github, добавление его в настройки аккаунта.

```bash
git add .
git commit -m "message"
git remote set-url origin git@github.com:is-devops-4sem/ansible-docker-role.git
git push -u origin master
```

### 5. Создание файла requirements.yml
Для автоматической установки роли Docker из репозитория был создан файл requirements.yml, где указан источник роли:

```yaml
---
roles:
  - name: Docker
    src: git+https://github.com/is-devops-4sem/ansible-docker-role.git
    version: master
```

### 6. Установка роли Docker через requirements.yml
`ansible-galaxy install -r requirements.yml`

### 7. Использование роли Docker в плейбуке
```yaml
---
- name: Install Docker and run the application
  hosts: app
  become: true
  roles:
    - Docker
  tasks:
    - name: Install necessary system packages
      apt:
        pkg:
        - python3-pip
        state: latest
        update_cache: true
        
    - name: Install Docker SDK for Python
      pip:
        name: docker
        state: present
        executable: pip3

    - name: Pull the Django Docker image
      docker_image:
        name: "{{ docker_image }}"
        tag: latest
        source: pull

    - name: Run Docker container
      docker_container:
        name: "{{ container_name }}"
        image: "{{ docker_image }}"
        state: started
        restart_policy: unless-stopped
        published_ports:
          - "{{ host_port }}:{{ container_port }}"
```
`ansible-playbook -i dev webservers.yml` - запуск плейбука.

### 8. Проверка работы приложения
После выполнения плейбука приложение доступно по адресу `http://localhost:8000/`.


