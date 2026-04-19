
Предварительно посмотри [[Главный файл конфигурации Apache|описание основных конфигов]].


``` 
###### apache2.conf 

Timeout 300
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5

User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}

ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn

IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf

Include ports.conf

<Directory />
	Options FollowSymLinks
	Require all denied
</Directory>

<Directory /usr/share>
	Require all granted
</Directory>

<Directory /var/www/>
	Options +Indexes
	Require all granted
</Directory>

IncludeOptional conf-enabled/*.conf
IncludeOptional sites-enabled/*.conf
```

``` 
###### ports.conf

Listen 80
Listen 8080
<IfModule ssl_module>
	Listen 443
</IfModule>

<IfModule rewrite_module>
	Listen 443
</IfModule>
```

Здесь замети, что используются такие переменные окружения, как `APACHE_RUN_USER`, `APACHE_RUN_GROUP`, `APACHE_LOG_DIR`.
Заметим, что именно здесь мы указываем, чтобы все дополнительные дополнительные файлы конфигурации подключались, а именно конфигурации портов `ports.conf`, конфигурации активных сайтов (все те файлы которые лежат в директории `sites-enabled`) и т.д. подключаются именно здесь при помощи команд `IncludeOptional` и `Include`.

Также, обратим внимание, что тут настроена прослушка порта 443, но только в случае работы одного из модулей: ssl_module или mod_gnutls.c.

Здесь наибольшего внимания требует настройка директорий.
Мы для начала, настраиваем все директории (через настройку директории /, происходит настройка и всех вложенных, то есть всех директорий на сервере). Таким образом мы как бы задаём базавые настройки для всех директорий, а именно запрещаем к ним доступ.
Затем мы точечно включаем доступ к директориям, с которыми пользователь будет коммуницировать.

Для чего нужна настройка изначальная настройка корневой директории? Это важно!! Иначе пользователь сможет например обратится к ресурсу `http://server/../../../etc/passwd` или же, если каким то образом, в директории сайта будет лежать символическая ссылка на секретный файл, и
1) В настройках директории сайта будет прописано `FollowSymLinks`
2) В настройках директории, на файл в которой символическая ссылка сылается, не будет прописан запрет
Пользователь сможет получить доступ к этому файлу.