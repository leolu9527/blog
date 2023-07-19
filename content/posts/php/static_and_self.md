---
title: "PHP 中的 new static 和 new self"
date: 2023-04-03T09:26:57+08:00
description: ""
tags: []
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: PHP
comment: false
draft: true
---

在使用 PHP 时，“new static”和“new self”都可以创建对象。但是，它们实际上有一些区别。

- “new static”在运行时创建对象，这意味着在执行子类时可以创建与子类关联的对象。
- “new self”在编译时创建对象，这意味着在执行子类时也会创建与父类关联的对象。

```php
class A {
    public static function get_self() {
        return new self();
    }

    public static function get_static() {
        return new static();
    }
}

class B extends A {}

echo get_class(B::get_self());  // A
echo get_class(B::get_static()); // B
echo get_class(A::get_self()); // A
echo get_class(A::get_static()); // A
```

总结

- new static与new self都可以用于在对象内创建新实例。
- new static创建的实例类由调用所处的类决定。
- new self创建的实例类始终为当前类本身。