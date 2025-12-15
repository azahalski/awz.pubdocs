# Crm Bitrix
<!-- desc-start -->

## Загрузить файл в поле crm сущности

```php
use Bitrix\Crm\Service;
$arFile = \CFile::MakeFileArray($fileId); //или массив описывающий файл
$field = $factory->getFieldsCollection()->getField('field_name');
$valueFile = Service\Container::getInstance()->getFileUploader()->saveFileTemporary($field, $arFile);
$item->set('field_name', $valueFile);
```



<!-- desc-end -->