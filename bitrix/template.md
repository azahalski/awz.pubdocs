# Плюшки шаблонов Bitrix
<!-- desc-start -->

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

<!-- desc-end -->