# Ошибки обновления Битрикс

## in_array(): Argument #2 ($haystack) must be of type array, bool given

добавить проверку на is_array

## count(): Argument #1 ($value) must be of type Countable|array, null given (0)

добавить проверку на is_array или is_countable

## Undefined constant "BX_RESIZE_PROPORTIONAL_ALT" (0)

ошибка в именовании константы BX_RESIZE_IMAGE_PROPORTIONAL_ALT

## Class "Bitrix\Highloadblock\HighloadBlockTable" not found (0)

подключить модуль HighloadBlockTable

```php 
\Bitrix\Main\Loader::includeModule('Highloadblock');
```