---
layout: post
title:  "利用 Git Hook 執行程式碼品質檢查"
date:   2014-03-14 14:34:25
tags: 
  - git hook
  - git
  - javascript
  - jshint
image: /assets/article_images/githook-quality-control.jpg
---

使用 Git 的時候常常會遇到一個狀況：commit 之後才發現 jshint 檢查不通過，像是少了個分號之類。當這是一個多人合作專案時，merge 之後一整排的 jshint 錯誤那更是歡樂無比。

#解決方式

###懶人包 || TL;DR
Git Pre-commit Hook + JSHint (or JSLint 或任何你在用的檢查工具)

###系統需求
須先安裝 `JSHint`

```bash
$ npm install -g jshint
```

###詳細說明及設定步驟
既然我們需要 commit 時後都能通過 jshint 檢查，那麼就可以使用 Git Hook 來完成這件任務。Git Hook 簡單的說就是一個 trigger：當某 git 動作發生的時候，會去執行對應的 hook。以本文來說，我們需要在 commit 之前，執行一個幫我們做 jshint 檢查的 shell script。

設定方式非常簡單，到專案底下的 .git/hooks 裏面，建立一個名為 pre-commit 的檔案，檔案的主要任務當然就是執行 jshint：

```bash
#!/bin/sh
#
# Run JSHint validation before commit.

files=$(git diff --cached --name-only --diff-filter=ACM | grep ".js$")
pass=true

echo "files:"
if [ "$files" != "" ]; then
    for file in ${files}; do
        echo $file
        result=$(jshint -c .jshintrc ${file})

        if [ "$result" != "" ]; then
            echo $result
            pass=false
        fi
    done
fi


if $pass; then
    exit 0
else
    echo ""
    echo "COMMIT FAILED:"
    echo "Some JavaScript files are invalid. Please fix errors and try committing again."
    exit 1
fi
```

這個 shell script 利用 git diff 把這次 commit 修改的檔案名稱抓出來，並以 grep 過濾出 .js 檔，然後對他們執行 jshint。（在這邊我有設定 jshint 的設定檔為 .jshintrc ）如果沒有問題就 exit 0 正常結束，git commit 就會繼續執行，如果有問題就印出錯誤訊息並 exit 1 中斷 commit。


#Grunt 進階版

如果你的專案有很多人在用，又剛好是使用 grunt，那麼可以新增下列 task 幫你自動安裝 git hook。

```javascript
grunt.registerTask('install-hooks', function () {
  var fs = require('fs');

  // my precommit hook is inside the repo as /pre-commit.hook
  // copy the hook file to the correct place in the .git directory
  grunt.file.copy('pre-commit.hook', '.git/hooks/pre-commit');

  // chmod the file to readable and executable by all
  fs.chmodSync('.git/hooks/pre-commit', '755');
});
```
記得把你的 pre-commit 檔放在 repo 的 `/pre-commit.hook`，這樣使用這個 repo 的人只要執行一次 `grunt install-hooks`，系統就會自動安裝我們寫好的 hooks。

當然 hooks 有很多種，pre-commit hook 也可以做更多事情，就看個人需求更改。