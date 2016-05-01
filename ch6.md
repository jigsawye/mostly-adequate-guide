# 第 6 章：範例應用程式

## 宣告式開發

從本章節開始，我們要開始轉變我們的觀念了，我們會停止告訴電腦該要怎麼去工作，而是透過撰寫規範來得到我們要的結果。我相信透過這種方式與無時無刻去關心所有程式碼細節相比，這會讓你感到輕鬆許多。

不同於命令式，宣告式意指我們將撰寫一些表達式的程式，而不是一步一步指示。

像是 SQL，就沒有「先做這個，再做那個」的命令。它有一個表達式來指定我們想要從哪個資料庫使用資料，我們不確定是如何執行的，要看表達式本身。當資料庫更新和 SQL 引擎優化，我們不需要改變我們查詢的方式。這是因為有許多方式來解析我們規範的表達式並得到相同的結果。

對於一些人來說，包含我自己，第一次很難掌握這種宣告式的概念，所讓我們列出一些範例來感受一下。

```js
// 命令式
var makes = [];
for (var i = 0; i < cars.length; i++) {
  makes.push(cars[i].make);
}


// 宣告式
var makes = cars.map(function(car) { return car.make; });
```

命令式第一次必須先實例化陣列。在執行後面的程式碼之前，直譯器必須先評估這個語句，然後才迭代整個 cars 的清單，在顯式的迭代中，手動增加計數器並顯示零碎的資訊給我們實在是不怎麼好。

`map` 版本是一個表達式。它對執行的順序沒有要求。在 map function 迭代並回傳的陣列集合，對於指定**做什麼**而不是**怎麼做**有很大的自由度。因此，它完全是一個宣告式的程式。

除了更簡潔明瞭外，map function 還可以進行優化，這樣我們的應用程式的程式碼就不需要改變了。

對那些認為「對啊，但是使用命令式迴圈比較快」的人，我建議你先去了解關於 JIT 優化的相關程式碼。這裡有一個[非常棒的影片](https://www.youtube.com/watch?v=65-RbBwZQdU)，或許可以有一些啟發。

這裡是另一個範例。

```js
// 命令式
var authenticate = function(form) {
  var user = toUser(form);
  return logIn(user);
};

// 宣告式
var authenticate = compose(logIn, toUser);
```

雖然命令式的版本不是絕對錯誤的，但還是像存在那種一步一步的硬編碼方式。`compose` 表達式只是簡單的指出一個事實：驗證是 `toUser` 和 `logIn` 兩個行為的組合。再者，宣告式的程式支援更新程式碼，使我們的應用程式可以成為一個高級的規範。

因為宣告式的程式不指定執行順序，所以適合運用在平行計算。它與 pure function 一起解釋了 functional programming 對於平行計算的未來是一個很好的選擇 - 我們真的不需要做什麼就能實現平行化的系統。

## 一個 FP 的 flickr

我們使用宣告式和可組合的方式來建立一個應用程式範例。我們現在還是會使到一些 side effects，但我們會把 side effects 降到最低，讓他與 pure function 的部份分離開來。我們要建立一個瀏覽器的 widget，從 flickr 上取得圖片並顯示。讓我們從 app 的 scaffolding 開始。這裡是 html 部份：


```html
<!DOCTYPE html>
<html>
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.1.11/require.min.js"></script>
    <script src="flickr.js"></script>
  </head>
  <body></body>
</html>
```

這裡是 flickr.js 的 skeleton：

```js
requirejs.config({
  paths: {
    ramda: 'https://cdnjs.cloudflare.com/ajax/libs/ramda/0.13.0/ramda.min',
    jquery: 'https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min'
  },
});

require([
    'ramda',
    'jquery',
  ],
  function(_, $) {
    var trace = _.curry(function(tag, x) {
      console.log(tag, x);
      return x;
    });
    // app 在此處
  });
```

這裡我們使用了 [ramda](http://ramdajs.com) 而不是 lodash 或其他的 library。它包含了 `compose`、`curry` 等等。我以前使用過 requirejs，或許看起來有點矯枉過正，但為了保持一致性，在本書我們會一直使用它。另外，我已經將 `trace` function 寫好，讓我們可以方便的 debug。

有點離題了，言歸正傳，我們的 app 需要做以下這四件事情：

1. 對於我們特定的搜尋條件來建構一個 url
2. 讓 flickr api 呼叫
3. 將回傳的 json 結果轉換成 html 的圖片
4. 將圖片放置在螢幕上

如上面所述，有兩個 impure 的行為。你看到了嗎？就是從 flickr api 取得資料和在螢幕上放置圖片。我們先來定義這兩個動作，這樣就可以隔離他們。

```js
var Impure = {
  getJSON: _.curry(function(callback, url) {
    $.getJSON(url, callback);
  }),

  setHtml: _.curry(function(sel, html) {
    $(sel).html(html);
  })
};
```

這裡我們簡單的將 jQuery 的方法包裝成 curry，這有便於幫助參數位置的交換。我已經使用了 `Impure` 的命名空間，這樣我們就知道這些 function 不安全。在之後的範例，我們將會讓這兩個 function 變為 pure function。

接下來我們必須建構一個 url 來傳送我們的 `Impure.getJson` function。

```js
var url = function(term) {
  return 'https://api.flickr.com/services/feeds/photos_public.gne?tags=' +
    term + '&format=json&jsoncallback=?';
};
```

使用 monoids（我們在之後會學習到）或 combinator 可以使用一些奇技淫巧讓 `url` function 成為 `pointfree` function。但是為了可讀性，我們還是選擇以普通非 pointfree 的方式拼接字串。

讓我們撰寫一個 app function 來發送呼叫，並將內容顯示在螢幕上。

```js
var app = _.compose(Impure.getJSON(trace('response')), url);

app('cats');
```

這會呼叫 `url` function，然後將字串傳送給 `getJSON` function，在某些部份上已經應用到了 `trace`。載入 app 後，從 api 呼叫的 response 會顯示在 console 上。

<img src="images/console_ss.png"/>

我們想要從 json 來建構圖片。看起来 src 都在 `items` 陣列中的每個 `media` 物件的 `m` 屬性上。

不管如何，如果要取得這些巢狀的屬性，我們可以從 ramda 中使用一個很棒的通用 getter function 叫做 `_.prop()`。不過為了讓你了解這個 function 做了些什麼，我們先自己實現一個 prop：

```js
var prop = _.curry(function(property, object) {
  return object[property];
});
```

這實際上有點傻，我們只是使用 `[]` 語法來存取物件的屬性。讓我們使用它來取得我們的 src：

```js
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var srcs = _.compose(_.map(mediaUrl), _.prop('items'));
```

一旦得到 `items`，我們必須使用 `map` 來提取每個 media 的 url。這樣就會得到一個 src 的陣列。讓我們將它加到 app 上，並將圖片顯示在螢幕上。

```js
var renderImages = _.compose(Impure.setHtml('body'), srcs);
var app = _.compose(Impure.getJSON(renderImages), url);
```

這裡所做的只不過是建立了一個組合，這個组合會呼叫 `srcs` function，並把回傳结果設定為 body 的 html。我們也把 `trace` 替換成了 `renderImages`，現在我們除了 render 原始的 json，會將我們的 src 直接顯示在我們的 body。

我們最後一步是將這些 src 轉換成真正的圖片。在大型應用程式中，我們使用像是 Handlebars 或 React 這樣的 template／dom library。但是對於這個應用程式來說，我們只需要一個 img 標籤，所以只要使用 jQuery 就可以了。

```js
var img = function(url) {
  return $('<img />', {
    src: url
  });
};
```

jQuery 的 `html()` 方法接受一個標籤陣列。我們只要將 src 轉換成圖片並傳送給 `setHtml` 就可以了。

```js
var images = _.compose(_.map(img), srcs);
var renderImages = _.compose(Impure.setHtml('body'), images);
var app = _.compose(Impure.getJSON(renderImages), url);
```

我們完成了！

<img src="images/cats_ss.png" />

這裡是完成後的程式碼：
```js
requirejs.config({
  paths: {
    ramda: 'https://cdnjs.cloudflare.com/ajax/libs/ramda/0.13.0/ramda.min',
    jquery: 'https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min',
  },
});

require([
    'ramda',
    'jquery',
  ],
  function(_, $) {
    ////////////////////////////////////////////
    // Utils

    var Impure = {
      getJSON: _.curry(function(callback, url) {
        $.getJSON(url, callback);
      }),

      setHtml: _.curry(function(sel, html) {
        $(sel).html(html);
      }),
    };

    var img = function(url) {
      return $('<img />', {
        src: url,
      });
    };

    var trace = _.curry(function(tag, x) {
      console.log(tag, x);
      return x;
    });

    ////////////////////////////////////////////

    var url = function(t) {
      return 'http://api.flickr.com/services/feeds/photos_public.gne?tags=' +
        t + '&format=json&jsoncallback=?';
    };

    var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

    var srcs = _.compose(_.map(mediaUrl), _.prop('items'));

    var images = _.compose(_.map(img), srcs);

    var renderImages = _.compose(Impure.setHtml('body'), images);

    var app = _.compose(Impure.getJSON(renderImages), url);

    app('cats');
  });
```

現在看看這些，多麼美妙的宣告式規範啊！現在我們可以把每一行程式碼都視為方程式和屬性。我們可以使用這些屬性去合理判斷關於我們應用程式以及重構。

## 有原則的重構

上面的程式碼還是可以優化的，我們 map 每個項目將它們轉換成 media url，然後我們再 map src 將它們轉換成 img 的標籤。關於 map 和組合是有定律的：


```js
// map 的結合律
var law = compose(map(f), map(g)) === map(compose(f, g));
```

我們可以利用這個定律優化程式碼，進行一次有原則的重構。

```js
// 原始程式碼
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var srcs = _.compose(_.map(mediaUrl), _.prop('items'));

var images = _.compose(_.map(img), srcs);

```

讓我們將 map 排序吧。感謝等式推導（equational reasoning）以及 pure function 的特性，我們可以在 `images` 呼叫 `srcs`。

```js
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var images = _.compose(_.map(img), _.map(mediaUrl), _.prop('items'));
```

把 `map` 排成一列後，就可以應用結合律了。

```js
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var images = _.compose(_.map(_.compose(img, mediaUrl)), _.prop('items'));
```

現在只需要一次迴圈，就可以把每個物件都轉換成一個 img 了。我們透過提取 function，讓 function 可以變得更具可讀性。

```js
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var mediaToImg = _.compose(img, mediaUrl);

var images = _.compose(_.map(mediaToImg), _.prop('items'));
```

## 總結

我們已經看到如何使用一個小而巧的新技術放入我們的真實應用的 app。我們使用我們的數學框架來思考和重構我們的程式碼。但是錯誤處理的程式碼部分呢？我們如何讓整個應用程式都是 pure 的，而不是將破壞性的 function 放入到命名空間下？我們如何使我們的應用程式更安全且更具有表現？這是本書在第二部分將要處理的問題。

[第 7 章：Hindley-Milner 與我](ch7.md)
