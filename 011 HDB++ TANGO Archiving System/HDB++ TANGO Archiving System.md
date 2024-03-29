 ![main](https://habrastorage.org/r/w1560/webt/tu/vr/ec/tuvrecn2a_yv1mikcfb39r9hlfa.png)  

## Что такое [HDB++](https://www.tango-controls.org/community/project-docs/hdbplusplus/)?

  

Это система архивирования TANGO, позволяет сохранять данные полученные с устройств в системе TANGO.

  

Здесь будет описана работа с Linux ([**TangoBox 9.3**](https://s2innovation.sharepoint.com/:f:/s/Developers/EovD2IBwhppAp-ZLXtawQ6gB9F6aXPPs2msr2hgPGTO-FQ?e=Ii3tnr) на основе Ubuntu 18.04), это уже готовая система где все настроено.

  

## О чем статья?

  

* Архитектура системы.
* Как настроить архивирование.

  

У меня ушло ~ 2 недели чтобы разобраться в архитектуре и написать свои скрипты для **python** под это дело.

  

## Для чего это нужно?

  

Позволяет хранить историю показаний Вашего оборудования.

  

* Вам не нужно думать о том как хранить данные в БД.
* Нужно только указать какие атрибуты с какого оборудования архивировать.

  

## Где взять?

  

* [**deb && sql**](https://www.tango-controls.org/community/project-docs/hdbplusplus/hdbplusplus-downloads/)
* [**исходники**](https://github.com/tango-controls-hdbpp)
* [**документация**](https://www.tango-controls.org/community/project-docs/hdbplusplus/hdbplusplus-doc/)
* [**образ TangoBox на базе Ubuntu**](https://tango-controls.readthedocs.io/en/latest/installation/virtualmachine.html)

  

## Архитектура

  

![image](https://habrastorage.org/r/w1560/webt/si/9f/sq/si9fsq0jjolmzn2t2-ka9n_svnk.png)

  

Здесь две самые важные вещи — это **Archiver** и **Archiving DB**. **HDB++ Configuration** графическая утилита для управления Archiver. **HDB++ Viewer** утилита просмотра Archiving DB.

  

**Archiver** опрашивает наши **Device Server**-а и записывает историю в **Archiving DB**.  

Archiver это такой же Device Server:

  

![image](https://habrastorage.org/r/w780q1/webt/wx/bw/4c/wxbw4cyo_occntl8jparsj2drbo.jpeg)

  

**Archiving DB** в нашей системе базируется на **MySQL**, она находится в докере **tangobox-hdbpp**.

  

![image](https://habrastorage.org/r/w780q1/webt/lx/zx/tf/lxzxtfz04nofarfshjy4jdh3hhy.jpeg)

  

## Archiver

  

За архивацию отвечает **archiving/hdbpp/eventsubscriber.1**

  

![image](https://habrastorage.org/r/w780q1/webt/bc/dd/mw/bcddmwci_lpsr7exia445op9nze.jpeg)

  

Посмотрим список команд этого сервера.

  

![image](https://habrastorage.org/r/w780q1/webt/e7/qk/ts/e7qktshbfap5gkk3dm_de-uqyxg.jpeg)

  

На картинке список команд управления этим Device Server-ом

  

Здесь нас интересуют:

  

* **AttibuteAdd** — добавить атрибут.
* **AttibutePause** — поставить архивацию на паузу.
* **AttibuteRemove** — удалить атрибут.
* **AttibuteStart** — начать архивацию.
* **AttibuteStatus** — состояние атрибута.
* **AttibuteStop** — остановить архивацию.

  

Посмотрим через **Jive** список атрибутов за которыми следит Archiver:

  


```bash
jive
```
  

![image](https://habrastorage.org/r/w780q1/webt/wx/bw/4c/wxbw4cyo_occntl8jparsj2drbo.jpeg)

  

Дважды щелкаем на устройство.

  

![image](https://habrastorage.org/r/w780q1/webt/gq/li/qa/gqliqanb2roaz98yqhl0b4lxhkk.jpeg)

  

Управнее этим Device Server-ом осуществляется через утилиту **HDB++ Configuration**, это графическая утилита которая отправляет выше показанные команды в **archiving/hdbpp/eventsubscriber.1**. Дальше будет показано как делать это программно.

  

## HDB++ Configuration

  


```bash
hdbpp-configurator -configure
```
  

![image](https://habrastorage.org/r/w780q1/webt/sv/nw/bh/svnwbh8yxo3l170gmibfly0gdzi.jpeg)

  

С помощью нее запускается/останавливается архивация и задаются параметры архивирования. Дважды щелкаем по атрибуту:

  

![image](https://habrastorage.org/r/w780q1/webt/ve/gq/qt/vegqqtoux5y-2cctdn1dugymqzw.jpeg)

  

* **TTL** — сколько дней будет длиться архивация истории.
* **absolute change** — изменение атрибута в единицах.
* **relative change** — изменение атрибута в процентах.
* **event period** — если значения изменились, то записывать каждые мс.
* **Attibute polling period** — записывать каждые мс.

  

## Archiving DB

  

В ней нас будет интересовать БД **hdbpp**:

  

![image](https://habrastorage.org/r/w780q1/webt/5e/hh/15/5ehh15mfnxsnbr_di1sc_oqfhzk.jpeg)

  

Также потребуется настроить доступ к БД, с какой машины и кому разрешено подключаться (Потому что изучать как система была уже настроена мне было лень, легче задать своего пользователя.):

  


```bash
docker exec -i -t 04e048 bash
mysql
```
  


```sql
GRANT ALL PRIVILEGES on *.* to 'root'@'172.18.0.1' IDENTIFIED BY 'tango';
FLUSH PRIVILEGES;
```
  

Наша основная система имеет адрес **172.18.0.1**, docker с БД находится на **172.18.0.7**.

  

Теперь перейдем к структуре БД, здесь основная таблица **att\_conf**. В ней прописаны атрибуты которые попали в систему архивации:

  

![image](https://habrastorage.org/r/w780q1/webt/p5/qb/jn/p5qbjnakml0npautm0rhfiz8ixw.jpeg)

  

Здесь важные поля **att\_conf\_id** и **att\_conf\_data\_type\_id**. По att\_conf\_data\_type\_id из таблицы **att\_conf\_data\_type** получим тип данных атрибута. Например **scalar\_devushort\_ro**, получив тип данных узнаем таблицу в которой хранится история. Имя таблицы будет **att\_scalar\_devushort\_ro**, из этой таблицы по att\_conf\_id получаем архив данных интересующего нас атрибута.

  

Команда **AttibuteAdd** archiving/hdbpp/eventsubscriber.1 не добавляет запись о необходимом атрибуте в att\_conf в отличие от команды добавления **hdbpp-configurator**. Поэтому пришлось делать это самому. Основная проблема найти тип атрибута (который хотим архивировать) в таблице **att\_conf\_data\_type**, взять его **id** и записать его в **att\_conf/att\_conf\_data\_type\_id**.

  

## Ошибки

  

Так же случается что по каким-то причинам атрибуты не архивируются. Такие атрибуты попадают в **AttributeNokList**:

  

![image](https://habrastorage.org/r/w780q1/webt/f1/nr/qn/f1nrqnn-kaeuybn-6t9q7ens0co.jpeg)

  

Посмотреть причину этого можно через **hdbpp-configurator**:

  

![](https://habrastorage.org/r/w780q1/webt/nr/8n/gx/nr8ngx2fhxhxyhedlv1qjaoyknq.jpeg)

  

![](https://habrastorage.org/r/w780q1/webt/zx/kg/ea/zxkgealu746_r0dxylb9h2y-mgu.jpeg)

  

![](https://habrastorage.org/r/w780q1/webt/eu/s0/tt/eus0ttx4qesv7wgapdc80w8rgj4.jpeg)

  

Здесь мы видим что наш Devise Server не запущен.

  

## Python

  

Механизмы python для работы с HDB++.

  

Есть официальная библиотека для python**2.7** для работы с HDB++ [PyTangoArchiving](https://github.com/tango-controls/PyTangoArchiving). Разобраться с ней удалось только когда написал свою библиотеку. По ней не хватает документации, что передается в методы, какие типы данных, что передавать в аргументах (**Это мое мнение**).

  

[**Модуль**](https://github.com/dvjdjvu/hdbpp) создан для версии >= **3.6**. Здесь все стандартные настройки для работы на **TangoBox 9.3** заданы по умолчанию.

  

### Установка модуля

  


```bash
sudo pip3 install -r requirements.txt
sudo python3 setup.py install
```
  

Зависимости:

  

* **mysql-connector**>=2.2.9
* **pytango**>=9.3.2

  

### Как использовать

  


```python
from hdbpp import HDBPP

if __name__ == '__main__':
    hdbpp = HDBPP()
    # семантика:
    # __init__(self, host="172.18.0.7", user="root", password="tango", database="hdbpp", 
    #    archive_server_name="archiving/hdbpp/eventsubscriber.1",
    #    server_default = "tango://tangobox:10000")

    # Подключиться к серверу архивации и к БД с архивами.
    if hdbpp.connect() == False :
        exit(0)

    # Получить историю атрибута за все время.
    archive = hdbpp.get_archive('tango://tangobox:10000/ECG/ecg/1/Lead')
    for a in archive:
        print(a)

    # Текущий статус архивации атрибута
    ret = hdbpp.archiving_status('tango://tangobox:10000/ECG/ecg/1/Lead')
    print(ret)

    # Добавить атрибут на сервер архивации
    hdbpp.archiving_add(['tango://tangobox:10000/ECG/ecg/1/Lead'])

    # Начать архивацию атрибута
    hdbpp.archiving_start('tango://tangobox:10000/ECG/ecg/1/Lead', 10 * 60 * 1000, 5 * 60 * 1000, 2, 1)
    # где:
    # 10 * 60 * 1000 - опрашивать и архивировать атрибут каждые мс
    # 5 * 60 * 1000 - архивировать атрибут каждые мс, если значение вышло за порог
    # 2 - порог изменения атрибута в единицах
    # 1 - порог изменения атрибута в процентах

    # Остановить архивацию атрибута
    hdbpp.archiving_stop('tango://tangobox:10000/ECG/ecg/1/Lead')

    # Закрыть соединение
    hdbpp.close()
```
  

Более подробная документация в коде.  

Ее можно посмотреть например так:

  


```bash
pydoc3 hdbpp.HDBPP
```
  

## Ссылки

  

* [Исходные коды](https://github.com/dvjdjvu/hdbpp)
* [Статья про Tango Controls](https://habr.com/ru/post/507144/)

  

Статью писал для себя, потому что спустя некоторое время начинаю забывать как и что делать.

  

Спасибо за внимание.

   