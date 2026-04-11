<!-- main-start -->

## Дополнительная фильтрация передаваемых данных

В параметрах метода можно добавлять фильтр на удаление и добавление ключей к парааметрам метода

Ниже функция на php используемая в решении

```php 
private function applyJsonChanges(array $target, array $changes): array {
    foreach ($changes as $key => $value) {
        $prefix = substr($key, 0, 1);

        if ($prefix === '-') {
            // Удаляем ключ (берем имя без первого символа)
            unset($target[substr($key, 1)]);
        }
        elseif ($prefix === '+') {
            // Добавляем или полностью перезаписываем значение
            $target[substr($key, 1)] = $value;
        }
        elseif (is_array($value)) {
            // Если префикса нет, идем вглубь только если это массив
            // Если ключа в оригинале нет, создаем пустой массив для обхода
            $target[$key] = $target[$key] ?? [];

            if (is_array($target[$key])) {
                $target[$key] = $this->applyJsonChanges($target[$key], $value);
            }
        }
        // Если префикса нет и это не массив (строка/число) — ничего не делаем
    }
    return $target;
}
```

## Примеры фильтров

### Базовое удаление и добавление
   Самый частый случай: убираем старое поле и принудительно записываем новое.

```php 
$old = [
    'id' => 10,
    'status' => 'active',
    'tags' => ['web', 'php']
];

$changes = [
    '-status' => '',      // Удалит 'status'
    '+role' => 'admin',   // Добавит 'role'
    'tags' => []          // Ничего не сделает (нет префикса и массив пустой)
];

$result = applyJsonChanges($old, $changes);
/* 
Результат:
[
    'id' => 10,
    'tags' => ['web', 'php'],
    'role' => 'admin'
]
*/
```

### Глубокое обновление (Вложенность)
Здесь мы меняем только одно поле внутри настроек, не затирая остальные.

```php 
$old = [
    'settings' => [
        'theme' => 'dark',
        'notifications' => [
            'email' => true,
            'sms' => false
        ]
    ]
];

$changes = [
    'settings' => [
        'notifications' => [
            '+sms' => true,      // Меняем только SMS на true
            '-email' => ''       // Удаляем Email
        ]
    ],
    'theme' => 'light'           // ИГНОРИРУЕТСЯ (нет префикса +)
];

$result = applyJsonChanges($old, $changes);
/*
Результат:
[
    'settings' => [
        'theme' => 'dark',       // Сохранилось
        'notifications' => [
            'sms' => true        // Обновилось
        ]
    ]
]
*/

```

### Полная перезамена ветки
Если нужно не «просачиваться» внутрь, а бахнуть весь блок целиком.

```php 
$old = [
    'profile' => [
        'name' => 'Ivan',
        'city' => 'Moscow'
    ]
];

$changes = [
    '+profile' => [              // Префикс + заменяет весь массив profile
        'name' => 'Dmitry'
    ]
];

$result = applyJsonChanges($old, $changes);
/*
Результат:
[
    'profile' => [
        'name' => 'Dmitry'       // 'city' исчез, так как была полная перезапись
    ]
]
*/

```

### Создание пути «на лету»
Если в исходном массиве ключа не было, но мы идем «вглубь», функция создаст цепочку.

```php
$old = [];

$changes = [
    'meta' => [
        'seo' => [
            '+title' => 'My Page'
        ]
    ]
];

$result = applyJsonChanges($old, $changes);
/*
Результат:
['meta' => ['seo' => ['title' => 'My Page']]]
*/

```

<!-- main-end -->