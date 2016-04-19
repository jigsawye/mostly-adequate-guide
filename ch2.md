# 第 2 章：First Class Funtcion

## 快速導覽
當我們說 function 是「First Class（頭等）」時，意指他們就像其他人一樣⋯所以就是 Normal Class（坐經濟艙的人？）。function 真的沒什麼特別的地方，你可以像對待其他資料型別一樣對待它們－他們可以被存在陣列中，當作參數傳遞，賦予至變數⋯等等。

這是 JavaScript 的基礎概念，不過還是值得一提的，因為在 Github 上隨便搜尋就能看到對這個概念的集體無視，或者也可能是無知。我們來看看假設的例子嗎：

```js
var hi = function(name) {
  return 'Hi ' + name;
};

var greeting = function(name) {
  return hi(name);
};
```

在這裡 `greeting` 將 `hi` 封裝一次的 function 完全是多餘的。為何？因為在 JavaScript 中 function 是可呼叫的（callable）。當 `hi` 後方接著 `()` 時就會執行並回傳一個值；若無，它就會簡單的回傳儲存在這個變數裡的 function。讓我們來確認一下：


```js
hi;
// function(name) {
//  return 'Hi ' + name
// }

hi('jonas');
// "Hi jonas"
```

`greeting` 只是轉個彎後使用相同的參數呼叫 `hi` 而已，所以我們可以簡單的寫成：

```js
var greeting = hi;


greeting('times');
// "Hi times"
```

換句換說，`hi` 已經是個只接受一個參數的 function 了，為何要將它再額外封裝進 function，而僅僅是使用同樣的參數呼叫 `hi`？完全沒有道理。這就像是在夏天穿上你最厚的大衣，只是為了跟熱空氣過意不去，然後再吃冰棒。簡直多此一舉。

用一個 function 將另一個 function 封裝起來的目的，只是為了延遲執行，這真的是非常糟糕的習慣。（稍後我會告訴你為何，這與可維護性密切相關。）

充分理解這個問題對讀懂本書後面的內容至關重要，所以我們再來看看幾個來自 npm module 的例子。

```js
// 無知
var getServerStuff = function(callback) {
  return ajaxCall(function(json) {
    return callback(json);
  });
};

// 合理
var getServerStuff = ajaxCall;
```

世界上到處都充斥著這樣的垃圾 ajax 程式碼。以下是為何兩種寫法等價的原因：

```js
// 這行
return ajaxCall(function(json) {
  return callback(json);
});

// 等價於這行
return ajaxCall(callback);

// 所以重構一下 getServerStuff
var getServerStuff = function(callback) {
  return ajaxCall(callback);
};

// ...就等於
var getServerStuff = ajaxCall; // <-- 看，沒有 () 喔
```

各位，以上才是寫 function 的正確方式。晚點再告訴你為何我對此如此執著。

```js
var BlogController = (function() {
  var index = function(posts) {
    return Views.index(posts);
  };

  var show = function(post) {
    return Views.show(post);
  };

  var create = function(attrs) {
    return Db.create(attrs);
  };

  var update = function(post, attrs) {
    return Db.update(post, attrs);
  };

  var destroy = function(post) {
    return Db.destroy(post);
  };

  return {
    index: index,
    show: show,
    create: create,
    update: update,
    destroy: destroy,
  };
})();
```

這個可笑的 controller 有 99% 的程式碼都是垃圾。我們可以將它重寫成這樣：

```js
var BlogController = {
  index: Views.index,
  show: Views.show,
  create: Db.create,
  update: Db.update,
  destroy: Db.destroy,
};
```

⋯或是直接全部刪掉，因為它的作用只是將 Views 與 DB 打包在一起而已。

## 為何鍾愛一等公民？

Okay，現在讓我們來看看鍾愛一等公民的原因為何。前面我們看過了 `getServerStuff` 與 `BlogController` 兩個例子，雖然增加一些沒有實際用途的間接層相當容易，但這樣做除了徒增加程式碼量，提高維護及查詢的成本外，沒有任何用處。

此外，如果一個 function 被不必要的封裝起來，當它發生變更時，我們也必須變更封裝它的 function。

```js
httpGet('/post/2', function(json) {
  return renderPost(json);
});
```

如果 `httpGet` 要改成可能送出一個 `err`，我們必須回頭修改「glue」。

```js
// 將應用程式中每個 httpGet 的呼叫都要改成這樣，才能傳遞 err
httpGet('/post/2', function(json, err) {
  return renderPost(json, err);
});
```

寫成 first class function，要做得更動將會少很多：

```js
// renderPost 會在 httpGet 中被呼叫，想要多少參數都可以
httpGet('/post/2', renderPost);
```

為了刪除不必要的 function，正確地為參數命名也必不可少。當然命名不是什麼大問題。但還是有可能存在一些不當的命名－尤其隨著程式碼的增長及需求的變更時。

專案中常見的混淆原因是，針對一個相同的概念使用不同的命名。在通用的程式碼也有相同的問題。舉例來說，下方兩個 function 做的事情是相同的，但後者比前者更加通用，可重用性也更高：


```js
// 針對目前的 blog
var validArticles = function(articles) {
  return articles.filter(function(article) {
    return article !== null && article !== undefined;
  });
};

// 對未來的專案友好多了
var compact = function(xs) {
  return xs.filter(function(x) {
    return x !== null && x !== undefined;
  });
};
```

在命名時，我們特別容易將自己限定在特定的資料（在本例中是 `articles`）。這種現象很常見，也是重複造輪子的一大原因。

有一點我必須指出，就像物件導向的程式碼，你必須非常小心 `this` 反咬你一口。如果一個底層 function 使用了 `this`，而且是以 first class 的方式被呼叫，我們都被這個充滿漏洞的抽象概念給惹怒。

```js
var fs = require('fs');

// 太可怕了
fs.readFile('freaky_friday.txt', Db.save);

// 好一些
fs.readFile('freaky_friday.txt', Db.save.bind(Db));

```

將 `Db` bind 到他自身後，你就可以隨心所欲的呼叫它的原型鏈式垃圾程式碼了。`this` 就像一片髒尿布，我盡可能地避免使用它，因為在 functional 的程式碼中根本用不到它。但是，在使用其它的 library 時，你卻不得不向這個瘋狂的世界低頭。

也有人會反駁說為了速度 `this` 是必須的。如果你是這種對速度吹毛求疵的人，那你還是合上這本書吧。如果你沒辦法退款，也許你能去換一本更入門的書來讀。

至此，我們才準備好繼續後面的章節。

[第 3 章：Pure Function－單純的幸福](ch3.md)
