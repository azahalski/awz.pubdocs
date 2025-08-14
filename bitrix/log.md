# Файловый PSR лог в Bitrix
<!-- desc-start -->

## 0. Пример использования

```php 
define("STOP_STATISTICS", true);
define('NO_AGENT_CHECK', true);
define('NOT_CHECK_PERMISSIONS', true);
define("DisableEventsCheck", true);
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");

CustomPayLogger::getInstance()->logRequest();

```

Если не нужен класс расширение, то можно обращаться по названию лога и перейти сразу к пункту 3

```php
define("STOP_STATISTICS", true);
define('NO_AGENT_CHECK', true);
define('NOT_CHECK_PERMISSIONS', true);
define("DisableEventsCheck", true);
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");

$log = \Bitrix\Main\Diag\Logger::create('CustomPayLogger', [null]);
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

## 1. Создаем trait с синглтоном, для удобства обращения к объекту лога

файл /bitrix/php_interface/AwzBaseInstance.php

```
<?php
trait AwzBaseInstance {

    use \Psr\Log\LoggerAwareTrait;

    public $requestId;

    private function __construct(){
        $this->requestId = md5(time().\Bitrix\Main\Security\Random::getString(10));
    }

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

trait awzBaseInstanceFunc {

    public function logRequest(string $name = "request"){

        $request = \Bitrix\Main\Application::getInstance()->getContext()->getRequest();
        $debugInfo = http_build_query($request->toArray());

        $this->getLogger()?->debug(
            "{requestId} | {name} | {date} | {debugInfo} | {php_input}\n",
            [
                'requestId'=>$this->requestId,
                'name'=>$name,
                'debugInfo' => $debugInfo,
                'php_input' => empty($debugInfo) ? file_get_contents('php://input') : '',
            ]
        );

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
    use awzBaseInstanceFunc;
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



<!-- desc-end -->