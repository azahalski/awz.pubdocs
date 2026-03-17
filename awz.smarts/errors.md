# Ошибки в работе приложения
<!-- desc-start -->
## Не работает встройка на редактирование в коробке Битрикс24

для работы встроек в urlrewrite.php должны быть правила 
(порядок расположения правил важен)
```php 
500 =>
    array (
        'CONDITION' => '#^/marketplace/view/([a-zA-Z0-9\\.\\_]+)/.*#',
        'RULE' => 'APP=$1',
        'ID' => 'bitrix:app.layout',
        'PATH' => '/marketplace/view/index.php',
        'SORT' => 100,
    ),
501 =>
    array (
        'CONDITION' => '#^/market/view/([a-zA-Z0-9\\.\\_]+)/.*#',
        'RULE' => 'APP=$1',
        'ID' => 'bitrix:app.layout',
        'PATH' => '/marketplace/view/index.php',
        'SORT' => 100,
    ),
502 =>
    array (
        'CONDITION' => '#^/extranet/marketplace/#',
        'RULE' => NULL,
        'ID' => 'bitrix:rest.marketplace',
        'PATH' => '/extranet/marketplace/index.php',
        'SORT' => 100,
    ),
503 =>
    array (
        'CONDITION' => '#^/marketplace/#',
        'RULE' => '',
        'ID' => 'bitrix:rest.marketplace',
        'PATH' => '/marketplace/index.php',
        'SORT' => 100,
    ),
```
<!-- desc-end -->