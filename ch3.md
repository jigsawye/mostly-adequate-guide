# 第 3 章：Pure Function－單純的幸福

## 再次強調「Pure」

首先，我們要釐清 pure function 的概念。

> Pure function 意指相同的輸入，永遠會得到相同的輸出，而且沒有任何顯著的副作用。

例如 `slice` 及 `splice`。這兩個 function 的作用並無二致－但是注意，它們各自的方式卻大不相同，但結果還是一樣的。為何說 `slice` 是 *pure*，因為對於相同的輸入它能保證回傳的輸出是相同的。但 `splice` 卻會嚼爛呼叫它的陣列，然後吐出來；這產生了顯著的副作用，即這個陣列被永久改變了。

```js
var xs = [1, 2, 3, 4, 5];

// pure（純）
xs.slice(0, 3);
//=> [1, 2, 3]

xs.slice(0, 3);
//=> [1, 2, 3]

xs.slice(0, 3);
//=> [1, 2, 3]


// impure（不純）
xs.splice(0, 3);
//=> [1, 2, 3]

xs.splice(0, 3);
//=> [4, 5]

xs.splice(0, 3);
//=> []
```

在 functional programming 中，我們討厭像 `splice` 這種會*改變*資料的笨 function。我們追求的是那種可靠的，每次都能回傳相同結果的 function，而不是像 `splice` 這種每次呼叫後留下一堆爛攤子的 function。

讓我們看看另一個例子。

```js
// impure
var minimum = 21;

var checkAge = function(age) {
  return age >= minimum;
};



// pure
var checkAge = function(age) {
  var minimum = 21;
  return age >= minimum;
};
```

在 impure 的版本中，`checkAge` 的結果將取決於 `minimum` 這個可變變數。換句話說，他將取決於系統狀態；這點令人失望，因為它引用了外部的環境，從而增加了認知負擔。

這個例子可能還不是那麼明顯，但這種依賴狀態是影響系統複雜度的罪魁禍首（http://www.curtclifton.net/storage/papers/MoseleyMarks06a.pdf）。輸入值之外的因素能夠左右 `checkAge` 的回傳值，不僅讓他變得不符合 pure，還導致每次我思考整個軟體時都痛苦不堪。

另一方面，使用 pure 的形式就能做到完全自給自足。我們可以讓 `minimum` 變為 immutable（不可變的），這樣就可保留純粹性，因為狀態不會有變化。我們可以建立一個 object 並 freeze 它來做到這件事。

```js
var immutableState = Object.freeze({
  minimum: 21,
});
```

## 副作用可能包含⋯

讓我們來仔細研究一下「副作用」以加深理解。那麼，在 `pure function` 定義中所提到萬惡的*副作用*指的是什麼？我們可以將*作用*理解為除了結果計算之外所發生的事。

作用本身並沒有什麼壞處，而且在本書後面的章節隨處可看到他的影子。真正帶有負面含義的是*副*。就像一灘死水的水並不是幼蟲的培養皿，*死*才是產生蟲群的主因。我向你保證，*副*作用是你程式中滋生 bug 的溫床。

>*副作用*是在計算結果的過程中，系統狀態的一種改變，或是外部世界可觀察的*交互作用*。

副作用可以包含，但不限於：

  * 更改檔案系統
  * 在資料庫寫入紀錄
  * 發送一個 http 請求
  * 可變資料
  * 印出至畫面 / log
  * 取得使用者輸入
  * DOM 查詢
  * 存取系統狀態

這個列表還能繼續寫下去。概括來說，只要與 function 外部環境發生交互作用的都是副作用，這可能會讓你懷疑在程式設計上無副作用的可行性。functional programming 的哲學就是假設副作用是造成不正確行為的主要原因

這並不代表我們要禁止使用一切的副作用，而是說要讓他們在可控制的範圍內發生。在後面說到 functor 及 monad 時我們會學習如何控制它們，但目前還是盡量遠離這些陰險的 function 較好。

副作用會讓一個 function 不再 *pure* 是有道理的：從定義上來說，pure function 必須要能夠根據相同的輸入回傳相同的輸出，若 function 必須與外部的事物來往時就無法保證此定義。

讓我們來仔細了解為何要堅持這種相同輸入得到相同輸出的原則。注意，我們要來複習一些八年級的數學了。

## 八年級數學

根據 mathisfun.com:

> function 與不同數值間的特殊關係：對每一個輸入值回傳一個輸出值。

換句話說，這只是兩個數值之間的關係：輸入及輸出。即使每個輸入都只會有一個輸出，但不同的輸入卻可以有相同的輸出。下圖展示了由 `x` 到 `y` 完全合法的 function。

<img src="images/function-sets.gif" />(http://www.mathsisfun.com/sets/function.html)

反之，下圖則展示了`非` function 的關係，因為輸入值的 `5` 指向了多個輸出：

<img src="images/relation-not-function.gif" />(http://www.mathsisfun.com/sets/function.html)

Function 可以被描述為一個集合，這個集合中的的內容為（輸入，輸出）：`[(1,2), (3,6), (5,10)]`（看來這個 function 是將輸入加倍）。

或是一張表：
<table> <tr> <th>Input</th> <th>Output</th> </tr> <tr> <td>1</td> <td>2</td> </tr> <tr> <td>2</td> <td>4</td> </tr> <tr> <td>3</td> <td>6</td> </tr> </table>

甚至是一個以 `x` 為輸入 `y` 為輸出的圖形：

<img src="images/fn_graph.png" width="300" height="300" />


如果輸入直接指名了輸出為何，那麼就不必實作細節了。因為 function 只是將輸入 mapping 至輸出，所以簡單的寫一個 object ，將 `[]` 代替 `()` 就能執行了。

```js
var toLowerCase = {
  'A': 'a',
  'B': 'b',
  'C': 'c',
  'D': 'd',
  'E': 'e',
  'D': 'd',
};

toLowerCase['C'];
//=> 'c'

var isPrime = {
  1: false,
  2: true,
  3: true,
  4: false,
  5: true,
  6: false,
};

isPrime[3];
//=> true
```

當然了，你可能需要計算而不是手動將這些東西寫出來，不過這樣說明了透過不同方式來思考 function。（你可能會想「要是 function 有多個參數呢？」。的確，這種情況表明了以數學方式思考的一些不便。我們可以暫時將他們打包進陣列中，或者把 `arguments` object 看成是輸入。當我們學習 `curry` 的概念後，你就知道如何直接為 function 在數學上的定義建立 model。）

戲劇性的事：Pure function *就是*數學上的 function，而且是 functional programming 的全部。使用這些 pure function 來進行程式設計會帶來大量的好處。讓我們來看看為何要不遺餘力的保留 function 純粹性的原因。

## 追求「Pure」的理由

### 可快取性（Cacheable）

首先，pure function 可以根據輸入來做快取。其中一種典型的方式就是透過 memoization。

```js
var squareNumber = memoize(function(x) {
  return x * x;
});

squareNumber(4);
//=> 16

squareNumber(4); // 回傳輸入為 4 的快取結果
//=> 16

squareNumber(5);
//=> 25

squareNumber(5); // 回傳輸入為 4 的快取結果
//=> 25
```

下方是一個簡單的實作，即便有很多更強大的版本可做選擇。

```js
var memoize = function(f) {
  var cache = {};

  return function() {
    var arg_str = JSON.stringify(arguments);
    cache[arg_str] = cache[arg_str] || f.apply(f, arguments);
    return cache[arg_str];
  };
};
```

值得注意的是可以透過延遲執行的方式將 impure 的 function 轉換為 pure function：

```js
var pureHttpCall = memoize(function(url, params) {
  return function() {
    return $.getJSON(url, params);
  };
});
```

這裡有趣的地方在於我們並沒有真正的發送 http 請求－只是回傳了一個 function，當呼叫它的時候才會發送請求。這個 function 之所以 pure 是因為它會根據相同的輸入回傳相同的輸出：給定了 `url` 及 `params` 後，它只會回傳同一個 http 請求的 function。

我們的 `memoize` function 執行起來沒有任何問題，雖然他快取的不是 http 請求的結果，而是產生的 function。

目前看來這種方式的意義不大，但我們很快會學習一些技巧來發掘它的用處。重點是我們可以快取任何一個 function，不管它們看起來多麽具有破壞性。

### 可移植性（Portable） / 本身即文件（Self-Documenting）

Pure function 完全是自給自足的，它所需要的所有東西都能輕易取得。仔細想想⋯這種特性的好處為何呢？首先，function 的依賴會很明確，因此更易於觀察和理解－沒有偷偷摸摸的小動作。

```js
//impure
var signUp = function(attrs) {
  var user = saveUser(attrs);
  welcomeUser(user);
};

var saveUser = function(attrs) {
    var user = Db.save(attrs);
    ...
};

var welcomeUser = function(user) {
    Email(user, ...);
    ...
};

//pure
var signUp = function(Db, Email, attrs) {
  return function() {
    var user = saveUser(Db, attrs);
    welcomeUser(Email, user);
  };
};

var saveUser = function(Db, attrs) {
    ...
};

var welcomeUser = function(Email, user) {
    ...
};
```

這個例子說明了 pure function 對於依賴必須要誠實，這樣我們就能知道他的目的。從 `signUp` 就可以得知它會用到 `Db`、`Email` 即 `attrs`，這在最小程度上給了我們足夠的訊息。

往後我們會學習如何不透過這種延遲執行的方式讓一個 function 變為 pure function，不過這裡的重點是，對比 impure function，pure function 能夠提供更多的訊息，前者背後做了什麼事只有老天爺才知道。

此外要注意的是，我們透過強制「注入」依賴，或把他們當作參數傳遞，會讓我們的 app 變成更加靈活，因為資料庫或 mail client 等等都已經參數化了（別擔心，我們會有別種方式讓它不那麼單調乏味）。若要使用另一個 Db，我們只需要將他傳給 function 即可。若想在新的應用程式中使用這個可靠的 function，只要在那時將 `Db` 與 `Email` 傳遞進 function 就行了，相當容易。

在 JavaScript 設定中，可移植性代表可將 function serializing（序列化）並透過 socket 傳動。也可以表示程式碼能夠在所有的 app worker 中執行。總之，可移植性是一個相當強大的特性。

Imperative programming 中「典型」的方式與過程都深深的根值在他們的環境中，透過狀態、依賴即有效作用（available effects）達成；Pure function 則與此相法，只要我們願意，可以在任何地方執行它。

你上一次將類別方法複製到新的 app 中是何時？我最喜歡的名言是 Erlang 的作者 Joe Armstrong 所說的一句話：「物件導向語言的問題在於，它們隨身攜帶那些癮式的環境。你只要一根香蕉，但卻得到一個拿著香蕉的大猩猩⋯以及整片叢林」。

### 可測試性（Testable）

下一點，pure function 讓測試更加的容易。我們不需 mock 一個「真實的」付款閘道，或每次測試前都要 setup，測試之後都要 assert 狀態。我們只需要給定一個輸入，在 assert 輸出即可。

事實上，我們會發現 functional 的社群正在開創一個新的測試工具，能夠幫助我們自動產生輸入並 assert 輸出。這超出了本書的範圍，不過我強烈建議你去試試 *Quickcheck*－一個為 pure functional 環境量身打造的測試工具。

### 合理性（Reasonable）

很多人相信使用 pure function 最大的好處就是*引用透明性（referential transparency）*。如果一段程式碼可以替換成它執行後所得到的結果，而且是在不改變整個程式行為的前提下替換的，那麼我們可以說這段程式碼是引用透明的。

由於 pure function 能夠根據相同的輸入回傳相同的輸出，所以它們就能夠保證總是回傳同一個結果，這也就保證了引用透明性。我們來看一個例子。

```js

var Immutable = require('immutable');

var decrementHP = function(player) {
  return player.set('hp', player.get('hp') - 1);
};

var isSameTeam = function(player1, player2) {
  return player1.get('team') === player2.get('team');
};

var punch = function(player, target) {
  return isSameTeam(player, target) ? target : decrementHP(target);
};

var jobe = Immutable.Map({
  name: 'Jobe',
  hp: 20,
  team: 'red',
});
var michael = Immutable.Map({
  name: 'Michael',
  hp: 20,
  team: 'green',
});

punch(jobe, michael);
//=> Immutable.Map({name:'Michael', hp:19, team: 'green'})
```

`decrementHP`、`isSameTeam` 及 `punch` 都是 pure function，所以是引用透明的。我們可以透過一種稱做為*等式推導（equational reasoning）*的技術來分析程式碼，這技術就是就是以「一對一」進行替換，有點像是在不考慮程式性執行的怪異行為的情況下，手動執行相關程式碼。我們借助引用透明性來解析這段程式碼。

首先我們將 `isSameTeam` function 替換。

```js
var punch = function(player, target) {
  return player.get('team') === target.get('team') ? target : decrementHP(target);
};
```

因為我們資料是 immutable，所以可以直接將 teams 替換成實際值。

```js
var punch = function(player, target) {
  return 'red' === 'green' ? target : decrementHP(target);
};
```

因為執行結果為 false，所以可以將整行 if 刪除。

```js
var punch = function(player, target) {
  return decrementHP(target);
};

```

如果我們替換 `decrementHP`，我們會發現在這個情況下，punch 變為一個讓 `hp` 遞減 1 的 function。

```js
var punch = function(player, target) {
  return target.set('hp', target.get('hp') - 1);
};
```

總之，等式推導所帶來分析程式碼能力對重構與理解程式碼非常重要。事實上，我們重構 flock of seagulls 程式使用的正是這種技巧：透過等式推導利用加與乘的特性。我們會使用這些技巧貫穿全書。

### 並行程式碼

最後，也是決定性的一點：我們可以並行執行任何 pure function，因為 pure function 根本不需要存取共享的記憶體，而且根據其定義，它也不會因副作用而進入競爭狀態（race condition）。

這在伺服器端 js 環境及使用 web worker 的瀏覽器中是相當容易實現的，因為他們使用的執行緒（thread）。不過出於對 impure function 複雜度的考慮，目前主流的觀點還是避免使用並行。


## 總結

我們已經瞭解了何為 pure function，也看到作為 functional programmer 的我們，為何深信它們是不同凡響的。從這裡開始，我們將盡力以 pure 的方式撰寫所有 function。對此我們需要一些額外的工具來達成這個目的，同時也盡量把 impure function 從 pure 程式碼中分離。

如果手邊沒有一些額外的工具，那麼撰寫 pure function 的程式就會有點費力。我們必須透過到處傳遞參數來操作資料，何況狀態是禁止使用的，更別說作用了。誰願意這樣自虐自己寫程式？所以讓我們來學習一個叫做 curry 的新工具。

[第 4 章：Curry（柯里化）](ch4.md)
