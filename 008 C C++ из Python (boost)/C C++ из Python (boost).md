 ![main](https://habrastorage.org/r/w780q1/webt/nf/o7/xa/nfo7xaba24kf94ay5hsqahnow2y.jpeg)  

Заключительная статья из серии как вызывать **C/C++** из **Python3**, перебрал все известные способы как можно это сделать. На этот раз добрался до [**boost**](https://www.boost.org/doc/libs/1_66_0/libs/python/doc/html/index.html). Что из этого вышло читаем ниже.

  

## C

  

За основу беру один и тот же пример тестовой библиотеки и делаю его вариацию для конкретного способа. Тестовая библиотека для демонстрации работы с глобальными переменными, структурами и функциями с аргументами различных типов.

  

test.c:

  


```cpp
#include "test.hpp"

int a = 5;
double b = 5.12345;
char c = 'X';

int
func_ret_int(int val) {
    printf("C get func_ret_int: %d\n", val);
    return val;
}

double
func_ret_double(double val) {
    printf("C get func_ret_double: %f\n", val);
    return val;
}

object
func_ret_str(char *val) {
    printf("C get func_ret_str: %s\n", val);

    return object(string(val));
}

char
func_many_args(int val1, double val2, char val3, short val4) {
    printf("C get func_many_args: int - %d, double - %f, char - %c, short - %d\n", val1, val2, val3, val4);
    return val3;
}

test_st_t *
func_ret_struct(test_st_t *test_st) {
    if (test_st) {
        printf("C get test_st: val1 - %d, val2 - %f, val3 - %c\n", test_st->val1, test_st->val2, test_st->val3);
    }

    return test_st;
}

// _test имя нашего модуля
BOOST_PYTHON_MODULE(_test) {

    /*
     * Функции библиотеки
     */

    def("func_ret_int", func_ret_int);
    def("func_ret_double", func_ret_double);
    def("func_ret_str", &func_ret_str);
    def("func_many_args", func_many_args);

    // Очень важно
    // manage_new_object C функция возвращает новый объект
    // reference_existing_object C функция возвращает существующий объект
    def("func_ret_struct", &func_ret_struct, return_value_policy<reference_existing_object>());

    /*
     * Глобальные переменные библиотеки
     */
    scope().attr("a") = a;
    scope().attr("b") = b;
    scope().attr("c") = c;

    /*
     * Структуры
     */
    class_<test_st_t>("test_st_t")
        .def_readwrite("val1", &test_st_t::val1)
        .def_readwrite("val2", &test_st_t::val2)
        .def_readwrite("val3", &test_st_t::val3)
    ;

}

```
  

test.h:

  


```cpp
using namespace boost::python;
using namespace std;

#ifdef  __cplusplus
extern "C" {
#endif

typedef struct test_st_s test_st_t;
typedef char * char_p;

extern int a;
extern double b;
extern char c;

int func_ret_int(int val);
double func_ret_double(double val);

object func_ret_str(char *val);
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
```
  

Как компилировать :

  


```plaintext
g++ -g -fPIC -I/usr/include/python3.6 -I./src/c  -o ./objs/test.o -c ./src/c/test.cpp
g++ -fPIC -g -shared -o ./lib/_test.so ./objs/test.o  -lboost_python3
```
  

Исходник компилируется в динамическую библиотеку.  

**python boost** похож в использовании на **pybind11**, так же нужно описать функции которые будут видны python. Но по моим ощущения boost более громоздкий и сложный. Например:

  


```cpp
def("func_ret_struct", &func_ret_struct, return_value_policy<reference_existing_object>());
```
  

Функция **func\_ret\_struct** принимает в качестве аргумента указатель на структуру и возвращает этот же указатель назад. Для нее нужно указать правила возвращаемого объекта **return\_value\_policy<reference\_existing\_object>()**. reference\_existing\_objec говорит, что возвращаемый объект уже существовал. Если указать manage\_new\_object, то это будет значить, что мы возвращаем новый объект. В таком случае вот такой скрипт упадет в **segmentation fault** на сборщике мусора:

  


```python
test_st = _test.test_st_t()

ret = _test.func_ret_struct(test_st)
```
  

Потому что сборщик мусора сначала очистит данные которые содержит test\_st, а потом захочет очистить данные которые содержит объект ret. Который содержит те же самые данные, что содержал test\_st, но они уже были очищены.

  

Интересно как в таком случае описать вот такую функцию(не стал углубляться)?:

  


```cpp
test_st_t *
func_ret_struct(test_st_t *test_st) {
    if (test_st) {
        return test_st;
    } else {
        return (test_st_t *) malloc(sizeof(test_st_t)); 
    }
}
```
  

Такая функция может вернуть как уже существующий объект, так и существовавший. 

  

Так же у меня возникла проблема с такой функцией:

  


```cpp
char *
func_ret_str(char *val) {
    return val;
}
```
  

Как я понял, получить из boost указатель в python на стандартный тип данных нельзя. Можно только на **struct**, **class** и **union**. Если кто знает способ просветите.

  

## Python

  

Для python модуль становится родным.  

main.py:

  


```python
#!/usr/bin/python3
#-*- coding: utf-8 -*-

import sys
import time

# Пути до модуля test
#sys.path.append('.')
sys.path.append('lib/')

# подключаем модуль
import _test

###
## C
###

print("boost\n")
print("C\n")

start_time = time.time()

##
# Работа с функциями
##

print('Работа с функциями:')
print('ret func_ret_int: ', _test.func_ret_int(101))
print('ret func_ret_double: ', _test.func_ret_double(12.123456789))
print('ret func_ret_str: ', _test.func_ret_str('Hello!'))
print('ret func_many_args: ', _test.func_many_args(15, 18.1617, 'X', 32000))

##
# Работа с переменными
##

print('\nРабота с переменными:')
print('ret a: ', _test.a)

# Изменяем значение переменной.
_test.a = 22
print('new a: ', _test.a)

print('ret b: ', _test.b)

print('ret c: ', _test.c)

##
# Работа со структурами
##

print('\nРабота со структурами:')

# Создаем структуру и заполняем её
test_st = _test.test_st_t()
test_st.val1 = 5
test_st.val2 = 5.1234567
test_st.val3 = 'Z'

print('val1 = {}\nval2 = {}\nval3 = {}'.format(test_st.val1, test_st.val2, test_st.val3))

ret = _test.func_ret_struct(test_st)

# Полученные данные из C
print('ret val1 = {}\nret val2 = {}\nret val3 = {}'.format(ret.val1, ret.val2, ret.val3))

# Время работы
print("--- {} seconds ---".format(time.time() - start_time))

```
  

### Плюсы и минусы boost

  

**Плюсы**:

  

* простой синтаксис при использовании в Python

  

**Минусы**:

  

* необходимо править C++ исходники, или писать обвязку для них
* boost сам по себе не простой

  

Код как обычно стараюсь комментировать понятно.

  

## Среднее время выполнения теста на каждом способе при 1000 запусках:

  

* ctypes: — 0.0004987692832946777 seconds ---
* CFFI: — 0.00038521790504455566 seconds ---
* pybind: — 0.0004547207355499268 seconds ---
* C API: — 0.0003561973571777344 seconds ---
* boost: — 0.00037789344787597656 seconds ---

  

## Ссылки

  

* [Исходные коды примеров](https://github.com/dvjdjvu/c_from_python)
* Способ через [**ctypes**](https://habr.com/ru/post/466499/)
* Способ через [**CFFI, pybind11**](https://habr.com/ru/post/468099/)
* Способ через [**C API**](https://habr.com/ru/post/469043/)
* [Python из C](https://habr.com/ru/post/466181/)
   