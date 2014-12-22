---
layout: post
title:  "CircleCI 與 GitHub 整合"
date:   2014-11-14 14:34:25
tags: 
  - GitHub
  - CircleCI
  - "continuos integration"
image: /assets/article_images/circleci-github-integration.jpg
---

[CircleCI](https://circleci.com/) 是個很強大的 SAAS CI。其中最重要的莫過於 GitHub 的支援，本文就簡單介紹如何整合這兩個工具。

# 材料準備

- GitHub 帳號一個
- AWS 帳號（依個人喜好）

# 做法

前往 [CircleCI](https://circleci.com/) 網站，點擊 `Sign Up Free` ，會要求輸入 GitHub 帳號密碼。
[CIRCLECI SIGN UP](http://4.bp.blogspot.com/-BBF1vyokxtY/VF7TZRUMNOI/AAAAAAAATJs/FheUp37PuhA/s800/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-11-09%2B%E4%B8%8A%E5%8D%8810.36.59.png)

帳號創建完成並授權 CircleCI 存取 GitHub 後，會回到 CircleCI 主控制頁面，點左方 `Add Projects` 並選擇 `Project，CircleCI` 即會取得 code 並開始第一次的建置。

[ADD PROJECTS](http://3.bp.blogspot.com/-pj2mcZGqtJ0/VF80oqJH6LI/AAAAAAAATJ8/kbyG1TwNFW4/s800/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-11-09%2B%E4%B8%8A%E5%8D%8810.50.22.png)

CircleCI 會自動判斷 project 語言並尋找建置設定檔，對於javascript 開發者來說就是 package.json。接著會依序安裝 package.json 所列的 package，並執行 npm test，如果一切順利通過則此次建置就算成功。一切的動作都可以即時在頁面上看到，上圖就是這個 project 可以看到因為沒有 test 可以跑所以失敗。

以上都是基本的狀態，以 CircleCI 預設的環境與參數來測試與建置。如果想要有更進一步的控制，就需要在 GitHub 專案的跟目錄加上一個檔案 `circle.yml`。這個檔案可以讓你自定系統環境、輸出測試報表、控制 branch、發佈等等。一個簡單的 circle.yml 長得像這樣：

```yaml
machine:
  node:
    version: 0.11.13 #自定node版本
general:
  branches:
    ignore:
      - /wip-.*/ # 忽略所有wip-開頭的branch
  artifacts:
    - "test/results" #測試報告放置目錄
deployment:
  dev:
    branch: dev
    commands:
      - grunt dev-deploy #dev branch測試成功後即會執行此指令
  stage:
    branch: stage
    commands:
      - grunt stage-deploy #stage branch測試成功後即會執行此指令
  production:
    branch: prod
    commands:
      - grunt prod-deploy #prod branch測試成功後即會執行此指令
```

[詳細設定參數可以他們的文件](https://circleci.com/docs/configuration)，重點設定有：

- **machine:** 主機環境設定，包含各種軟體版本，環境參數，服務等
- **general:** 一般設定，包含了要或不要測試哪些 branch，報表輸出目錄，建置目錄等等
- **deployment:** 如果需要發佈程式，則須有此設定，唯有 test 成功之後才會執行

### 注意事項
如果你有一些機密資訊比如 AWS key 或 SSH key，別存在 circle.yml，存到 project setting 的 environment variables 裏面，如下圖



[ENV VAR](http://1.bp.blogspot.com/-9473wz74kGc/VF81EVeJagI/AAAAAAAATKE/lC0HxMQJB9g/s800/circleci_project_settings.png)


[ENV VAR SETTING](http://1.bp.blogspot.com/-ss4fw1ZR8UA/VF81ERCBEUI/AAAAAAAATKI/BNsXSw02oY0/s800/circleci_env_settings.png)

主機的環境變數即會有存入的 key-value，任何建置工具只要能取得環境變數的都可以拿到，比如說在 grunt 裏面：

```javascript
aws = {
  key: process.env.AWS_ACCESS_KEY,
  secret: process.env.AWS_SECRET_KEY
};
```

經由 `process.env` 取得了預先設定的環境變數 AWS_ACCESS_KEY 以及 AWS_SECRET_KEY，簡單又安全的儲存機密資訊。

# 除錯

雖然說 CircleCI 的狀態頁面已經提供完整的建置訊息，但有時候總是需要一些更細不的測試，況且總不能每次改什麼小東西都要丟上 GitHub 然後讓機器重新跑一次。這時候SSH的功能就顯得非常好用了，開啓 SSH 的方式是在建置頁面的右上有個 `& enable SSH`。點下去就會出現以下頁面： 



[ENABLE SSH](http://2.bp.blogspot.com/-PykwCBRH7iU/VF81t9HPldI/AAAAAAAATKU/PMVXCpuGdLc/s800/circleci_ssh.png)

# 無用知識

CircleCI也支援 Status Badge（就是我們常在 GitHub 專案的 README 第一行看到小圖示），在 project setting 裡面，可以把 `badge=style` 改成 `badge=shield` 會有另外一種常見的 badge style 出現喔～

[BADGE SETTING](http://3.bp.blogspot.com/-gLXs_ysKhJA/VF810mZKmpI/AAAAAAAATKc/kghlOKkwXc8/s800/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-11-09%2B%E4%B8%8B%E5%8D%885.31.12.png)