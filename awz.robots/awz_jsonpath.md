# JsonPath парсер JSON
<!-- desc-start -->

Действие позволяет осуществить поиск по JSON в переменную

<!-- desc-end -->

<!-- docs-start -->

JsonPath Синтаксис
=================
Активити использует следующую спецификацию:
```
var_name¹   = /^\.([\p{L}\p{N}\_\$][\p{L}\p{N}\_\-\$]*|\*)(.*)/u
number      = ([0-9]+(\.[0-9]*) | ([0-9]*\.[0-9]+))
string      = ('\''.*?'\'' | '"'.*?'"')
boolean     = ('true' | 'false')
regpattern  = '/'.*?'/i?x?'
null        = 'null'
index       = -?[0-9]+

jsonpath    = '$' operator*
childpath   = '@' operator*

operator    = (childname | childfilter | recursive) operator*

childname   = '.' (var_name | '*')
childfilter = '[' ('*' | namelist | indexlist | arrayslice | filterexpr) ']'
recursive   = '..' (var_name | childfilter | '*')

namelist    = var_name (',' (var_name | '\'' .*? '\'' | '"' .*? '"'))*
indexlist   = index (',' index)*
arrayslice  = index? ':' index? ':' index?
filterexpr  = '?(' ors ' | regpattern)'

ors         = ands (' ' ( 'or' | '\|\|' ) ' ' ands)*
ands        = expr (' ' ( 'and' | '&&' ) ' ' expr)*
expr        = ( 'not ' | '! ' )? (value | comp | in_array)
comp        = value ('==' | '!=' | '<' | '>' | '<=' | '>=' | '=~') value
value       = (jsonpath | childpath | number | string | boolean | regpattern | null | length)
length      = (jsonpath | childpath) '.length'
in_array    = value 'in' '[' value (',' value)* ']'
```

¹`var_name`: регулярное выражение примерно переводится как «любое допустимое имя переменной JavaScript», плюс некоторые особенности, такие как имена, начинающиеся с цифр или содержащие дефисы (`-`).

### Ограничения по спецификации:
* Внутреннее значение jsonpath не может содержать or, and или какой-либо компаратор.
* Jsonpaths в значении возвращает первый элемент набора или, false если результата нет.
* Boolean операции нельзя группировать с помощью скобок.
* `and` запускаются перед `or`. Это означает, что `a and 1 = b or c != d` это то же самое, что
   `(a and 1) or (c != d)`

__`.length` Оператор__ может быть использован для:
* Получите количество дочерних элементов узла в JsonObject: `$..*[?(@.length > 3)]`
* Фильтр для узлов, имеющих дочерние элементы: `$..*[?(@.length)]`
* Для фильтрации узлов, не имеющие дочерних элементов (ветвлений): `$..*[?(not @.length)]`
* Проверки длины строки: `$.path.to[?(@.a.string.length > 10)]`
* Получение длины строки: `$.path.to.field.length`
* Получение размера массива: `$.path.to.array.length`
* Получить размер каждого массива внутри массива массивов: `$.path.to.array[*].array[*].length`
* Получить длину каждой строки внутри массива строк: `$.path.to.array[*].array[*].key.length`

__Компараторы__:  
`==`, `!=`, `<`, `>`, `<=`, `>=` делают то, что и ожидается (сравнивают по типу и значению).  
`=~` это компаратор регулярных выражений, сопоставляет левый операнд с шаблоном в правом операнде. Значение слева должно быть строкой , а справа — regpattern . В противном случае возвращается `false`.

JsonPath Примеры
================
Рассмотрим следующий JSON:
```json
{ "store": {
    "book": [
      { "category": "reference",
        "author": "Nigel Rees",
        "title": "Sayings of the Century",
        "price": 8.95,
        "available": true
      },
      { "category": "fiction",
        "author": "Evelyn Waugh",
        "title": "Sword of Honour",
        "price": 12.99,
        "available": false
      },
      { "category": "fiction",
        "author": "Herman Melville",
        "title": "Moby Dick",
        "isbn": "0-553-21311-3",
        "price": 8.99,
        "available": true
      },
      { "category": "fiction",
        "author": "J. R. R. Tolkien",
        "title": "The Lord of the Rings",
        "isbn": "0-395-19395-8",
        "price": 22.99,
        "available": false
      }
    ],
    "bicycle": {
      "color": "red",
      "price": 19.95,
      "available": true
    }
  },
  "authors": [
    "Nigel Rees",
    "Evelyn Waugh",
    "Herman Melville",
    "J. R. R. Tolkien"
  ]
}
```

JsonPath | Result
---------|---------
`$.store.bicycle.price` | Цена велосипеда.
`$.store.book[*]` | Все книги.
`$.store.book[1,3]` | Вторая и четвертая книга.
`$.store.book[1:3]` | От второй книги к третьей.
`$.store.book[:3]` | От первой книги к третьей.
`$.store.book[x:y:z]` | Книги от x до y с шагом z.
`$..book[?(@.category == 'fiction')]` | Все книги с категорией == «fiction».
`$..*[?(@.available == true)].price` | Все цены на имеющиеся товары.
`$..book[?(@.price < 10)].title` | Названия всех книг ценой ниже 10.
`$..book[?(@.author==$.authors[3])]` | Все книги "J. R. R. Tolkien"
`$[store]` | Магазин.
`$['store']` | Магазин.
`$..book[*][title, 'category', "author"]` | название, категория и автор всех книг.
`$..book[?(@.author in [$.authors[0], $.authors[2]])]` | Все книги "Nigel Rees" или "Herman Melville".
`$.store.book[?(@.category == 'fiction' and @.price < 10 or @.color == "red")].price` | Книги художественной литературы ценой ниже 10 или около того, имеющие красный цвет.  (`and` приоритетнее чем `or`)

<!-- docs-end -->