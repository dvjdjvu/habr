 ![main](https://habrastorage.org/r/w1560/webt/jf/9n/pl/jf9nplvejlhxgattedsrk33eyye.png)  

Эта статья продолжение статьи [HDB++ TANGO Archiving System](https://habr.com/ru/post/533040/), в которой рассказывалось об архитектуре и о том как настроить архивацию. Здесь речь пойдет о том как поднять и настроить docker в котором будет работать база архивирования.

  

## docker

  

Скачиваем и запускаем docker:

  


```bash
docker pull registry.gitlab.com/s2innovation/tangobox-docker/tangobox-hdbpp:latest

docker run -dit --name tangobox-hdbpp -h tangobox-hdbpp --network tango_nw --ip 172.18.0.7 --add-host scsc:172.18.0.1 -e TANGO_HOST=scsc:10000 --restart unless-stopped registry.gitlab.com/s2innovation/tangobox-docker/tangobox-hdbpp:latest
```
  

## Создание и настройка Device Server-ов архивации

  

Запускаем **jive**, и создаем два сервера **hdb++cm-srv** и **hdb++es-srv**:

  


```bash
jive
```
  

*Edit->Create Server*

  



| Name | Value |
| --- | --- |
| Server | hdb++cm-srv/1 |
| Class | HdbConfigurationManager |
| Devices | archiving/hdbpp/configurationmanager.1 |

  



| Name | Value |
| --- | --- |
| Server | hdb++es-srv/1 |
| Class | HdbEventSubscriber |
| Devices | archiving/hdbpp/eventsubscriber.1 |

  

![](https://habrastorage.org/r/w780q1/webt/gt/pz/py/gtpzpywzgpbatm_xdzuwjtazauw.jpeg)

  

![](https://habrastorage.org/r/w780q1/webt/qu/sa/1e/qusa1ezysmtnuwhcnvbomnqqmzk.jpeg)

  

### Заполняем properties

  

Во вкладке **Server** заполняем **properties** у **hdb++cm-srv**:

  



| Property name | Value |
| --- | --- |
| ArchiverList | tango://tangobox:10000/archiving/hdbpp/eventsubscriber.1 |
| \_\_SubDevices | tango://tangobox:10000/archiving/hdbpp/eventsubscriber.1 |

  

![](https://habrastorage.org/r/w780q1/webt/i6/kg/di/i6kgdiewz-j_cupuxdzywn1dfny.jpeg)

  

Далее переходим во вкладку **Class** и заполняем **properties** у **HdbConfigurationManager**:

  



| Property name | Value |
| --- | --- |
| InheritedFrom | TANGO\_BASE\_CLASS |
| LibConfiguration | * host=tangobox-hdbpp
* user=hdbpprw
* dbname=hdbpp
* port=3306
* libname=libhdb++mysql.so.6
 |
| MaxSearchSize | 2000 |
| ProjectTitle | Hdb++ configuration manager |

  

![](https://habrastorage.org/r/w780q1/webt/jf/dy/fo/jfdyfopvej92pf-fad8jtshft5y.jpeg)

  

И у **HdbEventSubscriber**:

  



| Property name | Value |
| --- | --- |
| CheckPeriodicTime | 150 |
| DefaultContext | ALWAYS |
| Description | This class is able to subscribe on archive events and store value in Historical DB |
| HdbppContext | 0:ALWAYS 1:RUN 2:SHUTDOWN 3:SERVICE |
| InheritedFrom | TANGO\_BASE\_CLASS |
| LibConfiguration | * host=tangobox-hdbpp
* user=hdbpprw
* password=hdbpprw
* dbname=hdbpp
* port=3306
* libname=libhdb++mysql.so.6
 |
| PollingThreadPeriod | 60 |
| ProjectTitle | Tango Device Server |
| StartArchivingAtStart | false |
| StatisticsTimeWindow | 60 |
| SubscribeRetryPeriod | 150 |

  

![](https://habrastorage.org/r/w780q1/webt/sq/ks/oo/sqksoowpgytzwdmg9nmt-omzvvs.jpeg)

  

## Astor

  

Запускаем **astor**:

  


```bash
astor
```
  

![](https://habrastorage.org/r/w780q1/webt/tr/ti/mz/trtimz26qfmnjh-5_rrrjtalow0.jpeg)

  

![](https://habrastorage.org/r/w780q1/webt/cv/hf/pc/cvhfpc05arozrce-ydkhkxmmpdo.jpeg)

  

Если все прошло успешно, то мы увидим что **tangobox-hdbpp** горит зеленым. docker запустил внутри себя процесс **Starter**, который отвечает за подключение к **astor** из docker-а в основную систему.

  


```bash
/usr/local/bin/Starter
```
  

Если что-то пошло не так, то нужно смотреть в docker-е почему он не запустился.

  

Так же нужно убедиться, что запускаются:

  


```bash
/usr/local/bin/hdb++es-srv
/usr/local/bin/hdb++cm-srv
```
  

![](https://habrastorage.org/r/w780q1/webt/r8/7n/6_/r87n6_h5ain5louen9jqex1fsrw.jpeg)

  

Далее 2-ым щелчком мыши подключаемся к **tangobox-hdbpp**, жмем кнопку **Start New** и добавляем 2-ва Device Server-а созданные нами ранее. Порядок запуска задаем как на картинке.

  

![](https://habrastorage.org/r/w780q1/webt/xn/ps/ao/xnpsaob7do2k6erhkuy7kgzh7wy.jpeg)

  

![](https://habrastorage.org/r/w780q1/webt/dl/9t/a8/dl9ta8i2sssubv0reufysmgrjf8.jpeg)

  

Спасибо за внимание.

   