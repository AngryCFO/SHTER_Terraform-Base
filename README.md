# Домашнее задание к занятию «Основы Terraform. Yandex Cloud»

### Цели задания

1. Создать свои ресурсы в облаке Yandex Cloud с помощью Terraform.
2. Освоить работу с переменными Terraform.


### Чек-лист готовности к домашнему заданию

1. Зарегистрирован аккаунт в Yandex Cloud. Использован промокод на грант.
2. Установлен инструмент Yandex CLI.
3. Исходный код для выполнения задания расположен в директории [**02/src**](https://github.com/netology-code/ter-homeworks/tree/main/02/src).


### Задание 0

1. Ознакомьтесь с [документацией к security-groups в Yandex Cloud](https://cloud.yandex.ru/docs/vpc/concepts/security-groups?from=int-console-help-center-or-nav). 
Этот функционал понадобится к следующей лекции.

------
### Внимание!! Обязательно предоставляем на проверку получившийся код в виде ссылки на ваш github-репозиторий!
------

### Задание 1
В качестве ответа всегда полностью прикладывайте ваш terraform-код в git.
Убедитесь что ваша версия **Terraform** ~>1.8.4

1. Изучите проект. В файле variables.tf объявлены переменные для Yandex provider.
2. Создайте сервисный аккаунт и ключ. [service_account_key_file](https://terraform-provider.yandexcloud.net).
4. Сгенерируйте новый или используйте свой текущий ssh-ключ. Запишите его открытую(public) часть в переменную **vms_ssh_public_root_key**.
5. Инициализируйте проект, выполните код. Исправьте намеренно допущенные синтаксические ошибки. Ищите внимательно, посимвольно. Ответьте, в чём заключается их суть.
6. Подключитесь к консоли ВМ через ssh и выполните команду ``` curl ifconfig.me```.
Примечание: К OS ubuntu "out of a box, те из коробки" необходимо подключаться под пользователем ubuntu: ```"ssh ubuntu@vm_ip_address"```. Предварительно убедитесь, что ваш ключ добавлен в ssh-агент: ```eval $(ssh-agent) && ssh-add``` Вы познакомитесь с тем как при создании ВМ создать своего пользователя в блоке metadata в следующей лекции.;
8. Ответьте, как в процессе обучения могут пригодиться параметры ```preemptible = true``` и ```core_fraction=5``` в параметрах ВМ.

В качестве решения приложите:

- скриншот ЛК Yandex Cloud с созданной ВМ, где видно внешний ip-адрес;
- скриншот консоли, curl должен отобразить тот же внешний ip-адрес;
- ответы на вопросы.

## Ответ на задание 1

1.  Был добавлен файл personal.auto.tfvars для передачи открытой части ключа авторизации, Cloud_id и Folder_id yandex cloud
2.  Били исправлены несколько ошибок в коде:
    - Был добавлен personal.auto.tfvars для передачи cloud и folder id
    - Исправлены повторяющиеся наименования блоков 
      ```
      resource "yandex_vpc_network" "develop"
      resource "yandex_vpc_subnet" "develop_sub" #(Было develop)
      ```
    - Исправлена строка platform_id, значения standart-v4 не существует, изменено на standard-v1.
    - Bзменено значение core на 2, так как это минимальное значение.
3.  ![alt text](https://github.com/VN351/ter_hw_02/raw/main/images/task-1-1.png)
4.  ![alt text](https://github.com/VN351/ter_hw_02/raw/main/images/task-1-2.png)
5.  Параметры ```preemptible = true``` и ```core_fraction=5``` в параметрах ВМ могут пригодиться, для экономии денег при создании VM.

### Задание 2

1. Замените все хардкод-**значения** для ресурсов **yandex_compute_image** и **yandex_compute_instance** на **отдельные** переменные. К названиям переменных ВМ добавьте в начало префикс **vm_web_** .  Пример: **vm_web_name**.
2. Объявите нужные переменные в файле variables.tf, обязательно указывайте тип переменной. Заполните их **default** прежними значениями из main.tf. 
3. Проверьте terraform plan. Изменений быть не должно. 

## Ответ на задание 2

1.  main.tf (измененная часть)
    ```
    data "yandex_compute_image" "ubuntu" {
      family = "${var.vm_web_os_family}"
    }
    resource "yandex_compute_instance" "platform" {
      name        = "${var.vm_web_name}"
      platform_id = "${var.vm_web_platform_id}"
      resources {
        cores         = var.vm_web_resources.cores
        memory        = var.vm_web_resources.memory
        core_fraction = var.vm_web_resources.core_fraction
      }
      boot_disk {
        initialize_params {
          image_id = data.yandex_compute_image.ubuntu.image_id
        }
      }
      scheduling_policy {
        preemptible = true
      }
      network_interface {
        subnet_id = yandex_vpc_subnet.develop_sub.id
        nat       = true
      }

      metadata = {
        serial-port-enable = 1
        ssh-keys           = "ubuntu:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC8bDGbiyUNa2k07/T9/4wqljOvFJOD nevzorovvlad@mail.ru"
      }

    }
    ```
2.  variables.tf (измененная часть)
    ```
    variable "vm_web_os_family" {
      type        = string
      default     = "ubuntu-2004-lts"
      description = "OS family"  
    }

    variable "vm_web_name" {
      type        = string
      default     = "netology-develop-platform-web"
      description = "Name" 
    }

    variable "vm_web_platform_id" {
      type        = string
      default     = "standard-v1" 
      description = "Platform ID"  
    }

    variable "vm_web_resources" {
      type        = map(number)
      default     = { cores = 2, memory = 1, core_fraction = 5 }
      description = "VM Resources"
    }
    ```
3. ![alt text](https://github.com/VN351/ter_hw_02/raw/main/images/task-2-1.png)

### Задание 3

1. Создайте в корне проекта файл 'vms_platform.tf' . Перенесите в него все переменные первой ВМ.
2. Скопируйте блок ресурса и создайте с его помощью вторую ВМ в файле main.tf: **"netology-develop-platform-db"** ,  ```cores  = 2, memory = 2, core_fraction = 20```. Объявите её переменные с префиксом **vm_db_** в том же файле ('vms_platform.tf').  ВМ должна работать в зоне "ru-central1-b"
3. Примените изменения.

## Ответ на задание 3
1.  vms_platform.tf
    ```
    variable "sub_name_db" {
      type        = string
      default     = "develop_sub_db"
      description = "subnet name"
    }

    variable "default_zone_db" {
      type        = string
      default     = "ru-central1-b"
      description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
    }
    variable "default_cidr_db" {
      type        = list(string)
      default     = ["10.1.1.0/24"]
      description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
    }

    variable "vm_db_name" {
      type        = string
      default     = "netology-develop-platform-db"
      description = "Name" 
    }

    variable "vm_db_platform_id" {
      type        = string
      default     = "standard-v2" 
      description = "Platform ID"  
    }

    variable "vm_db_resources" {
      type        = map(number)
      default     = { cores = 2, memory = 2, core_fraction = 20 }
      description = "VM Resources"
    }
    ```
2.  main.tf
    ```
    resource "yandex_vpc_network" "develop" {
      name = var.vpc_name
    }
    resource "yandex_vpc_subnet" "develop_sub"  { 
      name           = var.sub_name
      zone           = var.default_zone
      network_id     = yandex_vpc_network.develop.id
      v4_cidr_blocks = var.default_cidr
    }

    resource "yandex_vpc_subnet" "develop_sub_db"  { 
      name           = var.sub_name_db
      zone           = var.default_zone_db
      network_id     = yandex_vpc_network.develop.id
      v4_cidr_blocks = var.default_cidr_db
    }

    data "yandex_compute_image" "ubuntu" {
      family = "${var.vm_os_family}"
    }
    resource "yandex_compute_instance" "platform" {
      name        = "${var.vm_web_name}"
      platform_id = "${var.vm_web_platform_id}"
      zone        = var.default_zone
      resources {
        cores         = var.vm_web_resources.cores
        memory        = var.vm_web_resources.memory
        core_fraction = var.vm_web_resources.core_fraction
      }
      boot_disk {
        initialize_params {
          image_id = data.yandex_compute_image.ubuntu.image_id
        }
      }
      scheduling_policy {
        preemptible = true
      }
      network_interface {
        subnet_id = yandex_vpc_subnet.develop_sub.id
        nat       = true
      }

      metadata = {
        serial-port-enable = 1
        ssh-keys           = "ubuntu:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC8bDGbiyUNa2k07/T9/4wqljOvFJOD nevzorovvlad@mail.ru"
      }

      allow_stopping_for_update = true
    }

    resource "yandex_compute_instance" "platform_db" {
      name        = "${var.vm_db_name}"
      platform_id = "${var.vm_db_platform_id}"
      zone        = var.default_zone_db
      resources {
        cores         = var.vm_db_resources.cores
        memory        = var.vm_db_resources.memory
        core_fraction = var.vm_db_resources.core_fraction
      }
      boot_disk {
        initialize_params {
          image_id = data.yandex_compute_image.ubuntu.image_id
        }
      }
      scheduling_policy {
        preemptible = true
      }
      network_interface {
        subnet_id = yandex_vpc_subnet.develop_sub_db.id
        nat       = true
      }

      metadata = {
        serial-port-enable = 1
        ssh-keys           = "ubuntu:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC8bDGbiyUNa2k07/T9/4wqljOvFJOD nevzorovvlad@mail.ru"
      }

      allow_stopping_for_update = true
    }
    ```
3.  ![alt text](https://github.com/VN351/ter_hw_02/raw/main/images/task-3-1.png)  


### Задание 4

1. Объявите в файле outputs.tf **один** output , содержащий: instance_name, external_ip, fqdn для каждой из ВМ в удобном лично для вас формате.(без хардкода!!!)
2. Примените изменения.

В качестве решения приложите вывод значений ip-адресов команды ```terraform output```.

## Ответ на задание 4

1.  outputs.tf
    ```
    output "vm_details" {
      description = "Детали VM: имя, внешний IP, FQDN"
      value = [
        {
          VM_web_instance_name = yandex_compute_instance.platform.name
          VM_web_external_ip   = yandex_compute_instance.platform.network_interface[0].nat_ip_address
          VM_web_fqdn          = yandex_compute_instance.platform.fqdn
        },
        {
          VM_db = yandex_compute_instance.platform_db.name
          VM_db_external_ip   = yandex_compute_instance.platform_db.network_interface[0].nat_ip_address
          VM_db_fqdn          = yandex_compute_instance.platform_db.fqdn
        }
      ]
    }
    ```
2.  ![alt text](https://github.com/VN351/ter_hw_02/raw/main/images/task-4-1.png) 

### Задание 5

1. В файле locals.tf опишите в **одном** local-блоке имя каждой ВМ, используйте интерполяцию ${..} с НЕСКОЛЬКИМИ переменными по примеру из лекции.
2. Замените переменные внутри ресурса ВМ на созданные вами local-переменные.
3. Примените изменения.

## Ответ на задание 5
1.  locals.tf
    ```
    locals {
      web = "${var.vm_web_name}-${var.default_zone}-${var.vm_os_family}"
      db  = "${var.vm_db_name}-${var.default_zone_db}-${var.vm_os_family}"
    }
    ```
2.  ![alt text](https://github.com/VN351/ter_hw_02/raw/main/images/task-5-1.png) 
### Задание 6

1. Вместо использования трёх переменных  ".._cores",".._memory",".._core_fraction" в блоке  resources {...}, объедините их в единую map-переменную **vms_resources** и  внутри неё конфиги обеих ВМ в виде вложенного map(object).  
   ```
   пример из terraform.tfvars:
   vms_resources = {
     web={
       cores=2
       memory=2
       core_fraction=5
       hdd_size=10
       hdd_type="network-hdd"
       ...
     },
     db= {
       cores=2
       memory=4
       core_fraction=20
       hdd_size=10
       hdd_type="network-ssd"
       ...
     }
   }
   ```
3. Создайте и используйте отдельную map(object) переменную для блока metadata, она должна быть общая для всех ваших ВМ.
   ```
   пример из terraform.tfvars:
   metadata = {
     serial-port-enable = 1
     ssh-keys           = "ubuntu:ssh-ed25519 AAAAC..."
   }
   ```  
  
5. Найдите и закоментируйте все, более не используемые переменные проекта.
6. Проверьте terraform plan. Изменений быть не должно.

## Ответ на задание 6
1.  main.tf (Измененная часть)
    ```
    resource "yandex_compute_instance" "platform" {
      name        = local.web
      platform_id = var.vm_web_platform_id
      zone        = var.default_zone
      resources {
        cores         = var.vms_resurces.vm_web_resources.core
        memory        = var.vms_resurces.vm_web_resources.memory
        core_fraction = var.vms_resurces.vm_web_resources.core_fraction
      }
      boot_disk {
        initialize_params {
          image_id = data.yandex_compute_image.ubuntu.image_id
        }
      }
      scheduling_policy {
        preemptible = true
      }
      network_interface {
        subnet_id = yandex_vpc_subnet.develop_sub.id
        nat       = true
      }

      metadata = var.ssh_key
    /*
      metadata = {
        serial-port-enable = 1
          ssh-keys          = "ubuntu:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC8bDGbiyUNa2k07/T9/4wqljOvFJOD nevzorovvlad@mail.ru"
      }  
    */
      allow_stopping_for_update = true
    }

    resource "yandex_compute_instance" "platform_db" {
      name        = local.db
      platform_id = var.vm_db_platform_id
      zone        = var.default_zone_db
      resources {
        cores         = var.vms_resurces.vm_db_resources.core
        memory        = var.vms_resurces.vm_db_resources.memory
        core_fraction = var.vms_resurces.vm_db_resources.core_fraction
      }
      boot_disk {
        initialize_params {
          image_id = data.yandex_compute_image.ubuntu.image_id
        }
      }
      scheduling_policy {
        preemptible = true
      }
      network_interface {
        subnet_id = yandex_vpc_subnet.develop_sub_db.id
        nat       = true
      }

      metadata = var.ssh_key
      /*
      metadata = {
        serial-port-enable = 1
        ssh-keys           = "ubuntu:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC8bDGbiyUNa2k07/T9/4wqljOvFJOD nevzorovvlad@mail.ru"
      }
    */
      allow_stopping_for_update = true
    }
    ```
    variables.tf (Измененная часть)
    ```
    /*
    variable "vm_web_resources" {
      type        = map(number)
      default     = { cores = 2, memory = 1, core_fraction = 5 }
      description = "VM Resources"
    } */

    variable "vms_resurces" {
      type        = map(map(number))
      default = {
        vm_web_resources = {
          core = 2
          memory = 1
          core_fraction = 5
        }
        vm_db_resources = {
          core = 2
          memory = 2
          core_fraction = 20  
        }
      }
    }
    ###ssh vars

    variable "ssh_key" {
      type = map(string)
      default = {
        "serial-port-enable" = "1"
        "ssh-keys"           = "ubuntu:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC8bDGbiyUNa2k07/T9jlaRKD1gMcMT9/4wqljOvFJOD nevzorovvlad@mail.ru"
      }

    }
    ```
2.  ![alt text](https://github.com/VN351/ter_hw_02/raw/main/images/task-6-1.png) 


------

## Дополнительное задание (со звёздочкой*)

**Настоятельно рекомендуем выполнять все задания со звёздочкой.**   
Они помогут глубже разобраться в материале. Задания со звёздочкой дополнительные, не обязательные к выполнению и никак не повлияют на получение вами зачёта по этому домашнему заданию. 


------
### Задание 7*

Изучите содержимое файла console.tf. Откройте terraform console, выполните следующие задания: 

1. Напишите, какой командой можно отобразить **второй** элемент списка test_list.
2. Найдите длину списка test_list с помощью функции length(<имя переменной>).
3. Напишите, какой командой можно отобразить значение ключа admin из map test_map.
4. Напишите interpolation-выражение, результатом которого будет: "John is admin for production server based on OS ubuntu-20-04 with X vcpu, Y ram and Z virtual disks", используйте данные из переменных test_list, test_map, servers и функцию length() для подстановки значений.

**Примечание**: если не догадаетесь как вычленить слово "admin", погуглите: "terraform get keys of map"

В качестве решения предоставьте необходимые команды и их вывод.

## Ответ на задание 7

![alt text](https://github.com/VN351/ter_hw_02/raw/main/images/task-7-1.png)
------

### Задание 8*
1. Напишите и проверьте переменную test и полное описание ее type в соответствии со значением из terraform.tfvars:
```
test = [
  {
    "dev1" = [
      "ssh -o 'StrictHostKeyChecking=no' ubuntu@62.84.124.117",
      "10.0.1.7",
    ]
  },
  {
    "dev2" = [
      "ssh -o 'StrictHostKeyChecking=no' ubuntu@84.252.140.88",
      "10.0.2.29",
    ]
  },
  {
    "prod1" = [
      "ssh -o 'StrictHostKeyChecking=no' ubuntu@51.250.2.101",
      "10.0.1.30",
    ]
  },
]
```
2. Напишите выражение в terraform console, которое позволит вычленить строку "ssh -o 'StrictHostKeyChecking=no' ubuntu@62.84.124.117" из этой переменной.
------
## Ответ на задание 8

1.  variables.tf (часть кода)
    ```
    variable "test" {
      type = map(list(string))
      default = {
        "dev1" = [
          "ssh -o 'StrictHostKeyChecking=no' ubuntu@62.84.124.117",
          "10.0.1.7",
        ],
        "dev2" = [
          "ssh -o 'StrictHostKeyChecking=no' ubuntu@84.252.140.88",
          "10.0.2.29",
        ],
        "prod1" = [
          "ssh -o 'StrictHostKeyChecking=no' ubuntu@51.250.2.101",
          "10.0.1.30",
        ],
      }
    }
    ```

2.  команда ля вывода строки "ssh -o 'StrictHostKeyChecking=no' ubuntu@62.84.124.117" -  "var.test["dev1"][0]"
![alt text](https://github.com/VN351/ter_hw_02/raw/main/images/task-8-1.png)

------

### Задание 9*

Используя инструкцию https://cloud.yandex.ru/ru/docs/vpc/operations/create-nat-gateway#tf_1, настройте для ваших ВМ nat_gateway. Для проверки уберите внешний IP адрес (nat=false) у ваших ВМ и проверьте доступ в интернет с ВМ, подключившись к ней через serial console. Для подключения предварительно через ssh измените пароль пользователя: ```sudo passwd ubuntu```

### Правила приёма работы. Для подключения предварительно через ssh измените пароль пользователя: sudo passwd ubuntu
В качестве результата прикрепите ссылку на MD файл с описанием выполненой работы в вашем репозитории. Так же в репозитории должен присутсвовать ваш финальный код проекта.

## Ответ на задание 9
1.  main.tf (Измененная часть)
    ```
    resource "yandex_vpc_network" "develop" {
      name = var.vpc_name
    }

    resource "yandex_vpc_gateway" "nat_gateway" {
      name        = "nat-gateway" 
      shared_egress_gateway {}
    }

    resource "yandex_vpc_route_table" "rt" {
      name       = "route-table"
      network_id = yandex_vpc_network.develop.id

      static_route {
        destination_prefix = "0.0.0.0/0"
        gateway_id     = yandex_vpc_gateway.nat_gateway.id
      }
    }

    resource "yandex_vpc_subnet" "develop_sub"  { 
      name           = var.sub_name
      zone           = var.default_zone
      network_id     = yandex_vpc_network.develop.id
      v4_cidr_blocks = var.default_cidr
      route_table_id = yandex_vpc_route_table.rt.id
    }

    resource "yandex_vpc_subnet" "develop_sub_db"  { 
      name           = var.sub_name_db
      zone           = var.default_zone_db
      network_id     = yandex_vpc_network.develop.id
      v4_cidr_blocks = var.default_cidr_db
      route_table_id = yandex_vpc_route_table.rt.id
    }
    ```
    ![alt text](https://github.com/VN351/ter_hw_02/raw/main/images/task-9-1.png)

**Важно. Удалите все созданные ресурсы**.

## Ссылка на репозиторий с выполненой работой

[Ссылка на конфигурацию Terraform](https://github.com/VN351/ter-hw-2-vms/tree/master)

### Критерии оценки

Зачёт ставится, если:

* выполнены все задания,
* ответы даны в развёрнутой форме,
* приложены соответствующие скриншоты и файлы проекта,
* в выполненных заданиях нет противоречий и нарушения логики.

На доработку работу отправят, если:

* задание выполнено частично или не выполнено вообще,
* в логике выполнения заданий есть противоречия и существенные недостатки. 

