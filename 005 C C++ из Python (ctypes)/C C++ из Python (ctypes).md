 ![main](https://habrastorage.org/r/w780q1/webt/g9/gg/in/g9gginb-_l36uiheozsslmmfocw.jpeg)  

Про то как вызывать [Python из C](https://habr.com/ru/post/466181/) написал в прошлой статье, теперь поговорим как делать наоборот и вызывать **C/C++** из **Python3**. Раз начал писать об этом, то раскроем всю тему до конца. Тем более, что ни чего сложного здесь нет тоже.

  

## C

  

Здесь все просто, Python умеет вызывать C функции без каких либо проблем.

  

test.c:

  


```cpp
#include "test.h"

int a = 5;
double b = 5.12345;
char c = 'X';

int 
func_ret_int(int val) { 
    printf("get func_ret_int: %d\n", val);
    return val;
} 

double 
func_ret_double(double val) { 
    printf("get func_ret_double: %f\n", val);
    return val;
} 

char *
func_ret_str(char *val) { 
    printf("get func_ret_str: %s\n", val);
    return val;
} 

char
func_many_args(int val1, double val2, char val3, short val4) { 
    printf("get func_many_args: int - %d, double - %f, char - %c, short - %d\n", val1, val2, val3, val4);
    return val3;
} 
```
  

test.h:

  


```cpp
#ifndef _TEST_H_
#define _TEST_H_

#ifdef  __cplusplus
extern "C" {
#endif

#include <stdio.h>
#include <string.h>
#include <unistd.h>

int func_ret_int(int val);
double func_ret_double(double val);
char *func_ret_str(char *val);
char func_many_args(int val1, double val2, char val3, short val4);

#ifdef  __cplusplus
}
#endif

#endif  /* _TEST_H_ */
```
  

Как компилировать :

  


```plaintext
gcc -fPIC -shared -o libtest.so test.c
```
  

Исходник компилируется в динамическую библиотеку и готов к бою.  

Переходим к Python. В примере показывается как передать аргументы функции, получить результат работы от функции, а так же как получить и изменить значения глобальных переменных.

  

main.py:

  


```python
#!/usr/bin/python3
#-*- coding: utf-8 -*-

import ctypes 

# Загрузка библиотеки
test = ctypes.CDLL('./objs/libtest.so')

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

print('ret func_ret_int: ', test.func_ret_int(101))
print('ret func_ret_double: ', test.func_ret_double(12.123456789))

# Необходимо строку привести к массиву байтов, затем полученный массив байтов приводим к строке.
print('ret func_ret_str: ', test.func_ret_str('Hello!'.encode('utf-8')).decode("utf-8") )

print('ret func_many_args: ', test.func_many_args(15, 18.1617, 'X'.encode('utf-8'), 32000).decode("utf-8"))

print()

##
# Работа с переменными
##

# Указываем, что переменная типа int
a = ctypes.c_int.in_dll(test, "a")
print('ret a: ', a.value)

# Изменяем значение переменной.
a.value = 22
a = ctypes.c_int.in_dll(test, "a")
print('ret a: ', a.value)

# Указываем, что переменная типа double
b = ctypes.c_double.in_dll(test, "b")
print('ret b: ', b.value)

# Указываем, что переменная типа char
c = ctypes.c_char.in_dll(test, "c")
print('ret c: ', c.value.decode("utf-8"))
```
  

Все возможные типы данных и их обозначения можно посмотреть в [документации](https://docs.python.org/3/library/ctypes.html) Python.

  

#### Работа со структурами

  

C — объявление структуры в test.h:

  


```cpp
typedef struct test_st_s test_st_t;

struct test_st_s {
    int val1;
    double val2;
    char val3;
};
```
  

Функция по работе с нашей структурой:

  


```cpp
test_st_t *
func_ret_struct(test_st_t *test_st) {     
    if (test_st) {
        printf("C get test_st: val1 - %d, val2 - %f, val3 - %c\n", test_st->val1, test_st->val2, test_st->val3);
    }

    return test_st;
} 

```
  

Python:

  


```python
import sys
import struct

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
ret = test.func_ret_struct(None)
print('ret func_ret_struct: ', ret) # Если передали None, то его и получим назад
ret = test.func_ret_struct(ctypes.byref(test_st))

# Полученные данные из C
print('ret val1 = {}\nret val2 = {}\nret val3 = {}'.format(ret.contents.val1, ret.contents.val2, ret.contents.val3.decode("utf-8")))
```
  

## C++

  

Здесь немного сложнее, т.к. **ctypes** может только работать с **C** функциями. Это для нас не проблема, просто C обвяжем код C++.

  

Методы класса C++ и обвязка на C:

  


```cpp
#include "test.hpp"

/*
 * Методы класса
 */
std::string test::ret_str(std::string val) {
    std::cout << "get ret_str: " << val << std::endl;
    return val;
}

int test::ret_int(int val) {
    std::cout << "get ret_int: " << val << std::endl;
    return val;
}

double test::ret_double(double val) {
    std::cout << "get ret_double: " << val << std::endl;
    return val;
}

/*
 * Обвязка C для методов класса C++
 */

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
  

Но есть один нюанс, обвязку надо объявить как **extern C**. Чтобы ++ компилятор не перегрузил имена функций обвязки. Если он это сделает, то мы не сможем через ctypes работать с нашими функциями.  

test.hpp:

  


```cpp
#include <iostream>
#include <string.h>

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
```
  

Как компилировать :

  


```bash
g++ -fPIC -shared -o libtestpp.so test.cpp 
```
  

С Python все так же просто.

  


```python
# Загрузка библиотеки
testpp = ctypes.CDLL('./objs/libtestpp.so')

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
```
  

Для подключения **C/C++** дополнительных библиотек к **python**-у:

  


```python
ctypes.cdll.LoadLibrary('/abs/path/to/a.so')
ctypes.cdll.LoadLibrary('/abs/path/to/b.so')
```
  

Необходимо, если библиотека ./objs/libtestpp.so использует внутри себя функции из сторонних библиотек, например **a.so** & **b.so**. Без их подключения интерпретатор не знал бы где их искать.

  

Аналог подключения библиотек в **gcc**.

  


```bash
gcc -L/abs/path/to/ -la -lb test.c -o test
```
  

### Плюсы и минусы ctypes

  

**Плюсы**:

  

* Можно подключать любую уже скомпилированную **C** библиотеку

  

**Минусы**:

  

* В **Python** нужно описывать, что **C** функции возвращают и принимают в качестве аргументов.

  

Код постарался закомментировать понятно, чтобы здесь писать поменьше )

  

Надеюсь будет полезно.

  

## Благодарность

  

[DollaR84](https://habr.com/ru/users/DollaR84/) за его помощь.  

[Palich239](https://habr.com/ru/users/Palich239/) за найденные ошибки.

  

## Ссылки

  

* [Исходные коды примеров](https://github.com/dvjdjvu/c_from_python)
* Способ через [**CFFI, pybind11**](https://habr.com/ru/post/468099/)
* Способ через [**C API**](https://habr.com/ru/post/469043/)
* [Python из C](https://habr.com/ru/post/466181/)
   