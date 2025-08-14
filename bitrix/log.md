# Файловый PSR лог в Bitrix
<!-- desc-start -->

## 1. Создаем trait с синглтоном, для удобства обращения к объекту лога

файл /bitrix/php_interface/AwzBaseInstance.php

```
<?php
trait AwzBaseInstance {

    use \Psr\Log\LoggerAwareTrait;

    private function __construct(){}

    public static function getInstance(string $siteId='')
    {
        static $_instance;
        if($_instance === null){
            $_instance = new self();
        }
        return $_instance;
    }

    public function getLogger(): ?\Psr\Log\LoggerInterface
    {
        if ($this->logger === null)
        {
            $logger = \Bitrix\Main\Diag\Logger::create(static::class, [$this]);
            if ($logger !== null)
            {
                $this->setLogger($logger);
            }
        }
        return $this->logger;
    }

}
```

### и подключаем trait в init.php

```php 
include_once($_SERVER['DOCUMENT_ROOT']."/bitrix/php_interface/AwzBaseInstance.php");
```

## 2. Cоздаем класс-логер

Можно разместить в init.php, чтобы был доступен везде

```php
class CustomPayLogger {
    use AwzBaseInstance;
}
```

## 3. Включаем логирование

/bitrix/.settings_extra.php - [документация Битрикс](https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=43&LESSON_ID=15330&LESSON_PATH=3913.3516.5062.15330)

```php 
<?php
$finalSett = [];
$finalSett['loggers'] = $finalSett['loggers'] ?? [];
$finalSett['loggers']['readonly'] = $finalSett['loggers']['readonly'] ?? true;
$finalSett['loggers']['value'] = $finalSett['loggers']['value'] ?? [];

$finalSett['loggers']['value']['CustomPayLogger'] = [
    'className' => '\\Bitrix\\Main\\Diag\\FileLogger',
    'constructorParams' => function(){
        return [$_SERVER['DOCUMENT_ROOT'].'/local/logs/awz.log.CustomPayLogger.log'];
    },
    'level' => \Psr\Log\LogLevel::DEBUG
];
return $finalSett;
```

## 4. Пример использования

```php 
define("STOP_STATISTICS", true);
define('NO_AGENT_CHECK', true);
define('NOT_CHECK_PERMISSIONS', true);
define("DisableEventsCheck", true);
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");

$log = CustomPayLogger::getInstance()->getLogger();
$request = \Bitrix\Main\Application::getInstance()->getContext()->getRequest();
$debugInfo = http_build_query($request->toArray());

$log?->debug(
    "[sale_ps_result.php - request] | {date} | {debugInfo} | {php_input}\n",
    [
        'debugInfo' => $debugInfo,
        'php_input' => empty($debugInfo) ? file_get_contents('php://input') : '',
    ]
);
```

<!-- desc-end -->