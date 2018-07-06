---
layout: post
title:  "Pure function 早餐店"
date:   2018-07-07 07:27:59
tags:
  - "functional programming"
  - "pure function"
image: /assets/article_images/practical-pure-function/cover.jpg
excerpt: "用一個幻想的產品線發展來看 pure function 的使用時機"
---

# 前言

此文章以案例講述 pure function，如想了解在 function programming 內最重要的概念之一 - pure function 的概念的話， 推薦閱讀： [Mostly Adequate Guide to Functional Programming 第三章（中文）](https://jigsawye.gitbooks.io/mostly-adequate-guide/content/ch3.html)。

# 為什麼要用 pure function？

其實本人大腦的 working memory 很小，所以閱讀程式碼的時候對於太多外部相依的東西會很頭痛，常發生的情境是 trace 外部相依性 trace 到變成這樣：


![trace到傻了](/assets/article_images/practical-pure-function/freeze.jpg)

Pure function 是我的救星，只有輸入 / 輸出要管，其他都在函式裡面處理，大幅減低大腦的負擔。不管是外部呼叫函式或是在函式內，我們都只需要關注輸入 / 輸出有正確對應，大腦的能量就可以留在其他地方燃燒。


# 以做早餐當例子

我們知道 pure function 好棒棒了，但要怎麼用、什麼時候用、可以解決怎樣的問題？

假設我們有一間早餐店。抽象化一點的早餐產線會是這樣：

```
輸入：點餐
輸出：完成的菜色
```

經過分析之後這樣做早餐被切分成幾個部分：取得材料、備料、烹飪。（設備啊人力什麼假設是固定的，先不管）

![來做早餐](/assets/article_images/practical-pure-function/breakfast.png)

## 產品線

如果再往下倒產線內部的一些狀態/資料，可以有這樣子的流程圖：

![早餐產品線](/assets/article_images/practical-pure-function/breakfast-flow.png)

做一個早餐會經過幾個步驟（藍色），另外會有一些屬性（橘色）用來記錄早餐的狀態變化。

## 新的生意

早餐店做得不錯，除了原有的早餐店面產品線，也開始規劃冷凍食品的產線，準備在美式賣場販賣~~順便找民視鄉土劇冠名~~。


![冷凍食品產線](/assets/article_images/practical-pure-function/frozen.png)


因為資源不夠的關係，早餐店決定還是以現有產線處理一般早餐以及冷凍包產品，冷凍包除了既有的流程之外，還需要包裝的程序。另外冷凍食品的份量為一般產線的八成。

![產線很忙：一般與冷凍一起做](/assets/article_images/practical-pure-function/breakfast-flow-frozen.png)

開始變得有點複雜了，尤其是取得材料這一步驟，就算是同樣餐點，但是依據不同的點餐內容會需要不同的產出。

## 進軍平價超市

鄉土劇冠名的效果實在太好，平價超市開始與早餐店洽談合作。經過平價超市的輔導之後，早餐店決定另闢平價產線，分量不變但添加『客製化材料』以迎合平價客層需求。也因為添加不同材料的關係備料跟烹飪步驟也會不一樣。

![一般與冷凍與平價產品線](/assets/article_images/practical-pure-function/breakfast-flow-value.png)

可以想見隨著產品線越來越複雜，狀態（橘色）也變多，每個步驟的分支會越來越多。從步驟層面來看，這些步驟越來越與產品線綁死，除非知道這些產品線的狀態，無法簡單的去預測/驗證每一個步驟應有的產出。

## Functional Programming 作法

前述的流程分成四個步驟，我們也可以把每一個步驟都當成一個 pure function ，如下圖：

![Pure Functions?](/assets/article_images/practical-pure-function/fp-breakfast.png)

每個步驟負責把輸入的項目轉成輸出的產品，由於我們標準化了每一個步驟的輸入跟輸出，各產線要做的就是設定好把輸入項目傳入步驟，並接手輸出產品，再丟給下一個步驟，如下圖：

![模組化生產線](/assets/article_images/practical-pure-function/fp-breakfast-factory.png)

想像一下，今天如果又新增另一產線叫做『宅宅自己來』，是給喜歡在家下廚又不想花時間上市場/備料的人，我們就可以很快速的把『取得材料』、『備料』這兩個步驟組合起來變成新的『宅宅自己來』產品線。好像很簡單，不是嗎？


# 結語

Pure function 看起來很好用，但並不是萬靈丹，一般情況下越下層的元件越容易做成 pure function，而越上層因為會與許多其他元件接觸，要 pure 的成本就會越高。

Pure function 的目的是產生可以預測、可以重複的結果，其實跟 SOP 是一樣的概念，不過也別忘了不是所有事情都可以 SOP，很多事情是『視情況而定』的。


# 參考資料

- [Mostly Adequate Guide to Functional Programming](https://www.gitbook.com/book/jigsawye/mostly-adequate-guide)
