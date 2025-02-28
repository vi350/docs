# Создание сайта на WordPress

Создайте и настройте веб-сайт на базе CMS WordPress с помощью специального [образа](../../compute/concepts/image.md) [виртуальной машины](../../compute/concepts/vm.md).

Чтобы настроить веб-сайт на WordPress:
1. [Создайте группу безопасности](#create-security-group).
1. [Создайте ВМ для WordPress](#create-vm).
1. [Настройте DNS](#configure-dns).
1. [Настройте WordPress](#wordpress-setup).

Если сайт вам больше не нужен, [удалите ВМ с ним](#clear-out).

Также инфраструктуру для веб-сайта на базе CMS WordPress можно развернуть через {{ TF }} с помощью [готового файла конфигурации](#terraform).

## Подготовьте облако к работе {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

Убедитесь, что в выбранном [каталоге](../../resource-manager/concepts/resources-hierarchy.md#folder) есть [облачная сеть](../../vpc/concepts/network.md#network) с [подсетью](../../vpc/concepts/network.md#subnet) хотя бы в одной [зоне доступности](../../overview/concepts/geo-scope.md). Для этого на странице каталога выберите сервис **{{ vpc-short-name }}**. Если в списке есть сеть — нажмите на нее, чтобы увидеть список подсетей. Если нужных подсетей или сети нет, [создайте их](../../vpc/quickstart.md).


### Необходимые платные ресурсы {#paid-resources}

В стоимость поддержки веб-сайта на WordPress входит:
* Плата за постоянно запущенную ВМ (см. [тарифы {{ compute-full-name }}](../../compute/pricing.md)).
* Плата за использование динамического или статического [публичного IP-адреса](../../vpc/concepts/address.md#public-addresses) (см. [тарифы {{ vpc-full-name }}](../../vpc/pricing.md)).


## Создайте группу безопасности {#create-security-group}

Чтобы создать [группу безопасности](../../vpc/concepts/security-groups.md):

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите сервис **{{ vpc-name }}**.
  1. Откройте вкладку **Группы безопасности**.
  1. Нажмите кнопку **Создать группу**.
  1. Укажите **Имя** группы: `wordpress`.
  1. Выберите **Сеть**.
  1. В блоке **Правила** создайте следующие правила по инструкции под таблицей:

     Направление<br>трафика | Описание | Диапазон<br>портов | Протокол | Тип источника /<br>назначения | Источник /<br>назначение
     --- | --- | --- | --- | --- | ---
     Исходящий | any | Весь | Любой | CIDR | 0.0.0.0/0
     Входящий | ext-http | 80 | TCP | CIDR | 0.0.0.0/0
     Входящий | ext-https | 443 | TCP | CIDR | 0.0.0.0/0

     1. Выберите вкладку **Исходящий трафик** или **Входящий трафик**.
     1. Нажмите кнопку **Добавить правило**.
     1. В открывшемся окне в поле **Диапазон портов** укажите порт, куда будет поступать трафик. Для исходящего трафика не указывайте ничего.
     1. В поле **Протокол** укажите нужный протокол. Для исходящего трафика оставьте **Любой**, чтобы разрешить передачу трафика по всем протоколам.
     1. В поле **Назначение** или **Источник** выберите **CIDR** — правило будет применено к диапазону IP-адресов. В поле **CIDR блоки** укажите `0.0.0.0/0`.
     1. Нажмите кнопку **Сохранить**. Таким образом создайте все правила из таблицы.
  1. Нажмите кнопку **Сохранить**.

- {{ TF }} {#tf}

  См. раздел [Как создать инфраструктуру с помощью {{ TF }}](#terraform).

{% endlist %}

## Создайте ВМ для WordPress {#create-vm}

Чтобы создать ВМ:

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. На странице каталога в [консоли управления]({{ link-console-main }}) нажмите кнопку **Создать ресурс** и выберите **Виртуальная машина**.

     ![create-vm](../../_assets/tutorials/wordpress/vm-create-1.png)

  1. В поле **Имя** введите имя ВМ `wordpress`. Требования к имени:

     {% include [name-format](../../_includes/name-format.md) %}

  1. Выберите зону доступности, в которой будет находиться ВМ.
  1. В блоке **Выбор образа/загрузочного диска** перейдите на вкладку **{{ marketplace-name }}** и выберите публичный образ [WordPress](/marketplace/products/yc/wordpress).

     ![choose-image](../../_assets/tutorials/wordpress/vm-create-3.png)

  1. В блоке **Вычислительные ресурсы**:
     * Выберите [платформу](../../compute/concepts/vm-platforms.md).
     * Укажите необходимое количество vCPU и объем RAM.

     Для тестирования хватит минимальной конфигурации:
     * **Платформа** — Intel Ice Lake.
     * **vCPU** — 2.
     * **Гарантированная доля vCPU** — 20%.
     * **RAM** — 1 ГБ.
  1. В блоке **Сетевые настройки** выберите, к какой подсети необходимо подключить ВМ при создании.
  1. В пункте **Публичный адрес** выберите **Автоматически**.

     ![choose-network](../../_assets/tutorials/wordpress/vm-create-4.png)

  1. В пункте **Группа безопасности** выберите группу `wordpress`.
  1. Укажите данные для доступа на ВМ:
     * В поле **Логин** введите имя пользователя.
     * В поле **SSH-ключ** вставьте содержимое файла открытого ключа.

       Пару ключей для подключения по [SSH](../../glossary/ssh-keygen.md) необходимо создать самостоятельно. Подробнее см. [{#T}](../../compute/operations/vm-connect/ssh.md).
  1. Нажмите кнопку **Создать ВМ**.

  Создание ВМ может занять несколько минут. Когда ВМ перейдет в статус `RUNNING`, вы можете начать настраивать сайт.

  При создании ВМ назначается публичный IP-адрес и [имя хоста (FQDN)](../../compute/concepts/network.md#hostname). Эти данные можно использовать при настройке DNS и для доступа по SSH.

- {{ TF }} {#tf}

  См. раздел [Как создать инфраструктуру с помощью {{ TF }}](#terraform).

{% endlist %}

## Настройте DNS (если есть доменное имя) {#configure-dns}

Если у вас есть зарегистрированное доменное имя, воспользуйтесь сервисом [{{ dns-full-name }}](../../dns/) для управления доменом.

Также настроить DNS можно с помощью {{ TF }}, подробнее см. в разделе [Как создать инфраструктуру с помощью {{ TF }}](#terraform).

{% include [configure-a-record-and-cname](../_tutorials_includes/configure-a-record-and-cname.md) %}

## Настройка WordPress {#wordpress-setup}

После того как ВМ `wordpress` перейдет в [статус](../../compute/concepts/vm-statuses.md) `RUNNING`, выполните:
1. В блоке **Сеть** на странице ВМ в [консоли управления]({{ link-console-main }}) найдите публичный IP-адрес ВМ и внесите в [ресурсную запись](../../dns/concepts/dns-zone.md) [типа А](../../dns/concepts/resource-record.md#a), созданную ранее.

   ![add-ssh](../../_assets/tutorials/wordpress/vm-create-5.png)

1. Перейдите по доменному имени, которое вы настроили, или по адресу ВМ в браузере.
1. Выберите язык и нажмите кнопку **Продолжить**.

   ![choose-language](../../_assets/tutorials/wordpress/wordpress-1.png)

1. Заполните информацию для доступа к сайту:
   1. Укажите любое название сайта, например, `yc-wordpress`.
   1. Укажите имя пользователя, которое будет использоваться для входа в административную панель, например, `yc-user`.
   1. Укажите пароль, который будет использоваться для входа в административную панель.
   1. Укажите вашу электронную почту.

   ![credentials](../../_assets/tutorials/wordpress/wordpress-2.png)

1. Нажмите кнопку **Установить WordPress**.
1. Если установка прошла успешно, нажмите кнопку **Войти**.

   ![login](../../_assets/tutorials/wordpress/wordpress-3.png)

1. Войдите на сайт, используя указанные на прошлых шагах имя пользователя и пароль. После этого откроется административная панель, в которой можно приступать к работе с вашим сайтом.
1. Убедитесь, что сайт доступен, открыв публичный IP-адрес ВМ в браузере.

## Как удалить созданные ресурсы {#clear-out}

Чтобы перестать платить за развернутый сервер, достаточно [удалить](../../compute/operations/vm-control/vm-delete.md) ВМ `wordpress`.

Если вы зарезервировали статический публичный IP-адрес специально для этой ВМ:
1. Выберите сервис **{{ vpc-short-name }}** в вашем каталоге.
1. Перейдите на вкладку **IP-адреса**.
1. Найдите нужный IP-адрес, нажмите значок ![ellipsis](../../_assets/options.svg) и выберите пункт **Удалить**.

## Как создать инфраструктуру с помощью {{ TF }} {#terraform}

{% include [terraform-definition](../terraform-definition.md) %}

Чтобы разместить веб-сайт на базе CMS WordPress с помощью {{ TF }}:
1. [Установите {{ TF }}](../../tutorials/infrastructure-management/terraform-quickstart.md#install-terraform), [получите данные для аутентификации](../../tutorials/infrastructure-management/terraform-quickstart.md#get-credentials) и укажите источник для установки провайдера {{ yandex-cloud }} (раздел [{#T}](../../tutorials/infrastructure-management/terraform-quickstart.md#configure-provider), шаг 1).
1. Подготовьте файлы с описанием инфраструктуры.

   {% list tabs group=infrastructure_description %}

   - Готовый архив {#ready}

     1. Создайте папку для файлов.
     1. Скачайте [архив](https://{{ s3-storage-host }}/www.example.com/doc-files/wordpress.zip) (1 КБ).
     1. Разархивируйте архив в папку. В результате в ней должен появиться конфигурационный файл `wordpress.tf`.

   - Вручную {#manual}

     1. Создайте папку для файлов.
     1. Создайте в папке конфигурационный файл `wordpress.tf`:

        {% cut "wordpress.tf" %}

        {% include [wordpress-tf-config](../../_includes/web/wordpress-tf-config.md) %}

        {% endcut %}

   {% endlist %}

   Более подробную информацию о параметрах используемых ресурсов в {{ TF }} см. в документации провайдера:
    * [yandex_compute_instance]({{ tf-provider-resources-link }}/compute_instance)
    * [yandex_vpc_security_group]({{ tf-provider-resources-link }}/yandex_vpc_security_group)
    * [yandex_vpc_network]({{ tf-provider-resources-link }}/vpc_network)
    * [yandex_vpc_subnet]({{ tf-provider-resources-link }}/vpc_subnet)
    * [yandex_dns_zone]({{ tf-provider-resources-link }}/dns_zone)
    * [yandex_dns_recordset]({{ tf-provider-resources-link }}/dns_recordset)
1. В блоке `metadata` укажите метаданные для создания ВМ `<имя_пользователя>:<содержимое_SSH-ключа>`. Указанное имя пользователя не играет роли, ключ будет присвоен пользователю, который задан в конфигурации образа WordPress. В разных образах это разные пользователи. Подробнее см. в разделе [{#T}](../../compute/concepts/vm-metadata.md#keys-processed-in-public-images).
1. Создайте ресурсы:

   {% include [terraform-validate-plan-apply](../terraform-validate-plan-apply.md) %}

1. [Настройте WordPress](#wordpress-setup).