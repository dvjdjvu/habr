



maxzhurkin
Mon, 21 Dec 2020 17:48:41 GMT
 Наша основная система имеет адрес 172.18.0.1, docker с БД находится на 172.18.0.7.А что будет, когда адреса поменяются?]]>
https://habr.com/post/533040/#comment\_22448728
https://habr.com/post/533040/#comment\_22448728
21.12.2020 17:48:41 maxzhurkin


Jessy\_James
Mon, 21 Dec 2020 18:52:42 GMT
]]>

Здесь задать нужный ip.

  
```
                        server_default = "tango://tangobox:10000")
                        archive_server_name="archiving/hdbpp/eventsubscriber.1",
__init__(self, host="172.18.0.7", user="root", password="tango", database="hdbpp", 
```python


 Не будет соединения. Но внутри этой системы они статичны. А так по правильному хорошо бы авторизацию прописать. HDBPP() имеет следующую семантику:   
https://habr.com/post/533040/#comment\_22448962
https://habr.com/post/533040/#comment\_22448962
21.12.2020 18:52:42 Jessy\_James

Хабр
 https://habrastorage.org/webt/ym/el/wk/ymelwk3zy1gawz4nkejl\_-ammtc.png
https://habr.com/ru/

Sat, 14 May 2022 14:38:47 GMT
habr.com
editor@habr.com
ru
 
https://habr.com/ru/post/533040/
Комментарии к публикации «HDB++ TANGO Archiving System»

xml version="1.0" encoding="UTF-8"?