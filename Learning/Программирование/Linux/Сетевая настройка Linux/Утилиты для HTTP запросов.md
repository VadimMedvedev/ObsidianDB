
### Команда curl

`curl https://www.google.com` - простой запрос GET
`curl -I https://www.google.com` - возвращает заголовки ответа (метаинформацию)
`curl -vvv https://www.google.com` - подробный вывод (чем больше v, тем подробнее)
`curl -o file https://www.google.com/page1.html` - сохранить вывод в файл
`curl -O https://www.google.com/file.zip` - сохранить в файл с таким же названием
`curl -X POST -H "заголовок" -d "данные" https://www.google.com` - ключ `-X`  указывает какой метод использовать, `-H` - заголовки, метаданные и `-d` данные посылаемые.
`curl https://www.google.com` 

### Команда wget

Используется для загрузки
`wget https://www.google.com/page1.html` - скачать файл
`wget -r https://www.google.com` - рекурсивно скачать весь сайт
`wget -c https://www.google.com/page1.html` - продолжить прерваное скачивание
Если для скачивания требуется авторизация, то используються ключи `--user=<логин>` и `--password=<пароль>`.
