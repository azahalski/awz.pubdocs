# Плюшки шаблонов Bitrix
<!-- desc-start -->

## текущая страница

```php
$context = \Bitrix\Main\Application::getInstance()->getContext();
$uri = new \Bitrix\Main\Web\Uri($context->getRequest()->getRequestUri());
$uri->getPath();
```

<!-- desc-end -->