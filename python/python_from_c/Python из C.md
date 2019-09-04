<img alt="main" width="500" src="https://habrastorage.org/webt/la/_7/ji/la_7ji4yzcgayh2tad4yycia3yy.jpeg" align = "center"/>
В прошлом году появилась необходимость дополнить старый проект написанный на C функционалом на **python3**. Не смотря на то, что есть статьи на эту тему я помучился и в том году и сейчас когда писал программы для статьи. Поэтому приведу свои примеры по тому как работать с python3 из C под Linux (с тем что использовал). Опишу как создать класс и вызвать его методы, получить доступ к переменным. Вызов функций и получение переменных из модуля. А также проблемы с которыми я столкнулся и не смог их понять.

<cut/>

## Пакеты
Используем стандартное **Python API**. Необходимые пакеты python:   

 - **apt-get install python3**
 - **apt-get install python3-dev**
 - **apt-get install python3-all**
 - **apt-get install python3-all-dev**
 - **apt-get install libpython3-all-dev**


## Работа в интерпретаторе
Самое простое, загрузка интерпретатора python и работа в нем.

Для работы необходимо подключить заголовочный файл:

```cpp
 #include <Python.h>
```
Загружаем интерпретатор:

```cpp
Py_Initialize();
```

Далее идет блок работы с python, например:

```cpp
PyRun_SimpleString("print('Hello!')");
```

Выгружаем интерпретатор:

```cpp
Py_Finalize();
```
Полный пример:
```cpp
#include <Python.h>

void
main() {
	// Загрузка интерпретатора Python
	Py_Initialize();
	// Выполнение команды в интерпретаторе
	PyRun_SimpleString("print('Hello!')");
	// Выгрузка интерпретатора Python
	Py_Finalize();
}
```
Как компилировать и запустить:
```
gcc simple.c $(python3-config --includes --ldflags) -o simple && ./simple
Hello!
```
А вот так не будет работать:
```
gcc $(python3-config --includes --ldflags)  simple.c -o simple && ./simple
/tmp/ccUkmq57.o: In function `main':
simple.c:(.text+0x5): undefined reference to `Py_Initialize'
simple.c:(.text+0x16): undefined reference to `PyRun_SimpleStringFlags'
simple.c:(.text+0x1b): undefined reference to `Py_Finalize'
collect2: error: ld returned 1 exit status
```
Все из-за того, что **python3-config --includes --ldflags** раскрывается вот в такую штуку:
```
-I/usr/include/python3.6m -I/usr/include/python3.6m
-L/usr/lib/python3.6/config-3.6m-x86_64-linux-gnu -L/usr/lib -lpython3.6m -lpthread -ldl  -lutil -lm  -Xlinker -export-dynamic -Wl,-O1 -Wl,-Bsymbolic-functions
```
Здесь думаю важен порядок подключения линкера **-Wl**. Кто знает точнее напишите про это в коментах, дополню ответ.

Пример вызова функции из файла python:
simple.c
```cpp
#include <Python.h>

void 
python() {
    // Загрузка интерпретатора Python
    Py_Initialize();
    
    // Выполнение команд в интерпретаторе
	// Загрузка модуля sys
    PyRun_SimpleString("import sys");
	// Подключаем наши исходники python
    PyRun_SimpleString("sys.path.append('./src/python')");
    PyRun_SimpleString("import simple");
    PyRun_SimpleString("print(simple.get_value(2))");
    PyRun_SimpleString("print(simple.get_value(2.0))");
    PyRun_SimpleString("print(simple.get_value(\"Hello!\"))");
    
    // Выгрузка интерпретатора Python
    Py_Finalize();  
}

void
main() {
    puts("Test simple:");
    
    python();
}
```
simple.py
```python
#!/usr/bin/python3
#-*- coding: utf-8 -*-

def get_value(x):
    return x
```

Но это простые и неинтересные вещи, мы не получаем результат выполнения фунции.

## Работа с функциями и переменными модуля
Здесь немного сложнее.
Загрузка интерпретатора python и модуля func.py в него.:
```cpp
PyObject *
python_init() {
    // Инициализировать интерпретатор Python
    Py_Initialize();

    do {
        // Загрузка модуля sys
        sys = PyImport_ImportModule("sys");
        sys_path = PyObject_GetAttrString(sys, "path");
        // Путь до наших исходников python
        folder_path = PyUnicode_FromString((const char*) "./src/python");
        PyList_Append(sys_path, folder_path);

        // Загрузка func.py
        pName = PyUnicode_FromString("func");
        if (!pName) {
            break;
        }

        // Загрузить объект модуля
        pModule = PyImport_Import(pName);
        if (!pModule) {
            break;
        }

        // Словарь объектов содержащихся в модуле
        pDict = PyModule_GetDict(pModule);
        if (!pDict) {
            break;
        }

        return pDict;
    } while (0);

    // Печать ошибки
    PyErr_Print();
}
```
Освобождение ресурсов интерпретатора python:
```cpp
void
python_clear() {
    // Вернуть ресурсы системе
    Py_XDECREF(pDict);
    Py_XDECREF(pModule);
    Py_XDECREF(pName);

    Py_XDECREF(folder_path);
    Py_XDECREF(sys_path);
    Py_XDECREF(sys);

    // Выгрузка интерпретатора Python
    Py_Finalize();
}
```
Работа с переменными и функциями модуля.
```cpp
/**
 * Передача строки в качестве аргумента и получение строки назад
 */
char *
python_func_get_str(char *val) {
    char *ret = NULL;

    // Загрузка объекта get_value из func.py
    pObjct = PyDict_GetItemString(pDict, (const char *) "get_value");
    if (!pObjct) {
        return ret;
    }

    do {
        // Проверка pObjct на годность.
        if (!PyCallable_Check(pObjct)) {
            break;
        }

        pVal = PyObject_CallFunction(pObjct, (char *) "(s)", val);
        if (pVal != NULL) {
            PyObject* pResultRepr = PyObject_Repr(pVal);

            // Если полученную строку не скопировать, то после очистки ресурсов python её не будет.
            // Для начала pResultRepr нужно привести к массиву байтов.
            ret = strdup(PyBytes_AS_STRING(PyUnicode_AsEncodedString(pResultRepr, "utf-8", "ERROR")));

            Py_XDECREF(pResultRepr);
            Py_XDECREF(pVal);
        } else {
            PyErr_Print();
        }
    } while (0);

    Py_XDECREF(pObjct);

    return ret;
}

/**
 * Получение значения переменной содержащей значение типа int
 */
int
python_func_get_val(char *val) {
    int ret = 0;

    // Получить объект с именем val
    pVal = PyDict_GetItemString(pDict, (const char *) val);
    if (!pVal) {
        return ret;
    }
	
    // Проверка переменной на long
    if (PyLong_Check(pVal)) {
        ret = _PyLong_AsInt(pVal);
    } else {
        PyErr_Print();
    }

    Py_XDECREF(pVal);

    return ret;
}
```
На этом остановимся подробнее
```cpp
pVal = PyObject_CallFunction(pObjct, (char *) "(s)", val);
```
**"(s)"** означает, что передается 1 параметр типа **char * ** в качестве аргумента функции **get_value(x)**. Если бы нам нужно было передать несколько аргументов функции, то было бы так:
```cpp
pVal = PyObject_CallFunction(pObjct, (char *) "(sss)", val1, val2, val3);
```
Если необходимо передать **int**, то использовалась бы литера **i**, все возможные типы данных и их обозначения можно посмотреть в [документации](https://docs.python.org/3/c-api/arg.html#c.Py_BuildValue) python.
```cpp
pVal = PyObject_CallFunction(pObjct, (char *) "(i)", my_int);
```

func.py:
```python
#!/usr/bin/python3
#-*- coding: utf-8 -*-

a = 11
b = 22
c = 33

def get_value(x):
    return x

def get_bool(self, x):
    if x:
        return True
    else:
        return False
```
Проблема с которой я столкнулся и не смог пока понять:
```cpp
int
main() {
    puts("Test func:");

    if (!python_init()) {
        puts("python_init error");
        return -1;
    }

    puts("Strings:");
    printf("\tString: %s\n", python_func_get_str("Hello from Python!"));
    
    puts("Attrs:");
    printf("\ta: %d\n", python_func_get_val("a"));
    printf("\tb: %d\n", python_func_get_val("b"));
    printf("\tc: %d\n", python_func_get_val("c"));

    python_clear();

    return 0;
}
```
Если хочу получить **b** или **c** из **func.py**, то на:
```cpp
Py_Finalize();
```
получаю **segmentation fault**. С получением только **a** такой проблемы нет.
При получении переменных класса, тоже проблем нет.

## Работа с классом
Тут еще немножечко посложнее.
Загрузка интерпретатора python и модуля class.py в него.
```cpp
PyObject *
python_init() {
    // Инициализировать интерпретатор Python
    Py_Initialize();

    do {
        // Загрузка модуля sys
        sys = PyImport_ImportModule("sys");
        sys_path = PyObject_GetAttrString(sys, "path");
        // Путь до наших исходников python
        folder_path = PyUnicode_FromString((const char*) "./src/python");
        PyList_Append(sys_path, folder_path);

        // Создание Unicode объекта из UTF-8 строки
        pName = PyUnicode_FromString("class");
        if (!pName) {
            break;
        }

        // Загрузить модуль class
        pModule = PyImport_Import(pName);
        if (!pModule) {
            break;
        }

        // Словарь объектов содержащихся в модуле
        pDict = PyModule_GetDict(pModule);
        if (!pDict) {
            break;
        }

        // Загрузка объекта Class из class.py
        pClass = PyDict_GetItemString(pDict, (const char *) "Class");
        if (!pClass) {
            break;
        }

        // Проверка pClass на годность.
        if (!PyCallable_Check(pClass)) {
            break;
        }

        // Указатель на Class
        pInstance = PyObject_CallObject(pClass, NULL);

        return pInstance;
    } while (0);

    // Печать ошибки
    PyErr_Print();
}
```
Передача строки в качестве аргумента и получение строки назад
 ```cpp
char *
python_class_get_str(char *val) {
    char *ret = NULL;

    pVal = PyObject_CallMethod(pInstance, (char *) "get_value", (char *) "(s)", val);
    if (pVal != NULL) {
        PyObject* pResultRepr = PyObject_Repr(pVal);
        
        // Если полученную строку не скопировать, то после очистки ресурсов python её не будет.
        ret = strdup(PyBytes_AS_STRING(PyUnicode_AsEncodedString(pResultRepr, "utf-8", "ERROR")));
        
        Py_XDECREF(pResultRepr);
        Py_XDECREF(pVal);
    } else {
        PyErr_Print();
    }

    return ret;
}
```
Здесь ни каких проблем не было, все работает без ошибок. В исходниках примеры, как работать с **int**, **double**, **bool**.

Пока писал материал освежил свои знания )
Надеюсь будет полезно.

## Ссылки
[Исходные коды примеров](https://github.com/dvjdjvu/python_from_c)