# Плюшки шаблонов Bitrix
<!-- desc-start -->

## Подмена языковых переменных

разместить файл в bitrix/php_interface/user_lang/ru/lang.php

```php 
<?
$MESS['/bitrix/modules/awz.autform/lang/ru/lib/handlers.php']["AWZ_AUTFORM_HANDLERS_MLIFE_SMS_PHONE_ERR"] = 'Вводите телефон в формате: 79217776655';
$MESS['/bitrix/modules/awz.autform/lang/ru/lib/handlersv2.php']["AWZ_AUTFORM_HANDLERSV2_PHONE_ERR"] = 'Вводите телефон в формате: 79217776655';
```

## Прокинуть данные из кеша в component_epilog.php

### result_modifier.php

```php
$cp = $this->__component; // объект компонента
if (is_object($cp))
{
    $cp->arResult['OG_IMAGE'] = $pictSrc;
    $cp->arResult['OG_DESCR'] = \HTMLToTxt($arResult['PREVIEW_TEXT']);
    $cp->arResult['OG_TITLE'] = $arResult['NAME'];
    $cp->SetResultCacheKeys(['OG_IMAGE','OG_DESCR','OG_TITLE']);
}
```

### component_epilog.php

```php
if(isset($arResult['OG_TITLE']) && $arResult['OG_TITLE']){
    $APPLICATION->SetPageProperty('og-title', '<meta property="og:title" content="'.$arResult['OG_TITLE'].'" />');
}
```

## Подключение компонента внутри кеша (template.php) со скриптами

```php
//четвертый параметр объект родительского компонента $this
//вызов компонента всегда оборачиваем в <??>, чтобы html редактор ничего не ломал
?>
<?
$APPLICATION->IncludeComponent("awz:cookies.sett",".default",
    [], $this, array("HIDE_ICONS"=>"Y")
);
?>
```

## Буферизация контента через отложенные функции

### template.php



```php 
<?$APPLICATION->AddViewContent('min_description','content');?>
```

```php
$this->SetViewTarget('min_description');
?>
<div class="row">
    <div class="desc">
    <?=$arResult["PREVIEW_TEXT"]?>
    
</div>
</div>
<?php
$this->EndViewTarget();
```

### Вывод вне кеша

```php
<?$APPLICATION->ShowViewContent('min_description');?>
```

### Запрет перемещения скрипта

```
<script type=”text/javascript” data-skip-moving="true" src=""></script>
```

## Строки

### Безопасный вывод данных

```php
alt="<?=htmlspecialcharsEx($arResult['NAME'])?>"
```

<!-- desc-end -->