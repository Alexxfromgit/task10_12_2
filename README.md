# task10_12_2

Общие требования к выполнению практических заданий:

    1. Выполненное задание должно располагаться в отдельном репозитарии на github.com, т.е. если учетная запись на github называется ‘user’ и выполняется практическое задание с названием ‘task10_12_2’, то файлы задания должны быть доступны в репозитарии https://github.com/user/task10_12_2. 
    2. Для проверки выполнения ДЗ будет использована свежеустановленная VM из этого образа - https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img
    3. Для выполнения ДЗ разрешается устанавливать любые дополнительные пакеты, помимо тех, что явно указаны в задании. В случае использования пакетов, которые не установлены в образе, необходимо предусмотреть их установку.
    4. Запуск скриптов во время проекта будет осуществляться от имени root
    5. Практические Задания по ЛК10-12 должны быть выполнены и загружены в соответствующие репозитарии до 23:59 13/05/18.


Задание  
NGINX reverse proxy & Apache, но в контейнерах
Используя наработки из задания task6_7 необходимо развернуть docker контейнер с NGINX, который будет выступать в качестве reverse proxy для веб-сервера Apache2, также развернутого в контейнере. NGINX должен осуществлять терминирование HTTPS соединения используя сертификат, сгенерированный автоматически в процессе установки.

<p align="center">
  <img src="https://image.ibb.co/f2hZeS/lc10_12_2_jpg.png">
  
  # Рисунок 1 - Топология задания
  </p>

Необходимо написать bash скрипт, который:

    • Автоматически устанавливает docker-ce и docker-compose
    • Автоматически генерирует root CA сертификат и сертификат для nginx, подписанный корневым CA
        ◦ SubjectAltNames должен содержать имя докер-хоста и его IP-адрес (не контейнера). Эти параметры доступны из конфигурационного файла.
    • Разворачивает два контейнера (apache2 и nginx) с использованием возможностей docker-compose. При этом:
        ◦ В качестве базовых образов используются nginx:1.13 и httpd:2.4
        ◦ Сертификаты для NGINX а также конфигурационный файл nginx.com должны быть примонтированы в контейнер через volume.
        ◦ Логи NGINX должны писаться в /srv/log/nginx директорию на хосте
        ◦ NGINX порт на хосте определяется переменной из конфигурационного файла.

**Часть 1 - Необходимые файлы**

    1. В репозитории должен находиться файл с именем task10_12_2.sh. Именно он  является входной точкой и будет запущен
    2. В репозитории должен находиться файл с именем config. В нем содержатся конфигурируемые параметры, которые должен корректно прочитать и применить скрипт. Перечень конфигурируемых параметров приведен ниже.
    3. В репозитории могут находиться любые дополнительные файлы и папки на ваше усмотрение (например шаблон конфигурационного файла NGINX)
    4. В конечном результате после запуска скрипта необходимо обеспечить приведенную ниже иерархию файлов (допускается как генерация файлов скриптом, так и редактирование уже существующих в репозитории файлов; допускается наличие любых дополнительных файлов и директорий, например web-chain.crt в директории certs):

    WORKDIR				        # script working directory		
    ├── certs				    # directory with certificates
    │   ├── root.crt		    # root CA certificate
    │   ├── web.crt			    # nginx certificate
    │   └── web.key			    # nginx private key
    ├── config				    # parameters file
    ├── docker	-compose.yml	# docker compose file
    ├── etc				        # directory with NGINX config
    │   └── nginx.conf		    # NGINX configuration file
    └── task10_12_2.sh		    # main script file

Пример файла **config**:

    # Host parameters
    EXTERNAL_IP=10.14.254.15
    HOST_NAME=docker-vm.domain.tld

    # Docker parameters
    NGINX_IMAGE="nginx:1.13"
    APACHE_IMAGE="httpd:2.4"
    NGINX_PORT=17080
    NGINX_LOG_DIR=/srv/log/nginx

**Часть 2 - Рекомендации**

Ниже приведены некоторые рекомендации, которые помогут вам корректно выполнить текущее задание:

    1. Для установки docker-ce и docker-compose используйте официальный репозиторий docker 
    2. Можно воспользоваться официальной справкой по образам docker:
        a. NGINX
        b. Apache
    3. Ознакомьтесь с возможностями и синтаксисом compose- файлов:
        a. Docker compose file v2
    4. Используйте discovery сервис докера для проксирования трафика из контейнера NGINX в контейнер с apache2
    5. Используйте наработки из д/з task6_7 для автоматической генерации сертификатов
    6. Монтировать тома в докер контейнер можно как из папки проекта (в этом случае вам необходимо корректно определить полный путь к папке проекта), так и из любого другого места (например скрипт создает дополнительные директории в /srv или /etc). Во втором случае убедитесь, что все необходимые файлы также доступны в директории проекта (будет проверяться их наличие).

**Часть 3 - Проверка**

В ходе проверки будет копироваться репозиторий task10_12_2 и запускаться скрипт  task10_12_2.sh.
Проверка будет заключаться в https запросе к докер-хосту по порту, указанному в конфигурационном файле и с указанием корневого сертификата (из директории certs проекта, файл root.crt). Ожидается, что https соединение будет успешно установлено и в ответе будет страница apache2 по умолчанию (“It works!”).
Будет проверено наличие двух контейнеров, nginx и apache, а также их базовые образы (nginx:1.13 и httpd:2.4 соответственно).
Также будет проверена конфигурация точек монтирования для контейнера nginx (конфигурационный файл, сертификаты, логи). Будет проверено наличие записей в /srv/logs/nginx/access.log на докер хосте.
После запуска скрипта будет проверено наличие обязательных файлов и директорий в директории проекта (docker-compose.yml, etc/nginx.conf, certs/)


