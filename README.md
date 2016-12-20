SNMP to MQTT gateway
====================

Драйвер SNMP для Wiren Board.

Документация здесь: http://contactless.ru/wiki/index.php/Драйвер_SNMP

Зависимости
-----------

* snmp (Net-SNMP tools, в первую очередь - snmptranslate)
* snmp-mibs-downloader (не обязательно, если не планируется использовать стандартные MIB)


Конфигурация
------------

Конфигурационный файл демона содержит следующие поля настройки:

```
{
    "debug": false,
    "num_workers": 4,
    "devices": [...]
}
```

* *debug* - флаг включения режима отладки - в этом режиме генерируется дополнительный отладочный вывод;
* *num_workers* - максимальное количество одновременно посылаемых SNMP-запросов; по умолчанию 4;
* *devices* - массив опрашиваемых устройств.

Каждое устройство описывается следующим объектом:

```
{
    "name": "..",
    "id": "..",
    "address": "..",
    "device_type": "..",
    "enabled": true,
    "community": "..",
    "snmp_version": "..",
    "snmp_timeout": 5,
    "poll_interval": 1000,
    "oid_prefix": "..",
    "channels": []
}
```

Обязательные параметры:
* *address* - IP-адрес или имя хоста опрашиваемого устройства;
* *community* - SNMP Community name - "имя сообщества" для получения доступа к полям устройства;
* *channels* - список опрашиваемых каналов.

Необязательные параметры:
* *name* - человеко-читаемое имя устройства (генерируется из адреса хоста и имени сообщества);
* *id* - идентификатор устройства в MQTT (генерируется из адреса хоста и имени сообщества);
* *device_type* - тип устройства; по типу устройства выбирается шаблон;
* *enabled* - флаг активности устройства (true по умолчанию);
* *snmp_version* - версия SNMP, используемая при опросе устройства (на данный момент поддерживается только "2c");
* *snmp_timeout* - время ожидания ответа устройства (в секундах);
* *poll_interval* - минимальный интервал опроса каналов данного устройства по умолчанию (в миллисекундах);
* *oid_prefix* - префикс для текстовых OID каналов по умолчанию.


Для описания каналов используется следующая структура:

```
{
    "name": "..",
    "oid": "..",
    "control_type": "..",
    "scale": 1.0,
    "poll_interval": 1000
}
```

Обязательные параметры:
* *name* - имя канала (при использовании шаблонов может совпадать с одним из шаблонных, тогда данные будут наложены, см. Шаблоны);
* *oid* - OID канала (может быть в виде последовательности чисел через точку или текстовом);
* *control_type* - тип данных в канале (один из следующих: text, value, temperature, voltage, power);

Необязательные параметры:
* *scale* - коэффициент для полученных данных, если получаемые данные - число;
* *poll_interval* - минимальное время между двумя опросами канала (в миллисекундах), по умолчанию - 1000.


Шаблоны
-------

Шаблоны - это описания отдельных устройств, расположенные в специальной директории и имеющие имя config-[device_name].json.

По умолчанию, шаблоны располагаются в директории /usr/share/wb-mqtt-snmp/templates/. Можно выбрать любую другую директорию при
запуске демона с помощью ключа -templates.

Файл шаблона содержит описание одного устройства, при этом обязательно задаётся значение *device_type*.

При загрузке демона шаблоны "накладываются" на конфигурационный файл. Каналы "накладываются" при совпадении поля "name".
Если данные из конфигурационного файла конфликтуют с шаблоном, выбираются данные из конфигурационного файла.
