# Поднять приложение на Django, используя Nginx для обработки статических файлов и прокси используя роли, научиться шаблонизации конфигураций
[Кастомная Nginx роль](https://github.com/is-devops-4sem/ansible-nginx-role/tree/master)

## Ход работы

### 1. Поднять Vagrant-окружение при помощи файла Vagrantfile, используя команду `vagrant up`
![alt text](screenshots/1.png)

### 2. Инициализация структуры роли Nginx
Для создания структуры роли "Nginx" использовалась команда `ansible-galaxy init Nginx`. Эта команда автоматически создает шаблон файлов и каталогов для роли, что помогает упорядочить задачи и переменные.

![alt text](screenshots/2.png)

### 2. Написание роли для установки nginx
Задачи, связанные с установкой nginx, были вынесены в новую роль "Nginx". Теперь они находятся в файле `tasks/main.yml`.

`tasks/main.yml`:

![alt text](screenshots/3.png)

Для стандартной конфигурации nginx были указаны настройки в файле `nginx/defaults/main.yml`:

![alt text](screenshots/4.png)

- **`worker_connections`**  
  Указывает максимальное количество подключений, которое каждый рабочий процесс Nginx может одновременно обслуживать.  
  **По умолчанию:** `1024`.

- **`nginx_conf_template`**  
  Путь к Jinja2-шаблону, который используется для генерации основного файла конфигурации Nginx (`nginx.conf`).  
  **По умолчанию:** `"nginx.conf.j2"`.

- **`nginx_conf_dest`**  
  Путь, по которому будет размещён итоговый файл конфигурации Nginx после его генерации из шаблона. Обычно это основной файл конфигурации Nginx.  
  **По умолчанию:** `"/etc/nginx/nginx.conf"`.

- **`static_files_path`**  
  Указывает директорию, откуда будут обслуживаться статические файлы. Используется в секции `location /static/` для настройки алиаса.  
  **По умолчанию:** `"/var/www/static"`.

- **`proxy_pass_url`**  
  URL-адрес, на который будут проксироваться запросы, поступающие на корневую локацию (`/`). Используется в секции `location /`.  
  **По умолчанию:** `"http://127.0.0.1:8080"`.

- **`default_site_template`**  
  Путь к Jinja2-шаблону для конфигурации дефолтного сайта, который размещается в директории `sites-available`.  
  **По умолчанию:** `"default-site.conf.j2"`.

- **`nginx_site_template`**  
  Указывает шаблон конфигурации сайта Nginx, который будет использован для генерации файла конфигурации.  
  **По умолчанию:** `"nginx.conf.j2"`.

- **`nginx_site_available`**  
  Путь к файлу конфигурации сайта Nginx в папке `sites-available`, где хранятся конфигурации доступных сайтов.  
  **По умолчанию:** `"/etc/nginx/sites-available/default"`.

- **`nginx_site_enabled`**  
  Путь к символической ссылке на файл конфигурации сайта в папке `sites-enabled`, где хранятся активные сайты.  
  **По умолчанию:** `"/etc/nginx/sites-enabled/default"`.


Был подготовлен шаблон для конфигурационного файла для default сайта Nginx с настройками для отдачи статических файлов:

![alt text](screenshots/7.png)

А также был подготовлен шаблон для конфигурационного файла Nginx :

![alt text](screenshots/5.png)

### 3. Создание организации для репозиториев с ролями в Github и загрузка роли Nginx
Для хранения ролей Ansible была создана организация `is-devops-4sem` на Github. Роль "Nginx" была загружена в репозиторий `is-devops-4sem/ansible-nginx-role`.


### 4. Обновление файла requirements.yml
Для автоматической установки ролей Docker и Nginx из репозитория был обновлен файл requirements.yml, где указаны источники ролей:

![alt text](screenshots/6.png)

### 5. Установка ролей Docker и Nginx через requirements.yml
`ansible-galaxy install -r requirements.yml`
![alt text](screenshots/13.png)


### 6. Использование роли Docker и Nginx в плейбуке

Сначала в файл с хостами были добавлены хосты группы web:

![alt text](screenshots/9.png)

Также были указаны стандартные переменные для nginx и для docker:

![alt text](screenshots/10.png)

![alt text](screenshots/11.png)


Плейбук для запуска nginx на хостах группы web и django приложения на хостах группы app. 

![alt text](screenshots/12.png)

`ansible-playbook -i dev webservers.yml` - запуск плейбука.

![alt text](screenshots/14.png)

### 7. Проверка работы приложения

Проверка, что на хосте группы web запущенный nginx:

![alt text](screenshots/15.png)

После можно попробовать зайти в приложение, которое доступно по адресу `http://localhost:8000/` на хосте группы app.

![alt text](screenshots/16.png)


`lynx http://localhost:80/`
![alt text](screenshots/17.png)

Как можно видеть, что перейдя на `http://localhost:80/` мы попадаем на наше django приложение запущенное на хосте группы app.