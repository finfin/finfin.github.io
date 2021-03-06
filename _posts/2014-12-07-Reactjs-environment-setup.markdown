---
layout: post
title:  "React.js 開發環境設定"
date:   2014-12-07 14:34:25
tags: javascript
image: /assets/article_images/reactjs-environment.jpg
---

[React](http://facebook.github.io/react/) 可說是最近最熱門的一個 Library，用過之後都會被其簡單的程式流程所吸引。如果對 React 還不熟的可參考附錄，或直接至官網看教學。在這就分享一些環境設定，讓開發更有效率。 

#材料準備

- 安裝好 react-tools 的電腦一台

    ```bash
    $ npm install -g react-tools
    ```

- 支援 Grunt 的 Project 一個（Gulp 也 ok 設定很簡單沒有差很多自己修改一下） 

#目錄結構
一般的目錄結構大概是像這樣：

```
└── src
     └── js
         ├── app
         │   ├── collections
         │   ├── config
         │   ├── controllers
         │   ├── models
         │   └── views
         └── vendor
             ├── backbone
             |   ...
             └── requirejs
```

React 由於有自己的 jsx 檔案格式，目錄結構改成這樣： （變化不大只是把 js/app/views 拿掉，因為被 jsx 取代）

```
└── src
     ├── js
     |   ├── app
     |   │   ├── collections
     |   │   ├── config
     |   │   ├── controllers
     |   │   └── models
     |   └── vendor
     └── jsx
```

我的 jsx 目錄長得像這樣：

```
└── src
     └── jsx
         ├── atom
         ├── molecule
         ├── organism
         └── template
```

這樣子的分法是因為本身有使用 styleguide 工具——[PatternLab](http://patternlab.io/)，React 本身的元件概念跟 PatternLab 不謀而合，用一樣的目錄結構也可以方便樣式的對照。

#Grunt Task

Grunt Task 需要兩個部分： **A.把JSX 檔案編譯成 JS** , **B.JSX 檔案一旦修改就呼叫 A**

- **把 JSX 檔案編譯成 JS:** 在 Gruntfile.js 裡增加這樣一個task

```javascript
grunt.initConfig({
    shell: {
        jsx: {
            command: [
                'jsx -x jsx src/jsx/ src/js/app/views/',
                'rm -rf src/js/app/views/.module-cache/'
            ].join(' && '),
            stdout: true,
            failOnError: true
        }
    }
});
 
grunt.loadNpmTasks('grunt-shell');
```

這是一個 shell command，所以會需要 `grunt-shell`，內容就是執行 jsx 編譯並輸出至 `src/js/app/views/` 目錄下，第二個 command 僅只是把編譯過程的 cache 檔案清掉。（就是這個步驟需要 react-tools）

 - **JSX 檔案一旦修改就呼叫 A:** ——有了 JSX 轉 JS 的task ，再來就是用 watch 讓這動作更自動化 

```javascript
grunt.initConfig({
    shell: {
        jsx: {
            ...
        }
    },
    watch: {
        jsx: {
            files: 'jsx/**/*.jsx',
            tasks: ['shell:jsx']
        }
    }
});
```

有了這兩個 task 之後，只要 `grunt watch`，當你一修改 jsx 檔案系統就會自動幫你編譯成 js，之後就跟一般開發流程沒兩樣了。


#附註

###一些 React 的資源：

 - [Awesome React](https://github.com/enaqx/awesome-react): React 資源彙整
 - [Developing a React Edge: The JavaScript Library for User Interfaces](https://www.safaribooksonline.com/library/view/developing-a-react/9781939902122/): eBook
 - [JSDC 2014 #06 react/flux in Action](https://www.youtube.com/watch?v=UBWLr2i4MIg) / Jemery Lu<br>
投影片：[React/Flux in Action 實戰經驗分享](https://speakerdeck.com/coodoo/flux-in-action-shi-zhan-jing-yan-fen-xiang)