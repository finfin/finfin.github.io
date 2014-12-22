---
layout: post
title:  "Maintainable Javascript：作者Nicholas Zakas演說重點節錄"
date:   2013-09-04 14:34:25
tags: 
  - javascript
  - "code style"
  - 
image: /assets/article_images/maintainable-javascript.jpg
---

以下是Maintainable Javascript 作者 Nicholas Zakas 在 Fluent 2012 演說

<iframe src="//www.youtube.com/embed/c-kav7Tf834" frameborder="0" allowfullscreen></iframe>

<iframe src="//www.slideshare.net/slideshow/embed_code/13122173" width="100%" height="450px" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="margin-top: 1em; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>

這裏節錄一些重點，為什麼要讓程式易維護？ 因為大部份時間都是在針對既有的程式作修改加強，而非創造新的。而所謂的修改加強，就是維護的意思。由於90%的時間都是在做維護的動作，程式的可維護性自然是很重要的議題。好維護的程式一開始可能需要很多前置作業，但長期下來會減少許多不必要的時間浪費，進而加速開發的速度。

#誰在意維護性？

- **公司**：公司僱用員工是請員工來創造有價值的產物，不可維護的程式碼在員工離開之後就會失去其價值。
- **同事**：如果程式超過一個人碰觸，那麼可維護性就很重要，要保證所有接觸到的人都能有一個共通的語言。
- **你自己**：自己也可能維護自己曾寫過的程式，不好維護的程式過了一陣子之後有很大可能連作者自己都忘了為什麼要這麼寫。

由於前端工程（or 網頁工程)是新興產業，所以規則還在慢慢的成行中，但在可維護性的議題之下，統一規則是最重要的，確保了所有人都能看懂同一個程式碼。

#維護性

> *可維護的程式碼應該要能持續的運作五年而不需做任何重大的改變。(aka. 砍掉重練)*

- **直觀**： 看了就能大致瞭解在幹嘛
- **可理解**： 無法一眼就知道在幹嘛的，提供方法來解釋(注解或文件)
- **易修改**： 修改某段程式碼不容易導致其他地方出錯
- **可擴充**： 能一直增加功能而不需修改架構
- **易除錯**： 能輕易找出問題點
- **測試性**： Unit Test, Functional Test 能提升維護性

#如何提升維護性：Code Conventions

Code Conventions 可以說是程式的基本準則，一個專案或組織裡面只能有一種 code conventions，否則會引起混亂，也就降低了維護性。而 code conventions 分成兩種:

- **Code Style**：程式碼的格式，該長得什麼樣子
- **Programming Practice**：所謂的 best practices，寫程式時的準則或基準模式，能提升維護性、減少 bug 的產生

###Code Style

統一的 code style 讓你能用同一套方式去看所有的程式碼，不需額外花費不必要的力氣去理解。

一些比較常見的 style guide：

- [http://javascript.crockford.com/code.html](http://javascript.crockford.com/code.html)
- [http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml)
- [https://github.com/rwldrn/idiomatic.js/](https://github.com/rwldrn/idiomatic.js/)
-（其他請見投影片）

**重點**：style 只能有一種，否則就沒有意義。

####縮排
tabs 或是空白都可以，但要統一，不要混用。使用空白要確定每個縮排都是固定寬度（ex: 2 or 4)縮排的用法參考 code style，目的是讓檔案好讀，同樣階層的程式碼擁有同樣的縮排。

####每行長度
每行長度不超過80字元。過長的程式碼造成閱讀上的困難之外（勢必要左右捲動），使用版本空管系統（git, svn）時也更容易發生merge conflict。

####注解
注解就是要讓人瞭解這段程式碼在幹嘛。不要解釋很明顯的事項。每個函式的前面最好有注解解釋這個函式的用途，複雜的程式邏輯也需要注解來幫助人們理解。或是那些特異的程式寫法，看起來像是錯的寫法（但你知道是對的）也需要注解，避免後續修改時反而改"錯"了。

####命名
- 取名要貼切，能貼切的顯示變數/函數做什麼事情
- 變數通常為一個物品，名詞
- 函數通常是一個動作，故以動詞起始。布林值函數以 is/has 開頭 ex: isValid(), hasItem()
- 避免無意義的名稱：temp, foo（迴圈裡的變數例外）
- **CamelCase:** Javascript API 本身就使用 CamelCase，最好是沿用。縮寫可以全部都大寫或是首字大寫，重點在於*統一規範*！錯誤示範: XMLHttpRequest。

例外：常數全大寫，字與字間底線分隔 ex: MAX_LENGTH；Constructor 首字大寫。

###Programming Practices
>*There are two ways of constructing a software design: One way is to make it so simple that there are obviously no deficiencies and the other way is to make it so complicated that there are no obvious deficiencies.*

> ***--- C.A.R. Hoare, Quicksort 發明者***

####保持 JS/HTML/CSS 分離
js 檔就該只有 javascript 程式，html 裡面不該有 css 或 js，css 裡面不該有 js 的函式。

原因？簡化除錯流程。

提示: 很多 js 會需要 render DOM，因此會使用到 html 字串，但我們可以利用 [handlebar](http://handlebarsjs.com/), [mustache](https://github.com/janl/mustache.js/) 等工具把 html template 分離出來。重點就是屬於哪邊的東西就清楚的放在哪裡，混在一起就會增加複雜性，減低維護性。

####事件處理 Event Handler
每個函數都應該有自己明確的功能，而 event handler 能做的事情應該只有一件：處理使用者事件。

好處： 商業邏輯有可能不一定要被事件來觸發，分割邏輯之後能夠更有彈性的使用。（詳見投影片）

####別去修改不屬於你的物件
Javascript 很自由，你可以隨意修改任何物件，即使那物件不是你寫的。但隨意修改他人物件所產生的問題跟改變統一規範一樣，降低系統維護性。當大家預期 A 物件的功能是 B 時，隨意修改會造成很大的問題。也別在不屬於你的物件新增功能，你不會知道這個物件未來會不會新增一個與擬新增的功能完全同名且用法不同的功能。（[參考文章](http://www.nczonline.net/blog/2010/03/02/maintainable-javascript-dont-modify-objects-you-down-own/))

####避免全域變數/函數
如果大家都使用全域變數，將會很容易互相蓋台，也會很混亂，就像把家裡所有的物品都放在客廳桌上一樣。

####錯誤處理
對於預期可能會產生的失敗狀況，throw error。就可以簡單地知道錯誤出在哪裡。

####避免 x != null
`x != null`僅只判別 x 不是 null，但通常我們會希望變數有個形態，因此應該用 instanceof/typeof。

instanceof 是用來檢測物件是否為某種 object

typeof 用來檢測是否為某種 primitive type ( boolean/number/string/function/object )

**注意：** typeof null == "object"

####分離程式與設定
所有未來可能會被修改的數值，都最好與程式分離，才方便修改。以下是幾個應該分離出來的：

- URLs
- 顯示給使用者的字串
- HTML (前面有說過可以用 html templateing lib)
- 設定數值：如一頁顯示幾筆資料等
- 重複使用的特殊數值
- 任何未來可能需要修改的數值

#自動化

###自動化工具
- [ANT](https://ant.apache.org/)： [介紹文章](http://www.julienlecomte.net/blog/2007/09/16/)
- [GRUNT](https://github.com/cowboy/grunt)： [介紹文章](http://weblog.bocoup.com/introducing-grunt/)
- 更新： 這是2012年的文章，2014年的現在還有 Gulp, 甚至 [npm 也可以當成build tool](http://substack.net/task_automation_with_npm_run)

利用自動化工具達成自動化。這類工具可以完成的功能主要有下列七大類：

1. 加入/移除 debugging code
  - js-build-tools
2. 產生說明文件
  - jsdoc
  - yuidoc
  - docco
3. 檢測程式是否有問題
  - jslint
  - jshint
4. 縮小檔案大小
  - yuicompressor
  - uglifyjs
  - closure
5. 合併檔案
6. 測試
  - phantomJS
  - mocha
  - jasmine
7. 部署

#總結

> Always code as if the guy who ends up maintaining your code to be a violent psycho who knows where you live.

- Code Style 讓大家都說同一種語言
- Loose coupling 讓你修改＆除錯更輕鬆
- Programming practices 更容易除錯
- 自動化工具加速開發
