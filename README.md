# Дипломная работа по профессии "Системный администратор" SYS-22 Концевой Н.Ю.
## "Создание инфраструктуры в облаке"


## Задача
Ключевая задача — разработать отказоустойчивую инфраструктуру для сайта, включающую мониторинг, сбор логов и резервное копирование основных данных. Инфраструктура должна размещаться в Yandex Cloud и отвечать минимальным стандартам безопасности.



### 1. Для выполнения задания был написан файл Terraform [main.tf](https://github.com/Stitchzxz/final_work/blob/main/main.tf) для сооздания следующих ресурсов:
 
### 1.1. Виртуальные машины 
  - nginx1 
  - nginx2 
  - zabix 
  - elasticsearc
  - kibana
  - bastion


<details>
<summary> </summary>

![список вм](https://github.com/Stitchzxz/final_work/blob/main/img/VM_list.png)

</details>

  

</details>

### 1.2. Балансировщик нагрузки для вебсерверов

![балансировщик](https://github.com/Stitchzxz/final_work/blob/main/img/alb.png)


### 1.3. Группы безопасности для ограничения доступности хостов по определенным портам
  - nginx-sg (nginx1-2 разрешен доступ чере збалансировщик на порты 22, 80, 10050)
  - zabbix-sg  (разрешен доступ на порты 22, 8080, 10051)
  - elastic-sg (разрешен доступ на порты 22, 10050, 9200)
  - kibana-sg (разрешен доступ на порты 22, 10050, 5601)
  - bastion-sg (разрешен доступ на 22 порт)

![группы безопасности](https://github.com/Stitchzxz/final_work/blob/main/img/security_groups.png)

### 1.4. Расписание создания снапшотов дисков со всех виртуальных машин
  - создано расписание на ежедневное создание снапшотов в 20:00 по московскому времени
  - хранение снапшотов 7 дней

![снимки дисков](https://github.com/Stitchzxz/final_work/blob/main/img/schedule.png)

### 1.5. Сеть и 4 подсети в разных зонах доступности для обеспечения отказоустойчивости.
    распределение ВМ в сети:
  - nginx1 (зона а, приватная сеть, имеет внутренний IP 192.168.1.11)
  - nginx2 (зона b, приватная сеть, имеет внутренний IP 192.168.2.22)
  - zabix (зона c, публичная сеть, имеет внутренний IP 192.168.3.33 и внешний IP <назначается автоматически> )
  - elasticsearc (зона d, приватная сеть, имеет внутренний IP 192.168.4.44)
  - kibana (зона c, публичная сеть, имеет внутренний IP 192.168.3.34 и внешний IP <назначается автоматически> )
  - bastion (зона c, публичная сеть, имеет внутренний IP 192.168.33.33 и внешний IP <назначается автоматически> )

![Карта сети](https://github.com/Stitchzxz/final_work/blob/main/img/network_map.png)

### 1.6. Шлюз (для доступа в интернет вм расположенных в приватных сетях)


<скриншоты>



## 2. Установка необходимых программ для подготовленной инфраструктуры осуществлялась с помощью плэйбуков через Ansible:

*Ansible настроен на Bastion host и вся установка происходит с него.

### 2.1. [nginx_pb.yml](https://github.com/Stitchzxz/final_work/blob/main/ansible/nginx_pb.yml) 
  - устанавливает nginx на две виртуальные машины состоящие в группе web_servers
  - копирует c локального хоста страницу для отображения при обращении на ip адрес балансировщика
* Проверка работоспособности [тут](http://158.160.139.201:80)

### 2.2. [zabbix_pb.yml](https://github.com/Stitchzxz/final_work/blob/main/ansible/zabbix_pb.yml)
  - добавляет репозиторий zabbix
  - устанавливает на хост zabbix -  zabbix server, zabbix agent, mysql, nginx и прочие зависимости
  - создает базу данных, пользователя, задает пароль

### 2.3. [zabbix_agent_pb.yml](https://github.com/Stitchzxz/final_work/blob/main/ansible/zabbix_agent_pb.yml)
  - добавляет репозиторий zabbix
  - устанавливает zabbix agent на все хосты
  - вносит корректировку в файл конфигурации

### 2.4. [elasticsearch_pb.yml](https://github.com/Stitchzxz/final_work/blob/main/ansible/elasticsearch_pb.yml)
  - скачивает и добавляет ключ gpg elasticsearch
  - добавляет репозиторий elasticsearch 8.x
  - устанавливает elasticsearch и зависимости
  - корректирует конфигурационный файл

### 2.5. [kibana_pb.yml](https://github.com/Stitchzxz/final_work/blob/main/ansible/kibana_pb.yml)
  - скачивает и добавляет ключ gpg elasticsearch
  - добавляет репозиторий elasticsearch 8.x
  - устанавливает kibana и зависимости
  - корректирует конфигурационный файл 
*Для подключения kibana к elasticsearh необходимо сгенерировать токек на хосте elastik. Также необходимо сгенерировать новый пароль и ввести его вместе с логином в вебинтерфейсе kibana для авторизации http://<ip_kibana>:5601 

### 2.6. [filebeat_pb.yml](https://github.com/Stitchzxz/final_work/blob/main/ansible/filebeat_pb.yml)
  - скачивает и добавляет ключ gpg elasticsearch
  - добавляет репозиторий elasticsearch 8.x
  - устанавливает filebeat и зависимости
  - копирует с локального хоста конфигурационный файл  
*Для подключения filebeat к elasticsearh необходимо добавить вышеуказанный пароль в конфигурационный файл и перезагрузить сервис.

### 2.7. [ping_pb.yml](https://github.com/Stitchzxz/final_work/blob/main/ansible/ping_pb.yml) (был добавлен для удобства)
  - проверяет доступность хостов



### Инфраструктура готова к эксплуатации.

