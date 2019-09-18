<img alt="main" width="500" src="https://habrastorage.org/webt/up/e1/2v/upe12vokda_crnab9gfx9bv-qyu.jpeg" align = "center"/>
Продолжаем тему как вызывать **C** из **python**. Теперь ипользуем библиотеки **cffi**, **pybind11** и ** Boost.Python**. Способ через [**ctypes**](https://habr.com/ru/post/466499/) был уже ранее рассмотрен.
<cut/>

## C
Тестовая библиотека для демонстрации работы с глобальными переменными, структурами и функциями с аргументами различных типов.
test.h
```c++
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
```
test.c
```c++
#include <stdio.h>
#include <stdlib.h>
#include "test.h"

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

char *
func_ret_str(char *val) { 
    printf("C get func_ret_str: %s\n", val);
    return val;
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

```
Как компилировать :
```
gcc -fPIC -shared -o libtest.so test.c
```

Библиотека точно такая же как в статье про **ctypes**.

Теперь перейдем к библиотекам с помощью которых мы будем ее вызывать в **python**.

## CFFI

Для эксперимента использовалась версия **1.12.3**, почитать про неё можно [здесь](https://cffi.readthedocs.io/en/latest/). 
Немного об этой библиотеки в 2-х словах, CFFI генерирует поверх нашей библиотеки свою обвязку и компилирует её в библиотеку с которой мы и будем работать.

### Установка
**pip3 install cffi**

### Сборка 
Нужно написать скрипт сборки, который будет собирать обвязку вокруг нашей библиотеки.

build.py
```python
import os
import cffi

if __name__ == "__main__":
    ffi = cffi.FFI()
    # Путь расположение скрипта
    PATH = os.getcwd()
    
    # test.h заголовочный файл нашей библиотеки
    # указываем путь до него относительно build.py
    with open(os.path.join(PATH, "src/c/test.h")) as f:
        ffi.cdef(f.read())
    
    ffi.set_source("_test", # имя библиотеки собранной cffi, добавляем префикс _
        # Подключаем test.h, указываем путь относительно собираемой _test
        '#include "../src/c/test.h"',
        # Где libtest.so относительно _test.cpython-36m-x86_64-linux-gnu.so
        # Исходная собранная библиотека
        libraries=[os.path.join(PATH, "lib/test"), "./test"],
        library_dirs=[PATH, 'objs/'],
    )

    # компилируем _test в папку lib
    ffi.compile(tmpdir='./lib')
```
### python
Пример работы с **C** из **python** через **cffi**:
```python
from cffi import FFI
import sys
import time

# пути до модуля _test
sys.path.append('.')
sys.path.append('lib/')
sys.path.append('../../lib/')

# подключаем модуль скомпелированный cffi
import _test

###
## C
###

print("CFFI\n")
print("C\n")

start_time = time.time()

##
# Работа с функциями
##

print('Работа с функциями:')
print('ret func_ret_int: ', _test.lib.func_ret_int(101))
print('ret func_ret_double: ', _test.lib.func_ret_double(12.123456789))
# Необходимо строку привести из cdata к массиву байтов, и массив байтов к строке.
print('ret func_ret_str: ', _test.ffi.string(_test.lib.func_ret_str('Hello!'.encode('utf-8'))).decode("utf-8"))
print('ret func_many_args: ', _test.lib.func_many_args(15, 18.1617, 'X'.encode('utf-8'), 32000).decode("utf-8"))

##
# Работа с переменными
##

print('\nРабота с переменными:')
print('ret a: ', _test.lib.a)

# Изменяем значение переменной.
_test.lib.a = 22
print('new a: ', _test.lib.a)

print('ret b: ', _test.lib.b)

print('ret c: ', _test.lib.c.decode("utf-8"))

##
# Работа со структурами
##

print('\nРабота со структурами:')

# Создаем структуру и заполняем её
test_st = _test.ffi.new("test_st_t *")
test_st.val1 = 5
test_st.val2 = 5.1234567
test_st.val3 = 'Z'.encode('utf-8')

ret = _test.lib.func_ret_struct(test_st)

# Полученные данные из C
print('ret val1 = {}\nret val2 = {}\nret val3 = {}'.format(ret.val1, ret.val2, ret.val3.decode("utf-8")))

# Время работы
print("--- %s seconds ---" % (time.time() - start_time))
```

### Плюсы и минусы CFFI
**Плюсы**:
- простой синтаксис при использовании в python

**Минусы**:
- не удобная сборка, нужно прописывать пути до всех заголовочных файлов и библиотек. 
- создается еще 1-на динамическая библиотека, которая вызывает исходную. Так же библиотеке обвязке необходимо знать путь до исходной.
- не поддерживает директивы: 
```
#ifdef	__cplusplus
extern "C" {
#endif
...
#ifdef	__cplusplus
}
#endif
```
```
#ifndef _TEST_H_
#define	_TEST_H_
...
#endif	/* _TEST_H_ */
```

## pybind11

Для эксперимента использовалась версия **2.3.0**, почитать про неё можно [здесь](https://pybind11.readthedocs.io/en/stable/). 
Разрабатывалась для C++, но я пока приведу простые примеры сведенный к C функционалу.

### Установка
**pip3 install pybind11**
или
**git clone https://github.com/pybind/pybind11.git**
**python3 setup.py build**
**python3 setup.py install**

### Сборка 
Нужно написать скрипт сборки нашей библиотеки.
build.py
```python
import pybind11
from distutils.core import setup, Extension

ext_modules = [
    Extension(
        '_test', # имя библиотеки собранной pybind11
        ['src/c/test.cpp'], # Тестовый файлик который компилируем
        include_dirs=[pybind11.get_include()],  # не забываем добавить инклюды pybind11
        language='c++', # Указываем язык
        extra_compile_args=['-std=c++11'], # флаг с++11
    ),
]

setup(
    name='_test', # имя библиотеки собранной pybind11
    version='1.0.0',
    author='djvu',
    author_email='djvu@inbox.ru',
    description='pybind11 extension',
    ext_modules=ext_modules,
    requires=['pybind11'],  # Указываем зависимость от pybind11
    package_dir = {'': 'lib'}
)
```
Выполняем его:
```
python3 setup.py build --build-lib=./lib
```

### C
В исходники библиотеки притерпят изменения, в них нужно добавить:
- заголовочный файл **pybind11**
```
#include <pybind11/pybind11.h>
```
- макрос, который поволяет нам определить python модуль
```
PYBIND11_MODULE(_test, m)
```
Пример макроса для тестовой библиотеки:
```c++
namespace py = pybind11;

// _test имя нашего модуля
PYBIND11_MODULE(_test, m) {
    /*
     * Функции библиотеки
     */
    
    m.def("func_ret_int", &func_ret_int);
    m.def("func_ret_double", &func_ret_double);
    m.def("func_ret_str", &func_ret_str);
    m.def("func_many_args", &func_many_args);
    m.def("func_ret_struct", &func_ret_struct);
    
    /*
     * Глобальные переменные библиотеки
     */
    
    m.attr("a") = a;
    m.attr("b") = b;
    m.attr("c") = c;
    
    /*
     * Структуры
     */
    
    
    py::class_<test_st_t>(m, "test_st_t")
    .def(py::init()) // Указываем конструктор. Которого у структуры нет, но он нужен python
    .def_readwrite("val1", &test_st_t::val1) // переменные структуры
    .def_readwrite("val2", &test_st_t::val2)
    .def_readwrite("val3", &test_st_t::val3); 
};
```


### python
Пример работы с **C** из **python** через **cffi**:
```python
import sys
import time

# Пути до модуля _test
sys.path.append('lib/')

# подключаем модуль скомпелированный pybind11
import _test

###
## C
###

print("pybind11\n")
print("C\n")

start_time = time.time()

##
# Работа с функциями
##

print('Работа с функциями:')
print('ret func_ret_int: ', _test.func_ret_int(101))
print('ret func_ret_double: ', _test.func_ret_double(12.123456789))
# Необходимо строку привести из cdata к массиву байтов, и массив байтов к строке.
print('ret func_ret_str: ', _test.func_ret_str('Hello!'.encode('utf-8')))
print('ret func_many_args: ', _test.func_many_args(15, 18.1617, 'X'.encode('utf-8'), 32000))

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
_test_st = _test.test_st_t()
#print(dir(_test_st))
_test_st.val1 = 5
_test_st.val2 = 5.1234567
_test_st.val3 = 'Z'.encode('utf-8')

ret = _test.func_ret_struct(_test_st)

# Полученные данные из C
print('ret val1 = {}\nret val2 = {}\nret val3 = {}'.format(ret.val1, ret.val2, ret.val3))

# Время работы
print("--- %s seconds ---" % (time.time() - start_time))

```

### Плюсы и минусы pybind11
**Плюсы**:
- простой синтаксис при использовании в python

**Минусы**:
- Необходимо править искомые исходники, добавляя макрос для определения python модуля

