 ![main](https://habrastorage.org/r/w1560/webt/lu/ah/u_/luahu_ureofiaunaxa2gkdm2zai.png)  

Все работы произведены на Linux ([**TangoBox 9.3**](https://s2innovation.sharepoint.com/:f:/s/Developers/EovD2IBwhppAp-ZLXtawQ6gB9F6aXPPs2msr2hgPGTO-FQ?e=Ii3tnr) на основе Ubuntu 18.04), который является официальным дистрибутивом проекта.

  

## Содержание

  

1. [Установка](#install)  

	1. [Установка из репозитория](#install-repo)
	2. [Установка из исходников](#install-src)
2. [Отключение](#off)  

	1. [Отключение старого RestServer-а](#rest-off)
3. [API](#api)  

	1. [Документация](#doc)

  

## Установка

  

## Установка из репозитория

  

Получаем последнюю версию docker-а из репозитория и запускаем его.

  


```bash
docker pull tangocs/rest-server:rest-server-2.1
docker run --restart unless-stopped -p 8080:8080 -d tangocs/rest-server:rest-server-2.1
```
  

Смотрим результат выполнения. В список контейнеров добавился **tangocs/rest-server**.

  

![](https://habrastorage.org/r/w1560/webt/hp/vo/ch/hpvochw2wqyereteof63f-rxbn8.png)

  

Список запущенных контейнеров пополнил **tangocs/rest-server:rest-server-2.1**.

  

![](https://habrastorage.org/r/w1560/webt/jm/7l/yl/jm7lylie0tpm2kzitxo4orwh_6y.png)

  

Проверка работоспособности:

  

![](https://habrastorage.org/r/w1560/webt/tc/4x/-t/tc4x-tgnxlrjhdgjsldvbopwvzc.png)

  

**Важный акцент**, в системе **TangoBox 9.3** изначально работает старый RestServer. Работает он не в docker-е, а в **самой системе**! Их **API** отличается.

  

![](https://habrastorage.org/r/w1560/webt/zr/et/dn/zretdnrnmlvxtbmlr3vp0pp1slm.png)

  

Проверка его работы:

  

![](https://habrastorage.org/r/w1560/webt/mo/rr/op/morroplid6jp1oaa14wmi__7beo.png)

  

Работает на **10001** порту, и поскольку он работает в системе, то обратится может к **Tango Controls** как к **localhost**, чего не сможет docker.

  


```bash
http://localhost:10001/tango/rest/rc4/hosts/localhost/10000/devices/sys/tg_test/1/attributes/double_scalar/value
```
  

В docker-t RestServer работает на **8080** порту, и его порт проброшен в систему. Но обращаться к **Tango Controls** он должен по **ip** адресу системы **172.17.0.1** где тот работает!

  


```bash
http://localhost:8080/tango/rest/v10/hosts/172.17.0.1;port=10000/devices/sys/tg_test/1/attributes/double_scalar/value
```
  

Репозиторий содержит **не последнюю** версию. Установить последнюю версию можно из исходников, на текущий момент **2.2**.

  

### Установка из исходников

  

Последняя версия [TangoRestServer](https://github.com/tango-controls/rest-server).

  


```bash
git clone https://github.com/tango-controls/rest-server.git
cd rest-server
```
  

Собирается docker под **java** версии **11**, но все в системе работает под **8**-ой версий.  

Временно поменяем версию по умолчанию с 8 на 11.

  


```bash
sudo update-alternatives --config java
```
  

![](https://habrastorage.org/r/w1560/webt/0i/in/0p/0iin0pjssdz7skz2g2k6om09u6u.png)

  


```bash
mvn package
docker build -t tangocs/rest-server:rest-server-2.2 . 
```
  

Получил следующую ошибку:

  

`COPY failed: file not found in build context or excluded by .dockerignore: stat target/.war: file does not exist`

  

Открываем **Dockerfile** и меняем  

`COPY target/${REST_SERVER_VERSION}.war /usr/local/tomcat/webapps/tango.war`  

на  

`COPY target/rest-server-2.2-SNAPSHOT.war /usr/local/tomcat/webapps/tango.war`

  

Т.к. docker собрался с именем **rest-server-2.2-SNAPSHOT.war**

  

Смотрим список образов:

  

![](https://habrastorage.org/r/w1560/webt/wn/v9/5e/wnv95efh8dsmn9agbwhw8ciwa1i.png)

  

Теперь запускаем наш docker, он будет работать на **8080** порту.

  


```bash
docker run --restart unless-stopped -p 8080:8080 -d tangocs/rest-server:rest-server-2.2
```
  

Процедура проверки работоспособности такая же как и в 1-ом абзаце.

  

**Переключим версию java назад на 8-ую.**

  


```bash
sudo update-alternatives --config java
```
  

### Отключение старого RestServer-а

  

Запускаем **Astor**

  


```bash
astor
```
  

![](https://habrastorage.org/r/w780q1/webt/rg/tu/6h/rgtu6hcrhrysb8ajek23ir8hz7y.jpeg)

  

Выбираем ветку **Miscellaneous**, стартер **tangobox**, меню **Open control Panel**. Открывается окно управления стартером.

  

![](https://habrastorage.org/r/w780q1/webt/nt/x7/a5/ntx7a5r9tvdc0xgt49dhztzedt0.jpeg)

  

Ищем RestServer и выбираем **Set startup level**, и отключаем его.

  

![](https://habrastorage.org/r/w780q1/webt/pk/9k/an/pk9kan62t8qvyuxxxxvt2ve5uom.jpeg)

  

В итоге получается следующее:

  

![](https://habrastorage.org/r/w780q1/webt/3t/vv/jo/3tvvjo67gkvhvzsxetvgz2qq_ha.jpeg)

  

Старый RestServer выключен.

  

## API

  

### Документация

  

[**Документация**](https://github.com/tango-controls/rest-api)

  

Спасибо за внимание.

   