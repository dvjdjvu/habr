 ![main](https://habrastorage.org/r/w1560/webt/yw/uk/b4/ywukb4xd4bdxwnb1ns5gldo7f30.png)  

## Что такое [TANGO](https://www.tango-controls.org/)?

  

Это система для управления различным оборудованием и программным обеспечением.  

TANGO поддерживает 4 платформы на данный момент: Linux, Windows NT, Solaris и HP-UX.  

Здесь будет описана работа с Linux(Ubuntu 18.04)

  

## Для чего нужно?

  

Упрощает работу с различным оборудованием и софтом.

  

* Вам не нужно думать о том как хранить данные в БД, это уже сделано за Вас.
* Нужно только описать механизм опроса датчиков.
* Сводит весь Ваш код к одному стандарту.

  

## Где взять?

  

* [**исходники**](https://www.tango-controls.org/downloads/)
* [**инструкция по установке**](https://tango-controls.readthedocs.io/en/latest/installation/index.html)
* [**образ TangoBox на базе Ubuntu**](https://tango-controls.readthedocs.io/en/latest/installation/virtualmachine.html)

  

Из исходников не смог ее запустить, для работы использовал готовый образ TangoBox 9.3.  

В инструкции описано как ставить из пакетов.

  

## Из чего она состоит?

  

* **JIVE** — служит для просмотра и редактирования базы данных TANGO.
* **POGO** — генератор кода для серверов устройств TANGO.
* **Astor** — программный менеджер для системы TANGO.

  

Нас будут интересовать только первые два компонента.

  

## Поддерживаемые языки программирования

  

* **C**
* **C++**
* **Java**
* **JavaScript**
* **Python**
* **Matlab**
* **LabVIEW**

  

Я работал с ней на python & c++. Здесь в качестве примера будет использоваться c++.

  

Теперь перейдем к описанию как подключить устройство к TANGO и как с ним работать. В качестве примера будет взята плата ***GPS neo-6m-0-001***:

  

![image](https://habrastorage.org/r/w780q1/webt/dw/0y/di/dw0ydivfn97k481ros5rxplg3oy.jpeg)

  

Как видно на картинке плату к ПК подключаем через UART CP2102. При подключении к ПК появляется устройство **/dev/ttyUSB[0-N]**, обычно /dev/ttyUSB0.

  

## POGO

  

Теперь запустим **pogo**, и с генерируем скелет код для работы с нашей платой.

  


```bash
pogo
```
  

![image](https://habrastorage.org/r/w780q1/webt/a5/_g/kj/a5_gkjpv3rpbi62yambp-2vkgx0.jpeg)

  

У меня уже был создан код, создадим его заново *File->New*.

  

![image](https://habrastorage.org/r/w780q1/webt/sx/p6/k-/sxp6k-6jfutggptvtxb7qjhucvc.jpeg)

  

Получаем следующее:

  

![image](https://habrastorage.org/r/w780q1/webt/wc/t0/ss/wct0sst0gu49gw-xkczpuoejrt8.jpeg)

  

Наше устройство(под устройством в дальнейшем будет иметься ввиду программная часть) пустое и имеет две команды управления: **State** & **Status**.

  

Его нужно заполнить необходимыми атрибутами: 

  

**Device Property** — значения по умолчанию которые передаем в устройство для его инициализации, для платы GPS нужно передать имя платы в системе **com="/dev/ttyUSB0"** и скорость com порта **baudrade=9600**

  

**Commands** — команды управления нашим устройством, им можно задать аргументы и возвращаемое значение.

  

* **STATE** — возвращает текущее состояние, из **States**
* **STATUS** — возвращает текущий статус, это строковое дополнение к **STATE**
* **GPSArray** — возвращает **gps** строку в виде **DevVarCharArray**

  

Далее задаются атрибуты устройства которые можно читать/писать в/из него.  

**Scalar Attributes** — простые атрибуты (char, string, long и т.п.)  

**Spectrum Attributes** — одномерные массивы  

**Image Attributes** — двумерные массивы

  

**States** — состояния в котором находится наше устройство.

  

* **OPEN** — устройство открыто.
* **CLOSE** — устройство закрыто.
* **FAILT** — ошибка.
* **ON** — принимаем данные с устройства.
* **OFF** — нет данных с устройства.

  

Пример добавления атрибута **gps\_string**:

  

![image](https://habrastorage.org/r/w780q1/webt/4-/2s/f2/4-2sf2ywgzrpafgzcpfgpwkdchi.jpeg)

  

**Polling period** время в мс, как часто будет обновляться значение gps\_string. Если время обновления не задать, то атрибут будет обновляться только по запросу.

  

Получилось:

  

![image](https://habrastorage.org/r/w780q1/webt/ci/gp/9q/cigp9qv82egfdtzzlkghvx-ack0.jpeg)

  

Теперь нужно с генерировать код *File->Generate*

  

![image](https://habrastorage.org/r/w780q1/webt/nu/m7/6u/num76u0giqpc8vzbznow-pu7ryu.jpeg)

  

По умолчанию Makefile не генерируется, в 1-ый раз нужно поставить галочку что бы его создать. Это сделано для того что бы внесенные в него правки не удалялись при новой генерации. Создав его единожды и настроив под свой проект(прописать ключи компиляции, доп. файлы) можно забыть про него.

  

Теперь переходим непосредственно к программированию. pogo с генерировал нам следующее:

  

![image](https://habrastorage.org/r/w780q1/webt/bg/id/er/bgiderec9ohjcrp0yrbjlpymleq.jpeg)

  

Нас будут интересовать NEO6M.cpp & NEO6M.h. Рассмотрим для примера конструктор класса:

  


```cpp
NEO6M::NEO6M(Tango::DeviceClass *cl, string &s)
 : TANGO_BASE_CLASS(cl, s.c_str())
{
    /*----- PROTECTED REGION ID(NEO6M::constructor_1) ENABLED START -----*/
    init_device();

    /*----- PROTECTED REGION END -----*/    //  NEO6M::constructor_1
}
```
  

Что здесь есть и что здесь главное? В функции init\_device() происходит выделение памяти для наших атрибутов: **gps\_string** & **gps\_array**, но это не важно. **Самое важное здесь**, это комментарии:

  


```cpp
/*----- PROTECTED REGION ID(NEO6M::constructor_1) ENABLED START -----*/
    .......
/*----- PROTECTED REGION END -----*/    //  NEO6M::constructor_1
```
  

Все что находится внутри этого блока комментария при последующих перегенерациях кода в pogo не будет **удаляться!**. Все что в не блоках будет! Это те места где мы можем программировать и вносить свои правки.

  

Теперь какие главные функции содержит класс **NEO6M**:

  


```cpp
void always_executed_hook();
void read_attr_hardware(vector<long> &attr_list);
void read_gps_string(Tango::Attribute &attr);
void read_gps_array(Tango::Attribute &attr);
```
  

Когда мы захотим прочитать значение атрибута **gps\_string**, будут вызваны функции в следующем порядке: **always\_executed\_hook**, **read\_attr\_hardware** и **read\_gps\_string**. В read\_gps\_string произойдет заполнение gps\_string значением.

  


```cpp
void NEO6M::read_gps_string(Tango::Attribute &attr)
{
    DEBUG_STREAM << "NEO6M::read_gps_string(Tango::Attribute &attr) entering... " << endl;
    /*----- PROTECTED REGION ID(NEO6M::read_gps_string) ENABLED START -----*/
    //  Set the attribute value

    *this->attr_gps_string_read = Tango::string_dup(this->gps.c_str());

    attr.set_value(attr_gps_string_read);

    /*----- PROTECTED REGION END -----*/    //  NEO6M::read_gps_string
}
```
  

## Компиляция

  

Заходим в папку с исходниками и:

  


```bash
make
```
  

Программа скомпилируется в папку ~/DeviceServers.

  


```bash
tango-cs@tangobox:~/DeviceServers$ ls
NEO6M
```
  

## JIVE

  


```bash
jive
```
  

![image](https://habrastorage.org/r/w780q1/webt/xa/p8/u4/xap8u4kcs__3dai1pnx1g5bsriq.jpeg)

  

В БД уже есть какие-то устройства, создадим теперь наше *Edit->Create Server*

  

![image](https://habrastorage.org/r/w780q1/webt/9k/zg/hz/9kzghz8hsf5bcd9-xl1_jwudabk.jpeg)

  

Теперь попробуем подключиться к нему:

  

![image](https://habrastorage.org/r/w780q1/webt/-i/x_/xx/-ix_xxjm8xx3nv6yy8ht62ot7_a.jpeg)

  

Ни чего не выйдет, сначала надо запустить нашу программу:

  


```bash
sudo ./NEO6M neo6m -v2
```
  

Подключиться к com порту у меня можно только с правами **root**-а. **v** — уровень логирования.

  

### Как запустить несколько устройств.

  

Для того что бы запустить несколько таких устройств, необходимо создать нужное количество **Devices** в **Jive**:  

![image](https://habrastorage.org/r/w780q1/webt/cs/-y/yu/cs-yyuplob93bay3a051gv5dnre.jpeg)

  


```bash
NEO6M/neo6m/2
...
NEO6M/neo6m/N
```
  

Для каждого задать свои **Properties**. Адрес устройства и скорость подключения.  

Теперь запускаем:

  


```bash
sudo ./NEO6M neo6m -v2
```
  

У нас запуститься два устройства сразу.

  

Теперь можем подключиться:

  

![image](https://habrastorage.org/r/w780q1/webt/gc/iz/ab/gcizabt2ztocjsdrzm6jwjtybxw.jpeg)

  

## Клиент

  

В графике смотреть на картинки конечно хорошо, но нужно что-то более полезное. Напишем клиент который будет подключаться к нашему устройству и забирать с него показания.

  


```cpp
#include <tango.h>
using namespace Tango;

int main(int argc, char **argv) {
    try {

        //
        // create a connection to a TANGO device
        //

        DeviceProxy *device = new DeviceProxy("NEO6M/neo6m/1");

        //
        // Ping the device
        //

        device->ping();

        //
        // Execute a command on the device and extract the reply as a string
        //

        vector<Tango::DevUChar> gps_array;

        DeviceData cmd_reply;
        cmd_reply = device->command_inout("GPSArray");
        cmd_reply >> gps_array;

        for (int i = 0; i < gps_array.size(); i++) {            
            printf("%c", gps_array[i]);
        }
        puts("");

        //
        // Read a device attribute (string data type)
        //

        string spr;
        DeviceAttribute att_reply;
        att_reply = device->read_attribute("gps_string");
        att_reply >> spr;
        cout << spr << endl;

        vector<Tango::DevUChar> spr2;
        DeviceAttribute att_reply2;
        att_reply2 = device->read_attribute("gps_array");
        att_reply2.extract_read(spr2);

        for (int i = 0; i < spr2.size(); i++) {
            printf("%c", spr2[i]);
        }

        puts("");

        //
        // Write a device attribute (string data type)
        // Пример записи абстрактного атрибута доступного на запись.

        DeviceAttribute value;
        value = DeviceAttribute("attr_double", 32.3233);
        device->write_attribute(value);

    } catch (DevFailed &e) {
        Except::print_exception(e);
        exit(-1);
    }
}
```
  

Как компилировать:

  


```bash
g++ gps.cpp -I/usr/local/include/tango -I/usr/local/include -std=c++0x -Dlinux -L/usr/local/lib -ltango -lomniDynamic4 -lCOS4 -lomniORB4 -lomnithread -llog4tango -lzmq -ldl -lpthread -lstdc++
```
  

Результат:

  


```bash
tango-cs@tangobox:~/workspace/c$ ./a.out 
$GPRMC,,V,,,,,,,,,,N*53

$GPRMC,,V,,,,,,,,,,N*53

$GPRMC,,V,,,,,,,,,,N*53
```
  

Получили результат в качестве возврата команды, взятия атрибутов строки и массива символов.

  

## Ссылки

  

* [Исходные коды](https://github.com/dvjdjvu/tango_gps)

  

Статью писал для себя, потому что спустя некоторое время начинаю забывать как и что делать.

  

Спасибо за внимание.

   