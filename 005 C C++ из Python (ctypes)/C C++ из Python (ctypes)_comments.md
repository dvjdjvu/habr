



alec\_kalinin
Fri, 06 Sep 2019 18:13:21 GMT
 Numba. С простыми двойными циклами она должна справляться очень хорошо.]]>
https://habr.com/post/466499/#comment\_20595219
https://habr.com/post/466499/#comment\_20595219
06.09.2019 18:13:21 alec\_kalinin


ciklop
Fri, 06 Sep 2019 18:18:06 GMT
]]>

Под Windows можно посмотреть с помощью `link /dump /exports libtestcpp.dll`

  

И вот они, имена функций.

  
```
...
0000000000001370 T _Z8test_newv
00000000000014ee T _Z15test_ret_doubleP4testd
00000000000013ab T _Z12test_ret_strP4testPc
00000000000014cc T _Z12test_ret_intP4testi
...

nm -D libtestcpp.so
```plaintext


Под linux посмотреть имена функций можно с помощью **nm**. Для test.cpp без extern C будет примерно так:  

 Через ctypes можно вызывать с++ и без **extern C**. Но это будет "непереносимый код". Так как [name mangling](https://en.wikipedia.org/wiki/Name_mangling#C.2B.2B) не стандартизован, и каждый компилятор будет генерировать свои экспортируемые имена функций. Если их подсмотреть в библиотеке — их также можно спокойно вызвать.  
https://habr.com/post/466499/#comment\_20595223
https://habr.com/post/466499/#comment\_20595223
06.09.2019 18:18:06 ciklop


Jessy\_James
Fri, 06 Sep 2019 18:22:28 GMT
Про то что без extern прокатит писать не стал, а вы не поленились ). Спасибо.]]>

  

 nm писать, но не стал. Когда makefile писал, то накосячил и libtestpp.so собирал из test.c. С помощью nm разбирался, почему не находит вызываемые функции.   
https://habr.com/post/466499/#comment\_20595231
https://habr.com/post/466499/#comment\_20595231
06.09.2019 18:22:28 Jessy\_James


DollaR84
Fri, 06 Sep 2019 19:42:52 GMT
 
https://habr.com/post/466499/#comment\_20595395
https://habr.com/post/466499/#comment\_20595395
06.09.2019 19:42:52 DollaR84


alec\_kalinin
Fri, 06 Sep 2019 20:47:59 GMT
 
https://habr.com/post/466499/#comment\_20595545
https://habr.com/post/466499/#comment\_20595545
06.09.2019 20:47:59 alec\_kalinin


DollaR84
Sat, 07 Sep 2019 08:12:23 GMT
 
https://habr.com/post/466499/#comment\_20596227
https://habr.com/post/466499/#comment\_20596227
07.09.2019 08:12:23 DollaR84


alec\_kalinin
Sat, 07 Sep 2019 10:43:23 GMT
 Спасибо!]]>
https://habr.com/post/466499/#comment\_20596561
https://habr.com/post/466499/#comment\_20596561
07.09.2019 10:43:23 alec\_kalinin


Closius
Sat, 07 Sep 2019 11:13:18 GMT
 Boost.Python?]]>
https://habr.com/post/466499/#comment\_20596627
https://habr.com/post/466499/#comment\_20596627
07.09.2019 11:13:18 Closius


maxood
Sat, 07 Sep 2019 13:19:42 GMT
 
https://habr.com/post/466499/#comment\_20596825
https://habr.com/post/466499/#comment\_20596825
07.09.2019 13:19:42 maxood


DollaR84
Sat, 07 Sep 2019 19:26:04 GMT
несколько показал работу со Structure]]>

 Передача двумерных списков из python в DLL  
https://habr.com/post/466499/#comment\_20597375
https://habr.com/post/466499/#comment\_20597375
07.09.2019 19:26:04 DollaR84


Jessy\_James
Sat, 07 Sep 2019 21:35:40 GMT
 
https://habr.com/post/466499/#comment\_20597559
https://habr.com/post/466499/#comment\_20597559
07.09.2019 21:35:40 Jessy\_James


Jessy\_James
Sat, 07 Sep 2019 21:36:33 GMT
 
https://habr.com/post/466499/#comment\_20597563
https://habr.com/post/466499/#comment\_20597563
07.09.2019 21:36:33 Jessy\_James


iroln
Sat, 07 Sep 2019 22:09:03 GMT
 Лучше [pybind11](https://github.com/pybind/pybind11)]]>
https://habr.com/post/466499/#comment\_20597601
https://habr.com/post/466499/#comment\_20597601
07.09.2019 22:09:03 iroln


iroln
Sat, 07 Sep 2019 22:12:23 GMT
 Всё это здорово, конечно, но на самом деле это уже давно не нужно, потому что есть [pybind11](https://github.com/pybind/pybind11) и [CFFI](https://cffi.readthedocs.io/en/latest/). Эти инструменты разительно упрощают embedding/extending для Python без возни с медленным ctypes.]]>
https://habr.com/post/466499/#comment\_20597611
https://habr.com/post/466499/#comment\_20597611
07.09.2019 22:12:23 iroln


ser-mk
Sat, 07 Sep 2019 22:14:28 GMT
Мне кажется проще Ctypes уже нет]]>
 
https://habr.com/post/466499/#comment\_20597613
https://habr.com/post/466499/#comment\_20597613
07.09.2019 22:14:28 ser-mk


iroln
Sat, 07 Sep 2019 23:54:11 GMT
]]>

Boost.Python я вообще не рекомендую использовать ни для чего. Монстроузный и неудобный (удобнее, конечно, чем голый CPython API, но значительно менее удобный чем более современные штуки).

<https://qr.ae/TWy0op>  

 Для обычных C функций лучше CFFI.  
https://habr.com/post/466499/#comment\_20597713
https://habr.com/post/466499/#comment\_20597713
07.09.2019 23:54:11 iroln


grafalex
Sun, 08 Sep 2019 20:11:01 GMT
Я тут чуток затронул эту тему [в своей статье про автотестирование](https://habr.com/ru/post/273401/).]]>

А как же cython? там вообще без танцев с бубном и ctypes, и синтаксис почти питоновский.  

  
 
https://habr.com/post/466499/#comment\_20599321
https://habr.com/post/466499/#comment\_20599321
08.09.2019 20:11:01 grafalex


Jessy\_James
Sun, 08 Sep 2019 20:55:55 GMT
 
https://habr.com/post/466499/#comment\_20599415
https://habr.com/post/466499/#comment\_20599415
08.09.2019 20:55:55 Jessy\_James


grafalex
Mon, 09 Sep 2019 06:38:57 GMT
 
https://habr.com/post/466499/#comment\_20600097
https://habr.com/post/466499/#comment\_20600097
09.09.2019 06:38:57 grafalex


Jessy\_James
Mon, 09 Sep 2019 13:36:25 GMT
 
https://habr.com/post/466499/#comment\_20602265
https://habr.com/post/466499/#comment\_20602265
09.09.2019 13:36:25 Jessy\_James


DollaR84
Mon, 09 Sep 2019 14:12:13 GMT
 
https://habr.com/post/466499/#comment\_20602431
https://habr.com/post/466499/#comment\_20602431
09.09.2019 14:12:13 DollaR84


DollaR84
Mon, 09 Sep 2019 16:26:47 GMT
]]>
Постарался максимально приблизить к вашему примеру, но если где есть неточности, думаю поправите.  

Ну и конечно не забыть потом передать этот указатель в dll на удаление.  

Волшебное слово contents позволяет получить прямой доступ к данным структуры по указателю.  

  
```

print('val1 = {}\nval2 = {}\nval3 = {}'.format(ret.contents.val1, ret.contents.val2, ret.contents.val3))
ret = test.func_ret_struct()
test.func_ret_struct.restype = ctypes.POINTER(test_st_t)
test.func_ret_struct.argtypes = [ctypes.c_void_p]
```python


В коде python:  

  
```

}
    return res;
    res->val3 = 'z';
    res->val2 = 3.5;
    res->val1 = 19;
    test_st_t *res = new test_st_t;
func_ret_struct(void) {
test_st_t *
```cpp


Тогда в C:  

Вот вы в своей функции вернули тоже, что получили на вход. Не знаю насколько это нужно, поэтому упрощу вашу функцию. Пусть она на вход ничего не получает, а возвращает структуру, созданную в ней.  

Вот что меня смутило в вашем коде, буфер и копирование данных из C структуры в python/ Зачем лишние буфера, операции копирования и прочее. Так как dll может оперировать со структурами ctypes, созданными в самом python, так же можно оперировать структурами, созданными в dll, из самого python.  

  

 Как полученные данные скопировать в python структуру напрямую не додумался, кто знает напишите.  
https://habr.com/post/466499/#comment\_20602955
https://habr.com/post/466499/#comment\_20602955
09.09.2019 16:26:47 DollaR84


ser-mk
Mon, 09 Sep 2019 17:52:26 GMT
И можно наткнуться на очень неприятные баги)]]>

  
```

sizeof(GPIO_InitTypeDef) // 12 bytes

}GPIO_InitTypeDef;
  GPIOMode_TypeDef GPIO_Mode;
  GPIOSpeed_TypeDef GPIO_Speed; 
  uint16_t GPIO_Pin; 
{
typedef struct
```cpp


А С++ библиотека собрана с выравниванием структур по 4 байта, т.е. итоговый размер будет 12   

  

sizeof возвращает размер 4 байта.  

  
```

ctypes.sizeof(GPIO_InitTypeDef)
        ('GPIO_Mode',ctypes.c_uint8)]
        ('GPIO_Speed',ctypes.c_uint8),
        ('GPIO_Pin',ctypes.c_uint16),
    _fields_ = [
class GPIO_InitTypeDef(ctypes.Structure):
```python


Стоит наверно еще сказать про выравнивание структур. Не так давно столкнулся с такой особенностью.  

  

Возможно можно сделать по аналогии как [здесь](https://habr.com/ru/post/466575/) через ctypes.cast(). Но пока нет возможности проверить  

 Как полученные данные скопировать в python структуру напрямую не додумался, кто знает напишите.  
https://habr.com/post/466499/#comment\_20603285
https://habr.com/post/466499/#comment\_20603285
09.09.2019 17:52:26 ser-mk


Jessy\_James
Mon, 09 Sep 2019 17:56:27 GMT
 
https://habr.com/post/466499/#comment\_20603289
https://habr.com/post/466499/#comment\_20603289
09.09.2019 17:56:27 Jessy\_James


Closius
Mon, 16 Sep 2019 13:10:32 GMT
 
https://habr.com/post/466499/#comment\_20628395
https://habr.com/post/466499/#comment\_20628395
16.09.2019 13:10:32 Closius


tixo
Thu, 19 Sep 2019 20:54:13 GMT
 ]]>
https://habr.com/post/466499/#comment\_20644503
https://habr.com/post/466499/#comment\_20644503
19.09.2019 20:54:13 tixo


Jessy\_James
Thu, 19 Sep 2019 21:19:35 GMT
 
https://habr.com/post/466499/#comment\_20644585
https://habr.com/post/466499/#comment\_20644585
19.09.2019 21:19:35 Jessy\_James


Palich239
Sun, 24 Nov 2019 17:47:56 GMT
 
https://habr.com/post/466499/#comment\_20925804
https://habr.com/post/466499/#comment\_20925804
24.11.2019 17:47:56 Palich239


Palich239
Mon, 25 Nov 2019 22:08:01 GMT
]]>

> 
> ```
> testpp.test_get_c.argtypes = [ctypes.c_void_p]
> testpp.test_get_c.restype = ctypes.c_char
> # Указываем, что функция возвращает char
> testpp.test_get_b.argtypes = [ctypes.c_void_p]
> testpp.test_get_b.restype = ctypes.c_double
> # Указываем, что функция возвращает double
> testpp.test_get_a.argtypes = [ctypes.c_void_p]
> testpp.test_get_a.restype = ctypes.c_int
> # Указываем, что функция возвращает int
> ```python
> 


То же самое касается описания работы с переменными:  

  

> 
> ```
> testpp.test_del(test)
> # Удаляем класс
> ```python
> 


перед концом файла main\_cpp.py:  

  
```
testpp.test_del.argtypes = [ctypes.c_void_p]
testpp.test_del.restype = ctypes.c_int
```python

 
https://habr.com/post/466499/#comment\_20930630
https://habr.com/post/466499/#comment\_20930630
25.11.2019 22:08:01 Palich239


Jessy\_James
Tue, 26 Nov 2019 10:27:00 GMT
 github.com/dvjdjvu/c\_from\_python/tree/master/ctypes]]>
https://habr.com/post/466499/#comment\_20932362
https://habr.com/post/466499/#comment\_20932362
26.11.2019 10:27:00 Jessy\_James


Palich239
Tue, 26 Nov 2019 19:54:38 GMT
 
https://habr.com/post/466499/#comment\_20934856
https://habr.com/post/466499/#comment\_20934856
26.11.2019 19:54:38 Palich239


Jessy\_James
Tue, 26 Nov 2019 20:59:33 GMT
 
https://habr.com/post/466499/#comment\_20935060
https://habr.com/post/466499/#comment\_20935060
26.11.2019 20:59:33 Jessy\_James


im\_stD
Fri, 08 Jan 2021 06:43:54 GMT
… в самом конце запятая?]]>

  

(ctypes.c\_char)**,** ]  

  

  
```
test.func_ret_str.argtypes = [ctypes.POINTER(ctypes.c_char), ]
# Указываем, что функция принимает аргумент char *
```cpp


  
 
https://habr.com/post/466499/#comment\_22510000
https://habr.com/post/466499/#comment\_22510000
08.01.2021 06:43:54 im\_stD


Jessy\_James
Fri, 08 Jan 2021 10:14:42 GMT
 удалить этот комент.]]>
https://habr.com/post/466499/#comment\_22510680
https://habr.com/post/466499/#comment\_22510680
08.01.2021 10:14:42 Jessy\_James


Jessy\_James
Fri, 08 Jan 2021 10:16:25 GMT
В любом случае ее наличие или отсутствие ни на что не влияет. В python для упрощения жизни можно после последнего элемента в массиве и т.п. оставить запятую.]]>

 2 причины, либо ctrl-c & ctrl-v от куда-то делал, либо опечатался.  
https://habr.com/post/466499/#comment\_22510684
https://habr.com/post/466499/#comment\_22510684
08.01.2021 10:16:25 Jessy\_James


im\_stD
Fri, 08 Jan 2021 16:19:27 GMT
А не могли бы дополнить статью, или тут в комментах написать про **create\_string\_buffer**. В каких случаях это нужно использовать? Там что-то про изменяющиеся или не изменяющиеся данные, но я так и не понял.]]>

  
 
https://habr.com/post/466499/#comment\_22511826
https://habr.com/post/466499/#comment\_22511826
08.01.2021 16:19:27 im\_stD


Jessy\_James
Fri, 08 Jan 2021 17:03:30 GMT
 Сходу не знаю что это, попозже посмотрю.]]>
https://habr.com/post/466499/#comment\_22512024
https://habr.com/post/466499/#comment\_22512024
08.01.2021 17:03:30 Jessy\_James


im\_stD
Fri, 08 Jan 2021 19:49:38 GMT
 
https://habr.com/post/466499/#comment\_22512558
https://habr.com/post/466499/#comment\_22512558
08.01.2021 19:49:38 im\_stD


Jessy\_James
Sun, 10 Jan 2021 18:46:29 GMT
 [Здесь](https://python-scripts.com/extending-python-with-c-libraries) подробно написано для чего функция нужна и как ей пользоваться.]]>
https://habr.com/post/466499/#comment\_22519828
https://habr.com/post/466499/#comment\_22519828
10.01.2021 18:46:29 Jessy\_James


im\_stD
Mon, 11 Jan 2021 09:17:54 GMT
 
https://habr.com/post/466499/#comment\_22522344
https://habr.com/post/466499/#comment\_22522344
11.01.2021 09:17:54 im\_stD

Хабр
 https://habrastorage.org/webt/ym/el/wk/ymelwk3zy1gawz4nkejl\_-ammtc.png
https://habr.com/ru/

Sat, 14 May 2022 14:38:42 GMT
habr.com
editor@habr.com
ru
 
https://habr.com/ru/post/466499/
Комментарии к публикации «C/C++ из Python (ctypes)»

xml version="1.0" encoding="UTF-8"?