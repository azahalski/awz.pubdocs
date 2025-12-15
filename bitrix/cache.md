# Кеширование в Bitrix
<!-- desc-start -->

## Стандартное кеширование по ключу

```php 
$cacheId = md5(serialize(['list','1']));
$cacheTtl = 86400;
$cacheDir = "/awz/sections"; //без слеша на конце

$obCache = \Bitrix\Main\Data\Cache::createInstance();

if ($obCache->initCache($cacheTtl, $cacheId, $cacheDir)) {
    $vars = $cache->getVars(); 
}
elseif ($obCache->startDataCache()) {
    // получение данных
    $vars = [];
    if(empty($vars)){
        $obCache->abortDataCache(); //Отменяет создание текущего кэша
    }
    $obCache->endDataCache($vars);
}
```

## кеширование с добавлением тегированного кеша

```php 
$cacheId = md5(serialize(['list','1']));
$cacheTtl = 86400;
$cacheDir = "/awz/sections"; //без слеша на конце

$obCache = \Bitrix\Main\Data\Cache::createInstance();
$taggedCache = \Bitrix\Main\Application::getInstance()->getTaggedCache();

if ($obCache->initCache($cacheTtl, $cacheId, $cacheDir)) {
    $vars = $cache->getVars(); 
}
elseif ($obCache->startDataCache()) {
    $taggedCache->startTagCache($cachePath);
    $taggedCache->registerTag('iblock_5');
    // получение данных
    $vars = [];
    if(empty($vars)){
        $obCache->abortDataCache(); //Отменяет создание текущего кэша
    }
    $taggedCache->endTagCache();
    $obCache->endDataCache($vars);
}
```

## Очистка стандартного кеша

```php 
$cacheId = md5(serialize(['list','1']));
$cacheDir = "/awz/sections"; //без слеша на конце
$obCache = \Bitrix\Main\Data\Cache::createInstance();
$obCache->clean($cacheId, $cacheDir);
```

## Очистка тегированного кеша

```php 
$taggedCache = \Bitrix\Main\Application::getInstance()->getTaggedCache();
$taggedCache->clearByTag('iblock_5');
```

<!-- desc-end -->