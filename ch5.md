# 第 5 章：使用 Compose 開發

## Functional 飼養

這就是 `compose`：

```js
var compose = function(f, g) {
  return function(x) {
    return f(g(x));
  };
};
```

`f` 和 `g` 都是 function，`x` 則是通過它們之間「管道」的值。

Compose 感覺起來就像在飼養 function。你就是 function 的飼養員，選擇兩個有你喜歡特色的 function 並將它們組合，產生一個新的 function。使用起來如下：

```js
var toUpperCase = function(x) {
  return x.toUpperCase();
};
var exclaim = function(x) {
  return x + '!';
};
var shout = compose(exclaim, toUpperCase);

shout("send in the clowns");
//=> "SEND IN THE CLOWNS!"
```

組合兩個 function 並回傳一個新的 function 是很容易理解的：組合某種類型（在本例中為 function）的兩個元素應該產生一個該類型的新元素。你將兩個樂高積木組合起來並不會得到林肯積木。所以這是有跡可循的，我們會在適當的時候探討這方面的一些底層理論。

在 `composer` 的定義中，`g` 會在 `f` 之前執行，而建立一個由右至左的資料流。這麼做的可讀性遠高於巢狀的 function 呼叫。若不用 composer，那麼會像以下這樣：

```js
var shout = function(x) {
  return exclaim(toUpperCase(x));
};
```

程式由右而左執行，而不是由內而外，我認為這可以稱之為「左傾（left direction）」。讓我們看看一個順序重要的例子：

```js
var head = function(x) {
  return x[0];
};
var reverse = reduce(function(acc, x) {
  return [x].concat(acc);
}, []);
var last = compose(head, reverse);

last(['jumpkick', 'roundhouse', 'uppercut']);
//=> 'uppercut'
```

`reverse` 會將列表反轉，`head` 則會取得列表的第一個元素。結果就得到了一個效率不高的 `last` function。這個組合中 function 的執行順序是顯而易見的。雖然我們可以定義一個由左而右的版本，但是由右而左更能反映出數學上的含義。沒錯，compose 的概念直接來自於數學課本。事實上，現在是時候看看所有 compose 都有的一個特性了。

```js
// 結合律（associativity）
var associative = compose(f, compose(g, h)) == compose(compose(f, g), h);
// true
```

Compose 有結合律的特性，意指不管你將哪兩個分為一組都不重要。所以，如果我們想將字串轉為大寫，可以這樣寫：

```js
compose(toUpperCase, compose(head, reverse));

// 或
compose(compose(toUpperCase, head), reverse);
```

因為呼叫 `compose` 時的分組方式不重要，所以結果都會是相同的。因此，這也讓我們可以撰寫一個參數數量可變的 compose，用法如下：

```js
// 在前面的例子中我們寫了兩個 compose，不過因為 compose 符合結合律，我們可以讓 compose 接受多個 function，並讓它自己決定如何分組。
var lastUpper = compose(toUpperCase, head, reverse);

lastUpper(['jumpkick', 'roundhouse', 'uppercut']);
//=> 'UPPERCUT'


var loudLastUpper = compose(exclaim, toUpperCase, head, reverse);

loudLastUpper(['jumpkick', 'roundhouse', 'uppercut']);
//=> 'UPPERCUT!'
```

運用結合律的屬性能夠為我們帶來強大的靈活性，及當結果相同時的所帶來的安心感。稍微複雜一點，參數數量可變的 compose 都已經包含在本書的提供的 library 中，你也可以在像是 [lodash][lodash-website]、[underscore][underscore-website] 及 [ramda][ramda-website] 的 library 中也可以找到它們。

結合率的一大好處是任何一個 function 的分組都可以被拆開，然後再以他們自己的 compose 方式封裝在一起。讓我們來重構前面的例子：

```js
var loudLastUpper = compose(exclaim, toUpperCase, head, reverse);

// 或
var last = compose(head, reverse);
var loudLastUpper = compose(exclaim, toUpperCase, last);

// 或
var last = compose(head, reverse);
var angry = compose(exclaim, toUpperCase);
var loudLastUpper = compose(angry, last);

// 更多變種⋯
```

這沒有標準答案－我們只是以自己喜歡的方式玩樂高積木而已。一般來說，分組的最佳方式就是讓它可重用，像是 `last` 及 `angry`。如果熟悉的 Fowler 的《[Refactoring][refactoring-book]》這本書的話，你可能會知道這個過程稱之為「[extract method（抽出方法）][extract-method-refactor]」⋯只不過不需要擔心object的所有狀態。

## Pointfree

Pointfree 模式指的是永遠不必說出你的資料。呃抱歉（譯註：此處原文是「Pointfree style means never having to say your data」，源自 1970 年的電影 Love Story 裡的一句著名台詞「Love means never having to say you're sorry」。緊接著作者又說了一句「Excuse me」，大概是一種幽默）。意思是指，function 不必提及要操作的資料是什麼樣的。First Class Function、curry 及 compose 協作起來非常有助於建立這種模式。

```js
// 非 pointfree，因為我們提到資料：word
var snakeCase = function(word) {
  return word.toLowerCase().replace(/\s+/ig, '_');
};

// pointfree
var snakeCase = compose(replace(/\s+/ig, '_'), toLowerCase);
```

看到 `replace` 是如何被部分呼叫了嗎？這裡做的事情就是將資料傳遞至每個接收單一參數的 function。Curry 讓每個 function 都先接收資料，再操作資料，最後再將資料傳遞至下一個 function。另外要注意在 pointfree 的版本中，我們不需資料來建構 function，而在非 pointfree 的版本中，我們必須先擁有 `word` 才能進行其他操作。

讓我們看看另一個例子。

```js
// 非 pointfree，因為我們提到資料：name
var initials = function(name) {
  return name.split(' ').map(compose(toUpperCase, head)).join('. ');
};

// pointfree
var initials = compose(join('. '), map(compose(toUpperCase, head)), split(' '));

initials("hunter stockton thompson");
// 'H. S. T'
```

Pointfree 幫助我們移除不必要的命名，讓程式碼保持簡潔和通用。對 functional 的程式碼來說，pointfree 是個很好的試金石，因為它能告訴我們一個 function 是否為接受輸入回傳輸出的小 function。像是 compose 無法用於 while 迴圈上。不過請注意，pointfree 就像一把雙刃劍，有時會混淆視聽。並不是所有的 functional 程式碼都為 pointfree，不過這沒關係。可以使用他的時候就使用，不能使用的時候就用普通的 function。

## Debug
Compose 常見的錯誤就是，在沒有第一次部分呼叫前，就 compose 像是 `map` 接受兩個參數的 function。

```js
// 不正確－我們傳遞 array 給 angry，但不知道部分呼叫的 map 接收到什麼。
var latin = compose(map, angry, reverse);

latin(['frog', 'eyes']);
// error


// 正確－每個 function 都預期接受一個參數。
var latin = compose(map(angry), reverse);

latin(['frog', 'eyes']);
// ['EYES!', 'FROG!'])
```

如果你在 debug compose 時遇到了問題，我們可以使用下面這個實用，但 impure 的 trace function 追蹤執行情況。

```js
var trace = curry(function(tag, x) {
  console.log(tag, x);
  return x;
});

var dasherize = compose(join('-'), toLower, split(' '), replace(/\s{2,}/ig, ' '));

dasherize('The world is a vampire');
// TypeError: Cannot read property 'apply' of undefined
```

這裡出錯了，讓我們 `trace` 看看

```js
var dasherize = compose(join('-'), toLower, trace('after split'), split(' '), replace(/\s{2,}/ig, ' '));
// after split [ 'The', 'world', 'is', 'a', 'vampire' ]
```

啊！因為 `toLower` 執行於 array，我們需要透過 `map` 呼叫它。

```js
var dasherize = compose(join('-'), map(toLower), split(' '), replace(/\s{2,}/ig, ' '));

dasherize('The world is a vampire');

// 'the-world-is-a-vampire'
```

`trace` function 讓我們在某個特定的點觀察資料，以便進行 debug。像是 haskell 與 purescript 的語言為了方便開發，也都提供了相似的 function。

Compose 會成為我們建構程式的工具，且幸運的是，他背後有個強大的理論做支撐。讓我們來研究一下這個理論。


## 範疇論

範疇論（category theory）是個數學的抽象分支，能夠形式化集合論（set theory）、類型論（type theory）、群論（group theory）及邏輯學（logic）等數學分支的一些概念。範疇學主要處理 object、態射（morphism）及轉化（transformation），而這些概念跟程式設計的關係非常密切。下圖是一些同樣概念分別在不同理論下的形式：

<img src="images/cat_theory.png" />

抱歉，我沒有任何要嚇你的意思。我不假設你對這些概念瞭若指掌，我的重點只想讓你知道這裡有多少重複的內容，讓你知道為何範疇學要統一這些概念。

在範疇學中，有一個概念稱之為⋯範疇。有以下 component 的 collection 就構成一個範疇：

  * object 的 collection
  * 態射的 collection
  * 態射的組合
  * 一個名為 identity 獨特的態射

範疇學抽象到可以模擬任何事物，不過我們目前最關心的還是類型及 function，所以讓我們將範疇學運用到它們身上看看。

**Object 的 collection**
Object 就是資料類型。例如：``String``、``Boolean``、``Number`` 及 ``Object`` 等等。通常我們把資料類型作為所有可能值的一個集合（set）。像是 ``Boolean`` 就是 `[true, false]` 的集合，``Number`` 可以是所有實數的集合。把類型當作集合是有好處的，因為我們可以利用集合論處理類型。


**態射的 collection**
態射會是標準的 pure function。

**態射的組合**
你可以已經猜到了，這就是本章所介紹的新玩具－`compose`。我們已經討論過 `compose` function 是符合結合律，這並不是巧合，結合律是範疇學中對任何組合都適用的一個特性。

下圖展示了何為組合：

<img src="images/cat_comp1.png" />
<img src="images/cat_comp2.png" />

下方的程式碼是個具體的例子：

```js
var g = function(x) {
  return x.length;
};
var f = function(x) {
  return x === 4;
};
var isFourLetterWord = compose(f, g);
```

**一個名為 identity 獨特的態射**
讓我們來介紹一個名為 `id` 的實用 function。這個 function 只是接受隨便的輸入然後原封不動的還給你。如下：

```js
var id = function(x) {
  return x;
};
```

你可能想問「這到底哪裡有用了？」。我們會在之後的幾個章節擴增這個 function，暫時將它當作一個可以替代給定值的 function－一個假裝自己是資料的 function。

`id` 與 compose 簡直是天作之合。下面這個特性對所有的 unary function（一元 function：只接受一個參數的 function）f 都成立：

```js
// identity
compose(id, f) == compose(f, id) == f;
// true
```

嘿，這不就是實數的單一律（identity property）阿！如過這還不夠清楚明瞭，就慢慢理解它的無用性吧。我們很快會到處使用 `id`，但現在我們暫時將它當作是個替代給定值得 function 。這對於撰寫 pointfree 的程式碼相當有用。

好了，以上就是類型和 function 的範疇。如果這是你第一次聽說這些概念，我猜測你現在還有些不瞭解，不懂範疇為何及為何有用。沒關係，本書都會借助這些知識。至於現在，本章的本行中，你至少可以認為它向我們提供了有關 compose 的知識－例如結合律與單一律。

除了這些，還有哪些範疇呢？當然，我們可以定義一個向量圖，以結點為 object，邊為態射，以路徑連接為組合。我們可以定義一個實數為 object，`>=` 為態射（事實上任何偏序（partial order）及全序（total order）都可成為一個範疇）。範疇的總數是無上限的，但是要達到本書的目的，我們只需關心上方所定義的範疇即可。到目前我們已經瀏覽了一些表面的東西，接著必須進入下一章節了。


## 總結
Compose 將我們的的 function 連結在一起，就像一條管線一樣。資料也會在我們的應用程式中流動－畢竟 pure function 就是輸入對輸出，所以打破這個鏈結就是不遵重輸出，會讓我們的應用程式一無是處。

我們認為 compose 是高於其他原則的設計模式，這是因為 compose 讓我們的程式簡單而可讀。範疇學會在應用程式架構、模擬副作用及保證正確性方面扮演重要的角色。

現在我們已經有足夠的知識去進行一些實際的練習了，讓我們來撰寫一個範例應用程式。

[第 6 章：範例應用程式](ch6.md)

## 練習

```js
var _ = require('ramda');
var accounting = require('accounting');

// 範例資料
var CARS = [{
  name: 'Ferrari FF',
  horsepower: 660,
  dollar_value: 700000,
  in_stock: true,
}, {
  name: 'Spyker C12 Zagato',
  horsepower: 650,
  dollar_value: 648000,
  in_stock: false,
}, {
  name: 'Jaguar XKR-S',
  horsepower: 550,
  dollar_value: 132000,
  in_stock: false,
}, {
  name: 'Audi R8',
  horsepower: 525,
  dollar_value: 114200,
  in_stock: false,
}, {
  name: 'Aston Martin One-77',
  horsepower: 750,
  dollar_value: 1850000,
  in_stock: true,
}, {
  name: 'Pagani Huayra',
  horsepower: 700,
  dollar_value: 1300000,
  in_stock: false,
}];

// 練習 1：
// ============
// 使用 _.compose() 重寫以下的 function。提示： _.prop() 是 curry function。
var isLastInStock = function(cars) {
  var last_car = _.last(cars);
  return _.prop('in_stock', last_car);
};

// 練習 2：
// ============
// 使用 _.compose()、 _.prop() 及 _.head() 來取得第一筆 car 的 name。
var nameOfFirstCar = undefined;


// 練習 3：
// ============
// 使用 helper function _average 來重構 averageDollarValue 使之為 compose function。
var _average = function(xs) {
  return _.reduce(_.add, 0, xs) / xs.length;
}; // <- 不需改動

var averageDollarValue = function(cars) {
  var dollar_values = _.map(function(c) {
    return c.dollar_value;
  }, cars);
  return _average(dollar_values);
};


// 練習 4：
// ============
// 使用 compose 撰寫一個 function：sanitizeNames()，回傳一個 car 的 name 為全小寫及底線連接的列表：例如：sanitizeNames([{name: 'Ferrari FF', horsepower: 660, dollar_value: 700000, in_stock: true}]) //=> ['ferrari_ff']。

var _underscore = _.replace(/\W+/g, '_'); //<-- leave this alone and use to sanitize

var sanitizeNames = undefined;


// 加分題 1：
// ============
// 使用 compose 重構 availablePrices。

var availablePrices = function(cars) {
  var available_cars = _.filter(_.prop('in_stock'), cars);
  return available_cars.map(function(x) {
    return accounting.formatMoney(x.dollar_value);
  }).join(', ');
};


// 加分題 2：
// ============
// 重構使它 pointfree。提示: 你可以使用 _.flip()。

var fastestCar = function(cars) {
  var sorted = _.sortBy(function(car) {
    return car.horsepower;
  }, cars);
  var fastest = _.last(sorted);
  return fastest.name + ' is the fastest';
};
```

[lodash-website]: https://lodash.com/
[underscore-website]: http://underscorejs.org/
[ramda-website]: http://ramdajs.com/
[refactoring-book]: http://martinfowler.com/books/refactoring.html
[extract-method-refactor]: http://refactoring.com/catalog/extractMethod.html
