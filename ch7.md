# 第 7 章：Hindley-Milner 與我

## 你的型別？
如果你是第一次接觸到 functional programming，很容易深陷型別特徵（type signatures）的泥淖。類型是 meta 的語言，讓不同背景的人們可以簡潔有效的溝通。大多數情況下，他們寫了一個叫做「Hindley-Milner」的系統，我們會在這個章節研究這個系統。

當使用 pure function 時，型別特徵在表達上強而有力，這是英文不能比擬的。這些簽名在你耳邊低語，告訴你 function 的祕密。簡單一行，就可以暴露 function 的行為和意圖。型別特徵衍生出了 「free theorems」 的定理。因為類型是可以推斷的，所以不需要顯式的類型註釋，不過你可以撰寫精確度很高的型別特徵，讓他們保持通用和抽象。型別特徵不只可以用在編譯時進行檢查，而且變成是最好可用的文件。因此型別特徵在 functional programming 扮演一個重要的角色 - 重要程度超越你的想像。

JavaScript 是一門動態類型的語言，但這不代表我們要完全去避免所有類型。我們還是要使用到字串、數字、布林值等等。只不過，語言層面上沒有相關的集成讓我們時刻謹記各種資料類型。別擔心，因為我們使用簽名的文件，所以我們可以使用 comment 來達到這個目的。

JavaScript 也有一些類型檢查工具，像是 [Flow](http://flowtype.org/) 或是靜態類型的 [TypeScript](http://www.typescriptlang.org/)。由於本書目的是讓讀者使用工具去撰寫 functional 的程式碼，所以我們堅持使用跨 FP 語言的標準類型系統。


## 神祕的傳奇故事

從積塵已久的數學書籍，到浩如煙海的學術論文；不經意在週六上午看到的部落格文章，到原始碼本身，我們都能發現 Hindley-Milner 型別特徵的身影。Hindley-Milner 系統相當的簡單，但還是需要一些解釋和練習才能充分的掌握這個小語言的精髓。

```js
//  capitalize :: String -> String
var capitalize = function(s) {
  return toUpperCase(head(s)) + toLowerCase(tail(s));
}

capitalize("smurf");
//=> "Smurf"
```

這裡，`capitalize` 接受一個 `String` 並回傳了一個 `String`。先不管如何實現，我們感興趣的的是它的型別特徵。

在 Hindley-Milner 系统中，function 都寫成像是 `a -> b` 這樣子，其中 `a` 和 `b` 是任意類型的變數。所以 `capitalize` 的簽名可以解讀為「一個接受 `String` function 回傳一個 `String`」。換句話說，接受一個 `String` 類型作為輸入，然後回傳一個 `String` 類型作為輸出。

讓我們看更多 function 的簽名：

```js
//  strLength :: String -> Number
var strLength = function(s) {
  return s.length;
}

//  join :: String -> [String] -> String
var join = curry(function(what, xs) {
  return xs.join(what);
});

//  match :: Regex -> String -> [String]
var match = curry(function(reg, s) {
  return s.match(reg);
});

//  replace :: Regex -> String -> String -> String
var replace = curry(function(reg, sub, s) {
  return s.replace(reg, sub);
});
```

`strLength` 和 `capitalize` 類似，接受一個 `String` 然後回傳一個 `Number`。

至於其他的，第一次看起來可能會令人疑惑。不過在不了解的情況下，你可以把最後一個類型當作回傳值。所以 `match` 你可以解釋成：它接受一個 `Regex` 和一個 `String` 然後回傳 `[String]`。但是這裡有一個非常有趣的地方，稍後我會做解釋。

對於 `match` function，我們可以把它的型別特徵這樣分組：

```js
//  match :: Regex -> (String -> [String])
var match = curry(function(reg, s) {
  return s.match(reg);
});
```

是的，最後括號內的分組揭露了更多的資訊，現在我們可以看到 `match` function 接受一個 `Regex` 然後回傳一個然後回傳一個 `String` 到 `[String]` function。因為 curry 的緣故，所以造成如此的結果：給 `match` function 一個 `Regex`，然後得到一個處理 `String` 參數的新 function。當然，我們不一定要這麼思考，但這樣思考可以幫助我們理解為什麼最後一個類型是回傳值。

```js
//  match :: Regex -> (String -> [String])

//  onHoliday :: String -> [String]
var onHoliday = match(/holiday/ig);
```

每傳一個參數，就會彈出型別特徵最前面的那個類型。所以 `onHoliday` 就是已經有了 `Regex` 參數的 `match`

```js
//  replace :: Regex -> (String -> (String -> String))
var replace = curry(function(reg, sub, s) {
  return s.replace(reg, sub);
});
```

正如你所見的所有在，在 `replace` 外加上這麼多括號未免有些多餘，這裡的括號是可以省略的。如果可以的話，我們可以一次將所有參數傳入：`replace` 帶有一個 `Regex`、`String` 和另一個 `String`，回傳的還是一個 `String`。

這裡有最後幾件事情：


```js
//  id :: a -> a
var id = function(x) {
  return x;
}

//  map :: (a -> b) -> [a] -> [b]
var map = curry(function(f, xs) {
  return xs.map(f);
});
```

`id` function 接受任何類型的 `a`，並回傳相同類型的 `a`。我們可以在程式中使用類型的變數。變數名稱像是 `a` 和 `b` 只是一個慣例，但是他們可以任意被替換成你想要的名稱。如果它們是相同的變數，它們必須視同一個類型。這是很重要的規則，讓我們重申：`a -> b` 可以是任何的類型 `a` 到 任何類型 `b`，但是 `a -> a` 必須是要相同的類型。例如，`id` 可能是 `String -> String` 或 `Number -> Number`，而不是 `String -> Bool`。

`map` 同樣的使用了類型變數，但是這裡的 `b` 可能與類型 `a` 相同，也可能不同。我們可以這樣解讀：`map` 接受兩個參數，第一個是任意類型 `a` 到任意類型 `b` 的 function；第二個是一個陣列，元素是任意類型的 `a`；`map` 最後回傳的是一個 `b` 類型的陣列。

型別特徵的美妙令人印象深刻，希望你已經被他深深吸引。型別特徵從字面上告訴我們 function 做了哪些事情。給定一個從 `a` 到 `b` 的 function 和一個 `a` 的陣列作為參數，然後回傳一個 `b` 的陣列。`map` 唯一的明智之舉就是使用 function 去呼叫每一個 `a`，其他的操作都是多餘的。

辨別類型和它們的含義是一項重要的技能，這項技能可以讓你在 functional programming 的路上走得更遠。不只是論文、部落格、文件等等其他可以更容易理解，型別特徵本身基本上也能夠告訴你他的函式性（functionality）。要成為能夠熟練型別特徵的人，你必須要勤於練習；不過如果堅持下去，你將受益無窮。

這裡還有些例子，你可以試試看能不能理解它們。

```js
//  head :: [a] -> a
var head = function(xs) {
  return xs[0];
};

//  filter :: (a -> Bool) -> [a] -> [a]
var filter = curry(function(f, xs) {
  return xs.filter(f);
});

//  reduce :: (b -> a -> b) -> b -> [a] -> b
var reduce = curry(function(f, x, xs) {
  return xs.reduce(f, x);
});
```

`reduce` 也許是型別特徵中最具表現力的一個，但同時也是最複雜的一個，如果在理解它感到困難的話，也不要氣餒。為了滿足你的好奇心，我會嘗試去解釋，儘管我的解釋遠遠不如你自已去理解型別特徵來的有效。

咳咳，這裡不保證完全正確...注意到 `reduce` 的簽名，可以看到第一個參數是 function，接受一個 `b` 和 `a` 然後產生一個 `b`，那麼這些 `a` 和 `b` 是哪裡來的呢？很簡單，在簽名中的第二個和第三個參數就是 `b` 和 `a` 陣列，所以唯一合理的假設就是這裡的 `b` 和每一個 `a` 都將前面的 function 作為參數。我們也可以看到 reduce function 的最後回傳的結果是一個 `b`，也就是說 `reduce` 的第一個參數 function 就是 `reduce` function 的輸出。知道了 `reduce` 的含義，我們才敢說上面關於型別特徵的推理是正確的。


## 縮小可能性範圍

一旦引入了類型變數，就會出現一個奇怪的特性叫做 *parametricity*(http://en.wikipedia.org/wiki/Parametricity)。這個特性狀態表示一個 function 會*以一種統一的行為作用於所有的類型*。讓我們來探討：

```js
// head :: [a] -> a
```

注意到 `head`，它接受一個 `[a]` 然後回傳 `a`。除了知道參數是一個 `array` 外，其他的我們一概不知，所以，function 的操作只限於在這個陣列。在它對 `a` 一無所知的情況下，它能對 `a` 做什麼呢？換句話說，`a` 表示它沒有*指定*的類型，意思說他可是*任意*的類型，那麼我們對*每一個* function 的處理都必須保持一致。這就是關於 *parametricity* 的涵義。要讓我們來猜測 `head` 的實現的話，唯一合理的推斷就是它回傳陣列的第一個，或者最後一個，或者某個隨機的元素；當然，`head` 這個命名應該能給我們一些線索。

這裡有另一個例子：

```js
// reverse :: [a] -> [a]
```

只從型別特徵來看，`reverse` 的目的可能是什麼？它不能對 `a` 做任何特定的事情。它不能把 `a` 類型改變成其他類型，或者引入一個 `b`。那麼它可以排序嗎？答案是不行的，它沒有足夠的資訊去排序每個可能的類型。它可以重新排列嗎？可以的，我覺得它可以，但它必須以一種可預測的方式達成。另一種可能性是，它可能會刪除或重複某個元素。重點是，不管在哪種情況下，類型 a 的多態性（polymorphism）都會大幅縮小 reverse 函數可能的行為的範圍。

這種「可能性範圍的縮小」（narrowing of possibility）允許我們利用類似 [Hoogle](https://www.haskell.org/hoogle) 這樣的型別特徵搜尋引擎去搜索我們想要的 function。 型別特徵所能包含的資訊量真的非常大。

## 自由定理

型別特徵除了能夠説明我們推斷函數可能的實現，還能夠給我們帶來*自由定理*（free theorems）。下面是兩個直接從[Wadler 關於此主題的論文](http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf)中隨機播放的例子。

```js
// head :: [a] -> a
compose(f, head) == compose(head, map(f));

// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) === compose(filter(p), map(f));
```


你不需要任何程式碼也能了解這些定理，它們直接來自於類型本身。第一個意思是說如果我們要取得陣列的 `head`，然後執行 function `f`，這相等且相較於如果我們第一次使用 `map(f)` 來取得每個元素 `head` 的結果來的要快。

你可能會想，這些都只是常識。但是根據我的調查，電腦是沒有常識的。Indeed, 實際上，它們必須有正市的方式來自動化類似這些程式碼的優化。數學提供了這種方法，能夠形式化直觀的感覺，這對死板的電腦邏輯非常有用。

它說如果我們 compose `f` 和 `p` 來確認那些應該被過濾，然後實際上經由 `map` 來使用 `f`（別忘了 filter 是不會改變陣列的元素，這就保證 `a` 是保持不變），這會相等於 `map` 我們的 `f` 然後根據 `p` 過濾結果。

以上只是兩個例子，但你可以將這種推理應用到任何多態型別特徵。在 JavaScript，有一些工具可用來宣告重寫規則。一個可能就是藉由 `compose` function 本身。總之，這麼做的好處是顯而易見且唾手可得的，可能性則是無限的。

## 類型約束

最後要注意的一點是，簽名也可以把類型約束為一個特定的介面（interface）。

```js
// sort :: Ord a => [a] -> [a]
```

fat arrow 左邊表明這是一個事實：`a` 一定是個 `Ord` 物件。或是換句話說，`a` 必須實作 `Ord` 介面。`Ord` 是什麼，它從哪來的？在一門強型別語言中，它可能就是一個自訂的介面，能夠讓不同的值排序。這不僅告訴我們更多關於 `a` 資訊和 `sort` function 具體是在做什麼，而且還能限制函數的作用範圍。我們稱這種介面宣告叫*類型約束*。

```js
// assertEqual :: (Eq a, Show a) => a -> a -> Assertion
```

這裡我們有兩個約束：`Eq` 和 `Show`。它們確保我們可以檢查不同的 `a` 是否相等，並在有不相等的情況下列印出其中的差異。

我們將會在後面的章節中看到更多類型約束的例子，其含義也會更加清晰。


## 總結

Hindley-Milner 類型在 functional programming 的世界無所不在。它們簡單易讀，撰寫也不複雜，但僅僅憑簽名就能理解整個程式還是有一定難度的，要想精通這個技術就更需要花點時間了。 從這開始，我們將在每一行程式碼都加上型別特徵。

[第八章：Tupperware](ch8.md)
