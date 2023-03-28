---
title: "PHP中的依赖注入容器"
date: 2022-07-04T10:42:17+08:00
description: "PHP中基于反射的依赖注入原理"
tags: [依赖注入, 设计模式]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: PHP
comment: false
draft: false
---

## 什么是依赖注入？

在面向对象编程中，对象之间经常会有相互依赖的情况。例如，一个类可能需要使用另一个类的某个方法或属性。在这种情况下，我们可以通过将被依赖的类的实例传递给依赖类的构造函数或方法来解决依赖关系。这种方式被称为依赖注入。

依赖注入有助于解耦代码，使其更加模块化和可维护。它还使得代码更易于测试，因为我们可以轻松地替换依赖项的实现来模拟测试环境。

## 如何在PHP中实现依赖注入？

在PHP中，我们可以使用构造函数注入或方法注入来实现依赖注入。以下是一个使用构造函数注入的示例：

```php

class ClassA {
    private $classB;

    public function __construct(ClassB $classB) {
        $this->classB = $classB;
    }

    public function doSomething() {
        // 使用 $this->classB 中的方法和属性
    }
}

class ClassB {
    // ClassB 的方法和属性
}

$classB = new ClassB();
$classA = new ClassA($classB);

```

在上面的示例中，ClassA 的构造函数需要一个 ClassB 的实例作为参数。这使得 ClassA 可以使用 ClassB 中的方法和属性。

## 实现一个基于PHP反射机制实现的依赖注入容器，并实现`PSR-11`规范

上述逐个手工实例化依赖项的方法过于繁琐，不适合实际使用，下面是一个基于反射机制的简单实现：

```php
# file:Container.php

<?php

namespace Leolu9527\Container;

use Psr\Container\ContainerExceptionInterface;
use Psr\Container\ContainerInterface;
use Psr\Container\NotFoundExceptionInterface;

class Container implements ContainerInterface
{
    /**
     * @var array<string, callable|object|string>
     */
    private array $definitions;

    /**
     * @var array
     */
    private array $services = [];

    /**
     * Create a container object with a set of definitions.
     * @param array<string, callable|object|string> $definitions
     */
    public function __construct(array $definitions = [])
    {
        $this->definitions = $definitions;
    }

    public function set(string $id, $definition)
    {
        if (!array_key_exists($id, $this->definitions)) {
            $this->definitions[$id] = $definition;
        }
    }

    /**
     * @inheritDoc
     */
    public function get(string $id)
    {
        if (array_key_exists($id, $this->services)) {
            return $this->services[$id];
        }

        if (!$this->has($id)) {
            throw new NotInContainerException(sprintf(
                'There is not service with id "%s" in the container.',
                $id,
            ));
        }

        $definition = $this->definitions[$id];

        $service = $this->services[$id] = $this->getService($id, $definition);

        return $service;
    }

    /**
     * @inheritDoc
     */
    public function has(string $id): bool
    {
        return array_key_exists($id, $this->definitions);
    }

    private function getService($id, $definition)
    {
        if (is_callable($definition)) {
            try {
                return $definition();
            } catch (\Throwable $e) {
                throw new ContainerException(
                    sprintf('Error while invoking callable for "%s"', $id),
                    0,
                    $e,
                );
            }
        } elseif (is_object($definition)) {
            return $definition;
        } elseif (is_string($definition)) {
            if (class_exists($definition)) {
                try {
                    return $this->buildInstance($definition);
                } catch (\Throwable $e) {
                    throw new ContainerException(sprintf('Could not instantiate class "%s"', $id), 0, $e);
                }
            }

            throw new ContainerException(sprintf(
                'Could not instantiate class "%s". Class was not found.',
                $id,
            ));
        } else {
            throw new ContainerException(sprintf(
                'Invalid type for definition with id "%s"',
                $id,
            ));
        }
    }

    /**
     * @throws NotFoundExceptionInterface
     * @throws \ReflectionException
     * @throws ContainerExceptionInterface
     */
    private function buildInstance(string $className)
    {
        $reflectionClass = new \ReflectionClass($className);

        if (!$reflectionClass->isInstantiable()) {
            throw new \ReflectionException("{$className} is not instantiable");
        }

        $constructor = $reflectionClass->getConstructor();

        if ($constructor === null) {
            return $reflectionClass->newInstance();
        }

        $dependencies = $this->getDependencies($constructor->getParameters());

        return $reflectionClass->newInstanceArgs($dependencies);
    }


    /**
     * @param \ReflectionParameter[] $parameters
     * @return array
     * @throws ContainerExceptionInterface
     * @throws NotFoundExceptionInterface
     * @throws \ReflectionException
     */
    private function getDependencies($parameters): array
    {
        $dependencies = [];

        foreach ($parameters as $parameter) {
            $dependency = $parameter->getType();

            if (($dependency !== null && $dependency->isBuiltin()) || $dependency === null) {
                if ($parameter->isDefaultValueAvailable()) {
                    $dependencies[] = $parameter->getDefaultValue();
                } else {
                    throw new \ReflectionException("Unable to resolve dependency for parameter {$parameter->getName()}");
                }
            } else {
                $dependencies[] = $this->get($dependency->getName());
            }
        }

        return $dependencies;
    }
}
```

## 附录
代码：[https://github.com/leolu9527/php/blob/main/src/Container/Container.php](https://github.com/leolu9527/php/blob/main/src/Container/Container.php)

## References
1. https://www.php-fig.org/psr/psr-11/
2. https://laravel.com/
3. https://php-di.org/