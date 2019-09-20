---
layout: post
title: 什麼時候把類別宣告成 final
author: flamerecca
tags: [blog]
---

本文章翻譯自 [Marco Pivetta](https://ocramius.github.io/) 所寫的[When to declare classes final](https://ocramius.github.io/blog/when-to-declare-classes-final/)

如果有任何問題歡迎聯繫或者發 PR

----
**<abbr title="too long, didn't read">TL;DR</abbr>:  如果你的類別有實作介面，並且沒有宣告其餘的公開函式，那就把他宣告成 `final`**

上個月針對 PHP 類別上 `final` 該什麼時候宣告，我和其他人有一些討論。

下面的對話一直重複：

1. 我問新類別的作者是否可以將該類別宣告成 `final`
1. 作者不想改，提出宣告成 `final` 會限制程式的彈性
1. 我需要解釋彈性並不來自繼承，而是來自好的抽象

顯然工程師需要更清楚的說明**哪時**要用 `final`，還有哪時要避免。

有[許多](http://verraes.net/2014/05/final-classes-in-php/)[其他](http://www.javaworld.com/article/2073649/core-java/why-extends-is-evil.html)[文章](http://stackoverflow.com/questions/137868/using-final-modifier-whenever-applicable-in-java)討論過這個主題，不過這篇文章主要可以當作一個「快速摘要」，以免未來又有人問我相同的問題。

### 哪時候該用 `final`：

`final` 應該在**任何可能的時候**使用

### 為什麼要用 `final`？

用 `final` 有很多理由：下面會依據我認為的重要性依序列舉。

#### 1. 避免超長串的死亡繼承鍊

部分工程師有著在現有的類別下，加上新的子類別來解決問題的壞習慣。

你或許就看過類似的設計：

```php
<?php

class Db { /* ... */ }
class Core extends Db { /* ... */ }
class User extends Core { /* ... */ }
class Admin extends User { /* ... */ }
class Bot extends Admin { /* ... */ }
class BotThatDoesSpecialThings extends Bot { /* ... */ }
class PatchedBot extends BotThatDoesSpecialThings { /* ... */ }
```

很顯然的，你**絕不應該**把程式設計成這樣。

上面的解法很常出現在<a href="http://c2.com/cgi/wiki?AlanKaysDefinitionOfObjectOriented" target="_blank"><abbr title="Object Oriented Programming">物件導向程式設計</abbr></a>和「<cite>透過繼承來解決問題</cite>」（或許可以稱呼為「繼承導向設計」？）搞混的工程師身上

#### 2. 鼓勵合成

一般來說，預設禁止工程師繼承物件有個好處，就是鼓勵工程師盡量多想想怎麼合成。

這樣可以減少透過繼承不斷在原有的程式內增加功能。對我來說，這是一個習慣交差了事加上[功能蔓延](https://en.wikipedia.org/wiki/Feature_creep)的狀況。

看下面的簡單例子

```php
<?php

class RegistrationService implements RegistrationServiceInterface
{
    public function registerUser(/* ... */) { /* ... */ }
}

class EmailingRegistrationService extends RegistrationService
{
    public function registerUser(/* ... */) 
    {
        $user = parent::registerUser(/* ... */);
        
        $this->sendTheRegistrationMail($user);
        
        return $user;
    }
    
    // ...
}
```

如果讓 `RegistrationService` 變成 `final`，讓 `EmailingRegistrationService` 變成 `RegistrationService` 子類別的想法就不可能出現。前面所說的愚蠢錯誤就不會發生。

```php
<?php

final class EmailingRegistrationService implements RegistrationServiceInterface
{
    public function __construct(RegistrationServiceInterface $mainRegistrationService) 
    {
        $this->mainRegistrationService = $mainRegistrationService;
    }

    public function registerUser(/* ... */) 
    {
        $user = $this->mainRegistrationService->registerUser(/* ... */);
        
        $this->sendTheRegistrationMail($user);
        
        return $user;
    }
    
    // ...
}
```

#### 3. 強迫開發者構思用的人會怎麼使用公開 API

有的開發者習慣透過繼承在原有的類別裡面增加 API：

```php
<?php

class RegistrationService implements RegistrationServiceInterface
{
    protected $db;

    public function __construct(DbConnectionInterface $db) 
    {
        $this->db = $db;
    }

    public function registerUser(/* ... */) 
    {
        // ...
        
        $this->db->insert($userData);
        
        // ...
    }
}

class SwitchableDbRegistrationService extends RegistrationService
{
    public function setDb(DbConnectionInterface $db)
    {
        $this->db = $db;
    }
}
```
This example shows a set of flaws in the thought-process that led to the `SwitchableDbRegistrationService`:

* The `setDb` method is used to change the `DbConnectionInterface` at runtime, which seems to hide a different problem being solved: maybe we need a `MasterSlaveConnection` instead?
* `setDb` 沒有在 `RegistrationServiceInterface`裡面宣告，therefore we can only use it when we strictly couple our code with the `SwitchableDbRegistrationService`, which defeats the purpose of the contract itself in some contexts.
* The `setDb` method changes dependencies at runtime, and that may not be supported by the `RegistrationService` logic, and may as well lead to bugs.
* Maybe the `setDb` method was introduced because of a bug in the original implementation: why was the fix provided this way? Is it an actual fix or does it only fix a symptom?

`setDb` 這個範例還有其他的問題。但是上面這些問題，可以很好的解釋為什麼透過 `final` 可以事先解決掉一些程式結構的錯誤。

#### 4. 強迫開發者縮小物件的公開 API

既然有很多公開方法的類別，很容易就打破<abbr title="Single Responsibility Principle">SRP</abbr> 原則，通常會導致很多工程師使用時想要覆寫這一些公開方法。

一開始就宣告成 `final` 強迫工程師去思考新的 API，並且會想到盡可能的讓它們越小越好。

#### 5. `final` 在需要的時候還是可以拓展

將新的類別宣告成 `final`，你還是可以在**真正**需要的時候拓展它。

沒有任何實際的缺點，只是你會需要和團隊的其他人解釋改變的原因，而這番討論可能會引導出更好的解法。

#### 6. `extends` 破壞封裝

除非作者刻意將類別為了繼承而設計，不然使用時，即使類別沒有宣告成 `final`，你還是要把它當作是 `final` 的。

繼承類別會破壞該類別的封裝，並且可能會導致不可預見的後果，或者 <abbr title="Backwards Compatibility">向下相容</abbr> 被破壞：`extends` 某個類別之前請三思，或者，更好的做法是，把你的類別宣告成 `final` 避免其他人需要考慮繼承。

#### 7. 你不需要這種彈性

一個我常遇到，反對 `final` 的意見，是這會導致程式碼的彈性降低。

我的反駁非常的簡單：你不需要這種彈性

* 為什麼會需要這種彈性？
* 為什麼不能自己實作該介面？
* 為什麼不能用合成的方式？
* 你有詳細確認過問題嗎？

如果確認過之後，發現確實還是需要移除 `final`，那代表你的程式碼很可能有其他的壞味道。

#### 8. 你可以改程式碼

將程式改成 `final`，你還是可以在任何你想要的時候移除它。

Since encapsulation is guaranteed to be maintained, the only thing that you have to care about is that the public API.

Now you are free to rewrite everything, as many times as you want.

### 哪時要**避免** `final`：

類別宣告成 `final` **只有在下列假設時有效果**：

1. 宣告 `final` 的類別有實作某個抽象 (介面)
1. 該類別所有的公開 API 都在該介面內

如果其中某個條件不成立，那麼因為你的專案沒有真正的依賴抽象，有可能在某個時間點，你的專案會需要繼承某物件。

An exception can be made if a particular class represents a set of constraints or concepts that are totally immutable, inflexible and global to an entire system.A good example is a mathematical operation: `$calculator->sum($a, $b)` will unlikely change over time.In these cases, it is safe to assume that we can use the `final` keyword without an abstraction to rely on first.

另一個你無法使用 `final` 的時候是在既有的類別上。除非原本的專案遵守<a href="http://semver.org/" target="_blank">語意化版本號</a>，並且你可以對該專案往前推進一個主版本號，不然無法加上 `final`。

### 試看看！

After having read this article, consider going back to your code, and if you never did so, adding your first `final` marker to a class that you are planning to implement.

You will see the rest just getting in place as expected.
