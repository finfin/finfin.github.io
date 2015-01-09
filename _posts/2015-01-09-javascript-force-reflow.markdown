---
layout: post
title:  "用 Javascript 操作 DOM：批次修改屬性須知"
date:   2015-01-09 23:00:00
tags: 
  - javascript
  - DOM
  - reflow
image: /assets/article_images/javascript-force-reflow.jpg
---

有時候我們在程式碼中修改 DOM 的樣式，尤其是動畫，例如從哪邊開始往哪裡去等等的設定。在修改 DOM 樣式的時候有個地方需要注意：瀏覽器對於 DOM 操作，並不是一行一行的執行，而是會『儘量』一次性的執行，以維持瀏覽器的高效能。在大部份情況下這當然是對開發者最有利的。

```javascript
var el = document.getElementById('my-element-id'); 

el.style.transition = 'margin 1s';
el.style.margin = '50px';
```

上面的程式碼很簡單，以一秒漸變的方式把 margin 變成 50px。但是，讓我們看另一種比較複雜的狀況：假設我們要把一個元素 B 加至元素 A 裡面，並且以漸變方式增加 margin，程式碼會是這樣子：

<a class="jsbin-embed" href="http://jsbin.com/bodide/2/embed?js,output">JS Bin</a><script src="http://static.jsbin.com/js/embed.js"></script>

很簡單的幾行程式，然而我們會發現無論你怎麼把程式碼調換 transition 的效果都出不來。不過既然文章看到這，相信可以輕易地猜到就是跟開頭所說的『一次性執行』有關。是的，聰明的瀏覽器繪把所有的 style 設定好，等到 js 執行完之後一次性的重繪至頁面。所以這邊看到的就是一個 `style="transition: all 1s;margin:40px"` 的元素直接被放至頁面上，當然看不到任何的動態效果。

解決的方式其實只要一行程式碼，看看下面的修正版本：

<a class="jsbin-embed" href="http://jsbin.com/bodide/4/embed?js,output">JS Bin</a><script src="http://static.jsbin.com/js/embed.js"></script>

跟之前的版本比起來，只多了一行看起來無意義的程式碼： `child.offsetHeight`，看起來什麼也沒做卻能夠造成如此不同的效果，為什麼？

看看[這篇文章](http://blog.letitialew.com/post/30425074101/repaints-and-reflows-manipulating-the-dom)，就可以大概知道發生了什麼事情。如果對於 repaint 跟 reflow 不是太了解的，推薦看看 [How Browsers Work (簡體翻譯)](http://blog.csdn.net/zzzaquarius/article/details/6532299) ，英文能力可以的推薦看原文 [How Browsers Work (原文)](http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)。

這裡節錄一個重點，reflow 會發生在以下情況：

1. 插入、刪除、變更 DOM 元素
2. 變更頁面內容，比如文字匡輸入文字
3. 移動 DOM 元素
4. 利用 animation/transition 操控 DOM 元素
5. **取得元素的數值比如 offsetHeight 或是 getComputedStyle**
6. 捲動頁面、視窗放大縮小、新增/刪除 stylesheet

雖然我們的範例中一直在變更元素的樣式（符合第1項），但這種變更瀏覽器會累積起來一次更新，所以只能利用第5項，我們在 `child` 加至 `parent` 之後就呼叫 `child.offsetHeight`，強迫瀏覽器執行 reflow，此時 child 已經在頁面上，這時候再修改 `margin`就會看到 transition 的效果。而為什麼這樣就會造成 reflow 的原因也不難理解，因為如果沒有把元素繪至頁面上（或至少知道元素要畫在哪裡），是沒有辦法得知offsetHeight的數值的。

最後再次推薦 [How Browsers Work (原文)](http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/) 這篇文章，對於想要了解
 DOM, 減低不必要的 reflow 以增加效能等等等的，必讀。


