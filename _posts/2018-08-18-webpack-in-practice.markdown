---
layout: post
title:  "實戰中的 WEBPACK"
date:   2018-08-18 08:15:22
tags:
  - "webpack"
  - "frontend"
image: /assets/article_images/webpack-in-practice.jpg
excerpt: "Webpack 基礎以及實際應用於前端專案時的大小事"
---

今天來介紹一下 webpack。說到 webpack 最常一起提的應該是 react 了，因 react 專案通常會透過 webpack 來建置，但 webpack 本身就是個單純的 js 建置工具，因此實際上只要是 js 專案都可以使用。

既然標題有兩個字『實戰』，我們就從實際前端專案開發上的一些考量來開始吧。一般的開發流程大概是這樣子：


```javascript
while(haveBugs() || haveNewFeature()) {
  testAndDev('localhost');
  buildAndDeploy('stage');
  buildAndDeploy('production');
}

```


不管是開發新功能或者維護既有功能，在正式上版前通常會經過三個階段：開發、STAGE以及正式環境。逐一來看每個環境的議題：

# 開發環境

理想的開發環境會分成開發與測試兩個不同的配置，這兩個在本機端開發時可以同時執行。此環境下的幾個議題：

## 程式碼品質 / 拼字檢查
確保程式碼品質的第一道防線： coding style + 拼字檢查，維持最基本的可維護性。

## 引入各種不同檔案類型
前端專案需要跟各種不同檔案類型為伍，引入各種檔案類型的時候會需要：複製檔案到輸出資料夾、使用正確的路徑引用、幫檔案做一些後處理之類的重複性的工作。舉例來說：react 中引用 sass 檔案的某個 class。如果能有機制把這些相依的檔案拉進此專案可以節省很多管理上的麻煩。

## 開發用網頁伺服器
開發時很重要的一點就是確認程式碼的執行是否如預期，因此會需要一個網頁伺服器來讓開發者看到現有的程式碼實際執行的狀態。

## 跨瀏覽器相容性
處理跨瀏覽器相容性是前端很重要的一部份，通常我們可以透過 shim / polyfill 等手段把各瀏覽器的差異彌平，這樣後續的開發上就不用煩惱有沒有這些功能可以使用。


## 用來執行的程式碼要能夠對應到原始碼
一般正式上線的網頁，其執行檔是經過編譯與壓縮過的，因此執行時無法直接知道相對應於原始檔是哪行程式碼，如果能對應到原始檔就可以更清楚地瞭解目前執行狀態或是除錯。

## 即時執行單元測試
修改功能時會想要知道相對應的單元測試狀態，以確保模組基本行為是我們所預期的。又如果是使用 TDD 的話，就可以先把 test 定義好，開發的目標就是把這些測試都變成綠燈，這情境下即時反饋測試的狀態可以大大提升開發的效率。

# STAGE 環境

STAGE 很重要的一點是盡量模擬正式環境，同時提供除錯工具以期能夠早期發現早期治療。

## 盡可能減少建置完的檔案大小，提升網頁載入速度
其實是正式環境的考量，我們都知道[網頁載入越快越好](https://developers.google.com/web/fundamentals/performance/user-centric-performance-metrics)，而減少檔案大小就是一種優化的方式。當然還有一些其他好處比如網路花費減少、伺服器負擔相對比較輕。

## 減少網路請求數量
同樣也可以提昇網頁載入速度以及減輕伺服器負擔。

## 用來執行的程式碼要能夠對應到原始碼
既然想要早期發現早期治療，這點是不可少的。基本行為跟開發環境沒什麼不同。要注意的是 STAGE 雖然希望模擬整是環境，但這議題是開發需求因此不在正式環境出現。

# 正式環境

正式環境該考量的基本上應該都在 STAGE 測過，除了『用來執行的程式碼要能夠對應到原始碼』這點之外，正式環境另外還有商業安全性上的考量所以這點還是需要做得比較隱密一點。

---

上面各環境的議題可以看到每個開發階段的考量點都不太一樣，也因此通常需要依照不同環境有不同的建置配置。以前面提到的範疇來說就會有四種建置配置：開發、測試、STAGE、正式。在深入瞭解如何設定 webpack 針對不同環境建置之前，我們先來瞭解一些基礎。

# WEBPACK 簡介

嗯就是一個 JS 的 compiler。一個很強大且彈性的 compiler。

Compiler 擅長的事情是把原始碼打包成執行檔，其他工作例如把建置好的檔案放到伺服器、建置後重啟伺服器之類的工作，就不是 webpack 擅長的地方。通常這些工作還是透過 npm / shell script 之類的來完成。

![webpack.config.js](/assets/article_images/webpack-in-practice/config.png)
## 使用方式

簡單 Webpack 設定流程

1. 安裝 Webpack 
2. 專案中新增一個用來告訴 Webpack 要如何建置的 `webpack.config.js`
3. 執行建置

詳細可以看這邊：
https://webpack.js.org/guides/getting-started/

所有的建置設定都可以透過 `webpack.config.js` 去調整。另外有一條路是直接用命令列去操作 Webpack 但這在設定複雜的情況之下基本上不可行，大多數都還是使用設定檔來執行建置。

## 核心

![The core four](/assets/article_images/webpack-in-practice/core.png)

上圖標出了設定檔中最重要的四個部分：

### Entry

建置的進入點，也可以說是應用程式的進入點，建置時會從進入點開始把所有相關的 dependency 全部打包進來。除了較常用的單一進入點之外，其實[也可以設定多個建置進入點](https://webpack.js.org/concepts/entry-points/#multi-page-application)，適用於一次要打包多種版本或是單元測試的狀況。

### Output

建置完的產出設定，包含檔案名稱、輸出資料夾等。

### Loader

應用程式中有可能會需要引入各種不同種類的檔案如：json, typescript, jpg, sass/css 等，而 loader 的工作就是把引入的檔案變成可以在 js 檔案中存取的轉接頭。

### Plugin

在建置的過程中如果需要做一些客製化，就透過 plugin 來完成。例如透過 DefinePlugin 來設定全域變數。

### 其他重要設定

devtool: sourcemap 支援，以方便未來除錯。
watch: true 時只要檔案更改就會及時建置。
mode: webpack 提供 production / development / none 三種模式，分別針對不同情境提供一些預設的最佳化調整。[詳情](https://webpack.js.org/concepts/mode/)

## Webpack Dev Server

基於 webpack 上的一個工具，負責在本機端起一台 server 來讓開發者及時檢視 webpack 編譯好的檔案。也因為編譯好的檔案是直接放在記憶體中存取，幾乎是修改原始碼之後就可以馬上看到變更後的效果。而個人最喜愛的就是 hot reload 的功能了，可以在頁面不用重新整理的情況底下直接更新修改過的模組，在需要複雜操作/狀態的介面修改上非常有幫助。

---

簡略的介紹了 webpack 的功能，我們再回來看看開發時的議題以及可能的解決方式：

# 開發環境

## 程式碼品質 / 拼字檢查

webpack 提供 [eslint-loader](https://github.com/webpack-contrib/eslint-loader) 可以做這樣的檢查，然而 lint/spelling 畢竟是原始碼本身的品質問題，個人習慣用 IDE 來做 lint。目前 eslint 已支援各大 IDE，只要加上一個 auto fix 的選項，就可以讓 IDE 幫你處理一些基本的語法錯誤。

至於 spell check 也是原始碼層級的問題，所以也是在 IDE 裝個 [Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker) 來解決。

簡言之，lint 跟拼字檢查不需要看實際程式執行的結果，因此雖然 webpack 可以做個處理，但在編輯原始碼的當下就把這些問題先處理掉是更有效率的處理方式。

## 引入各種不同檔案類型

這裡是 webpack loader 功能發揮專長的地方。[看看有哪些 loader](https://webpack.js.org/loaders/)，較常用的有 

- json-loader (這個已經內建)：直接把 json 檔變成 js object 可以存取
- file-loader/url-loader： 載入檔案並提供 local URL，比如在專案資料夾下的系統圖片/圖示就可以透過 url loader 引入並設定給 <img />。
- babel-loader： 開發時可以用 es6+ 而不用管相容性問題，由 babel 在載入時幫我們解決。
- css-loader / style-loader: react 專案可以用它們來引入 css 並指定其中的 class 給特定元件。
- postcss-loader: CSS 後處理，可以用來處理一些 css 上的相容性議題，當作 css 界的 babel 使用。

## 本機端開發用網頁伺服器

既然 webpack 都提供了 [webpack dev server](https://webpack.js.org/configuration/dev-server/#devserver) 那就好好利用嘍。webpack dev server 跟 expressjs 一樣有非常彈性的客製化空間，可以透過 middleware 來達成，比如說 hot reload 等。

## 跨瀏覽器相容性

用 babel-loader 解決 js 相容性問題，post-css 可（部分）解決 css 相容性問題。剩下的要靠測試去挖掘。


## 用來執行的程式碼要能夠對應到原始碼

devtool 這個屬性可以告訴 webpack 我們要產生什麼樣的 sourcemap，在目前瀏覽器都有支援 sourcemap 的情況下，就可以很簡單的追溯回原始碼。

## 即時執行單元測試

跑單元測試時可以設定 watch 屬性，這樣只要檔案修改就會重新建置，以即時看到測試結果。watch 時可以只針對修改過的檔案會重新建置，這樣可以更快速的看到結果。


[開發環境設定範例](https://gist.github.com/finfin/25702983244e7311e0c38067b0865948#file-webpack-dev-config-js)

# STAGE 環境

## 模擬正式環境

建置時會盡量用跟正式環境一樣的參數以達到幾乎相同的效果。

## 用來執行的程式碼要能夠對應到原始碼

承上，雖然會盡量用正式環境的參數，但 stage 為測試用因此需要把 sourcemap 的功能開啟。

## 盡可能減少建置完的檔案大小，提升網頁載入速度
## 減少網路請求數量

模擬正式環境的需求。

透過 uglify / minify 把檔案壓縮，甚或比較進階的 [Tree Shaking](https://webpack.js.org/guides/tree-shaking/)，都可以幫我們減少檔案大小。而 webpack 會把所有資源整併建置，單一進入點情況下一般最終只會產生一個 js 檔。而如果檔案太大也可以透過 [CommonChunksPlugin](https://webpack.js.org/plugins/commons-chunk-plugin/) 來做分段載入。


[STAGE環境設定範例](https://gist.github.com/finfin/25702983244e7311e0c38067b0865948#file-webpack-dev-config-js)

# 正式環境

## 盡可能減少建置完的檔案大小，提升網頁載入速度
## 減少網路請求數量

與 STAGE 相同。

## 安全性考量

開發 / 測試 / STAGE 環境下都會開啟的 devtool 參數，通常不會（公開的）用在正式環境，以避免商業邏輯外洩。



# 總結

最後想跟大家分享的是，工具本身並不是重點，重點是我們想要解決的問題，也因此從一開始我選擇從實際的環境上的需求開始介紹。

總結一下使用了 webpack 我們想要解決什麼樣的問題，達成什麼樣的目標：

## 快速反饋

比如透過 webpack dev server 達到 hot reload 或是即時執行 unit test。我們可以馬上看到變更後的結果，加快迭代/改進的速度。

## 自動化重複性的工作

工程師的基本美德就是可以給電腦做我就不想做，很多雜事但有明確規則的就都交給電腦吧，比如 copy webpack plugin 可以代替需要手動 copy 檔案到某個資料夾的動作

當想解決的問題跟想達成的目標清楚之後，工具對我們來說就是一個達成目標的手段，有效評估是否有辦法達成目標的第一步是了解這個工具，也才能讓工具能真正地發揮他們的功用。

如果想開始了解 webpack 的話網路上有滿多教學資源的[例如這個](https://neighborhood999.github.io/webpack-tutorial-gitbook/Part1/)。