 ![main](https://habrastorage.org/r/w780q1/webt/yj/-c/sm/yj-csmwltho22f1-38_w8exbryu.jpeg)  

Ранее я писал статью [C/C++ из Python (ctypes)](https://habr.com/ru/post/466499/), в ней описывается процесс запуска на **Linux**. На этот раз мне понадобилось повторить это уже на **Android**. В этой статье речь пойдет о сборке, необходимых инструментах, механизмах отладки и установки.

  

Код на **C/C++** ни каких изменений не претерпел. Подробнее ознакомиться с описанием кода можно по ссылке на статью приведенной в начале данного материала. 

  

## C

  

test.c:

  


```cpp
#include "test.h"

int a = 5;
double b = 5.12345;
char c = 'X';

int 
func_ret_int(int val) { 
    return val;
} 

double 
func_ret_double(double val) { 
    return val;
} 

char *
func_ret_str(char *val) { 
    return val;
} 

char
func_many_args(int val1, double val2, char val3, short val4) { 
    return val3;
} 

test_st_t *
func_ret_struct(test_st_t *test_st) {         
    return test_st;
} 

```
  

test.h:

  


```cpp
#ifndef _TEST_H_
#define _TEST_H_

#ifdef  __cplusplus
extern "C" {
#endif

typedef struct test_st_s test_st_t;

extern int a;
extern double b;
extern char c;

int func_ret_int(int val);
double func_ret_double(double val);
char *func_ret_str(char *val);
char func_many_args(int val1, double val2, char val3, short val4);
test_st_t *func_ret_struct(test_st_t *test_st);

struct test_st_s {
    int val1;
    double val2;
    char val3;
};

#ifdef  __cplusplus
}
#endif

#endif  /* _TEST_H_ */
```
  

Процесс компиляции претерпел изменения, теперь нам понадобится **arm-linux-gnueabi-gcc**.

  

Как компилировать :

  

#### linux-gnu-gcc

  


```bash
aarch64-linux-gnu-gcc -c -Wl,-hash-style=sysv -g -O2 -march=armv8-a -fPIC -I./src/c  -o ./objs/test.o ./src/c/test.c
aarch64-linux-gnu-gcc -Wl,-hash-style=sysv -g -O2 -march=armv8-a -shared -o ./objs/libtest.so ./objs/test.o
aarch64-linux-gnu-strip --strip-unneeded ./objs/libtestpp.so
```
  

Флаг **-march=armv8-a** определяет архитектуру. Но особую важность представляют собой два флага **-Wl, -hash-style=sysv**, дело в том что компилятор по умолчанию компилирует с флагом **-hash-style=gnu**. Но такой формат hash таблицы не нравится android:

  


```bash
I/python  (13157): dlopen failed: empty/missing DT_HASH in "libtest.so" (built with --hash-style=gnu?)
```
  

Без **Wl** тоже ничего работать не будет, т.к. компоновщик соберет все с -hash-style=gnu. Пишут, что еще подойдет **hash-style=both**, но я не проверял.

  

Для архитектуры **armv7-a** компилятор **arm-linux-gnueabi-gcc**. 

  

При использовании aarch64-linux-gnu-gcc/arm-linux-gnueabi-gcc **нельзя** использовать **printf**, в разделе проблемы написано почему.

  

#### clang

  


```bash
$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/clang -c -target aarch64-linux-android21 -fPIC -I./src/c  -o ./objs/test.o ./src/c/test.c
$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/clang -target aarch64-linux-android21 -shared -o ./objs/libtest.so ./objs/test.o
$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip --strip-unneeded ./objs/libtest.so
```
  

*$ANDROID\_NDK* использую от **buildozer**-а:

  


```bash
~/.buildozer/android/platform/android-ndk-r19c/
```
  

На этом работа с библиотекой на C закончена. 

  

## C++

  

test.cpp:

  


```cpp
#include "test.hpp"

/**
 * Методы класса
 **/
std::string test::ret_str(std::string val) {
    std::cout << "C get ret_str: " << val << std::endl;
    return val;
}

int test::ret_int(int val) {
    std::cout << "C get ret_int: " << val << std::endl;
    return val;
}

double test::ret_double(double val) {
    std::cout << "C get ret_double: " << val << std::endl;
    return val;
}

/**
 * Обвязка C для методов класса C++
 **/

// Создаем класс test, и получаем указатель на него.
test *test_new() {
    return new test();
}

// Удаляем класс test.
void test_del(test *test) {
    delete test;
}

/*
 * Вызов методов класса.
 */

// Обертка над методом ret_str
char *test_ret_str(test *test, char *val) {
    // char * к std::string
    std::string str = test->ret_str(std::string(val));

    // std::string к char *
    char *ret = new char[str.length() + 1];
    strcpy(ret, str.c_str());

    return ret;
}

// Обертка над методом ret_int
int test_ret_int(test *test, int val) {
    return test->ret_int(val);
}

// Обертка над методом ret_double
double test_ret_double(test *test, double val) {
    return test->ret_double(val);
}

/*
 * Получение переменных класса.
 */

// Обертка для получения a
int test_get_a(test *test) {
    return test->a;
}

// Обертка для получения b
double test_get_b(test *test) {
    return test->b;
}

// Обертка для получения c
char test_get_c(test *test) {
    return test->c;
}

```
  

test.hpp:

  


```cpp
#ifndef _TEST_HPP_
#define _TEST_HPP_

#include <iostream>
#include <string>

class test {
public:
    int a = 5;
    double b = 5.12345;
    char c = 'X';

    std::string ret_str(std::string val);
    int ret_int(int val);
    double ret_double(double val);
};

#ifdef __cplusplus
extern "C" {
#endif

    test *test_new();
    void test_del(test *test);
    char *test_ret_str(test *test, char *val);
    int test_ret_int(test *test, int val);
    double test_ret_double(test *test, double val);

    int test_get_a(test *test);
    double test_get_b(test *test);
    char test_get_c(test *test);

#ifdef __cplusplus
}
#endif

#endif
```
  

Как компилировать :

  

#### clang++

  


```bash
$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/clang++ -c -target aarch64-linux-android21 -std=c++17 -stdlib=libc++ -fPIC -I./src/c  -o ./objs/test.pp.o ./src/c/test.cpp
$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/clang++ -target aarch64-linux-android21 -std=c++17 -stdlib=libc++ -shared -o ./objs/libtestpp.so ./objs/test.pp.o
$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-strip --strip-unneeded ./objs/libtestpp.so
```
  

Для С++ понадобится [libc++\_shared.so](https://github.com/urho3d/android-ndk/tree/master/sources/cxx-stl/llvm-libc%2B%2B/libs), она уже скомпилирована для нас.

  

Вот и все, не забудем скопировать **.so** файлы в папку проекта kivy.

  

## Проблемы С/C++

  

* На aarch64-linux-gnu-gcc/arm-linux-gnueabi-gсс ругается на **printf**, для его работы пересобрал **libc.so.6**. Но все равно не заработало из-за зависимости на **ld-linux-aarch64.so.1**, которую **ни как** не смог подключить к проекту.
* Ни как не получилось заставить **C++** работать на aarch64-linux-gnu-g++/arm-linux-gnueabi-g++, только на **clang++** из **NDK**. Из-за того что:

  


```bash
I/python  (13637):  OSError: dlopen failed: empty/missing DT_HASH in "libstdc++.so.6" (built with --hash-style=gnu?)
```
  

libstdc++.so.6 не стал пытаться собирать после неудачи с libc.so.6 и т.к. на clang++ заработало.

  

## GLIBC

  

Как собрать glibc с помощью aarch64-linux-gnu-gcc/g++ под armv8-a. Мне это ни как не помогло, но опишу процесс на всякий случай т.к. потратил на него много времени.

  


```bash
git clone git://sourceware.org/git/glibc.git
cd glibc
mkdir build
cd build
export glibc_install="$(pwd)/install"
LDFLAGS="-Wl,-hash-style=sysv -march=armv8-a" ../configure --host=aarch64-linux-gnu --prefix "$glibc_install"
make -j 4
make install -j 4
```
  

Наши библиотеки в: **glibc/build/install/lib**

  

## Python

  

Здесь нам понадобится фреймворк **Kivy** и **buildozer** — утилита для создания apk пакетов. Установку и настройку проводил по статье: [Kivy. Сборка пакетов под Android и никакой магии](https://habr.com/ru/post/479236/)

  

### Установка kivy & buildozer

  


```bash
sudo pip3 install kivy
sudo pip3 install buildozer
```
  

Установив kivy и buildozer приступим к созданию тестовой программы. Создадим папку под нее:

  


```bash
mkdir android_python
cd android_python
```
  

Теперь создадим **main.py**, это точка запуска нашей программы.

  


```bash
touch main.py
```
  

И заполним его:

  


```python
#!/usr/bin/python3
#-*- coding: utf-8 -*-

import os
import sys
import ctypes

import kivy
kivy.require("1.9.1")
from kivy.app import App
from kivy.uix.button import Button

# class in which we are creating the button
class ButtonApp(App):

    def build(self):
        # use a (r, g, b, a) tuple
        btn = Button(text ="Push Me !",
                   font_size ="20sp",
                   background_color = (1, 1, 1, 1),
                   color = (1, 1, 1, 1),
                   size_hint = (.2, .1),
                   pos_hint = {'x':.4, 'y':.45})

        # bind() use to bind the button to function callback
        btn.bind(on_press = self.callback)
        return btn

    # callback function tells when button pressed
    def callback(self, event):
        exit(0)

##
#  Старт.
##
if __name__ == "__main__":

    test = None
    # Загрузка библиотеки
    try:
        test = ctypes.CDLL('libtest.so')
    except OSError as e:
        print(str(e))
        exit(0)

    ###
    ## C
    ###

    print("ctypes\n")
    print("C\n")

    ##
    # Работа с функциями
    ##

    # Указываем, что функция возвращает int
    test.func_ret_int.restype = ctypes.c_int
    # Указываем, что функция принимает аргумент int
    test.func_ret_int.argtypes = [ctypes.c_int, ]

    # Указываем, что функция возвращает double
    test.func_ret_double.restype = ctypes.c_double
    # Указываем, что функция принимает аргумент double
    test.func_ret_double.argtypes = [ctypes.c_double]

    # Указываем, что функция возвращает char *
    test.func_ret_str.restype = ctypes.c_char_p
    # Указываем, что функция принимает аргумент char *
    test.func_ret_str.argtypes = [ctypes.POINTER(ctypes.c_char), ]

    # Указываем, что функция возвращает char
    test.func_many_args.restype = ctypes.c_char
    # Указываем, что функция принимает аргументы int, double. char, short
    test.func_many_args.argtypes = [ctypes.c_int, ctypes.c_double, ctypes.c_char, ctypes.c_short]

    print('Работа с функциями:')
    print('ret func_ret_int: ', test.func_ret_int(101))
    print('ret func_ret_double: ', test.func_ret_double(12.123456789))
    # Необходимо строку привести к массиву байтов, и массив байтов к строке.
    print('ret func_ret_str: ', test.func_ret_str('Hello!'.encode('utf-8')).decode("utf-8"))
    print('ret func_many_args: ', test.func_many_args(15, 18.1617, 'X'.encode('utf-8'), 32000).decode("utf-8"))

    ##
    # Работа с переменными
    ##

    print('\nРабота с переменными:')
    # Указываем, что переменная типа int
    a = ctypes.c_int.in_dll(test, "a")
    print('ret a: ', a.value)

    # Изменяем значение переменной.
    a.value = 22
    a = ctypes.c_int.in_dll(test, "a")
    print('new a: ', a.value)

    # Указываем, что переменная типа double
    b = ctypes.c_double.in_dll(test, "b")
    print('ret b: ', b.value)

    # Указываем, что переменная типа char
    c = ctypes.c_char.in_dll(test, "c")
    print('ret c: ', c.value.decode("utf-8"))

    ##
    # Работа со структурами
    ##

    print('\nРабота со структурами:')

    # Объявляем структуру в Python аналогичную в C
    class test_st_t(ctypes.Structure):
        _fields_ = [('val1', ctypes.c_int),
                    ('val2', ctypes.c_double),
                    ('val3', ctypes.c_char)]

    # Указываем, что функция возвращает test_st_t *
    test.func_ret_struct.restype = ctypes.POINTER(test_st_t)
    # Указываем, что функция принимает аргумент void *
    test.func_ret_struct.argtypes = [ctypes.c_void_p]

    # Создаем структуру
    test_st = test_st_t(19, 3.5, 'Z'.encode('utf-8'))

    # Python None == Null C
    # ret = test.func_ret_struct(None)
    # print('ret func_ret_struct: ', ret) # Если передали None, то его и получим назад
    ret = test.func_ret_struct(ctypes.byref(test_st))

    # Полученные данные из C
    print('ret val1 = {}\nret val2 = {}\nret val3 = {}'.format(ret.contents.val1, ret.contents.val2,
                                                               ret.contents.val3.decode("utf-8")))

    ###
    ## C++
    ###

    start_time = time.time()

    print("\n\nC++\n")

    # Загрузка библиотеки
    testpp = ctypes.CDLL('libs/libs_arm64_v8a/libtestpp.so')

    # Указываем, что функция возвращает указатель
    testpp.test_new.restype = ctypes.c_void_p
    # Создание класса test
    test = testpp.test_new()

    ##
    # Работа с методами
    ##

    # Указываем, что функция возвращает char *
    testpp.test_ret_str.restype = ctypes.c_char_p
    # Указываем, что функция принимает аргумент void * и char *
    testpp.test_ret_str.argtypes = [ctypes.c_void_p, ctypes.c_char_p]

    # Указываем, что функция возвращает int
    testpp.test_ret_int.restype = ctypes.c_int
    # Указываем, что функция принимает аргумент void * и int
    testpp.test_ret_int.argtypes = [ctypes.c_void_p, ctypes.c_int]

    # Указываем, что функция возвращает double
    testpp.test_ret_double.restype = ctypes.c_double
    # Указываем, что функция принимает аргумент void * и double
    testpp.test_ret_double.argtypes = [ctypes.c_void_p, ctypes.c_double]

    print('Работа с методами:')
    # В качестве 1-ого аргумента передаем указатель на наш класс
    print('ret test_ret_str: ', testpp.test_ret_str(test, 'Hello!'.encode('utf-8')).decode("utf-8"))
    print('ret test_ret_int: ', testpp.test_ret_int(test, 123))
    print('ret test_ret_double: ', testpp.test_ret_double(test, 9.87654321))

    ##
    # Работа с переменными
    ##

    # Указываем, что функция возвращает int
    testpp.test_get_a.restype = ctypes.c_int
    # Указываем, что функция принимает аргумент void *
    testpp.test_get_a.argtypes = [ctypes.c_void_p]
    # Указываем, что функция возвращает double
    testpp.test_get_b.restype = ctypes.c_double
    # Указываем, что функция принимает аргумент void *
    testpp.test_get_b.argtypes = [ctypes.c_void_p]
    # Указываем, что функция возвращает char
    testpp.test_get_c.restype = ctypes.c_char
    # Указываем, что функция принимает аргумент void *
    testpp.test_get_c.argtypes = [ctypes.c_void_p]

    print('\nРабота с переменными:')
    print('ret test_get_a: ', testpp.test_get_a(test))
    print('ret test_get_b: ', testpp.test_get_b(test))
    print('ret test_get_c: ', testpp.test_get_c(test).decode("utf-8"))

    # Указываем, что функция принимает аргумент void *
    testpp.test_del.argtypes = [ctypes.c_void_p]
    # Удаляем класс
    testpp.test_del(test)

    ButtonApp().run()
```
  

Здесь создается простая графическая программа с одной кнопкой при нажатии которой произойдет закрытие приложения. Основная задача статьи показать как запускать C библиотеки, результат работы увидим в консоли.

  

Так же нам понадобится файл спецификации buildozer, описывающий правила сборки пакета **apk**. Создаем его:

  


```bash
touch buildozer.spec
```
  

Заполняем:

  


```plaintext
[app]

# (str) Title of your application
title = KivyTest

# (str) Package name
package.name = kivy_test

# (str) Package domain (needed for android/ios packaging)
package.domain = com.heattheatr

# (str) Source code where the main.py live
source.dir = .

# (list) Source files to include (let empty to include all the files)
source.include_exts = py,png,jpg,jpeg,ttf,so,6

# (list) Application version
version = 0.0.1

# (list) Application requirements
# comma separated e.g. requirements = sqlite3,kivy
requirements = python3, kivy==2.0.0

# (str) Custom source folders for requirements
# Sets custom source for any requirements with recipes
# requirements.source.kivy = ../../kivy
#requirements.source.libtest = lib/libtest

# (str) Supported orientation (one of landscape, sensorLandscape, portrait or all)
orientation = portrait

# (bool) Indicate if the application should be fullscreen or not
fullscreen = 1

# (list) Permissions
android.permissions = INTERNET,WRITE_EXTERNAL_STORAGE

# (int) Target Android API, should be as high as possible.
android.api = 28

# (int) Minimum API your APK will support.
android.minapi = 21

# (str) Android NDK version to use
android.ndk = 19c

# (bool) If True, then skip trying to update the Android sdk
# This can be useful to avoid excess Internet downloads or save time
# when an update is due and you just want to test/build your package
android.skip_update = False

# (bool) If True, then automatically accept SDK license
# agreements. This is intended for automation only. If set to False,
# the default, you will be shown the license when first running
# buildozer.
android.accept_sdk_license = True

# (str) The Android arch to build for, choices: armeabi-v7a, arm64-v8a, x86, x86_64
android.arch = arm64-v8a

# (list) Android additionnal libraries to copy into libs/armeabi
android.add_libs_arm64_v8a = /home/djvu/workspace/c_from_python/ctypes/src/python/android/libs/libs_arm64_v8a/*.*

[buildozer]

# (int) Log level (0 = error only, 1 = info, 2 = debug (with command output))
log_level = 2

# (int) Display warning if buildozer is run as root (0 = False, 1 = True)
warn_on_root = 0

# (str) Path to build artifact storage, absolute or relative to spec file
build_dir = ./.buildozer

# (str) Path to build output (i.e. .apk, .ipa) storage
bin_dir = ./bin
```
  

Здесь выделю следующие поля:

  

* **source.include\_exts** — файлы с перечисленными расширениями будут включены в проект.
* **requirements** — список python модулей которые будут скачены через pip и включены в проект.
* **android.arch** — архитектура под которую производится сборка
* **android.add\_libs\_arm64\_v8a** — папка с библиотеками под конкретную архитектуру, которые будут включены в проект. Под каждую архитектуру свой ключ:  

	+ android.add\_libs\_armeabi = libs/android/\*.so
	+ android.add\_libs\_armeabi\_v7a = libs/android-v7/\*.so
	+ android.add\_libs\_arm64\_v8a = libs/android-v8/\*.so
	+ android.add\_libs\_x86 = libs/android-x86/\*.so
	+ android.add\_libs\_mips = libs/android-mips/\*.so

  

Данный buildozer.spec собирает приложения под архитектуру **arm64-v8a**.

  

Здесь у меня вопрос к **знающим людям**, с помощью **android.add\_libs\_arm64\_v8a = /home/djvu/workspace/c\_from\_python/ctypes/src/python/android/libs/libs\_arm64\_v8a/*.*** я ни как не смог добавить в apk пакет библиотеку **libstdc++.so.6**. Добавляются только с расширениями **so**. **Подскажите как это сделать?**

  

Теперь соберем apk пакет:

  


```bash
buildozer android debug
```
  

Операция очень долгая и растянется на несколько десятков минут. Так же потребуется порядка **1.5 GB** свободного места, т.к. buildozer подтянет все необходимые библиотеки для сборки.  

После удачного завершения в папке **bin** соберется пакет **kivy\_test-0.0.1-arm64-v8a-debug.apk**.

  

## Телефон

  

На телефоне нужно включить режим отладки по **USB** и разрешить установку через **USB**.  

Можно сбрасывать себе пакеты через **telegram** (так делал на телефоне со сломанным USB).

  

![main](https://habrastorage.org/r/w780q1/webt/53/dg/bj/53dgbjcchbcypis05uzqdyi7n58.jpeg)  

Установим на телефон через провод:

  


```bash
adb install -r ./bin/kivy_test-*.apk
```
  

Убедимся что пакет установился и все наши библиотеки внутри:

  


```bash
adb shell
run-as com.heattheatr.kivy_test
ls files/app/libs
```
  

![](https://habrastorage.org/r/w1560/webt/xh/hn/yr/xhhnyrwgqaiqtjz_wmayytjs1u8.png)

  

Находим приложение на телефоне:

  

![main](https://habrastorage.org/r/w780q1/webt/rw/9v/wm/rw9vwmh-reotpkzhj9yjsw1hkpi.jpeg)  

Подключаемся к консоли телефона и мониторим работу приложения:

  


```bash
adb logcat | grep python
```
  

Запускаем и получаем следующее — **C/C++** отработал без проблем:

  


```bash
I/python  (15497): ctypes
I/python  (15497): C
I/python  (15497): Работа с функциями:
I/python  (15497): ret func_ret_int:  101
I/python  (15497): ret func_ret_double:  12.123456789
I/python  (15497): ret func_ret_str:  Hello!
I/python  (15497): ret func_many_args:  X
I/python  (15497): Работа с переменными:
I/python  (15497): ret a:  5
I/python  (15497): new a:  22
I/python  (15497): ret b:  5.12345
I/python  (15497): ret c:  X
I/python  (15497): Работа со структурами:
I/python  (15497): ret val1 = 19
I/python  (15497): ret val2 = 3.5
I/python  (15497): ret val3 = Z
I/python  (15497): C++
I/python  (15497): Работа с методами:
I/python  (15497): ret test_ret_str:  Hello!
I/python  (15497): ret test_ret_int:  123
I/python  (15497): ret test_ret_double:  9.87654321
I/python  (15497): Работа с переменными:
I/python  (15497): ret test_get_a:  5
I/python  (15497): ret test_get_b:  5.12345
I/python  (15497): ret test_get_c:  X
```
  

На экране телефона видим следующую картинку:

  

![main](https://habrastorage.org/r/w780q1/webt/dv/xg/qj/dvxgqjdrunzpibh8ormbrhbe6l4.jpeg)  

Все отработало как надо. Нажатие на кнопку закрывает приложение. Больше оно ни чего не умеет делать ;)

  

Спасибо за внимание.

  

## Ссылки

  

* [Исходные коды примеров](https://github.com/dvjdjvu/c_from_python)
* C/C++ из Python [**ctypes**](https://habr.com/ru/post/466499/)
* [Kivy. Сборка пакетов под Android и никакой магии](https://habr.com/ru/post/479236/)
   