---
layout: post
title:  "Javascript 中的 this "
date:   2015-01-11 21:59:00
tags: 
  - javascript
  - this
image: /assets/article_images/this-in-javascript.jpg
excerpt: "為什麼 Javascript 裡的 this 常常不是（一般認為的） this？說穿了只是因為 javascript 裡的 this 的行為跟一般物件導向語言的行為不一致所造成。this = 呼叫 function 的物件，就這麼簡單。"
---

先學習 Java（或任何其他物件導向語言）再開始學 JavaScript 的人，一定會遇到這個問題：

>為什麼 Javascript 裡的 this 常常不是（一般認為的） this？

要了解這問題需要先理解一個基本觀念：

>Java 是 class-based 語言，this 指向的是該段程式碼所屬的 class。
>Javascript 是 prototype-based 語言，class 實際上不存在，this 指向的是呼叫該 function 的主人 

進一步解釋，Java 中所有東西都被定義在一個 class 內，this 代表的就是所在的 class，就算這 method 是繼承而來的，只要你是在這個 class 內呼叫，那麼 this 就會指到這個 class 而不是被繼承的 class，也因此 this 是（相較於 Javascript）在物件初始化之後就不會變動。Javascript 就比較動態，一個 function 可以跟著定義好的物件，也可以跟著物件的 prototype，甚至你可以（在執行期間）拿別人的 prototype 來用、修改，這樣的彈性間接的導致了 this 會隨著呼叫的方式而變動。

儘管 this 不像 Java 是固定的，但其實規則也不複雜，簡單一句話

>this = 呼叫 function 的物件

# 規則

基於這樣的規則，以下是可能出現的狀況：


|  執行方式  |  範例語法 |  this等於 |
|:----------|:---------|:---------|
| Global | `this;` | Global object(eg. window) |
| Global Function | `foo();` | Global object |
| Object Function | `myObject.foo();` | myObject |
| Function using call | `foo.call(myCall, myArg);` | myCall |
| Function using apply | `foo.apply(myApply, [myArgs]);` | myApply |
| Constructor Function | `var newObj = new Foo();` | newObj |
| Evaluation | `eval(thing_to_eval);` | 等同eval層級 |


# 範例一

現在我們知道 this 在不同情況下會是什麼，但實際運用上更多時候遇到的狀況是 this 不是我們想要的：

```javascript
var myObj = {
  id: "rettamkrad",
  printId: function() {
    console.log('The id is '+ this.id + ' '+ this.toString());
  }
};
setTimeout(myObj.printId, 100);
```

`myObj.printId` 的功能很簡單，利用 this.id 取得物件id並顯示。而這麼簡單的功能，我們透過 setTimeout 延遲個0.1秒執行，結果卻是........找不到id？？（或是變成global 的 id）

原因是，setTimeout 的 callback function 裡面的 this 都會變成 global object，對照一開始說的，可以知道呼叫這個 callback 的應該就是這個 global object。不過知道呼叫的是誰其實沒太大的幫助，我們想要的是 this 維持指向 myObj，這裏有兩種解法：

### closure

```javascript
setTimeout(function() { myObj.printId() }, 100);
```

由於此匿名函式會保存myObj，printId也就會正常執行，因此在setTimeout執行時就不會有先前的問題了。

### bind

```javascript
setTimeout(myObj.printId.bind(myObj), 100);
```

ES5 的函式都有一個叫做 bind 的函式（[詳細用法點這裡](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)），bind 後的函式不管是誰來呼叫， this 都指向 bind 的參數，這裏就是指向 myObj。需要注意的是 bind 是 ES5 才加入的功能，所以必須注意開發的環境下有沒有這個功能。那麼針對不支援 ES5 的環境要怎麼做呢？連結裡面有 polyfill 的方法，或是如果剛好有用 underscore/lodash 之類的函式庫，他們也有提供 bind 函式。


# 範例二

另一種常見的錯誤情形：

```javascript
myObj = {
    name: 'siri',
    callme: function() {
        console.log('hello this is ' + this.name + ' speaking');
    },
    init: function() {
        var t = document.getElementById('t');
        t.addEventListener('click', this.callme);
    }
}
```


這個範例中我們希望可以在 #t 這個元素被點擊的時候，呼叫 myObj 中的 callme。在程式碼中可以看到 callback 函式為 `this.callme`，但因為是由 DOM 元素執行 callback，在該函數裡的 this 也就跑掉變成該元素，所以這樣的寫法因而會出錯。

那如果把 this 包在匿名函式裡呢？

```javascript
myObj = {
    name: 'siri',
    callme: function() {
        console.log('hello this is ' + this.name + ' speaking');
    },
    init: function() {
        var t = document.getElementById('t');
        t.addEventListener('click', function() {
            this.callme();
        });
    }
}
```

this 會指向呼叫該 function 的物件，而在這裡呼叫匿名函式的人是 click 事件發生的 DOM 元素，也就是this跟原來版本一樣，還是會錯誤。

### **解法 1：** 用 closure 把 this 存下來就好啦！

```javascript
init: function() {
    var self = this, t = document.getElementById('t');
    t.addEventListener('click', function() {
        self.callme();
    });
}
```

其它不變，只要我們另外用個變數 self 把 this 存下來，再用 self 呼叫 callme，此時 callme 裡的 this 就會指向 self，也就是我們的`myObj`。

### **解法 2：** 又或者也可以使用 bind 來解

```javascript
init: function() {
    t.addEventListener('click', this.callme.bind(this));
}
```

前面已經提到 bind 的注意事項了，可以看到這裏的解法也是 closure vs bind，那麼哪一種比較好呢？根據 [JSPerf](http://jsperf.com/bind-vs-closure-23)，closure 是比較快的但差異非常的小，而 bind 的易讀性比較優，哪個比較好就看個人喜好了。

# 結語

一般遇到的 this 問題大多跟這裡的範例一樣起因於函式呼叫者的改變，處理方式自然也是一樣。由此可知 Javascript 的 this 看起來不太容易掌握，但還是有一套固定的規則在，只要能理解基本原理就能快速地找出其因應方式。