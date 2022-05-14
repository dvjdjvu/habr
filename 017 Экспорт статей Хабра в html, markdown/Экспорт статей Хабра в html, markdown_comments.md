



MentalBlood
Wed, 11 May 2022 20:09:59 GMT
]]>

Если решите развивать, было бы круто добавить многопоточность и оформить в виде питоновского пакета (чтобы можно было `pip install habrArticleSrcDownloader`)

 Отличная идея  
https://habr.com/post/665254/#comment\_24335752
https://habr.com/post/665254/#comment\_24335752
11.05.2022 20:09:59 MentalBlood


Jessy\_James
Wed, 11 May 2022 20:17:55 GMT
 Спасибо. Многопоточность из головы выпала. Добавлю ваши замечания.]]>
https://habr.com/post/665254/#comment\_24335790
https://habr.com/post/665254/#comment\_24335790
11.05.2022 20:17:55 Jessy\_James


Ploni
Wed, 11 May 2022 21:51:13 GMT
 А вот чтоб еще и с комментами - можно такое?]]>
https://habr.com/post/665254/#comment\_24335956
https://habr.com/post/665254/#comment\_24335956
11.05.2022 21:51:13 Ploni


Jessy\_James
Wed, 11 May 2022 21:51:57 GMT
 Да, доработаю.]]>
https://habr.com/post/665254/#comment\_24335962
https://habr.com/post/665254/#comment\_24335962
11.05.2022 21:51:57 Jessy\_James


kt97679
Thu, 12 May 2022 04:39:44 GMT
 
https://habr.com/post/665254/#comment\_24336284
https://habr.com/post/665254/#comment\_24336284
12.05.2022 04:39:44 kt97679


dot22
Thu, 12 May 2022 05:22:22 GMT
]]>

А так - да, действительно было бы замечательно - иметь возможность скачивать не все статьи пользователя сразу, а только какую-то одну конкретную. Ну, и как выше уже писали - если бы еще вместе с комментариями - было бы еще лучше.Обсидиан - который работает с файлами в формате md, - в таком случае вообще можно было бы сделать что-то типа локальной базы знаний избранных статей с хабра - с поиском, тегами, пометками, графами и всякими другими плюшками.
 Если нужна только одна статья именно в формате md и прямо сейчас, то можно использовать плагин (работает в хроме и опере (через прокладку) <https://chrome.google.com/webstore/detail/markdownload-markdown-web/pcmpcfapbekmbjjkdalcgopdkipoggdi>  
https://habr.com/post/665254/#comment\_24336322
https://habr.com/post/665254/#comment\_24336322
12.05.2022 05:22:22 dot22


kt97679
Thu, 12 May 2022 07:22:11 GMT
 
https://habr.com/post/665254/#comment\_24336632
https://habr.com/post/665254/#comment\_24336632
12.05.2022 07:22:11 kt97679


dot22
Thu, 12 May 2022 10:15:40 GMT
]]>

Или я опять делаю поспешные выводы?

Если же интересует именно формат md, то после (или даже во время загрузки по пайпу) файлы в формате html уже локально можно переконвертировать в md (вроде как pandoc это умеет искаропки)

Если есть список - почему бы не использовать специализированные инструменты, именно и предназначенные для скачивания файлов любого типа из интернета - первое, что приходит на ум - wget или curl. 
[https://habr.com/ru/post/665254/](https://habr.com/ru/post/665254/comments/#comment_24336322)  
Как я понимаю, т.е. уже есть какой-то подготовленный список статей, которые Вам интересны, с url-ами, содержащими ID статьи?, как, например, обсуждаемая статья  

 Хотелось бы уточнить по поводу трекера. Не совсем понимаю, что Вы имеете в виду?Из Вашего комментария - "возможность скачать только одну статью по айди"
https://habr.com/post/665254/#comment\_24337290
https://habr.com/post/665254/#comment\_24337290
12.05.2022 10:15:40 dot22


Jessy\_James
Thu, 12 May 2022 11:30:25 GMT
 
https://habr.com/post/665254/#comment\_24337498
https://habr.com/post/665254/#comment\_24337498
12.05.2022 11:30:25 Jessy\_James


Jessy\_James
Thu, 12 May 2022 11:32:21 GMT
]]>
```
./src/main.py -f jessy_james
```bash


  

Скачиваем закладки пользователя:  

  
```
./src/main.py -u jessy_james
```bash


Скачиваем статьи пользователя:  

  
 
https://habr.com/post/665254/#comment\_24337508
https://habr.com/post/665254/#comment\_24337508
12.05.2022 11:32:21 Jessy\_James


kuaniv
Thu, 12 May 2022 13:58:42 GMT
]]>

 Спасибо! Отличный скрипт! Скачал уже 2 Гб статей.Столкнулся с такой особенностью. Если заголовок статьи содержит, например, знак вопроса, то имя папки и файлов будут включать экранированный знак вопроса '/?'. Если эти файлы потом перенести в Win, то их имена не читаются. Можно добавить в скрипт игнорирование спецсимволов при создании файлов/папок?
https://habr.com/post/665254/#comment\_24337998
https://habr.com/post/665254/#comment\_24337998
12.05.2022 13:58:42 kuaniv


Jessy\_James
Thu, 12 May 2022 14:19:35 GMT
 
https://habr.com/post/665254/#comment\_24338072
https://habr.com/post/665254/#comment\_24338072
12.05.2022 14:19:35 Jessy\_James


Jessy\_James
Thu, 12 May 2022 15:02:14 GMT
 
https://habr.com/post/665254/#comment\_24338220
https://habr.com/post/665254/#comment\_24338220
12.05.2022 15:02:14 Jessy\_James


kt97679
Thu, 12 May 2022 16:24:21 GMT
В результате я вижу 2 проблемы: как скачать прочие ресурсы, например картинки, не выкачивая весь хабр. И как склеить тело статьи с комментариями.]]>

$`  

./habr.com/robots.txt  

./habr.com/ru/post/665254/comments/index.html  

./habr.com/ru/post/665254/index.html  

`$ find . -type f  

 wget -r -np https://habr.com/ru/post/665254/, то скачиваются 2 файла: с содержимым статьи и с комментариями:  
https://habr.com/post/665254/#comment\_24338452
https://habr.com/post/665254/#comment\_24338452
12.05.2022 16:24:21 kt97679


kt97679
Thu, 12 May 2022 16:24:45 GMT
 
https://habr.com/post/665254/#comment\_24338456
https://habr.com/post/665254/#comment\_24338456
12.05.2022 16:24:45 kt97679


arboozof
Thu, 12 May 2022 19:36:52 GMT
 Великолепный инструмент! [Jessy\_James](https://habr.com/ru/users/jessy_james/), спасибо вам!]]>
https://habr.com/post/665254/#comment\_24338910
https://habr.com/post/665254/#comment\_24338910
12.05.2022 19:36:52 arboozof


AquariusStar
Thu, 12 May 2022 20:19:55 GMT
 Хороший инструмент! Я обычно сохраняю в pdf. Но это не всегда удобно. Если есть анимированные картинки, то уже совсем грустно становится. Кстати, в некоторых статьях есть картинки с форматом jpeg и с форматом PNG. Вот jpeg почему-то портит картинку в браузере и VS Code. Только отдельным просмотрщиком можно смотреть. А вот PNG уже нормально. Есть ли возможность указать на принудительное сохранение картинок в нужном формате? А также сделать сохранение только конкретной статьи, а не всех?]]>
https://habr.com/post/665254/#comment\_24338996
https://habr.com/post/665254/#comment\_24338996
12.05.2022 20:19:55 AquariusStar


arboozof
Fri, 13 May 2022 11:08:35 GMT
]]>

![image](https://habrastorage.org/webt/3k/us/6k/3kus6kqlqhuwsqbzreyoujziseu.png)

 Есть предложение использовать в именовании каталогов 3-значную порядковую нумерацию, дополняя слева незначащими нулями (например, 001 — каталог с публикацией #1, 078 — каталог с публикацией #78). Дабы на выходе, при отображении каталогов в файловом менеджере, была сортировка, как вы и задумывали — от последней написанной к первой (хотя при загрузке своих публикаций я получил обратную сортировку, которая как мне кажется и правильная — от первой написанной к последней).  
https://habr.com/post/665254/#comment\_24340702
https://habr.com/post/665254/#comment\_24340702
13.05.2022 11:08:35 arboozof


Jessy\_James
Fri, 13 May 2022 13:26:44 GMT
 Я писал, что скачивается от последней статьи к первой. Нумерация же как раз шла от первой написанной к последней. Сейчас же из за многопоточности порядок скачивания как получится.]]>
https://habr.com/post/665254/#comment\_24341252
https://habr.com/post/665254/#comment\_24341252
13.05.2022 13:26:44 Jessy\_James


arboozof
Fri, 13 May 2022 14:24:44 GMT
 В моем случае они скачались ровно по порядку от старой к новой, с соответствующей нумерацией. По трехзначной нумерации, эстетики ради, подумайте...]]>
https://habr.com/post/665254/#comment\_24341466
https://habr.com/post/665254/#comment\_24341466
13.05.2022 14:24:44 arboozof


Jessy\_James
Fri, 13 May 2022 17:42:52 GMT
 
https://habr.com/post/665254/#comment\_24341854
https://habr.com/post/665254/#comment\_24341854
13.05.2022 17:42:52 Jessy\_James


Jessy\_James
Fri, 13 May 2022 17:43:13 GMT
 
https://habr.com/post/665254/#comment\_24341856
https://habr.com/post/665254/#comment\_24341856
13.05.2022 17:43:13 Jessy\_James


Jessy\_James
Fri, 13 May 2022 17:45:04 GMT
 
https://habr.com/post/665254/#comment\_24341858
https://habr.com/post/665254/#comment\_24341858
13.05.2022 17:45:04 Jessy\_James


Jessy\_James
Fri, 13 May 2022 20:01:48 GMT
]]>
```
./src/main.py -s 665634
```bash

 
https://habr.com/post/665254/#comment\_24342134
https://habr.com/post/665254/#comment\_24342134
13.05.2022 20:01:48 Jessy\_James


arboozof
Fri, 13 May 2022 22:46:03 GMT
 Супер. Отдельное спасибо за реализацию пожеланий!]]>
https://habr.com/post/665254/#comment\_24342324
https://habr.com/post/665254/#comment\_24342324
13.05.2022 22:46:03 arboozof

Хабр
 https://habrastorage.org/webt/ym/el/wk/ymelwk3zy1gawz4nkejl\_-ammtc.png
https://habr.com/ru/

Sat, 14 May 2022 14:38:39 GMT
habr.com
editor@habr.com
ru
 
https://habr.com/ru/post/665254/
Комментарии к публикации «Экспорт статей Хабра в html, markdown»

xml version="1.0" encoding="UTF-8"?