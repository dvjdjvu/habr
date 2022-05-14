 ![main](https://habrastorage.org/r/w780q1/webt/fp/ig/sb/fpigsboyg1gykkbxi1sm7wwyv_q.jpeg)  

Продолжаем тему как вызывать **C/C++** из **Python3**. Теперь используем **C API** для создания модуля, на этом примере мы сможем разобраться как работает **cffi** и прочие библиотеки упрощающие нам жизнь. Потому что на мой взгляд это самый трудный способ.

  

## C

  

Тестовая библиотека для демонстрации работы с глобальными переменными, структурами и функциями с аргументами различных типов. В своих статьях использую вариации одной и той же библиотеки, в зависимости от способа которым я теперь пользуюсь. Ссылки на предыдущие способы внизу.  

test.h

  


```cpp
typedef struct test_st_s test_st_t;

extern int a;
extern double b;
extern char c;

static PyObject *func_hello(PyObject *self, PyObject *args);
static PyObject *func_ret_int(PyObject *self, PyObject *args);
static PyObject *func_ret_double(PyObject *self, PyObject *args);
static PyObject *func_ret_str(PyObject *self, PyObject *args);
static PyObject *func_many_args(PyObject *self, PyObject *args);
static PyObject *func_ret_struct(PyObject *self, PyObject *args);

struct test_st_s {
    PyObject_HEAD // Макрос объявления нового типа, объекта фиксированного размера
    int val1;
    double val2;
    char val3;
};
```
  

test.c

  


```cpp
// Список функций модуля
static PyMethodDef methods[] = {
    {"func_hello", func_hello, METH_NOARGS, "func_hello"}, // Функция без аргументов
    {"func_ret_int", func_ret_int, METH_VARARGS, "func_ret_int"}, // Функция с аргументами
    {"func_ret_double", func_ret_double, METH_VARARGS, "func_ret_double"},
    {"func_ret_str", func_ret_str, METH_VARARGS, "func_ret_str"},
    {"func_many_args", func_many_args, METH_VARARGS, "func_many_args"},
    {"func_ret_struct", func_ret_struct, METH_VARARGS, "func_ret_struct"},
    {NULL, NULL, 0, NULL}
};

// Описание модуля
static struct PyModuleDef module = {
    PyModuleDef_HEAD_INIT, "_test", "Test module", -1, methods
};

// Инициализация модуля
PyMODINIT_FUNC 
PyInit__test(void) {
    PyObject *mod = PyModule_Create(&module);

    // Добавляем глобальные переменные
    PyModule_AddObject(mod, "a", PyLong_FromLong(a)); // int
    PyModule_AddObject(mod, "b", PyFloat_FromDouble(b)); // double
    PyModule_AddObject(mod, "c", Py_BuildValue("b", c)); // char

    // Добавляем структуру

    // Завершение инициализации структуры
    if (PyType_Ready(&test_st_t_Type) < 0)
        return NULL;

    Py_INCREF(&test_st_t_Type);
    PyModule_AddObject(mod, "test_st_t", (PyObject *) &test_st_t_Type);

    return mod;
}

/**
 * Тестовые функции, тестовые переменные.
 */

int a = 5;
double b = 5.12345;
char c = 'X'; // 88

static PyObject *
func_hello(PyObject *self, PyObject *args) { // Можно без args, но будет warning при компиляции.
    puts("Hello!");
    Py_RETURN_NONE;
}

/**
 * Получение значения переменной содержащей значение типа int и возврат его.
 */
static PyObject *
func_ret_int(PyObject *self, PyObject *args) {
    int val;

    // Проверка кол-ва аргументов
    if (PyTuple_Size(args) != 1) {
        PyErr_SetString(self, "func_ret_int args error");
    }

    PyArg_ParseTuple(args, "i", &val);
    /* 
     * Альтернативный вариант.
     * 
    // Получаем аргумент
    PyObject *obj = PyTuple_GetItem(args, 0);
    // Проверяем его на тип int/long
    if (PyLong_Check(obj)) {
        PyErr_Print();
    }
    // Приводим (PyObject *) к int
    val = _PyLong_AsInt(obj);
     */
    printf("C get func_ret_int: %d\n", val);
    return Py_BuildValue("i", val);
}

/**
 * Получение значения переменной содержащей значение типа double и возврат его.
 */
static PyObject *
func_ret_double(PyObject *self, PyObject *args) {
    double val;

    if (PyTuple_Size(args) != 1) {
        PyErr_SetString(self, "func_ret_double args error");
    }

    PyArg_ParseTuple(args, "d", &val);

    printf("C get func_ret_double: %f\n", val);
    return Py_BuildValue("f", val);
}

/**
 * Получение string и возврат его.
 */
static PyObject *
func_ret_str(PyObject *self, PyObject *args) {
    char *val;

    if (PyTuple_Size(args) != 1) {
        PyErr_SetString(self, "func_ret_str args error");
    }

    PyArg_ParseTuple(args, "s", &val);
    /* 
     * Альтернативный вариант.
     * 
    PyObject *obj = PyTuple_GetItem(args, 0);

    PyObject* pResultRepr = PyObject_Repr(obj);
    val = PyBytes_AS_STRING(PyUnicode_AsEncodedString(pResultRepr, "utf-8", "ERROR"));
     */
    printf("C get func_ret_str: %s\n", val);
    return Py_BuildValue("s", val);
}

/**
 * Получение значения переменных содержащих значения типа int, double, char *.
 */
static PyObject *
func_many_args(PyObject *self, PyObject *args) {
    int val1;
    double val2;
    char *val3;

    if (PyTuple_Size(args) != 3) {
        PyErr_SetString(self, "func_ret_str args error");
    }

    PyArg_ParseTuple(args, "ids", &val1, &val2, &val3);

    printf("C get func_many_args: int - %d, double - %f, string - %s\n", val1, val2, val3);
    return Py_BuildValue("ifs", val1, val2, val3);
}

static PyObject *
func_ret_struct(PyObject *self, PyObject *args) {

    test_st_t *st;

    // Получаем структуру из Python
    if (!PyArg_ParseTuple(args, "O", &st)) // O - объект данных
        Py_RETURN_NONE;

    printf("C get test_st: val1 - %d, val2 - %f, val3 - %d\n", st->val1++, st->val2++, st->val3++);

    return Py_BuildValue("O", st);
}
```
  

Модулю требуется указать что он в себя будет включать: функции, глобальные переменные и структуры. Каждую такую вещь нужно описать, самое сложное для своих типов данных(структуры...) Примерно такой файл генерирует **cffi**.

  

Для работы необходимо подключить заголовочные файлы:

  


```cpp
#include <Python.h>
#include <structmember.h> // Для пользовательских типов данных
```
  

Флаги компиляции:

  


```plaintext
$(python3-config --includes --ldflags) -fPIC
```
  

За обработку аргументов отвечает следующая функция:

  


```cpp
PyArg_ParseTuple(args, "ids", &val1, &val2, &val3);
```
  

1-ым идет аргумент типа int, он имеет литерное обозначение **i**  

2-ым float/double — **d**  

3-им string — **s**  

Все возможные литерные обозначения типов данных можно посмотреть [здесь](https://docs.python.org/3/c-api/arg.html#c.Py_BuildValue)

  

Теперь перейдем к описанию как описать структуру.  

struct.c:

  


```cpp
// Освобождение структуры
static void
test_st_t_dealloc(test_st_t* self) {
    Py_TYPE(self)->tp_free((PyObject*)self);
}

// Создание структуры
static PyObject *
test_st_t_new(PyTypeObject *type, PyObject *args, PyObject *kwds) {
    test_st_t *self;

    self = (test_st_t *)type->tp_alloc(type, 0);
    if (self != NULL) {
        self->val1 = 0;
        self->val2 = 0.0;
        self->val3 = 0;
    }

    return (PyObject *)self;
}

// Инициализация структуры, заполняем её переданными значениями
static int
test_st_t_init(test_st_t *self, PyObject *args, PyObject *kwds) {
    static char *kwlist[] = {"val1", "val2", "val3", NULL};

    if (! PyArg_ParseTupleAndKeywords(args, kwds, "|idb", kwlist, &self->val1, &self->val2, &self->val3))
        return -1;

    return 0;
}

// Описываем аттрибуты из которых состоит структура
static PyMemberDef test_st_t_members[] = {
    {"val1", T_INT, offsetof(test_st_t, val1), 0, "int"},
    {"val2", T_DOUBLE, offsetof(test_st_t, val2), 0, "double"},
    {"val3", T_CHAR, offsetof(test_st_t, val3), 0, "char"},
    {NULL}
};

// Метод структуры, который печатает структуру
static PyObject* test_st_print(PyObject *self, PyObject *args)
{
    test_st_t *st;

    // Получаем структуру из Python
    if (!PyArg_ParseTuple(args, "O", &st)) // O - объект данных
        Py_RETURN_NONE;

    printf("method: val1 - %d, val2 - %f, val3 - %d\n", st->val1++, st->val2++, st->val3++);
    Py_RETURN_NONE;
}

// Описание методов стрктуры, но у классической структуры не может быть методов!
// А здесь может!
static PyMethodDef test_st_t_methods[] = {
    {"print", test_st_print, METH_VARARGS, "doc string"},
    {NULL}  /* Sentinel */
};

// Структура описывающая нашу структуру. Какие атрибуты, методы, конструкторы, деструкторы и т.д. и т.п.
PyTypeObject test_st_t_Type = {
    PyVarObject_HEAD_INIT(NULL, 0)
    "_test.test_st_t",         /* tp_name */
    sizeof(test_st_t),         /* tp_basicsize */
    0,                         /* tp_itemsize */
    (destructor) test_st_t_dealloc, /* tp_dealloc */
    0,                         /* tp_print */
    0,                         /* tp_getattr */
    0,                         /* tp_setattr */
    0,                         /* tp_reserved */
    0,                         /* tp_repr */
    0,                         /* tp_as_number */
    0,                         /* tp_as_sequence */
    0,                         /* tp_as_mapping */
    0,                         /* tp_hash  */
    0,                         /* tp_call */
    0,                         /* tp_str */
    0,                         /* tp_getattro */
    0,                         /* tp_setattro */
    0,                         /* tp_as_buffer */
    Py_TPFLAGS_DEFAULT |
        Py_TPFLAGS_BASETYPE,   /* tp_flags */
    "test_st_t objects",       /* tp_doc */
    0,                         /* tp_traverse */
    0,                         /* tp_clear */
    0,                         /* tp_richcompare */
    0,                         /* tp_weaklistoffset */
    0,                         /* tp_iter */
    0,                         /* tp_iternext */
    test_st_t_methods,         /* tp_methods */
    test_st_t_members,         /* tp_members */
    0,                         /* tp_getset */
    0,                         /* tp_base */
    0,                         /* tp_dict */
    0,                         /* tp_descr_get */
    0,                         /* tp_descr_set */
    0,                         /* tp_dictoffset */
    (initproc) test_st_t_init, /* tp_init */
    0,                         /* tp_alloc */
    test_st_t_new,             /* tp_new */
};
```
  

И это все для:

  


```cpp
struct test_st_s {
    PyObject_HEAD // Макрос объявления нового типа, объекта фиксированного размера
    int val1;
    double val2;
    char val3;
};
```
  

Согласитесь, не мало. Причем для структуры можно определять методы (в качестве примера **test\_st\_print**).  

В коде стараюсь делать больше комментариев, что бы меньше описывать отдельно.

  

### Python

  

Пример работы с **C** модулем из **Python**:

  


```python
import sys
import time

# пути до модуля _test
sys.path.append('.')
sys.path.append('lib/')
sys.path.append('../../lib/')

import _test 

###
## C
###

print("C API\n")
print("C\n")

start_time = time.time()

##
# Работа с функциями
##

print('Работа с функциями:')
print('ret func_hello: ', _test.func_hello())
print('ret func_ret_int: ', _test.func_ret_int(101))
print('ret func_ret_double: ', _test.func_ret_double(12.123456789))
print('ret func_ret_str: ', _test.func_ret_str('Hello!'))
print('ret func_many_args: ', _test.func_many_args(15, 18.1617, "Many arguments!"))

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

# Создаем структуру
st = _test.test_st_t(1, 2.3456789, 88)

print('st.val1 = {}\nst.val2 = {}\nst.val3 = {}'.format(st.val1, st.val2, st.val3))
st = _test.func_ret_struct(st)
print("ret func_ret_struct:")
print('st.val1 = {}\nst.val2 = {}\nst.val3 = {}'.format(st.val1, st.val2, st.val3))
# Вызывай метод print нашей структуры, только по скольку C частично ООП
# То нужно в этод метод передать указатель на нашу структуру
st.print(st)

# Время работы
print("--- {} seconds ---".format((time.time() - start_time)))
```
  

Модуль стал родным.

  

### Плюсы и минусы C API

  

**Плюсы**:

  

* легко использовать в **Python**

  

**Минусы**:

  

* сложно описывать свои типы данных на **C API**
* сложно реализовать чистым **Python** программистам, да и не им тоже… (по мне самое простое через **ctypes**)
* модуль(библиотека) будет только для **Python**

  

## Среднее время выполнения теста на каждом способе при 1000 запусках:

  

* ctypes: — 0.0004987692832946777 seconds ---
* CFFI: — 0.00038521790504455566 seconds ---
* pybind: — 0.0004547207355499268 seconds ---
* C API: — 0.0003561973571777344 seconds ---

  

## Ссылки

  

* [Исходные коды примеров](https://github.com/dvjdjvu/c_from_python)
* Способ через [**CFFI, pybind11**](https://habr.com/ru/post/468099/)
* Способ через [**ctypes**](https://habr.com/ru/post/466499/)
* [Python из C (C API)](https://habr.com/ru/post/466181/)
   