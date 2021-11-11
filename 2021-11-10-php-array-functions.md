---
title: PHP Array Functions
date: 2021-11-10 15:30:00
description: 复习或学习PHP数组相关函数
categories:
- php
tags:
- php
---



- [array_pad](#method-array-pad)
- [array_reduce](#method-array-reduce)
- [array_reverse](#method-array-reverse)

<a id="method-array-pad" href="#method-array-pad"></a>

# array_pad

> 以下翻译来自于：https://www.php.net/manual/en/function.array-pad.php

## 描述

```
array_pad(array $array, int $length, mixed $value): array
```

`array_pad()` 返回一个在`array`上用`value`填充到`length`指定长度的数组副本。如果`length`为正数时填充到数组右边，负数则在左侧。如果`length`的绝对值小于`array`的长度则不会发生填充。一次操作最多可以填充1048576个元素。

## 参数
### array
用于填充的初始数组。
### length
新数组的长度
### value
当`array`长度小于`length`用于填充的值

## 返回值
(参看描述)

## 例子
#1
```
$input = array(12, 10, 9);

$result = array_pad($input, 5, 0);
// 结果是: array(12, 10, 9, 0, 0)

$result = array_pad($input, -7, -1);
// 结果是: array(-1, -1, -1, -1, 12, 10, 9);

$result = array_pad($input, 2, "noop");
// 未填充
```

<a id="method-array-reduce" href="#method-array-reduce"></a>

# array_reduce

> 以下翻译来自于：https://www.php.net/manual/en/function.array-reduce.php

## 描述

```
array_reduce(array $array, callback $callback, mixed $initial = null): mixed
```

`array_reduce()` 遍历`array`中的每个元素并应用 `callback`方法，以便将这个数组缩减成一个值。

## 参数

### array

输入数组。

### callback

回调函数，方法签名如下：

```
callback(mixed $carry, mixed $item): mixed
```

#### carray

保存上一次遍历的值，在第一次遍历的时候保存`initial`的值。

#### item

保存当前遍历的值。

### initial

如果可选的`initial`值有效，其将会被用于开始的处理，或者当输入数组为空时作为最终值。

## 返回值
返回最终结果。
如果数组为空且`initial`未传入，`array_reduce()`返回`null`。

## 例子
### #1

```php
function sum($carry, $item)
{
    $carry += $item;
    return $carry;
}

$a = array(1, 2, 3, 4, 5);
var_dump(array_reduce($a, 'sum')); // int(15) null+1+2+3+4+5
```



<a id="method-array-reverse" href="#method-array-reverse"></a>

# array_reverse

> 以下翻译来自于：https://www.php.net/manual/en/function.array-reverse.php

## 描述

```
array_reverse(array $array, bool $preserve_keys = false): array
```

将输入`array`的元素翻转后得到一个新的数组。

## 参数

### array

输入数组。

### preserve_keys

保留数组key，当为`true`时数字键将会被保留。非数字键不受该参数影响而且总是会被保留。

## 返回值

返回被翻转后的数组。

## 例子
### #1
```
$input = array("php", 4.0, array("green", "red"));
$reversed = array_reverse($input);
$preserved = array_reverse($input, true);

print_f($input);
print_r($reversed);
print_r($preserved);
```

输出结果如下：

```
// input
Array
(
    [0] => php
    [1] => 4
    [2] => Array
        (
            [0] => green
            [1] => red
        )
)
// $reversed
Array
(
    [0] => Array
        (
            [0] => green
            [1] => red
        )
    [1] => 4
    [2] => php
)
// $preserved
Array
(
    [2] => Array
        (
            [0] => green
            [1] => red
        )
    [1] => 4
    [0] => php
)
```

### #2 当存在非数字键时

```
$input = array("php", 4.0, "name" => "shawly", array("green", "red"));
$reversed = array_reverse($input);
$preserved = array_reverse($input, true);

print_f($input);
print_r($reversed);
print_r($preserved);
```

结果如下

```
// input
Array
(
    [0] => php
    [1] => 4
    [name] => shawly
    [2] => Array
        (
            [0] => green
            [1] => red
        )
)
// $reversed
Array
(
    [0] => Array
        (
            [0] => green
            [1] => red
        )
    [name] => shawly
    [1] => 4
    [2] => php
)
// $preserved
Array
(
    [2] => Array
        (
            [0] => green
            [1] => red
        )
    [name] => shawly
    [1] => 4
    [0] => php
)
```



