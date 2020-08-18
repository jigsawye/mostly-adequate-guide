# 第 4 章：Curry（柯里化）

## Curry 的重要性
我爸以前跟我說過，有些東西在你得到以前是可有可無的，但得到之後就不可或缺了。微波爐、智慧型手機皆是如此。老人在沒有網路時也過得很充實。對我來說，curry 也是這樣。

Curry 的概念很簡單：你可以只透過部分的參數呼叫一個 function，它會回傳一個 function 去處理剩下的參數。

你可以一次性的呼叫 curry function，也可以每次只傳遞一個參數。

```js
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12
```

這裡我們建立了一個 `add` function，它接受一個參數並回傳一個新的 function。當呼叫它後，回傳的 function 會透過 closure（閉包）的方式儲存 `add` 第一個參數。一次透過兩個參數呼叫實在是有點繁複，幸虧我們可以使用一個名為 `curry` 的特殊 helper function 讓定義並呼叫這種 function 變得更容易。

讓我們建立一些 curry function 來享受一下。

```js
var curry = require('lodash/curry');

var match = curry(function(what, str) {
  return str.match(what);
});

var replace = curry(function(what, replacement, str) {
  return str.replace(what, replacement);
});

var filter = curry(function(f, ary) {
  return ary.filter(f);
});

var map = curry(function(f, ary) {
  return ary.map(f);
});
```

上方程式碼所遵循的是一種簡單，但也重要的模式。這裡戰略性的將欲操作的資料（String，Array）作為最後一個參數傳入。當使用時就知道為何要這麼做了。

```js
match(/\s+/g, 'hello world');
// [ ' ' ]

match(/\s+/g)('hello world');
// [ ' ' ]

var hasSpaces = match(/\s+/g);
// function(x) { return x.match(/\s+/g) }

hasSpaces('hello world');
// [ ' ' ]

hasSpaces('spaceless');
// null

filter(hasSpaces, ['tori_spelling', 'tori amos']);
// ['tori amos']

var findSpaces = filter(hasSpaces);
// function(xs) { return xs.filter(function(x) { return x.match(/\s+/g) }) }

findSpaces(['tori_spelling', 'tori amos']);
// ['tori amos']

var noVowels = replace(/[aeiouy]/ig);
// function(replacement, x) { return x.replace(/[aeiouy]/ig, replacement) }

var censored = noVowels("*");
// function(x) { return x.replace(/[aeiouy]/ig, '*') }

censored('Chocolate Rain');
// 'Ch*c*l*t* R**n'
```

這裡示範的是一種「pre-load（預先載入）」的能力，透過傳遞一至兩個參數，就能得到一個記住這些參數的新 function。

我建議你透過 `npm install lodash` 安裝 lodash，複製上方的程式碼至 REPL 執行看看。你也可以在能使用 lodash 或 ramda 的瀏覽器環境中執行。

## 不只是雙關語 / 咖哩

Curry 的用處非常廣泛。就像在 `hasSpaces`、`findSpaces` 及 `censored` 看到的，只需傳遞一些參數至 function，就能得到一個新的 function。

只要透過 `map` 封裝單一元素的 function，即可將它轉換成參數維陣列的 function：

```js
var getChildren = function(x) {
  return x.childNodes;
};

var allTheChildren = map(getChildren);
```

只傳遞一部分參數至 function 通常稱做*部分應用（partial application）*，能夠大量減少樣板程式碼（boiler plate code）。考慮上方的 `allTheChildren` function 若使用 lodash 非 curry 的 `map` 來寫會如何（請注意參數的順序也不同）：

```js
var allTheChildren = function(elements) {
  return _.map(elements, getChildren);
};
```

一般來說我們不會定義直接操作陣列的 function，因為我們只需要行內呼叫 `map(getChildren)` 即可。此點也同樣適用於 `sort`、`filter` 及其他高階 function（Higher order function：一個 function 使用或者是回傳另一個 function）。

當我們討論 *pure function* 時，我們會說它接受一個輸入並對應一個輸出。Curry 所做的事也是如此：每傳遞一個參數就會回傳一個新的 function 處理剩餘的參數。這就是一個輸入對應一個輸出。

不論輸出是否為另一個 function，它也是 pure function。我們也接受一次傳遞多個參數，不過這樣也只是為了方便減少多餘的 `()`。


## 總結

Curry 使用起來相當得心應手，每天使用它對我來說簡直是一種享受。它是一種必備的工具，讓 functional programming 不那麼繁冗。

透過簡單的傳遞一些參數，就能夠動態的建立實用的新 function，即便有多個參數，也保留了數學 function 的定義。

讓我們來介紹另一個必備工具 `compose（組合）`

[第 5 章：透過 Compose 開發](ch5.md)

## 練習

在開始前先說明一下。我們預設會使用名為 [Ramda](http://ramdajs.com) 的 library 將 function 轉換為 curry function。或者你能使用由 lodash 撰寫與維護的 [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide) 做到一樣的事。兩者都運作得相當好，你可以根據偏好做選擇。

你還能對自己的練習程式碼進行[unit test（單元測試）](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises)，或者只將程式碼複製到 javascript REPL 執行看看。

練習的答案已經放在[本書的 repository](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises/answers)。做本練習最佳方式就是透過[及時回饋環線（immediate feedback loop）](feedback_loop.md)。

```js
var _ = require('ramda');


// 練習 1
//==============
// 透過部分套用（partially applying）重構此 function 移除所有參數。

var words = function(str) {
  return _.split(' ', str);
};

// 練習 1a
//==============
// 使用 map 建立一個新的 words fn，讓它可以操作字串的陣列。

var sentences = undefined;


// 練習 2
//==============
// 透過部分套用（partially applying）重構此 function 移除所有參數。

var filterQs = function(xs) {
  return _.filter(function(x) {
    return match(/q/i, x);
  }, xs);
};


// 練習 3
//==============
// 使用 helper function _keepHighest 重構 max，讓它不需參考任何參數。

// 不需更動：
var _keepHighest = function(x, y) {
  return x >= y ? x : y;
};

// 重構這段：
var max = function(xs) {
  return _.reduce(function(acc, x) {
    return _keepHighest(acc, x);
  }, -Infinity, xs);
};


// 加分題 1：
// ============
// 封裝陣列的 slice 讓它變為 functional 的 curry function。
// //[1, 2, 3].slice(0, 2)
var slice = undefined;


// 加分題 2：
// ============
// 使用 slice 定義一個「take」function，讓它擷取字串從頭開始的的 n 個元素。此 function 必須為 curry function。
// // 輸入「Something」且 n=4 時結果必須為「Some」
var take = undefined;
```
