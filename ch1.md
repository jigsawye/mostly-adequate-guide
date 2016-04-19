# 第 1 章：我們在做些什麼？

## 簡介

嗨！我是 Franklin Risby 教授，很高興認識你。接下來我們將共度一段時光，因為我要教你一些 functional programming 的知識。關於我就到此為止，那麼你呢？我希望你已經熟悉了 JavaScript 語言，關於物件導向也有一些經驗，而且自認為是一名合格的程式設計師。你並不需要擁有昆蟲學博士學位，你只需要知道如何找到並殺死一些 bugs。

我不假設你之前有任何 functional programming 的知識，我們都知道假設的後果是什麼。但我猜想你在使用可變狀態（mutable state）、不受限的副作用（unrestricted side effects）及無原則設計（unprincipled design）的過程中已經遇過一些麻煩。現在我們已經介紹的差不多了，接著讓我們切入正題吧。

本章節的目的是讓你了解為何我們要撰寫 functional 的程式。我們必須了解為何讓讓一個程式 *functional*，否則我們會發現自己漫無目的的避免使用物件－無疑是在做白工。寫程式時需要遵照一定的原則，就像在《激戰》遊戲中當水變成石頭時你需要天國羅盤來指引。

現在已經有一些通用的程式開發原則－各種縮寫詞帶領我們在應用程式的黑暗隧道中前進：DRY（don't repeat yourself，不重複程式），YAGNI（ya ain't gonna need it，你不會需要它），高內聚低耦合（loose coupling high cohesion），最少意外原則（principle of least surprise），單一責任原則（single responsibility）等等。

我當然不會囉唆的把這些年我所聽到的原則都列舉出來⋯重點是這些原則都適用於 functional 的情況，只是它們與我們的目的不太有關係。在我們深入主題前，我想先讓你有一種感覺，當你在敲打鍵盤時內心就能強烈感受到 functional 的氛圍。

<!--BREAK-->

## 相見恨晚

讓我們以一個簡單的例子開始。這是一個海鷗（seagull）的應用程式。當鳥群合併（conjoin）後牠們會變成更大的鳥群，繁殖（breed）則會增加他們的數量，所增加的數目為他們自身所繁殖出的數量。目前這不是物件導向的良好實踐，只是為了強調這種賦值方式所造成的弊端。看看吧：

```js
var Flock = function(n) {
  this.seagulls = n;
};

Flock.prototype.conjoin = function(other) {
  this.seagulls += other.seagulls;
  return this;
};

Flock.prototype.breed = function(other) {
  this.seagulls = this.seagulls * other.seagulls;
  return this;
};

var flock_a = new Flock(4);
var flock_b = new Flock(2);
var flock_c = new Flock(0);

var result = flock_a.conjoin(flock_c)
    .breed(flock_b).conjoin(flock_a.breed(flock_b)).seagulls;
//=> 32
```

誰的手法會寫出這麼的令人退步三舍的程式？內部的可變狀態相當的難以追蹤，而且我的天啊，答案居然還是錯的！正確答案應該是 `16`，但是因為 `flock_a` 在計算過程中被永久改變了，可憐的 `flock_a`。這是 I.T. 部門混亂的表現，非常粗暴的計算方式！

如果你看不懂這支程式，是沒關係的，因為我也看不懂。重點是狀態及可變值非常難追蹤，即使這是一個小小的範例。

讓我們試試另一種更 functional 的方式：

```js
var conjoin = function(flock_x, flock_y) { return flock_x + flock_y; };
var breed = function(flock_x, flock_y) { return flock_x * flock_y; };

var flock_a = 4;
var flock_b = 2;
var flock_c = 0;

var result = conjoin(
  breed(flock_b, conjoin(flock_a, flock_c)), breed(flock_a, flock_b)
);
//=>16
```

很好，這次我們得到正確的答案。而且少了很多程式碼。不過巢狀 function 有點讓人難以理解⋯（我們會在第 5 章解決這個情形）。這種寫法更好，不過程式碼肯定是越直白越好，所以讓我們更深入探討。瞭解之後，我們會發現我們只是很簡單的進行相加（`conjoin`）及相乘（`breed`）。

除了兩個 function 的名稱比較特殊外，其他沒有任何難以理解之處。讓我們重新命名這些 function 來揭曉它們的真面目。

```js
var add = function(x, y) { return x + y; };
var multiply = function(x, y) { return x * y; };

var flock_a = 4;
var flock_b = 2;
var flock_c = 0;

var result = add(
  multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b)
);
//=>16
```
這麼一來，我們會發現這些只是古人所流傳下來的知識：

```js
// 結合率（associative）
add(add(x, y), z) === add(x, add(y, z));

// 交換律（commutative）
add(x, y) === add(y, x);

// 同一律（identity）
add(x, 0) === x;

// 分配綠（distributive）
multiply(x, add(y,z)) === add(multiply(x, y), multiply(x, z));
```

是的，這些古老又堅定的數學特性早晚會派上用場。若你一時想不起來也沒關係，大多數的人已經很久沒複習這些資訊了。讓我們看看是否能利用這些定律簡化這個海鷗程式。

```js
// 原有程式碼
add(multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b));

// 套用同一律來移除多餘的相加 (add(flock_a, flock_c) == flock_a)
add(multiply(flock_b, flock_a), multiply(flock_a, flock_b));

// 套用分配律來達到我們的結果
multiply(flock_b, add(flock_a, flock_a));
```

漂亮！除了呼叫的 function 外我們不必再多寫多餘的程式碼。我們定義了 `add` 及 `multiply` 只是為了完整性，但實際上我們並不需要撰寫－在某些已經寫好的 library 裡一定包含了 `add` 及 `multiply`。

你可能在想「你前面舉這種助學例子也太曲解論點了吧」。或者「真實的程式才不會這麼簡單，不能以這樣的方式來推斷」。我會選擇這個例子是因為大部分的人已經知道如何相加及相乘，所以這很容易讓我們發現可以在這裡使用數學。

別感到絕望－在本書中還會穿插一些範疇論（category theory）、集合論（set theory）及 lamdba 運算的知識，教你著寫更複雜的程式碼，而且一點也不輸本章海鷗程式的簡潔性與準確性。你也不需成為一名數學家，本書要交給你的程式設計模式實踐起來，就像是使用一個普通的框架或 API 一般。

你也許會感到訝異，我們可以向上例那樣遵循 functional 的模式去撰寫完整且日常的應用程式，有著優異效能的程式、簡潔易推理的程式，以及不用每次都重新造輪子的程式。如果你是罪犯，那違法對你來說是件好事；但在本書中，我們希望能夠承認並遵守數學之法。

我們希望去實踐每一部份都能完美結合的理論，希望能以一種通用、可組合的元件來描述我們的特定問題，然後利用這些元件的特性來解決這些問題。對於 imperative programming（指令式程式開發，稍後本書將會介紹 imperative 的經確定義，我們暫時還是先將重點放在 functional 上）那種「某某去做某事」的方式，functional programming 將會有更多的約束，不過你會震攝於這種強約束、數學性的框架所帶來的回報。

我們已經看到 functional 的點點星光了，但在真正開始我們的旅程之前，必須先掌握一些具體的概念。

[第 2 章：First Class Function](ch2.md)
