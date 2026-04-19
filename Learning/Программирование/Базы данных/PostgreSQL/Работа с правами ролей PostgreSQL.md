
Для выдчи прав тому или иному пользователю на объект или группу объекто используется два шаблона запросов:
``` 
GRANT <privileges>
ON <object_type> <object_name>
TO <role>;
```
Для выдачи прав на конкретный объект и
```
GRANT <privileges>
ON ALL <object_type_plural> IN SCHEMA <schema>
TO <role>;
```
для выдачи прав на все объекты одного типа в схеме.

В качетсве привелегий можно выдать все возмодные привелегии указав значение `ALL PRIVILEGES`.

### Типы объектов и их права

##### Базы данных (DATABASE)
Права:
* `CONNECT` — подключаться
- `CREATE` — создавать схемы
- `TEMP` — временные таблицы
Пример:
```
GRANT CONNECT, CREATE ON DATABASE demo TO ilya;
```
##### Схемы (SCHEMA)
Права:
- `USAGE` — “видеть” объекты внутри
- `CREATE` — создавать таблицы и др.
Пример:
```
GRANT ALL PRIVILEGES ON SCHEMA public TO ilya;
```

##### Таблицы (TABLE)
Права:
* Права на использование запросов `SELECT`, `INSERT`, `UPDATE`, `DELETE`
Пример:
```
GRANT SELECT, INSERT ON ALL TABLES IN SCHEMA public TO ilya;
```

##### Функции (FUNCTION)
Права:
* `EXECUTE` - использовать функцию
Пример:
```
GRANT EXECUTE ON FUNCTION myfunc() TO ilya;
```

### Наследование прав

```
GRANT parent_role TO child_role
```
Передаёт все права от `parent_role` к `child_role`.

### Отобрать права

Запрос SQL выглядит так:
```
REVOKE <privileges> ON <type> <object name> FROM <role>;
```
Синтаксис похож на `GRANT` и может притерпевать похожие обобщения, например использовать `ALL PRIVILEGES` и `ALL type IN SCHEMA ...`.

### DEFAULT PRIVILEGES

Мы выдавали ролям права на разные объекты. Но если появится новый объект то что с ним делать? Пока что на него никто не будет иметь права, так как права выдавались только на старые объекты (даже `ALL type_obj IN SCHEMA` так работает).
Чтобы настроить права по умолчанию для новых создаваемых таблиц мы можем поменять DEFAULT PRIVILEGES. Вообщем синтаксис SQL выглядит так:
```
ALTER DEFAULT PRIVILEGES  
[FOR ROLE role_name]  
[IN SCHEMA schema_name]  
GRANT <privileges>  
ON <object_type>  
TO <role>;
```
Если указать ещё `FOR ROLE` то такие права будут выдаваться, только если объект был создан этим пользователем (по умолчанию вписывается пользователь, который отпарвляет запрос). Также можно указать объекты в отдельной схеме.

### Вспомогательные команды 
Команда `\dp` позволяет посмотреть права пользователя. Просмотр всех привелегий по умолчанию `\ddp`.