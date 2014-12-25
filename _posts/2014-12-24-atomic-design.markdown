---
layout: post
title:  "網頁設計系統：ATOMIC DESIGN 介紹"
date:   2014-12-24 18:33:00
tags: 
  - "style guide"
  - design
  - atomic design
  - methology
image: /assets/article_images/atomic-design.jpg
excerpt: "當網頁的規模逐漸變得複雜，建立一個又一個的頁面耗費越來越多時間，此時需要開始以系統化的角度來重新思考頁面設計這回事。Atomic design 是個設計方法，目的就是用來解決這樣子的問題。"
---

> We’re not designing pages, we’re designing systems of components.

> [***——Stephen Hay***](http://bradfrost.com/blog/mobile/bdconf-stephen-hay-presents-responsive-design-workflow/)

當網頁的規模逐漸變得複雜，建立一個又一個的頁面會耗費更多時間，此時需要開始以系統化的角度來重新思考頁面設計這回事。換句話說，需要建立一個設計系統。

現階段關於[設計系統](http://24ways.org/2012/design-systems/)的討論大多關注在顏色、字型、grid、素材之上。這些是最基本也最重要的元素，但忽略了介面是如何組成、由什麼組成這些問題，而這些問題應該用一種更系統化的方式去建立、討論並解決。

如果將化學的概念應用到介面設計上會是如何呢？在化學的世界所有物質都是由原子組成，原子組成分子、分子組成物體。相似的，網頁的介面是由基本的 `html` 元素所組成，也就是說所有的網頁設計都可以被分解到以 html 元素為單位並開始組合，這就是本文所要探討的 `ATOMIC DESIGN`。

[![HTML元素週期表](http://bradfrost.com/wp-content/uploads/2012/11/Screen-Shot-2012-11-13-at-5.15.05-PM.png)](http://madebymike.com.au/html5-periodic-table/)

# 什麼是 Atomic Design

Atomic design 是個設計方法，目的是用來建立整個網頁的架構與風格。分成五個階層：

1. Atoms / 原子
2. Molecules / 分子
3. Organisms / 機構
4. Templates / 範本
5. Pages / 頁面

![Atomic Design 階層](/assets/article_images/atomic-design/atomic-design.png)


# ATOMS / 原子

原子是最基本的元素，對應到網頁介面的話就是 HTML 元素如 `label`, `input`, `button`。另外原子還包括了字型、配色或動畫等 HTML tag 沒有包括的部分。

![三種不同“原子”： label, input, button](/assets/article_images/atomic-design/atoms.jpg)

這些原子本身的角色是一個基本的組成元件，就像一個樂高積木一樣，本身沒有太大的功用。但把所擁有的積木一次性地列出可以提供一個風格上的概觀。


# MOLECULES / 分子

當我們把原子組合在一起（尤其是不同的原子），就慢慢形成一個一個的網頁的部件，我們稱作分子。這樣的一個部件是以原子所組成的最小的組合單位，在這個設計系統裡分子的角色就如同樑柱對於建築物一樣。

舉例來說，前述的 `label`, `input`, `button` 本身來說並不具有實質的意義，而如果我們把這三個組合在一起就可以變成如下圖這樣一個分子：

![分子：搜尋列](/assets/article_images/atomic-design/molecule.jpg)

從原子到分子的建構過程中有一個重要的觀念：*一個元件做一件事情，並把事情做好*。儘管分子有可能很複雜，但在這樣的建構概念下分子是為了一個單純功能而建構並可以重複使用的。

附註：可以試著對應到物件導向設計中的物件


# ORGANISMS / 機構

有了分子這樣子的功能性部件，我們就可以開始建構更複雜的介面。簡單來說機構是由分子組成的一個複雜區塊，成為一個功能明確的部分比如功能列、側欄等。

![機構：頁首](/assets/article_images/atomic-design/organism2.jpg)

到了這個階層已經可以很明確地建構一個使用者／客戶可以理解的功能性區塊。機構（理所當然的）是由分子組成，可以是相似的分子也可能是各種相異的成員，端看機構的功能需求。一般網頁的頁首，以機構的觀點來看，會有 logo、功能列、搜尋列、使用者資訊等；另一種機構 — 產品列表，就僅僅是由重複的*產品分子*所組成。

分子建構機構的概念與原子建構分子的概念相去不遠，我們的目標是建構一個獨立的、能夠重複使用的區塊。


# TEMPLATES / 範本

這個階層就跳出化學系統的類比，用一個大家都聽得懂的名詞：範本。範本是由各機構拼湊而成的一個基本頁面架構。也因此範本非常的具體，幾乎就是頁面的呈現。到了這個階段，範本可以拿來做 HTML wireframe，再慢慢的把細節加入，最後變成實實在在的產品。

![範本](/assets/article_images/atomic-design/template1.jpg)


# PAGES / 頁面

把範本填入真實的資料，就會變成一個頁面，也就是最終使用者會看到的成品。也因為如此，這會是最常被檢討、修改的部分。藉由不斷的測試整個頁面來修正其下的範本、機構、分子甚至原子等，朝著呈現正確資訊的設計方向來進步。

![頁面](/assets/article_images/atomic-design/page1.jpg)

頁面也用做測試範本的工具，舉例來說，一個範本的標題如果40個字長得如何、如果400個字又長得如何？購物車沒有東西的時候是什麼樣子？10樣商品時又是什麼樣子？透過不同的情境可以讓我們知道範本在各狀況下的呈現，用以進一步的調整。


# 為什麼要使用 ATOMIC DESIGN?

其實仔細想想，在物件導向設計時我們也是這麼做的，一個一個元件建構成一個大系統，只是今天對象不是程式語言而已。

Atomic design 把這樣子的方法做了一個清楚的規範，讓我們在建立網頁時能有一個基準可以遵從，對於小組成員或客戶來說，一套標準的步驟放在面前會讓大家知道該做什麼，同時也顯得我們很專業（？）

利用這系統裡面從較為抽象的小元素直至確切的頁面的步驟，我們可以建立一個具有一致性、易擴充的成品，同時在修改時也可以快速地產出修改後的結果。


# PATTERN LAB

如果想加入這套設計系統，*Brad Frost* 建立了一套工具—— [Pattern Lab](http://demo.patternlab.io/)，也可以 [前往 GitHub](https://github.com/bradfrost/patternlab) 抓取 Pattern Lab 來試試 Atomic design。



# 參考資料

本文轉譯自 [Atomic Design](http://bradfrost.com/blog/post/atomic-web-design/)

文中提到的其他參考連結如下：

 - [Element Collage](http://danielmall.com/articles/rif-element-collages/) 
 - [Obtaining Signoff without Mockups](http://alistapart.com/article/responsive-comping-obtaining-signoff-with-mockups) 
 - [Rock Hammer](http://stuffandnonsense.co.uk/blog/about/rock-hammer-a-curated-responsive-project-library) 
 - [Web Components](https://www.youtube.com/watch?v=fqULJBBEVQE) 
 - [Modularity](http://www.w3.org/DesignIssuesPrinciples.html#Modular) 
 - [Responsive Deliverables](http://daverupert.com/2013/04/responsive-deliverables/) 

此文作者在 [2013 Beyond Tellerrand](http://2013.beyondtellerrand.com/) 的演講

<iframe src="//player.vimeo.com/video/67476280?title=0&amp;byline=0&amp;portrait=0&amp;color=9a151b" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> <p><a href="http://vimeo.com/67476280">Brad Frost – Atomic Design – beyond tellerrand 2013</a> from <a href="http://vimeo.com/beyondtellerrand">beyond tellerrand</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

<iframe src="//www.slideshare.net/slideshow/embed_code/22077743" width="700" height="400" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/bradfrostweb/atomic-design" title="Atomic design" target="_blank">Atomic design</a> </strong> from <strong><a href="//www.slideshare.net/bradfrostweb" target="_blank">Brad Frost</a></strong> </div>














