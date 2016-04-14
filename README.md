[![cover](images/cover.png)](SUMMARY.md)

## 關於本書

此書主題為 functional 模式。我們將使用世界上最熱門的 functional programming 語言：JavaScript 來講述此主題。有些人可以認為這個選擇並不明智，因為目前的主流觀點認為他是一個命令式（imperative）的語言，並不適合用來講述 functional。但我認為這是學習 FP 最好的方式，有幾點原因：

 * **你可能每天都在工作中使用到它。**

    這讓你有機會在實際的程式開發過程中學以致用，而不是在空閒時間將一門深奧的 FP 語言用在玩票性質的專案上。


 * **我們不必從頭學習所有東西就能開始撰寫程式。**

    在一個純 functional 的語言中，你必須使用 monads 才能印出變數或讀取 DOM 節點。JavaScript 則簡單多了，可以作弊走捷徑。JavaScript 也更容易入門，因為他是一門混合模式的語言，你可以隨時在感到吃力之時回去按你原有的習慣開發。


 * **這門語言完全有能力撰寫高級的 functional 程式碼。**

    只需藉助一兩個小型的 library 就能幫助你模擬 Scala 或 Haskell 這類語言的所有特性。雖然物件導向程式設計（Object-oriented programing）主導著業界，但很明顯這種模式在 JavaScript 中相當的笨重，用起來像在高速公路上露營或像穿著橡膠鞋跳著踢踏舞一般。我們不得不到處使用 `bind` 以避免 `this` 在不知不覺中改變，語言中也沒有 class 可用（目前），我們還發明了各種變通的方式來避免忘記 `new` 關鍵字的怪異行為，private 成員目前只能透過閉包（closure）實現。對大多數人來說，FP 感覺起來反而更加自然。

以上說明，強型別的 functional 語言毫無疑問將會是本書所提供程式類型的最佳實驗場所。JavaScript 會是我們學習這種模式的手段之一，將它運用在何處則完全取決於你。最後你會發現你習慣了 swiftz、scalaz、haskell、purescript 及其他數學導向的語言。


### Gitbook（較佳的閱讀體驗）

* [線上閱讀](https://drboolean.gitbooks.io/mostly-adequate-guide/content/)
* [Download EPUB](https://www.gitbook.com/download/epub/book/drboolean/mostly-adequate-guide)
* [Download Mobi (Kindle)](https://www.gitbook.com/download/mobi/book/drboolean/mostly-adequate-guide)

### 自己做

```
git clone https://github.com/jigsawye/mostly-adequate-guide.git

cd mostly-adequate-guide/
npm install gitbook-cli -g
gitbook init

brew update
brew install Caskroom/cask/calibre

gitbook mobi . ./functional.mobi
```


# 目錄

請見 [SUMMARY.md](SUMMARY.md)

### 貢獻

請見 [CONTRIBUTING.md](CONTRIBUTING.md)

### 翻譯

請見 [TRANSLATIONS.md](TRANSLATIONS.md)

### FAQ

請見 [FAQ.md](FAQ.md)



# 未來計劃

* **第 1 部分**（目前的章節 1 至 7）是指南的基礎知識。這是初版草稿，我會在找到錯誤時及時更正。歡迎提供幫助！
* **第 2 部分**（目前的章節 8+）會講述 class 型別，像是 functors 及 monads，最後會帶到 traversable。我希望能塞一些 transformers 及一個純的應用程式。
* **第 3 部分**會開始遊走於程式開發實踐與學術研究之間。我們將學習 comonad、f-algebra、free monad、yoneda 及其他類型的結構。
