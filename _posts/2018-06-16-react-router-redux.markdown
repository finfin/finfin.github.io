---
layout: post
title:  "React 下與 URL 參數讀取與變更"
date:   2018-06-16 22:51:59
tags:
  - "react-router-redux"
  - "redux-logic"
  - "react"
  - "redux"

image: /assets/article_images/redux-logic.jpg
excerpt: "React 下與 URL 參數讀取與變更"
---

# 問題

試想如果你是使用者，下列情境是否很熟悉：

在某網站點來點去，網頁顯示了一些資訊，但下次來的時候又要重新點來點去進入同樣的頁面。想弄成書籤也不行，存下來的 URL 只會讓你回到該功能的最初狀態頁面。於是每次使用的時候都要耗費許多時間重複做一樣的動作。

我們在單一頁面會有比較複雜的使用者互動時，通常不希望使用者重整網頁或是下次再來的時候還要再重新做一些複雜的操作，為了後續使用方便，一種方式是把使用者的輸入反應回 URL 上。在 URL 路徑層級的 (ex: http://finfin.github.io/urlpath/subpath) 可以透過 react-router 處理，但 react-router v4 已經[取消對 query string 的支援](https://github.com/ReactTraining/react-router/issues/4410)，所以必須用別的方式來處理。

# 範例頁面

直接用個範例來看看我們可以如何處理。


這範例中，我們有一個多功能的報表介面，提供查詢時間區間資料、過濾器、顯示時間區間內數據曲線圖、表格，如下圖。

![範例報表介面](/assets/article_images/react-router-redux/mockup.png)

## 設計

因為頁面操作很複雜，且使用者經常性的需要與他人分享特定時段特定篩選條件的報表。故希望可以把查詢時間、篩選條件都能透過 URL 帶出。


## 解法一

直接在 react component 中存取 `react-router` 的 `context` 物件。

```js
componentDidMount() {
  if (this.context.router.history.location.search !== '') {
    // 讀取 url query param
    // 將對應的 query param 參數傳遞至相關的 react component
  }
}
```

使用者有相關輸入的時候當然也要更新 URL

```js
handleUserInput(value) {
  // 看看使用者改了什麼
  this.changeURL() // 負責把頁面狀態更新到 URL 上
}
```

### 流程圖

![解法一流程圖](/assets/article_images/react-router-redux/solution-one-flow.png)


### 分析

直觀好寫的架構，但會帶來一些副作用：

- 商業邏輯跑到 react component 內
- 不支援 上一步、下一步
- react component 本身要多依賴一個外部變數 (URL)，失去 redux 的 SSOT 精神，也比較難測試。

## 解法二

透過 react-router-redux 做 URL 的 middleware 處理。以下用範例會搭配 redux-logic 使用，但基本概念可相通其他類似 redux-logic 的套件。

基本流程圖會像這樣

![解法二流程圖](/assets/article_images/react-router-redux/solution-two-flow.png)

```js
// logic.js
import {push, LOCATION_CHANGE} from 'react-router-redux';
import queryString from 'query-string';

// 針對 URL 變化攔截
const locationChangeLogic = createLogic({
  type: LOCATION_CHANGE, // react-router-redux 在 URL 改變時所會發送的事件
  latest: true,
  process({getState, action, history}, dispatch, done) {
    // TODO: history back/forward support
    const search = queryString.parse(action.payload.search); // 讀取 query param 參數
    
    // 因為是 global action 所以針對某個頁面需要去判斷 path 是否相符
    if(action.payload.pathname === '/report') { 
      // 依照 queryString 的參數決定發出的 action ，以改變頁面狀態
    }
    done();
  }
});

// 針對想改變 URL 的 action 攔截
// 此例中 改變日期、改變篩選條件 我們想要反映到 URL 上
const handleChangeURLLogic = createLogic({
  type: [CHANGE_DATE, CHANGE_FILTER, REMOVE_FILTER],
  // latest: true,
  process({getState, action, history}, dispatch, done) {
    // 從目前的 redux state / action 計算出想要的 query param
    let search = getSearchParams(); 
    dispatch(push({search: queryString.stringify(search)})); // 更新 URL
    done();
  }
});
```

透過攔截 `LOCATION\CHANGE` 來讀取 URL 參數，並且模擬發出相對應的 action，達成與使用者操作（也是會發 action）一樣的效果，進而讓 react component 變成我們想要的狀態。

### 分析

- 可以針對已有的 react component 去加強，只要在 logic.js 層動手腳
- react component 不需變動，故不影響到原有測試方式。
- URL 變化被 `react-router-redux` 抽象化成簡單的 action ，故 針對 URL 變化的測試也簡化成對 `LOCATION_CHANGE` 的影響測試
- 無痛支援上一步、下一步

要注意的是有可能 LOCATION_CHANGE 造成 logic dispatch `CHANGE_DATE`, `CHANGE_FILTER`, `REMOVE_FILTER` 等的 action, 這些 action 又會反過來造成 `LOCATION_CHANGE` 的無窮迴圈。故中間一定要有地方去對比是否與改變前狀態相同，如果相同就不做事。

# 總結

解法一跟解法二其實是一個重構的過程，在這過程中最有幫助的是把畫流程圖給畫出來。流程圖對於整個功能的資料流視覺化非常有幫助，大致步驟如下：

1. 先針對現況做
2. 有哪些主要元件
3. 有哪些主要事件
4. (來自外部)事件的引發者
5. 把 2-4 之間的關係畫出來

如此就得到現狀的流程圖，有流程圖之後就可以很容易的看出資料流雜亂或是不合理的地方。接下來就是重新整理資料流並進行重構，而重構的時候也因為已經有一張藍圖所以可以很明確的知道要做什麼。

整體大致是依照 functional programming 的概念去重構，每個部分都只負責做一件事情，目標是希望可以比較好測試，後續維護上也會比較簡單。