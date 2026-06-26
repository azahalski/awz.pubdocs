# Критическая уязвимость в модулях отправки смс

<!-- desc-start -->

**Модули подверженные уязвимости**:

mlife.smsservices, targetsms.sms, sms96ru.sms

В старой реализации функции compileTemplate отсутствовала фильтрация пользовательских данных, 
поступающих через макросы ($macros). 
Злоумышленник мог передать опасный PHP-код или символы управления переменными внутри макросов. 
Поскольку далее шаблон обрабатывался методом self::executePhp(), это приводило к произвольному исполнению кода (RCE) на сервере.

**Кто находится в зоне риска:** 
Уязвимости подвержены те пользователи, которые используют в смс шаблоне макросы с данных без дополнительной фильтрации, отправленные пользователем (например, имя).

## Пошаговое руководство по исправлению

### Шаг 1: Откройте целевой файл

Перейдите в директорию вашего проекта и откройте для редактирования следующий файл:bitrix/modules/<модуль>/lib/events.php

### Шаг 2: Найдите старый метод

Найдите в коде уязвимую функцию compileTemplate. Она выглядит так:

```php 
public static function compileTemplate($template, &$macros){
    $arParams = array();
    foreach($macros as $k=>$v){
        $arParams[str_replace("#","",$k)] = $v;
    }
    $template = str_replace(array_keys($macros), $macros, $template);
    $template = self::executePhp($template,$macros,$arParams);
    foreach($arParams as $k=>$v){
        $macros['#'.$k.'#'] = $v;
    }
    //$template = preg_replace('/(\#[^#]+\#)/is',"",$template);
    return $template;
}
```

### Шаг 3: Замените код на безопасную версию

Замените старое тело функции на новый безопасный вариант. В нем реализована многоуровневая очистка данных (удаление PHP-тегов, знаков доллара и экранирование спецсимволов):

```php 
public static function compileTemplate($template, &$macros){
    $arParams = array();
    foreach($macros as $k=>&$v){
        if(is_array($v)) continue;
        $arParams[str_replace("#","",$k)] = $v;
        
        // 2. УДАЛЯЕМ теги <?php
        $vClean = preg_replace('/<\?(php)?/i', '', $v);
        $vClean = str_replace('?>', '', $vClean);
        
        // 3. УДАЛЯЕМ знак доллара (запрет вызова переменных)
        $vClean = str_replace('$', '', $vClean);
        
        // 4. ЭКРАНИРУЕМ кавычки и теги через Битрикс-функцию
        $v = \htmlspecialcharsEx($vClean);
    }
    unset($v);
    
    // ПРОВЕРКА: Вызываем executePhp только при наличии PHP-кода в шаблоне
	if (stripos($template, '<?') !== false) {
		$template = str_replace(array_keys($macros), $macros, $template);
		$template = self::executePhp($template, $macros, $arParams);
	}else{
		$template = str_replace(array_keys($macros), $macros, $template);
	}
    foreach($arParams as $k=>$v){
        $macros['#'.$k.'#'] = $v;
    }
    //$template = preg_replace('/(\#[^#]+\#)/is',"",$template);
    return $template;
}
```

## Я ничего не понял как исправлять, и касается ли это меня?

**Крайне рекомендуется обновить модули до последний версий**

Если вы боитесь вносить изменения в код или не можете обновить модуль через систему обновлений, 
то проверьте шаблоны смс на предмет появления в них потенциально опасных данных и подкорректируйте шаблоны.

**Например**, не безопасный шаблон:
```
#NAMЕ#, ваш заказ на сумму #PRICE# принят.
```
Замените на безопасный:
```
Ваш заказ на сумму #PRICE# принят.
```

<!-- desc-end -->