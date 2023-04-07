---
title: "Laravel中的Macroable trait原理"
date: 2023-04-06T21:03:47+08:00
description: ""
tags: [Laravel]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: PHP
comment: false
draft: false
---

> Laravel 提供的 Macroable 可以在不改变类结构的情况为其扩展功能。

## 基本原理

利用PHP中的匿名函数可以动态绑定作用域来扩展类或者实例的功能。

定义类：

```php
class A
{
}
```

定义匿名函数：

```php

$sum = function (...$i) {
    $sum = 0;
    foreach ($i as $n) {
        $sum += $n;
    }
    return $sum;
};
```

使用匿名函数基类`Closure`的三个方法`bindTo`、`bind`、`call` 为类的实例添加 sum 功能

```php

$ASum = $sum->bindTo(new A(), A::class);

$ASum(1, 2, 3);  // 6

// or
// $sum->call(new A(), 1, 2, 3);

// or 静态绑定
// $staticASum = \Closure::bind($sum, null, A::class);
// $staticASum(1, 2, 3);
```

##  Laravel中的 `Macroable` trait的实现

`Illuminate\Support\Traits\Macroable`是对上述基本原理的封装，`$macros`属性数组用来保存用户添加的扩展功能

```php
trait Macroable
{
    // 保存用户添加的扩展功能
    protected static $macros = [];

    // 新注册一个扩展功能
    public static function macro($name, $macro)
    {
        static::$macros[$name] = $macro;
    }
}
```

当需要为特定类添加扩展功能时，添加`Macroable` trait

```php
class A
{
    use Macroable;
}
```

添加`sum`功能到 `A`

```php
A::macro("sum", function (...$i) {
    $sum = 0;
    foreach ($i as $n) {
        $sum += $n;
    }
    return $sum;
});
```

调用方法如下

```php
A::sum(1,2,3);

// or
(new A())->sum(1, 2, 3);
```

调用上述A不存在的方法需要使用魔术方法`__callStatic` 、`__call`

```php
public static function __callStatic($method, $parameters)
{
    if (! static::hasMacro($method)) {
        throw new BadMethodCallException(sprintf(
            'Method %s::%s does not exist.', static::class, $method
        ));
    }

    $macro = static::$macros[$method];

    if ($macro instanceof Closure) {
        $macro = $macro->bindTo(null, static::class);
    }

    return $macro(...$parameters);
}

public function __call($method, $parameters)
{
    if (! static::hasMacro($method)) {
        throw new BadMethodCallException(sprintf(
            'Method %s::%s does not exist.', static::class, $method
        ));
    }

    $macro = static::$macros[$method];

    if ($macro instanceof Closure) {
        $macro = $macro->bindTo($this, static::class);
    }

    return $macro(...$parameters);
}
```

上面两个方法同时支持扩展匿名函数和类实例对象，只需要将匿名函数换成支持函数调用的类即可

```php
class Sum
{
    public function __invoke(int ...$i): int
    {
        $sum = 0;
        foreach ($i as $n) {
            $sum += $n;
        }
        return $sum;
    }
}
```

这样使用

```php
A::macro("sum", new Sum());
A::sum(1,2,3);

// or
(new A())->sum(1, 2, 3);
```

## `mixin` 注入对象的多个方法

`mixin`静态方法通过反射机制注入对象的多个非私有方法

```php
public static function mixin($mixin, $replace = true)
{
    $methods = (new ReflectionClass($mixin))->getMethods(
        ReflectionMethod::IS_PUBLIC | ReflectionMethod::IS_PROTECTED
    );

    foreach ($methods as $method) {
        if ($replace || ! static::hasMacro($method->name)) {
            $method->setAccessible(true);
            static::macro($method->name, $method->invoke($mixin));
        }
    }
}
```

举例

```php
class Math
{
    public function sum(): Closure
    {
        return function (int ...$i) {
            $sum = 0;
            foreach ($i as $n) {
                $sum += $n;
            }
            return $sum;
        };
    }

    public function minus(): Closure
    {
        return function (int $a, $b) {
            return $a - $b;
        };
    }
}

A::mixin(new Math());

echo A::sum(1, 2, 3);  //6
echo A::minus(20, 1);  //19
```

最后给类加上标签方便IDE代码提示

```php
use Illuminate\Support\Traits\Macroable;

/**
 * @method static int sum(int ...$i)
 * @method int sum(int ...$i)
 * @method static int minus(int $a, $b)
 */
class A
{
    use Macroable;
}
```