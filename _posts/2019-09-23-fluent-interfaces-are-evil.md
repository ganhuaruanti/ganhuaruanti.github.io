---
layout: post
title: 什麼時候把類別宣告成 final
author: flamerecca
tags: [blog]
---

本文章翻譯自 [Marco Pivetta](https://ocramius.github.io/) 所寫的 [Fluent Interfaces are Evil](https://ocramius.github.io/blog/fluent-interfaces-are-evil/)

如果有任何問題，歡迎聯繫或者發 PR

----

今天，我在 IRC 上面討論為什麼 [Doctrine EntityManager](https://github.com/doctrine/doctrine2/blob/2.4/lib/Doctrine/ORM/EntityManager.php)

沒有（未來也不會）實作流式接口。以下是我的想法以及說明。

<hr/>

## 提要：什麼是流式接口？

[流式接口](https://en.wikipedia.org/wiki/Fluent_interface)
是一個可以提供「可讀性更高」程式碼的設計。

概略地說，流式接口是如下的程式碼：

```php
<?php

interface {InterfaceName}
{
    /** @return self */
    public function {MethodName}({Params});
}
```

顯然的，因為 PHP 不支援回傳型態的宣告，所以我只能宣告一個 `/** @return self */` 的 [docblock](http://www.phpdoc.org/docs/latest/for-users/tags/return.html)。

（譯註：現在支援回傳型態宣告了，不過這跟本文無關。）

流式接口讓你可以串接方法的呼叫，當你對同一個物件進行多個操作時，可以輸入更少的字：

```php
<?php

$foo
    ->doBar()
    ->doBaz()
    ->setTaz('taz')
    ->otherCall()
    ->allTheThings();
```

----

## 什麼時候流式接口有道理？

流式接口在某些 API 上面使用很合理，像是 [QueryBuilder](https://github.com/doctrine/doctrine2/blob/2.4/lib/Doctrine/ORM/QueryBuilder.php)，或者其他的 builder 都很適合。特別是, especially when it comes to putting together nodes into a hierarchical structure.

下面是一個使用流式接口的好範例：

```php
<?php

$queryBuilder
    ->select('u')
    ->from('User u')
    ->where('u.id = :identifier')
    ->orderBy('u.name', 'ASC')
    ->setParameter('identifier', 100);
```

----

## 流式接口有什麼問題？

我發現了流式接口的一些問題。以下依據重要性列出這些問題：

1. 流式接口破壞[封裝](https://en.wikipedia.org/wiki/Encapsulation_%28object-oriented_programming%29)
1. 流式接口破壞裝飾者模式，有時會破壞合成。
1. 流式接口比較難 Mock
1. 流式接口讓差異比對更難讀
1. 流式接口讓可讀性更差（個人意見）
1. 流式接口在開發早期，會破壞向下相容

<hr/>

## 流式接口破壞封裝

流式接口的概念基於以下假設：

>在流式接口裡面，方法的回傳值，會是運行該方法的物件本身。

首先，**假設**語言設計上無法保證的事情，本身就是個問題。

Additionally, in OOP, you cannot rely on the identity of the returned value of an object, but just on its
interface.


<p>
    What does that mean? Let's make an example with a <code>Counter</code> interface:
</p>

```php
<?php

interface Counter
{
    /** @return self */
    public function count();

    /** @return int */
    public function getCount();
}
```

<p>Here's a fluent implementation of the interface:</p>

```php
<?php

class FluentCounter implements Counter
{
    private $count = 0;

    public function count()
    {
        $this->count += 1;

        return $this;
    }

    public function getCount()
    {
        return $this->count;
    }
}
```

<p>
    Here's an Immutable implementation of the interface:
</p>

```php
<?php

class ImmutableCounter implements Counter
{
    private $count;

    public function __construct($count = 0)
    {
        $this->count = (int) $count;
    }

    public function count()
    {
        return new ImmutableCounter($this->count + 1);
    }

    public function getCount()
    {
        return $this->count;
    }
}
```

<p>
    Here is how you <a href="https://3v4l.org/l5rr0" target="_blank">use a <code>FluentCounter</code></a>:
</p>

```php
<?php

$counter = new FluentCounter();

echo $counter->count()->count()->count()->getCount(); // 3!
```

<p>
    Here is how you <a href="https://3v4l.org/AP62m" target="_blank">use an <code>ImmutableCounter</code></a>:
</p>

```php
<?php

$counter = new ImmutableCounter();

$counter = $counter->count()->count()->count();

echo $counter->getCount(); // 3!
```

<p>
    We managed to implement an immutable counter even though the author of <code>Counter</code> maybe
    assumed that all implementations should be mutable.
</p>
<p>
    The same can be seen in the opposite direction: interface author may want to have all implementations
    immutable, but then people implement a mutable version of it.
</p>
<p>
    Turns out that the only correct way of
    <a href="https://3v4l.org/fILUc" target="_blank">using such an interface</a>
    is the "immutable" way, so:
</p>

```php
<?php

$counter = $counter->count()->count()->count();

echo $counter->getCount(); // 3!
```

<p>
    This ensures that <code>FluentCounter#getCount()</code> works as expected, but obviously defeats the
    purpose of the fluent interface.
</p>

<p>
    On top of that, there is nothing that the author of <code>Counter</code> can do to enforce either one or
    the other way of implementing the contract, and that's a limitation of the language itself (and it's
    most probably for good!).
</p>

<p>
    None of the implementors/implementations are wrong. What is wrong here is the interface by trying to
    force implementation details, therefore breaking encapsulation.
</p>

<p>Wrapping it up:</p>

<ul>
    <li>In OOP, a contract cannot guarantee the identity of a method return value</li>
    <li>Therefore, In OOP, fluent interfaces cannot be guaranteed by a contract</li>
    <li>Assumptions not backed by a contract are wrong</li>
    <li>Following wrong assumptions leads to wrong results</li>
</ul>

<hr/>

<h2>Fluent Interfaces break Decorators (and Composition)</h2>

<p>
    As some of you may know, I'm putting a lot of effort in writing libraries that
    <a href="https://github.com/Ocramius/ProxyManager/" target="_blank">generate decorators and proxies</a>.
    <br/>
    While working on those, I came to a very complex use case where I needed to build a generic wrapper around
    an object.
</p>

<p>
    I'm picking the <code>Counter</code> example again:
</p>

```php
<?php

interface Counter
{
    /** @return self */
    public function count();

    /** @return int */
    public function getCount();
}
```

<p>
    Assuming that the implementor of the wrapper doesn't know anything about the implementations of this
    interface, he goes on and builds a wrapper.
</p>
<p>
    In this example, the implementor simply writes a wrapper that echoes every time one of the methods is called:
</p>

```php
<?php

class EchoingCounter implements Counter
{
    private $counter;

    public function __construct(Counter $counter)
    {
        $this->counter = $counter;
    }

    public function count()
    {
        echo __METHOD__ . "\n";

        return $this->counter->count();
    }

    public function getCount()
    {
        echo __METHOD__ . "\n";

        return $this->counter->getCount();
    }
}
```

<p>
    Let's <a href="https://3v4l.org/i5m5r" target="_blank">try it out with our fluent counter</a>:
</p>

```php
<?php

$counter = new EchoingCounter(new FluentCounter());

$counter = $counter->count()->count()->count()->count();

echo $counter->getCount();
```

<p>
    Noticed anything wrong? Yes, the string
    <strong><code>"EchoingCounter::count"</code> is echoed only once!</strong>
</p>

<p>
    That happens because we're just trusting the interface, so the <code>FluentCounter</code> instance
    gets "un-wrapped" when we call <code>EchoingCounter::count()</code>.
</p>

<p>
    Same happens when
    <a href="https://3v4l.org/bUMJ7" target="_blank">using the <code>ImmutableCounter</code></a>
</p>

```php
<?php

$counter = new EchoingCounter(new ImmutableCounter());

$counter = $counter->count()->count()->count()->count();

echo $counter->getCount();
```

<p>Same results. Let's try to fix them:</p>

```php
<?php

class EchoingCounter implements Counter
{
    private $counter;

    public function __construct(Counter $counter)
    {
        $this->counter = $counter;
    }

    public function count()
    {
        echo __METHOD__ . "\n";

        $this->counter->count();

        return $this;
    }

    public function getCount()
    {
        echo __METHOD__ . "\n";

        return $this->counter->getCount();
    }
}
```

<p>And now let's <a href="https://3v4l.org/AilJu" target="_blank">retry</a>:</p>

```php
<?php

$counter = new EchoingCounter(new FluentCounter());

$counter = $counter->count()->count()->count()->count();

echo $counter->getCount();
```

<p>
    Works! We now see the different <code>EchoingCounter::count</code> being echoed.
    <br/>
    What about the immutable implementation?
</p>

```php
<?php

$counter = new EchoingCounter(new ImmutableCounter());

// we're using the "SAFE" solution here
$counter = $counter->count()->count()->count()->count();

echo $counter->getCount();
```

<p>
    <a href="https://3v4l.org/FuX4X" target="_blank">Seems to work</a>, but if you look closely,
    the reported count is wrong. Now the wrapper is working, but not the real logic!
</p>

<p>
    Additionally, we cannot fix this with a generic solution.
    <br/>
    We don't know if the wrapped instance is supposed to return itself or a new instance.
    <br/>
    We <em>can</em> manually fix the wrapper with some assumptions though:
</p>

~~~php
<?php

class EchoingCounter implements Counter
{
    private $counter;

    public function __construct(Counter $counter)
    {
        $this->counter = $counter;
    }

    public function count()
    {
        echo __METHOD__ . "\n";

        $this->counter = $this->counter->count();

        return $this;
    }

    public function getCount()
    {
        echo __METHOD__ . "\n";

        return $this->counter->getCount();
    }
}
~~~

<p>
    As you can see, we have to manually patch the <code>count()</code> method, but then again, this breaks
    the case when the API is neither Immutable nor Fluent.
    <br/>
    Additionally, our wrapper is now opinionated about the usage of the <code>count()</code> method, and it is
    not possible to build a generic wrapper anymore.
</p>

<p>
    I conclude this section by simply stating that fluent interfaces are problematic for wrappers, and require
    a lot of assumptions to be catched by a human decision, which has to be done per-method, based on
    assumptions.
</p>

<hr/>

<h2>Fluent Interfaces are harder to Mock</h2>

<p>
    Mock classes (at least in PHPUnit) are
    <a href="https://en.wikipedia.org/wiki/Null_Object_pattern" target="_blank">null objects</a> by default,
    which means that all the return values of methods have to be manually defined:
</p>

```php
<?php

$counter = $this->getMock('Counter');

$counter
    ->expects($this->any())
    ->method('count')
    ->will($this->returnSelf());
```

<p>
    There are 2 major problems with this:
</p>

<ol>
    <li>All fluent methods need explicit mocking</li>
    <li>We are assuming that a fluent interface is implemented, whereas the implementation may be immutable (as shown before)</li>
</ol>

<p>
    That basically means that we have to code assumptions in our unit tests (bad, and hard to follow).
</p>

<p>
    Also, we have to make decisions on the implementation of a mocked object
</p>

<p>
    The correct way of mocking the <code>Counter</code> interface would be something like:
</p>

```php
<?php

$counter = $this->getMock('Counter');

$counter
    ->expects($this->any())
    ->method('count')
    ->will($this->returnValue($this->getMock('Counter')));
```

<p>
    As you can see, we can break our code by making the mock behave differently, but still respecting the
    interface. Additionally, we need to mock every fluent method regardless of the parameters or even when
    we don't have expectations on the API.
</p>

<p>
    That is a lot of work, and a lot of <strong>wrong</strong> work to be done.
</p>

<hr/>

## Fluent Interfaces make diffs harder to read

<p>
    The problem with diffs is minor, but it's something that really
    <a href="http://knowyourmeme.com/memes/that-really-rustled-my-jimmies" target="_blank">rustles my jimmies</a>,
    especially because people abuse fluent interfaces to write giant chained method calls like:
</p>

```php
<?php

$foo
    ->addBar('bar')
    ->addBaz('baz')
    ->addTab('tab')
    ->addBar('bar')
    ->addBaz('baz')
    ->addTab('tab')
    ->addBar('bar')
    ->addBaz('baz')
    ->addTab('tab')
    ->addBar('bar')
    ->addBaz('baz')
    ->addTab('tab')
    ->addBar('bar')
    ->addBaz('baz')
    ->addTab('tab')
    ->addBar('bar')
    ->addBaz('baz')
    ->addTab('tab')
    ->addBar('bar')
    ->addBaz('baz')
    ->addTab('tab')
    ->addBar('bar')
    ->addBaz('baz')
    ->addTab('tab')
    ->addBar('bar')
    ->addBaz('baz')
    ->addTab('tab');
```

<p>
    Let's assume that a line is changed in the middle of the chain:
</p>

```shell
$ diff -p left.txt right.txt
```

```diff
*** left.txt    Fri Nov  8 15:05:09 2013
--- right.txt   Fri Nov  8 15:05:22 2013
***************
*** 11,16 ****
--- 11,17 ----
    ->addTab('tab')
    ->addBar('bar')
    ->addBaz('baz')
+     ->addBaz('tab')
    ->addTab('tab')
    ->addBar('bar')
    ->addBaz('baz')
```

<p>
    Not really useful, huh? Where do we see the object this <code>addBaz()</code> is being called on?
</p>

<p>
    This may look like nitpicking, but it makes code reviews harder, and I personally do a lot of code reviews.
</p>

<hr/>

## Fluent Interfaces are less readable

<p>
    This is a personal feeling, but when reading a fluent interface, I cannot recognize if what is going on
    is just a massive violation of the
    <a href="https://en.wikipedia.org/wiki/Law_of_Demeter" target="_blank">Law of Demeter</a>, or if we're
    dealing with the same object over and over again.
    <br/>
    I'm picking an obvious example to show where this may happen:
</p>

```php
<?php
return $queryBuilder
    ->select('u')
    ->from('User u')
    ->where('u.id = :identifier')
    ->orderBy('u.name', 'ASC')
    ->setFirstResult(5)
    ->setParameter('identifier', 100)
    ->getQuery()
    ->setMaxResults(10)
    ->getResult();
```

<p>
    This one is quite easy to follow: <code>getQuery()</code> and <code>getResult()</code> are returning
    different objects that have a different API.
    <br/>
    The problem occurs when a method does not look like a getter:
</p>


```php
<?php
return $someBuilder
    ->addThing('thing')
    ->addOtherThing('other thing')
    ->compile()
    ->write()
    ->execute()
    ->gimme();
```

<p>
    Which of these method calls is part of a fluent interface? Which is instead
    returning a different object? You can't really tell that...
</p>

<hr/>


## Fluent Interfaces cause <abbr title="Backwards Compatibility">BC</abbr> breaks

<p>
    Declaring an <code>@return void</code> type allows you to change the method behavior later on.
</p>

<p>
    Eagerly declaring a <code>@return self</code> precludes that choice, and changing the signature of the
    method will inevitably result in a <abbr title="Backwards Compatibility">BC</abbr> break.
</p>

<p>
    Using <code>void</code> as a default return type allows for more flexible and rapid design while 
    we are prototyping around a piece of our codebase.
</p>

<hr/>

## 結論

<p>
    I know the article is titled <q>"Fluent Interfaces are Evil"</q>, but that doesn't mean it's an absolute.
</p>

<p>
    Fluent interfaces are useful and easy to read in <strong>some contexts</strong>.
    What I am showing here is a set of problems that raise when inheriting them or making every piece of your
    code fluent.
</p>

<p>
    I just want you to think carefully next time you want fluent interfaces in your libraries,
    especially about the downsides that I have just exposed.
</p>

<p>
    You must have a <strong>very good reason</strong> to implement a fluent interface, otherwise it's just
    a problem that you are possibly dragging into your codebase.
</p>
