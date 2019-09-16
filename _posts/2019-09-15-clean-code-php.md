---
layout: post
title: Clean Code PHP
author: flamerecca
tags: [blog]
---
## 目錄

  1. [介紹](#介紹)
  2. [變數](#變數)
     * [使用容易理解的變數名](#使用容易理解的變數名)
     * [同一個實體要用相同的變數名](#同一個實體要用相同的變數名)
     * [使用便於搜索的名稱 (part 1)](#使用便於搜索的名稱-part-1)
     * [使用便於搜索的名稱 (part 2)](#使用便於搜索的名稱-part-2)
     * [使用自解釋型變數](#使用自解釋型變數)
     * [避免深層嵌套，盡早返回 (part 1)](#避免深層嵌套盡早返回-part-1)
     * [避免深層嵌套，盡早返回 (part 2)](#避免深層嵌套盡早返回-part-2)
     * [少用無意義的變數名](#少用無意義的變數名)
     * [不要添加不必要上下文](#不要添加不必要上下文)
     * [合理使用參數默認值，沒必要在方法裡再做默認值檢測](#合理使用參數默認值沒必要在方法裡再做默認值檢測)
  3. [表達式](#表達式)
     * [使用恆等式](#使用恆等式)
  4. [函式](#函式)
     * [函式參數（最好少於2個）](#函式參數-最好少於2個)
     * [函式應該只做一件事](#函式應該只做一件事)
     * [函式名應體現他做了什麼事](#函式名應體現他做了什麼事)
     * [函式裡應當只有一層抽像abstraction](#函式裡應當只有一層抽像abstraction)
     * [不要用flag作為函式的參數](#不要用flag作為函式的參數)
     * [避免副作用](#避免副作用)
     * [不要寫全局函式](#不要寫全局函式)
     * [不要使用單例模式](#不要使用單例模式)
     * [封裝條件語句](#封裝條件語句)
     * [避免用反義條件判斷](#避免用反義條件判斷)
     * [避免條件判斷](#避免條件判斷)
     * [避免類型檢查 (part 1)](#避免類型檢查-part-1)
     * [避免類型檢查 (part 2)](#避免類型檢查-part-2)
     * [移除僵屍代碼](#移除僵屍代碼)
  5. [對像和數據結構 Objects and Data Structures](#對像和數據結構)
     * [使用 getters 和 setters Use object encapsulation](#使用-getters-和-setters)
     * [給對像使用私有或受保護的成員變數](#給對像使用私有或受保護的成員變數)
  6. [類](#類)
     * [少用繼承多用組合](#少用繼承多用組合)
     * [避免連貫接口](#避免連貫接口)
     * [推薦使用 final 類](#推薦使用-final-類)
  7. [類的SOLID原則](#solid)
     * [S: 單一職責原則 Single Responsibility Principle (SRP)](#單一職責原則)
     * [O: 開閉原則 Open/Closed Principle (OCP)](#開閉原則)
     * [L: 里氏替換原則 Liskov Substitution Principle (LSP)](#里氏替換原則)
     * [I: 接口隔離原則 Interface Segregation Principle (ISP)](#接口隔離原則)
     * [D: 依賴倒置原則 Dependency Inversion Principle (DIP)](#依賴倒置原則)
  8. [別寫重複代碼 (DRY)](#別寫重複代碼-dry)
  9. [翻譯](#翻譯)

## 介紹

本文參考自 Robert C. Martin 的[*Clean Code*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)  書中，針對軟體工程師的原則，適用於PHP。

這篇文章不是風格指南。 這是一個關於開發可讀、可復用並且可重構的PHP軟件指南。

並不是這裡所有的原則都必須遵循，這些原則只有很少能得到所有人的認同。

這些原則雖然只是指導，但是都是 *Clean Code* 作者多年總結出來的。

本文受到 [clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript) 的啟發

雖然很多開發者還在使用 PHP5，但是本文中的大部分示例的運行環境需要 PHP 7.1+。

## 翻譯說明

從 [php-cpm 版本](https://github.com/php-cpm/clean-code-php)的 [clean-code-php](https://github.com/jupeter/clean-code-php) 簡繁轉換過後，將用詞修正成更符合繁體中文的用法。

閱讀過程中，如果遇到各種連接失效、內容老舊、術語使用錯誤和其他翻譯錯誤等問題，歡迎大家積極提交 PR。

## **變數**

### 使用容易理解的變數名

**壞：**

```php
$ymdstr = $moment->format('y-m-d');
```

**好：**

```php
$currentDate = $moment->format('y-m-d');
```

**[⬆ 返回頂部](#目錄)**

### 同一個類型的變數要用相同的單字

**壞：**

```php
getUserInfo();
getUserData();
getUserRecord();
getUserProfile();
```

**好：**

```php
getUser();
```

**[⬆ 返回頂部](#目錄)**

### 使用便於搜索的名稱 (part 1)

程式碼是用來讀的。所以寫出可讀性高、便於搜索的程式碼至關重要。命名變數時如果沒有有意義、不好理解，那就是在傷害讀者。請讓你的程式碼便於搜索。

**壞：**
```php
// 448 ™ 干啥的?
$result = $serializer->serialize($data, 448);
```

**好：**

```php
$json = $serializer->serialize(
    $data, JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE
);
```

### 使用便於搜索的名稱 (part 2)

**壞：**

```php
class User
{
    // 7 ™ 干啥的?
    public $access = 7;
}

// 4 ™ 干啥的?
if ($user->access & 4) {
    // ...
}

// 這裡會發生什麼?
$user->access ^= 2;
```

**好：**

```php
class User
{
    const ACCESS_READ = 1;
    const ACCESS_CREATE = 2;
    const ACCESS_UPDATE = 4;
    const ACCESS_DELETE = 8;

    // 默認情況下用戶 具有讀、寫和更新權限
    public $access = self::ACCESS_READ | self::ACCESS_CREATE | self::ACCESS_UPDATE;
}

if ($user->access & User::ACCESS_UPDATE) {
    // do edit ...
}

// 禁用創建權限
$user->access ^= User::ACCESS_CREATE;
```

**[⬆ 返回頂部](#目錄)**

### 使用自解釋型變數

**壞：**

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches[1], $matches[2]);
```

**不錯：**

好一些，但強依賴於開發者對正則表達式的熟悉程度

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

[, $city, $zipCode] = $matches;
saveCityZipCode($city, $zipCode);
```

**好：**

使用帶名字的子規則，不用懂正則也能看的懂

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(?<city>.+?)\s*(?<zipCode>\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches['city'], $matches['zipCode']);
```

**[⬆ 返回頂部](#目錄)**

### 避免深層嵌套，盡早返回 (part 1)

太多的if else語句通常會導致你的程式碼難以閱讀，直白的程式碼優於隱晦的

**糟糕：**

```php
function isShopOpen($day): bool
{
    if ($day) {
        if (is_string($day)) {
            $day = strtolower($day);
            if ($day === 'friday') {
                return true;
            } elseif ($day === 'saturday') {
                return true;
            } elseif ($day === 'sunday') {
                return true;
            } else {
                return false;
            }
        } else {
            return false;
        }
    } else {
        return false;
    }
}
```

**好：**

```php
function isShopOpen(string $day): bool
{
    if (empty($day)) {
        return false;
    }

    $openingDays = [
        'friday', 'saturday', 'sunday'
    ];

    return in_array(strtolower($day), $openingDays, true);
}
```

**[⬆ 返回頂部](#目錄)**

### 避免深層嵌套，盡早返回 (part 2)

**糟糕的：**

```php
function fibonacci(int $n)
{
    if ($n < 50) {
        if ($n !== 0) {
            if ($n !== 1) {
                return fibonacci($n - 1) + fibonacci($n - 2);
            } else {
                return 1;
            }
        } else {
            return 0;
        }
    } else {
        return 'Not supported';
    }
}
```

**好：**

```php
function fibonacci(int $n): int
{
    if ($n === 0 || $n === 1) {
        return $n;
    }

    if ($n >= 50) {
        throw new \Exception('Not supported');
    }

    return fibonacci($n - 1) + fibonacci($n - 2);
}
```

**[⬆ 返回頂部](#目錄)**

### 少用無意義的變數名

別讓讀你的程式碼的人猜你寫的變數是什麼意思。
寫清楚好過模糊不清。

**壞：**

```php
$l = ['Austin', 'New York', 'San Francisco'];

for ($i = 0; $i < count($l); $i++) {
    $li = $l[$i];
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
  // 等等, `$li` 又代表什麼?
    dispatch($li);
}
```

**好：**

```php
$locations = ['Austin', 'New York', 'San Francisco'];

foreach ($locations as $location) {
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    dispatch($location);
}
```

**[⬆ 返回頂部](#目錄)**

### 不要添加不必要上下文

如果從你的類別名、對像名已經可以得知一些訊息，就別在變數名內重複。

**壞：**

```php
class Car
{
    public $carMake;
    public $carModel;
    public $carColor;

    //...
}
```

**好：**

```php
class Car
{
    public $make;
    public $model;
    public $color;

    //...
}
```

**[⬆ 返回頂部](#目錄)**

### 合理使用參數預設值，沒必要在方法裡再做預設值檢測

**不好：**

不好，`$breweryName` 可能為 `NULL`.

```php
function createMicrobrewery($breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```

**還行：**

比上一個好理解一些，但最好能控制變數的值

```php
function createMicrobrewery($name = null): void
{
    $breweryName = $name ?: 'Hipster Brew Co.';
    // ...
}
```

**好：**

如果你的程序只支持 PHP 7+，那你可以用 [type hinting](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration) 保證變數 `$breweryName` 不是 `NULL`.

```php
function createMicrobrewery(string $breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```

**[⬆ 返回頂部](#目錄)**

## 表達式

### [使用恆等式](http://php.net/manual/en/language.operators.comparison.php)

**不好：**

簡易對比會轉型後比對

```php
$a = '42';
$b = 42;

if( $a != $b ) {
   //這裡始終執行不到
}
```

對比 $a != $b 返回了 `FALSE` 但應該返回 `TRUE` !
字串 '42' 跟整數 42 不相等

**好：**

使用恆等判斷檢查類型和數據

```php
$a = '42';
$b = 42;

if ($a !== $b) {
    // The expression is verified
}
```

`$a !== $b` 比對會回傳 `TRUE`.

**[⬆ 返回頂部](#目錄)**

## 函式

### 函式參數（最好少於2個）

限制函式參數個數極其重要，這樣測試你的函式會更容易。有超過三個可選參數，會導致一個爆炸式組合增長，你會有成噸的獨立參數情形要測試。

無參數是理想情況。1個或2個都可以，最好避免3個。再多就需要重構了。通常，如果你的函式有超過兩個參數，說明他要處理的事太多了。

如果必須要傳入很多數據，建議封裝一個高級別對像作為參數。

**壞：**

```php
function createMenu(string $title, string $body, string $buttonText, bool $cancellable): void
{
    // ...
}
```

**好：**

```php
class MenuConfig
{
    public $title;
    public $body;
    public $buttonText;
    public $cancellable = false;
}

$config = new MenuConfig();
$config->title = 'Foo';
$config->body = 'Bar';
$config->buttonText = 'Baz';
$config->cancellable = true;

function createMenu(MenuConfig $config): void
{
    // ...
}
```

**[⬆ 返回頂部](#目錄)**

### 函式應該只做一件事

這是迄今為止，軟體工程裡最重要的一個規則。

當一個函式做超過一件事的時候，他們就難於實現、測試和理解。

當你把一個函式拆分到只剩一個功能時，他們就容易被重構，然後你的程式碼讀起來就更清晰。

光遵循這條規則，你就領先於大多數開發者了。

**壞：**

```php
function emailClients(array $clients): void
{
    foreach ($clients as $client) {
        $clientRecord = $db->find($client);
        if ($clientRecord->isActive()) {
            email($client);
        }
    }
}
```

**好：**

```php
function emailClients(array $clients): void
{
    $activeClients = activeClients($clients);
    array_walk($activeClients, 'email');
}

function activeClients(array $clients): array
{
    return array_filter($clients, 'isClientActive');
}

function isClientActive(int $client): bool
{
    $clientRecord = $db->find($client);

    return $clientRecord->isActive();
}
```

**[⬆ 返回頂部](#目錄)**

### 函式名應體現他做了什麼事

**壞：**

```php
class Email
{
    //...

    public function handle(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// 啥？handle 處理一個消息幹嘛了？是往一個文件裡寫嗎？
$message->handle();
```

**好：**

```php
class Email 
{
    //...

    public function send(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// 簡單明了
$message->send();
```

**[⬆ 返回頂部](#目錄)**

### 函式裡應當只有一層抽像（abstraction）

當你抽像層次過多時，函式處理的事情太多了。需要拆分功能來提高可重用性和易用性，以便簡化測試。

**壞：**

```php
function parseBetterPHPAlternative(string $code): void
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            // ...
        }
    }

    $ast = [];
    foreach ($tokens as $token) {
        // lex...
    }

    foreach ($ast as $node) {
        // parse...
    }
}
```

**壞：**

我們把一些事情從函式中提取出來，但是`parseBetterPHPAlternative()`方法還是很複雜，而且不利於測試。

```php
function tokenize(string $code): array
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            $tokens[] = /* ... */;
        }
    }

    return $tokens;
}

function lexer(array $tokens): array
{
    $ast = [];
    foreach ($tokens as $token) {
        $ast[] = /* ... */;
    }

    return $ast;
}

function parseBetterPHPAlternative(string $code): void
{
    $tokens = tokenize($code);
    $ast = lexer($tokens);
    foreach ($ast as $node) {
        // 解析邏輯...
    }
}
```

**好：**

最好的解決方案是把這些功能移出 `parseBetterPHPAlternative()` 所在的類別。

```php
class Tokenizer
{
    public function tokenize(string $code): array
    {
        $regexes = [
            // ...
        ];

        $statements = explode(' ', $code);
        $tokens = [];
        foreach ($regexes as $regex) {
            foreach ($statements as $statement) {
                $tokens[] = /* ... */;
            }
        }

        return $tokens;
    }
}

class Lexer
{
    public function lexify(array $tokens): array
    {
        $ast = [];
        foreach ($tokens as $token) {
            $ast[] = /* ... */;
        }

        return $ast;
    }
}

class BetterJSAlternative
{
    private $tokenizer;
    private $lexer;

    public function __construct(Tokenizer $tokenizer, Lexer $lexer)
    {
        $this->tokenizer = $tokenizer;
        $this->lexer = $lexer;
    }

    public function parse(string $code): void
    {
        $tokens = $this->tokenizer->tokenize($code);
        $ast = $this->lexer->lexify($tokens);
        foreach ($ast as $node) {
            // 解析邏輯...
        }
    }
}
```

**[⬆ 返回頂部](#目錄)**

### 不要用 flag 作為函式的參數

flag 就是在告訴大家，這個方法裡處理很多事。前面剛說過，一個函式應當只做一件事。 把不同flag的代碼拆分到多個函式裡。

**壞：**
```php
function createFile(string $name, bool $temp = false): void
{
    if ($temp) {
        touch('./temp/'.$name);
    } else {
        touch($name);
    }
}
```

**好：**

```php
function createFile(string $name): void
{
    touch($name);
}

function createTempFile(string $name): void
{
    touch('./temp/'.$name);
}
```
**[⬆ 返回頂部](#目錄)**

### 避免副作用

如果一個函式做了比讀取輸入，然後回傳一筆或多筆輸出，要更多的事情的話，那就會產生副作用。副作用可能是寫入某個檔案，修改某個全域變數，或者不小心把全部的錢都給了陌生人。

有時候，你確實會需要副作用，比方說剛剛的例子，你或許真的需要在輸出資料的同時寫入某個文件。我們該做的事情，是集中做這些事情的位置，不要有好幾個不同的類別或者函式，都寫入同一個檔案。找個 Service 物件來處理，而且只產生一個。

這邊的重點是，要避免副作用常見的陷阱，例如說毫無結構化的分享物件間的狀態，或者使用任何人都可能改寫的可變形態資料⋯⋯等

如果做到這點，你會比大多數工程師要快樂。


**壞：**

```php
// 被其他函式改變型態的全域變數
// 如果我們在 splitIntoFirstAndLastName() 之後使用這個名稱，我們會拿到陣列而不是字串，可能會導致程式當機
$name = 'Ryan McDermott';

function splitIntoFirstAndLastName(): void
{
    global $name;

    $name = explode(' ', $name);
}

splitIntoFirstAndLastName();

var_dump($name); // ['Ryan', 'McDermott'];
```

**好：**

```php
function splitIntoFirstAndLastName(string $name): array
{
    return explode(' ', $name);
}

$name = 'Ryan McDermott';
$newName = splitIntoFirstAndLastName($name);

var_dump($name); // 'Ryan McDermott';
var_dump($newName); // ['Ryan', 'McDermott'];
```

**[⬆ 返回頂部](#目錄)**

### 不要寫全域函式

在大多數語言中，污染全域的任何東西都是一個壞的做法，因為你可能和其他類別衝突，並且調用你 api 的人要一直到他們捕獲異常，才知道踩坑了。

讓我們思考一種場景：如果你想用一個陣列來設置環境，你可能會寫一個全域函式 `config()`。但是，這可能和試著做同樣事的其他類庫衝突。

**壞：**

```php
function config(): array
{
    return  [
        'foo' => 'bar',
    ]
}
```

**好：**

```php
class Configuration
{
    private $configuration = [];

    public function __construct(array $configuration)
    {
        $this->configuration = $configuration;
    }

    public function get(string $key): ?string
    {
        return isset($this->configuration[$key]) ? $this->configuration[$key] : null;
    }
}
```

加載配置，並建立 `Configuration` 類的實例

```php
$configuration = new Configuration([
    'foo' => 'bar',
]);
```

現在你必須在程序中用 `Configuration` 的實例了

**[⬆ 返回頂部](#目錄)**

### 不要使用單例模式

單例（Singleton）是一種 [反模式](https://en.wikipedia.org/wiki/Singleton_pattern).  以下是解釋：
 1. 總是被用成**全域實例**。為什麼這很糟？因為你在程式碼裡面隱藏了**相依的對象**，而不是透過實作了哪些介面而標記出來。讓個東西是全域來避免這個東西被傳來傳去是一種[壞味道](https://en.wikipedia.org/wiki/Code_smell)。
 2. 違反了[單一職責原則](#單一職責模式)：本質上，單例模式必定會**控制自己的建立，同時控制自己的生命週期**。所以這個模式先天就違反了單一職責原則。
 3. 導致程式碼強[耦合](https://en.wikipedia.org/wiki/Coupling_%28computer_programming%29)：這導致很多情況下，在測試中假造他們**非常困難**。
 4. 在整個程式的生命週期中，始終攜帶狀態。這是另一個對測試的重大打擊，因為這會導致你的測試最終會有前後順序，而這對單元測試是很致命的。為什麼？因為每個單元測試都應該獨立於其他的單元測試。
 
這裡有一篇討論單例模式[根本問題](http://misko.hevery.com/2008/08/25/root-cause-of-singletons/) 很好的文章，是[Misko Hevery](http://misko.hevery.com/about/) 寫的。

**壞：**

```php
class DBConnection
{
    private static $instance;

    private function __construct(string $dsn)
    {
        // ...
    }

    public static function getInstance(): DBConnection
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }

        return self::$instance;
    }

    // ...
}

$singleton = DBConnection::getInstance();
```

**好：**

```php
class DBConnection
{
    public function __construct(string $dsn)
    {
        // ...
    }

     // ...
}
```

建立 `DBConnection` 類的實例，並通過 [DSN](http://php.net/manual/en/pdo.construct.php#refsect1-pdo.construct-parameters) 配置.

```php
$connection = new DBConnection($dsn);
```

現在你必須在程序中 使用 `DBConnection` 的實例了

**[⬆ 返回頂部](#目錄)**

### 封裝條件語句

**壞：**

```php
if ($article->state === 'published') {
    // ...
}
```

**好：**

```php
if ($article->isPublished()) {
    // ...
}
```

**[⬆ 返回頂部](#目錄)**

### 避免用反義條件判斷

**壞：**

```php
function isDOMNodeNotPresent(\DOMNode $node): bool
{
    // ...
}

if (!isDOMNodeNotPresent($node))
{
    // ...
}
```

**好：**

```php
function isDOMNodePresent(\DOMNode $node): bool
{
    // ...
}

if (isDOMNodePresent($node)) {
    // ...
}
```

**[⬆ 返回頂部](#目錄)**

### 避免條件判斷

這看起來像一個不可能的任務。當人們第一次聽到這句話是都會這麼說。「沒有 `if語句` 我還能做啥？」 答案是，你可以使用多態來實現多種場景的相同任務。

第二個問題也很常見，「這麼做可以，但為什麼我要這麼做？」答案是前面我們學過的一個 Clean Code 原則：一個函式應當只做一件事。當你有很多含有`if`語句的類和函式時,你的函式做了不止一件事。

記住，只做一件事。

**壞：**

```php
class Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        switch ($this->type) {
            case '777':
                return $this->getMaxAltitude() - $this->getPassengerCount();
            case 'Air Force One':
                return $this->getMaxAltitude();
            case 'Cessna':
                return $this->getMaxAltitude() - $this->getFuelExpenditure();
        }
    }
}
```

**好：**

```php
interface Airplane
{
    // ...

    public function getCruisingAltitude(): int;
}

class Boeing777 implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getPassengerCount();
    }
}

class AirForceOne implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude();
    }
}

class Cessna implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getFuelExpenditure();
    }
}
```

**[⬆ 返回頂部](#目錄)**

### 避免類型檢查 (part 1)

PHP 是弱型別的，這意味著你的函式可以接收任何類型的參數。有時候你會為了這自由度所痛苦，並且在你的函式漸漸加入型態檢查。

有很多方法可以避免這麼做。第一種方法是統一 API。

**壞：**

```php
function travelToTexas($vehicle): void
{
    if ($vehicle instanceof Bicycle) {
        $vehicle->pedalTo(new Location('texas'));
    } elseif ($vehicle instanceof Car) {
        $vehicle->driveTo(new Location('texas'));
    }
}
```

**好：**

```php
function travelToTexas(Vehicle $vehicle): void
{
    $vehicle->travelTo(new Location('texas'));
}
```

**[⬆ 返回頂部](#目錄)**

### 避免類型檢查 (part 2)

如果你正使用基本原始值，比如字符串、整數和陣列等等，要求版本是PHP 7+，無法使用多型別，但還是需要類型檢測，那你可以考慮使用[型態宣告](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration)或者嚴格模式。

這樣可以基於標準 PHP 語法，又增加型態的檢查。手動檢查類型的問題，是需要加上很多的檢查程式碼，但是所增加的安全性並不足以彌補所喪失的可讀性。
 
要避免類型檢查，保持程式碼整潔，寫好測試，並常常做好 code reviews。不然，就加上 PHP 靜態型別檢查或者嚴格模式。
 
**壞：**

```php
function combine($val1, $val2): int
{
    if (!is_numeric($val1) || !is_numeric($val2)) {
        throw new \Exception('Must be of type Number');
    }

    return $val1 + $val2;
}
```

**好：**

```php
function combine(int $val1, int $val2): int
{
    return $val1 + $val2;
}
```

**[⬆ 返回頂部](#目錄)**

### 移除僵屍代碼

程式碼代碼和重複程式碼一樣糟糕。沒有理由保留在你的程式中。如果從來沒被使用過，就刪掉！
不用擔心這些程式碼之後還會用到，因為些程式碼還在版控裡面，因此很安全。

**壞：**
```php
function oldRequestModule(string $url): void
{
    // ...
}

function newRequestModule(string $url): void
{
    // ...
}

$request = newRequestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```

**好：**

```php
function requestModule(string $url): void
{
    // ...
}

$request = requestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```

**[⬆ 返回頂部](#目錄)**


## 對像和數據結構

### 使用 getters 和 setters

在PHP中你可以對方法使用`public`, `protected`, `private` 來控制對像屬性的變更。

* 當你想對對像屬性做獲取之外的操作時，你不需要在代碼中去尋找並修改每一個該屬性訪問方法
* 當有`set`對應的屬性方法時，易於增加參數的驗證
* 封裝內部的表示
* 使用set* 和 get* 時，易於增加 Log 和錯誤控制
* 繼承當前類別時，可以覆寫默認的方法功能
* 當對像屬性是從遠端服務器獲取時，get* ，set* 易於使用延遲加載

此外，這樣的方式也符合 OOP 開發中的[開閉原則](#開閉原則)

**壞：**

```php
class BankAccount
{
    public $balance = 1000;
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->balance -= 100;
```

**好：**

```php
class BankAccount
{
    private $balance;

    public function __construct(int $balance = 1000)
    {
      $this->balance = $balance;
    }

    public function withdraw(int $amount): void
    {
        if ($amount > $this->balance) {
            throw new \Exception('Amount greater than available balance.');
        }

        $this->balance -= $amount;
    }

    public function deposit(int $amount): void
    {
        $this->balance += $amount;
    }

    public function getBalance(): int
    {
        return $this->balance;
    }
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->withdraw($shoesPrice);

// Get balance
$balance = $bankAccount->getBalance();
```

**[⬆ 返回頂部](#目錄)**

### 給對像使用私有或受保護的成員變數

* 對`public`方法和屬性進行修改非常危險，因為外部代碼容易依賴他，而你沒辦法控制。**對之修改影響所有這個類的使用者。** `public` methods and properties are most dangerous for changes, because some outside code may easily rely on them and you can't control what code relies on them. **Modifications in class are dangerous for all users of class.**
* 對`protected`的修改跟對`public`修改差不多危險，因為他們對子類可用，他倆的唯一區別就是可調用的位置不一樣，**對之修改影響所有集成這個類的地方。**  `protected` modifier are as dangerous as public, because they are available in scope of any child class. This effectively means that difference between public and protected is only in access mechanism, but encapsulation guarantee remains the same. **Modifications in class are dangerous for all descendant classes.**
* 對`private`的修改保證了這部分代碼**只會影響當前類**`private` modifier guarantees that code is **dangerous to modify only in boundaries of single class** (you are safe for modifications and you won't have [Jenga effect](http://www.urbandictionary.com/define.php?term=Jengaphobia&defid=2494196)).

所以，當你需要控制類裡的代碼可以被訪問時才用`public/protected`，其他時候都用`private`。

可以讀一讀這篇 [文章](http://fabien.potencier.org/pragmatism-over-theory-protected-vs-private.html) ，[Fabien Potencier](https://github.com/fabpot)寫的.

**壞：**

```php
class Employee
{
    public $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->name; // Employee name: John Doe
```

**好：**

```php
class Employee
{
    private $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function getName(): string
    {
        return $this->name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->getName(); // Employee name: John Doe
```

**[⬆ 返回頂部](#目錄)**

## 類

### 少用繼承多用組合

正如 the Gang of Four 所著的[*設計模式*](https://en.wikipedia.org/wiki/Design_Patterns)之前所說，我們應該盡量優先選擇組合，而不是繼承的方式。使用繼承和組合都有很多好處。

這個準則的主要意義，在於當你本能的使用繼承時，試著思考一下`組合`是否能更好對你的需求建模。在一些情況下，`組合`是更好的。

接下來你或許會想，「那我應該在什麼時候使用繼承？」
這得要看你的問題，當然下面有一些何時繼承比組合更好的說明：

1. 你的繼承表達了「是一個」（is-a）而不是「有一個」(has-a)的關係（人類-》動物，用戶-》用戶詳情）
2. 你可以重複使用基類別的程式碼（人類可以像動物一樣移動）
3. 你想通過修改基類別，對所有繼承的類別做全局的修改（當動物移動時，修改她們的能量消耗）

**糟糕的：**

```php
class Employee 
{
    private $name;
    private $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    // ...
}


// 不好，因為 Employees "有" taxdata
// 而 EmployeeTaxData 不是 Employee 類型的


class EmployeeTaxData extends Employee 
{
    private $ssn;
    private $salary;
    
    public function __construct(string $name, string $email, string $ssn, string $salary)
    {
        parent::__construct($name, $email);

        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}
```

**好：**

```php
class EmployeeTaxData 
{
    private $ssn;
    private $salary;

    public function __construct(string $ssn, string $salary)
    {
        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}

class Employee 
{
    private $name;
    private $email;
    private $taxData;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    public function setTaxData(string $ssn, string $salary)
    {
        $this->taxData = new EmployeeTaxData($ssn, $salary);
    }

    // ...
}
```

**[⬆ 返回頂部](#目錄)**

### 避免連貫接口

[連貫接口 Fluent interface](https://en.wikipedia.org/wiki/Fluent_interface)是一種希望在面向對像編程時，能提高可讀性的 API 設計模式，他基於[方法鏈 Method chaining](https://en.wikipedia.org/wiki/Method_chaining)的做法。

這種設計方式，可以有效的降低程式碼複雜度（例如[PHPUnit Mock Builder](https://phpunit.de/manual/current/en/test-doubles.html)
和[Doctrine Query Builder](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/query-builder.html)
）。更多的情況下，這方法會帶來較大代價：

1. 破壞了 [對像封裝](https://en.wikipedia.org/wiki/Encapsulation_%28object-oriented_programming%29)
2. 破壞了 [裝飾器模式](https://en.wikipedia.org/wiki/Decorator_pattern)
3. 在測試組件中不好做[mock](https://en.wikipedia.org/wiki/Mock_object)
4. 導致提交的 diff 不好閱讀

想了解更多，請閱讀 [連貫接口為什麼不好](https://ocramius.github.io/blog/fluent-interfaces-are-evil/)這篇文章，作者 [Marco Pivetta](https://ocramius.github.io/)

**壞：**

```php
class Car
{
    private $make = 'Honda';
    private $model = 'Accord';
    private $color = 'white';

    public function setMake(string $make): self
    {
        $this->make = $make;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function setModel(string $model): self
    {
        $this->model = $model;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function setColor(string $color): self
    {
        $this->color = $color;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = (new Car())
  ->setColor('pink')
  ->setMake('Ford')
  ->setModel('F-150')
  ->dump();
```

**好：**

```php
class Car
{
    private $make = 'Honda';
    private $model = 'Accord';
    private $color = 'white';

    public function setMake(string $make): void
    {
        $this->make = $make;
    }

    public function setModel(string $model): void
    {
        $this->model = $model;
    }

    public function setColor(string $color): void
    {
        $this->color = $color;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = new Car();
$car->setColor('pink');
$car->setMake('Ford');
$car->setModel('F-150');
$car->dump();
```

**[⬆ 返回頂部](#目錄)**

### 推薦使用 final 類

能用時盡量使用 `final` 關鍵字：

1. 阻止不受控的繼承鏈
2. 鼓勵 [組合](#少用繼承多用組合)
3. 鼓勵 [單一職責模式](#單一職責模式)
4. 鼓勵開發者用你的公開方法，而非通過繼承類獲取受保護方法的訪問權限
5. 使得修改代碼時，可以保證不破壞使用你的類別的其他程式

唯一不使用 `final` 關鍵字的情況，是你的類別繼承了其他的介面，並且沒有宣告其他函式。

想了解更多，請閱讀 [什麼時候宣告 final](https://ocramius.github.io/blog/when-to-declare-classes-final/)這篇文章，作者 [Marco Pivetta](https://ocramius.github.io/)

**壞：**

```php
final class Car
{
    private $color;
    
    public function __construct($color)
    {
        $this->color = $color;
    }
    
    /**
     * @return string The color of the vehicle
     */
    public function getColor() 
    {
        return $this->color;
    }
}
```

**好：**

```php
interface Vehicle
{
    /**
     * @return string The color of the vehicle
     */
    public function getColor();
}

final class Car implements Vehicle
{
    private $color;
    
    public function __construct($color)
    {
        $this->color = $color;
    }
    
    /**
     * {@inheritdoc}
     */
    public function getColor() 
    {
        return $this->color;
    }
}
```

**[⬆ 返回頂部](#目錄)**

## SOLID

**SOLID** 是 Michael Feathers 推薦的好記簡寫，它代表了 Robert Martin 所命名，最重要的五個物件程式設計原則

 * [S: 單一職責原則 (SRP)](#職責原則)
 * [O: 開閉原則 (OCP)](#開閉原則)
 * [L: 里氏替換原則 (LSP)](#里氏替換原則)
 * [I: 接口隔離原則 (ISP)](#接口隔離原則)
 * [D: 依賴倒置原則 (DIP)](#依賴倒置原則)


### 單一職責原則

Single Responsibility Principle (SRP)

正如在 Clean Code 所述，「修改一個類別，應該只有一個理由」。人們總是很容易用一堆方法塞滿一個類別。就好像我們只能在飛機上，只能攜帶一個行李箱，所以就會把所有的東西都塞到箱子裡。

這樣做的問題是：從概念上，這樣的類別不是高內聚的，並且留下了很多理由去修改它。

將你需要修改類別的次數降低到最小很重要。這是因為，當有很多方法在類別中時，修改其中一處，你很難知道在程式碼中，哪些依賴的模塊會被影響到。

**壞：**

```php
class UserSettings
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function changeSettings(array $settings): void
    {
        if ($this->verifyCredentials()) {
            // ...
        }
    }

    private function verifyCredentials(): bool
    {
        // ...
    }
}
```

**好：**

```php
class UserAuth 
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }
    
    public function verifyCredentials(): bool
    {
        // ...
    }
}

class UserSettings 
{
    private $user;
    private $auth;

    public function __construct(User $user) 
    {
        $this->user = $user;
        $this->auth = new UserAuth($user);
    }

    public function changeSettings(array $settings): void
    {
        if ($this->auth->verifyCredentials()) {
            // ...
        }
    }
}
```

**[⬆ 返回頂部](#目錄)**

### 開閉原則

Open/Closed Principle (OCP)

正如 Bertrand Meyer 所說，「軟體的元件（類別，模組，函式⋯⋯等）應該對擴展開放，對修改關閉。」 然而這句話意味著什麼呢？這個原則大體上表示，你應該在僅新增但是不修改原本程式碼的情況下，要有辦法增加新的功能

**壞：**

```php
abstract class Adapter
{
    protected $name;

    public function getName(): string
    {
        return $this->name;
    }
}

class AjaxAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'ajaxAdapter';
    }
}

class NodeAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'nodeAdapter';
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        $adapterName = $this->adapter->getName();

        if ($adapterName === 'ajaxAdapter') {
            return $this->makeAjaxCall($url);
        } elseif ($adapterName === 'httpNodeAdapter') {
            return $this->makeHttpCall($url);
        }
    }

    private function makeAjaxCall(string $url): Promise
    {
        // request and return promise
    }

    private function makeHttpCall(string $url): Promise
    {
        // request and return promise
    }
}
```

**好：**

```php
interface Adapter
{
    public function request(string $url): Promise;
}

class AjaxAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class NodeAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        return $this->adapter->request($url);
    }
}
```

**[⬆ 返回頂部](#目錄)**

### 里氏替換原則

Liskov Substitution Principle (LSP)

這是一個簡單的原則，卻用了一個不好理解的術語。它的正式定義是"如果S是T的子類型，那麼在不改變程序原有既定屬性（檢查、執行任務等）的前提下，任何T類型的對像都可以使用S類型的對像替代（例如，使用S的對像可以替代T的對像）" 這個定義更難理解:-)。

對這個概念最好的解釋是：如果你有一個父類和一個子類，在不改變原有結果正確性的前提下，父類和子類可以互換。這個聽起來依舊讓人有些迷惑，所以讓我們來看一個經典的正方形-長方形的例子。

從數學上講，正方形是一種長方形。但是當你的模型通過繼承，使用了「is-a」的關係時，就不對了。

**壞：**

```php
class Rectangle
{
    protected $width = 0;
    protected $height = 0;

    public function setWidth(int $width): void
    {
        $this->width = $width;
    }

    public function setHeight(int $height): void
    {
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle
{
    public function setWidth(int $width): void
    {
        $this->width = $this->height = $width;
    }

    public function setHeight(int $height): void
    {
        $this->width = $this->height = $height;
    }
}

function printArea(Rectangle $rectangle): void
{
    $rectangle->setWidth(4);
    $rectangle->setHeight(5);
 
    // BAD: Will return 25 for Square. Should be 20.
    echo sprintf('%s has area %d.', get_class($rectangle), $rectangle->getArea()).PHP_EOL;
}

$rectangles = [new Rectangle(), new Square()];

foreach ($rectangles as $rectangle) {
    printArea($rectangle);
}
```
（譯註：這是因為，雖然說正方形是一種長方形，但是正方形不能夠做長方形能做到的「改變長度不改變寬度」這件事情。所以當你讓正方形繼承了長方形，然後放進會改變長寬的函式內，正方形就會出現原本長方形不會出現的錯誤了。）

**好：**

最好是將這兩種四邊形分別對待，用一個適合兩種類型的更通用子類型來代替。

儘管正方形和長方形看起來很相似，但他們是不同的。正方形更接近菱形，而長方形更接近平行四邊形。但他們不是子類型。儘管相似，正方形、長方形、菱形、平行四邊形，都是有自己屬性的不同形狀。

```php
interface Shape
{
    public function getArea(): int;
}

class Rectangle implements Shape
{
    private $width = 0;
    private $height = 0;

    public function __construct(int $width, int $height)
    {
        $this->width = $width;
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square implements Shape
{
    private $length = 0;

    public function __construct(int $length)
    {
        $this->length = $length;
    }

    public function getArea(): int
    {
        return $this->length ** 2;
    }
}

function printArea(Shape $shape): void
{
    echo sprintf('%s has area %d.', get_class($shape), $shape->getArea()).PHP_EOL;
}

$shapes = [new Rectangle(4, 5), new Square(5)];

foreach ($shapes as $shape) {
    printArea($shape);
}
```

（譯註：所以這邊的關係變成了，正方形長方形都「has-a」面積，也就是都可以取得面積。這樣就避免了繼承邏輯上的問題）

**[⬆ 返回頂部](#目錄)**

### 接口隔離原則

Interface Segregation Principle (ISP)

接口隔離原則表示：「調用方不應該被強制依賴於他不需要的介面」

有一個清晰的例子來示範這條原則：當一個類需要一個大量的設置條件，為了方便不會要求調用方去設置所有的條件，因為在大多數時候，他們不需要這麼多條件。你應該要製作很多小介面。

使設置變成可選項目。有助於我們避免產生「胖介面」

**壞：**

```php
interface Employee
{
    public function work(): void;

    public function eat(): void;
}

class HumanEmployee implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        // ...... eating in lunch break
    }
}

class RobotEmployee implements Employee
{
    public function work(): void
    {
        //.... working much more
    }

    public function eat(): void
    {
        //.... robot can't eat, but it must implement this method
    }
}
```

**好：**

不是每一個工人都是雇員，但是每一個雇員都是工人

```php
interface Workable
{
    public function work(): void;
}

interface Feedable
{
    public function eat(): void;
}

interface Employee extends Feedable, Workable
{
}

class HumanEmployee implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        //.... eating in lunch break
    }
}

// robot can only work
class RobotEmployee implements Workable
{
    public function work(): void
    {
        // ....working
    }
}
```

（譯註：機器人雇員不需要吃飯，所以不繼承雇員，但是我們有 `Workable` 可以繼承）

**[⬆ 返回頂部](#目錄)**

### 依賴倒置原則

Dependency Inversion Principle (DIP)

這條原則說明兩個基本的要點：
1. 高階的模塊不應該依賴低階的模塊，它們都應該依賴於抽像
2. 抽像不應該依賴於實現，實現應該依賴於抽像

這條起初看起來有點晦澀難懂，但是如果你使用過 PHP 框架（例如 Symfony），你應該見過依賴注入（DI），它是對這個概念的實現。雖然它們不是完全相等的概念，依賴倒置原則使高階模塊與低階模塊的實現細節和創建分離。可以使用依賴注入（DI）這種方式來實現它。最大的好處是它使模塊之間解耦。耦合會導致你難於重構，它是一種非常糟糕的的開發模式。

**壞：**

```php
class Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot extends Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
```

**好：**

```php
interface Employee
{
    public function work(): void;
}

class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot implements Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
```

**[⬆ 返回頂部](#目錄)**

## 別寫重複代碼 (DRY)

試著去遵循[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 原則.

盡你最大的努力去避免複製代碼，它是一種非常糟糕的行為，複製代碼通常意味著當你需要變更一些邏輯時，你需要修改不止一處。

試想一下，如果你在經營一家餐廳並且你在記錄你倉庫的進銷記錄：所有的土豆，洋蔥，大蒜，辣椒等。如果你有多個列表來管理進銷記錄，當你用其中一些土豆做菜時你需要更新所有的列表。如果你只有一個列表的話只有一個地方需要更新。

通常情況下你複製代碼是應該有兩個或者多個略微不同的邏輯，它們大多數都是一樣的，但是由於它們的區別致使你必須有兩個或者多個隔離的但大部分相同的方法，移除重複的代碼意味著用一個 function/module/class 創建一個能處理差異的抽像。

用對抽像非常關鍵，這正是為什麼你必須學習遵守在[類](#類)章節寫的 SOLID 原則，不合理的抽像比複製代碼更糟糕，所以務必謹慎！說了這麼多，如果你能設計一個合理的抽像，那就這麼做！別寫重複代碼，否則你會發現任何時候當你想修改一個邏輯時，你必須同時修改多個地方。

**壞：**

```php
function showDeveloperList(array $developers): void
{
    foreach ($developers as $developer) {
        $expectedSalary = $developer->calculateExpectedSalary();
        $experience = $developer->getExperience();
        $githubLink = $developer->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}

function showManagerList(array $managers): void
{
    foreach ($managers as $manager) {
        $expectedSalary = $manager->calculateExpectedSalary();
        $experience = $manager->getExperience();
        $githubLink = $manager->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}
```

**好：**

```php
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        $expectedSalary = $employee->calculateExpectedSalary();
        $experience = $employee->getExperience();
        $githubLink = $employee->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}
```

**極好：**

最好讓代碼緊湊一點

```php
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        render([
            $employee->calculateExpectedSalary(),
            $employee->getExperience(),
            $employee->getGithubLink()
        ]);
    }
}
```

**[⬆ 返回頂部](#目錄)**

## 翻譯

其他語言的翻譯：

* :cn: **Chinese：**
   * [php-cpm/clean-code-php](https://github.com/php-cpm/clean-code-php)
* :ru: **Russian：**
   * [peter-gribanov/clean-code-php](https://github.com/peter-gribanov/clean-code-php)
* :es: **Spanish：**
   * [fikoborquez/clean-code-php](https://github.com/fikoborquez/clean-code-php)
* :brazil: **Portuguese：**
   * [fabioars/clean-code-php](https://github.com/fabioars/clean-code-php)
   * [jeanjar/clean-code-php](https://github.com/jeanjar/clean-code-php/tree/pt-br)
* :thailand: **Thai：**
   * [panuwizzle/clean-code-php](https://github.com/panuwizzle/clean-code-php)
* :fr: **French：**
   * [errorname/clean-code-php](https://github.com/errorname/clean-code-php)
* :vietnam: **Vietnamese**
   * [viethuongdev/clean-code-php](https://github.com/viethuongdev/clean-code-php)
* :kr: **Korean：**
   * [yujineeee/clean-code-php](https://github.com/yujineeee/clean-code-php)
* :tr: **Turkish：**
   * [anilozmen/clean-code-php](https://github.com/anilozmen/clean-code-php)
   
**[⬆ 返回頂部](#目錄)**
