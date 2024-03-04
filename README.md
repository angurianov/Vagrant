Создание виртуальной машины с помощью Vagrant, в которой через Ansible provision устанавливается Docker, node-exporter, запускается docker-compose с Prometheus и Grafana.
После выполнения команды vagrant up в браузере хостовой машины по адресу http://localhost:3000 открывается Grafana с Node Exporter Full. Имя и пароль для входа в веб-интерфейс графаны: admin/admin.

Для запуска использовать команду:
```
git clone git@github.com:angurianov/Vagrant.git && \
cd Vagrant && \
vagrant up
```

### Vagrant
Так как в условиях задачи нет требований к используемому провайдеру vagrant и выделяемым системным ресурсам (RAM, CPU), то этот блок конфигурации отсутствует в решении.
Так как требуется входить на страницу с графаной через адрес localhost, то между хостовой ОС и виртуальной машиной vagrant требуется только проброс портов (NAT, настройка по-умолчанию). Для доступа к 3000-порту графаны выполняем проброс этого порта.
Для копирования всех файлов проекта внутрь виртуальной машины используется общая папка (config.vm.synced_folder) и копирование через provision file.

### Ansible
Для использования Ansible provision требуется установленный на хостовую машину Ansible. Так как в условиях задачи не указано, есть ли ansible на хостовой ОС, то лучше использовать Ansible Local Provisioner, устанавливаемый внутрь виртуальной машины.
Вместо использования inventory принято решение использовать настройку hosts: внутри плейбука, так как настраиваться будет только локальный хост.
Инструкции ансибл написаны в виде четырёх плейбуков: основного playbook.yml, и еще трёх (для установки докера, для установки node-exporter, для запуска docker-compose), указанных в основном через import_playbook.

### Docker
Докер можно установить несколькими способами, в том числе с помощью пустого docker provision из vagrant. Но в этом случае будут установлены не все зависимости, и часть модулей докера не будет корректно работать. Поэтому принято решение установить докер способом, описанным в официальной документации, с использованием ansible playbook.
В docker-compose используются образы тех паблишеров с докерхаба, которые указаны в официальной документации к графане (grafana/grafana-oss) и прометеусу (prom/prometheus). В качестве версии (тэга) не рекомендуется использовать latest, во избежания проблем и различий в версиях при запуске контейнеров (пуле образов) в разные даты. В условии задачи нет условий по используемым версиям, поэтому используются последние доступные. Аналогично с node-exporter.

Для гибкого управления файлами между виртуальной машиной и контейнерами, запущенной на ней, используются volumes. Через них передаются конфигурационные файлы для прометеуса и графаны. Используется созданная ансиблом сеть для докера, необходимая для того, чтобы прометеус мог соединиться с Node-exporter, расположенном на виртуальной машине, а не в контейнере. Для этого используется j2 template. Подробнее ниже в блоке о прометеусе.

### Node-exporter
Для того, чтобы node exporter мог собирать все метрики хоста (например, метрики сетевых соединений), его следует устанавливать на хостовую машину, а не контейнером через docker-compose.
Для установки node exporter нужно использовать релиз для корректной архитектуры процессора (amd64 / arm64 / 386). Для этого в плейбуке выводится архитектура машины и записывается в качестве переменной, используемой в дальнейшем для скачивания нужного релиза, распаковки архива с нужным именем, копирования файла из нужной директории.
Чтобы node-exporter запустился в фоне, лучше всего запустить его как systemd unit file. Для этого предварительно подготовлен данный файл, который с помощью ansible playbook копируется в /etc/systemd/systemd. Бинарный файл node_exporter из распакованного архива копируется в /usr/local/bin.

### Prometheus
Для того, чтобы контейнерный prometheus собирал данные с хостового node-exporter, нужно прописать это в targets в /etc/prometheus/prometheus.yml, указав адрес шлюза для докер сети, в которой находится контейнер, и порт 9100. Докер по-умолчанию использует сеть 172.17.0.0/16 (с интерфейсом docker0). При создании новых контейнеров через docker-compose, если не указано иное, создатся новая сеть 172.18.0.0/16. Но для избежания возможных проблем, если эта сеть окажется занята, принято решение создать докер-сеть вручную (через плейбук), узнать ip-адрес бриджевого интерфейса в этой сети с помощью ansible gather facts, и затем с помощью Jinja template подставить ip-адрес этого интерфейса в prometheus.yml для коннекта с node-exporter. Полученный prometheus.yml с помощью общей папки (volume) копируется внутрь контейнера, и прометеус сможет пулить данные метрик с виртуальной машины, а не только изнутри сети контейнера.

### Grafana
Так как графана находится в одной докер-сети с прометеусом, то для коннекта к нему с помощью datasources, достаточно указать его контейнерное имя и порт. Для создания дашбордов с помощью файлов конфигурации используется графана провижн (/etc/grafana/provisioning). Template дашборда node-exporter-full скачивается с сайта графаны в виде yaml-файла, и остается только поместить его в папку с провиженом.








