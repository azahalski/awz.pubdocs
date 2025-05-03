# Подпись параметра ключом
<!-- desc-start -->

Действие подписывает параметры секретным ключом по стандартам БУС.

**Внимание!** Параметры передаются в открытом доступе, а само действие предназначено для защиты от подмены этих данных.

## 1. параметры действия

| Параметр                      | Тип     | Обязателен |
|-------------------------------|---------|------------|
| Строка для подписи            | string  | да         |
| Секретная соль для подписи    | string  | да         |
| Секретный ключ для подписи    | string  | нет        |

## 2. возвращаемые значения

| код       | Тип       | Описание                                  |
|-----------|-----------|-------------------------------------------|
| sign      | string    | Подпись                                   |
| errorText | string    | Текст ошибки, если действие не отработало |

## 3. Примеры

### 3.1 Защитим ссылку с ид сделки от подмены

#### 3.1.1. Добавим действие в БП по сделке

![](https://zahalski.dev/images/modules/awz.robots/001.png)

параметр id={{ID}}
секретный ключ: `sdfgfgdfgtest`
секретная соль: `sdfgfgdfgtest`

#### 3.1.2. получим значение 

sign = `aWQ9MTg=.50c9c734613731259eee7304a8d300b5ce53f8b1e2bb39151e92a2e39d30a913`

#### 3.1.3. отправим данные в наш БУС по хуку

Добавляем действие исходящий вебхук, например
`https://api.zahalski.dev/local/test.php?sign={=A22_1271_21243_61926:sign}`

#### 3.1.4. на принимающей стороне

```php
define('BX_SECURITY_SESSION_VIRTUAL', true);
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");

$signVal = '';
try{
    $saltKey = 'sdfgfgdfgtest';
    $signKey = 'sdfgfgdfgtest';
    $sign = $_REQUEST['sign'];

    $signer = new \Bitrix\Main\Security\Sign\Signer();
    $signer->setKey($signKey);
    $signVal = base64_decode($signer->unsign($sign, $saltKey));
}catch (\Exception $e){
}

if($signVal){
    echo $signVal; //id=18
}else{
    //ошибка проверки подписи
}
```

или

```php
$saltKey = 'sdfgfgdfgtest';
$signKey = 'sdfgfgdfgtest';
$sign = $_REQUEST['sign'];
$signData = explode('.',$sign);

$signVal = '';
if(is_array($signData) && $signData[0] && count($signData)>1){
    $genHash = bin2hex(hash_hmac("sha256", $signData[0], $saltKey.$signKey, true));

    if($genHash === $signData[1]){
        //проверка пройдена
        $signVal = base64_decode($signData[0]);
    }
}

if($signVal){
    echo $signVal; //id=18
}else{
    //ошибка проверки подписи
}
```

или сделать запрос на api приложения

https://api.zahalski.dev/bitrix/services/main/ajax.php?action=awz:bxapi.api.fullactivity.check

| параметр | Тип       | Описание                          |
|----------|-----------|-----------------------------------|
| salt     | string    | Соль для подписи                  |
| key      | string    | Ключ для подписи                  |
| sign     | string    | Подпись сгенерированная действием |

например: 
&key=sdfgfgdfgtest&salt=sdfgfgdfgtest&sign=aWQ9MTg=.2dc36a9cc5c1522a8448000c3eb059dd6dbb95b87ba394eff46df389cfd664fd

успешный ответ:
```json
{"status":"success","data":"id=18","errors":[]}
```
ошибка проверки:
```json
{"status":"error","data":null,"errors":[{"message":"bad signature","code":0,"customData":null}]}
```

* если не задать `key` - ключ для подписи, то проверка будет пройдена на секретном ключе приложения (известному только разработчику)

<!-- desc-end -->

