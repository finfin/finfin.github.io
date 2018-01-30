---
layout: post
title:  "使用 redux-logic 簡化複雜的流程與測試"
date:   2018-01-30 21:04:59
tags:
  - "react"
  - "redux"
image: /assets/article_images/redux-logic.jpg
excerpt: "redux-logic 使用心得"
---

# 前提知識

- react
- redux 
- mocha

# 待解決的問題

原本專案使用 redux-thunk + redux-api-middleware 來處理複雜的（通常是非同步的）流程。當專案開始有一些複雜的商業邏輯時，redux-thunk 的寫法會開始造成 action creator 肥大，並且商業邏輯會綁死在 action creator 內。

同時，個人很阿雜的一個點就是，這樣子 action creator 就不是 pure function ，想寫一個單元測試也開始需要 mock 更多的物件才有辦法完成。

是時候尋找更好的解決方案了，是吧？

# 選項

survey 之後試過以下幾個解決方案，就順便把優缺點列一列

## [redux-thunk](https://github.com/gaearon/redux-thunk) + [redux-api-middleware](https://github.com/agraboso/redux-api-middleware) (既有的)

### Pros

- 簡單易懂
- 輕量的 lib

### Cons

- 只能 listen action 並做後續動作，無法在 action 發出後到達 store 之前動手腳
- 商業邏輯與 action creator 綁在一起
- 難測試

## 自幹 middleware

一開始想說乾脆把邏輯全部都寫成一個一個 middleware 就好

### Pros

- 想做什麼就做什麼，想什麼時候發 action 就什麼時候發

### Cons

- middleware 的複雜度高，且流程寫不好很容易把後面的 middleware 堵死，一個小螺絲就可以癱瘓整個系統
- 跟 redux-thunk 一樣難測試

看起來不是個容易 scale 的方式，那就下一個選項吧

## [redux-saga](https://github.com/redux-saga/redux-saga)

曾經紅過一陣子的 saga，做法是把商業邏輯抽出來變成 saga 檔然後透過 redux-saga 組合成 middleware 

### Pros

- 商業邏輯抽出來另外自己管理
- 提供許多工具諸如 effect, channel 等
- 好測試

### Cons

- 需支援 generator
- 強大的功能也同時帶來了許多新的知識點，要花較長時間學習
- saga 只能 listen action 並做後續動作，無法在 action 發出後到達 store 之前動手腳

### [redux-logic](https://github.com/jeffbski/redux-logic)

在討論 saga 的討論串偶然看到 redux-logic

### Pros

- 商業邏輯抽出來另外自己管理
- API 簡單
- 好測試
- 可對 action 動手腳

### Cons

嗯目前還沒遇到

反正標題已經破梗最後的選擇了，那就 show me the (psuedo) code 吧！

# 實作

我們用一個建立影片紀錄的流程範例來說明，這個範例的流程：

1. 跟 api server 要一個 cloud storage 的 access token
2. 拿 access token 上傳影片至 cloud storage
3. 告訴 api server 建立新的影片紀錄，其中包含影片的 cloud storage url

![建立影片紀錄流程](/assets/article_images/create_video_record_flow.png)

此流程可以寫成以下 psuedo code

```javascript
//logic.js
export const newVideoRecord = createLogic({
  type: 'START_CREATING',
  validate({ getState, action }, allow, reject) {
    const videoFile = action.payload.videoFile;
    
    // 檢查檔案大小
    if (videoFile.size > MAX_VIDEO_SIZE) {
      // TODO: SHOW ERROR
      reject({
        type: 'CREATING_FAILURE',
        payload: 'video too large'
      });
    }
    // ... 其他想要 validate 的事情

    allow(action);
  },

  process({getState, action}, dispatch, done) {
    // dispatch({type: START_CREATING});
    dispatch({type: 'GET_UPLOAD_TOKEN_REQUEST'});
    const {videoFile} = action.payload;

    // getToken 會跟 api server 要 cloud storage 的 token
    return getToken()
      .catch((err) => {
        dispatch({
          type: 'GET_UPLOAD_TOKEN_FAILURE',
          payload: err
        });
      })
      .then((token) => {
        // 要到 token ，
        dispatch({
          type: 'GET_UPLOAD_TOKEN_SUCCESS',
          payload: token
        });

        // 開始上傳 video 到 cloud storage 之前發一個 action
        dispatch({
          type: 'UPLOAD_CLOUD_STORAGE_REQUEST',
          payload: {
            token,
            videoFile
          }
        });

        // 上傳到 cloud storage
        return createBlob(videoFile, token)
          .catch((err) => {
            // 上傳失敗了
            dispatch({
              type: 'UPLOAD_CLOUD_STORAGE_FAILURE',
              payload: err
            });
          })
          .then((url) => { // 上傳成功，得到 video 的 blob url

            // 先發個 action 
            dispatch({ type: 'UPLOAD_CLOUD_STORAGE_SUCCESS' });
            return {
              name: videoFile.name,
              size: videoFile.size,
              url
            };
          });
      })
      .then((video) => {
        dispatch({
          type: 'CREATING_VIDEO_RECORD_REQUEST',
          payload: video
        });

        // 跟 api server 建立一筆影片紀錄
        return postVideoRecord(video);
      })
      .catch((err) => {
        dispatch({
          type: 'CREATING_VIDEO_RECORD_FAILURE',
          payload: err
        });
      })
      .then(() => {
        // 建立成功
        done();
        return { type: 'CREATING_VIDEO_RECORD_SUCCESS' };
      });
  }
});

export default [newVideoRecord];
```

然後把此 `logic.js` 註冊到 redux-logic 裡面去

```javascript
import logic from 'logic';

const logicMiddleware = createLogicMiddleware([logic]);
// 把此 logicMiddleware 塞到 redux middleware 裡面去
```

在 react component 上面使用是這樣

```javascript
class CreateVideoRecord extends React.Component {
  ...
  onSubmit() {
    // startCreating 是 connect 進來的 action creator
    startCreating({
      videoFile: this.state.videoFile // 使用者選擇的影片檔案
    });
  }

  render() {
    <Form>
      ... 
      <Button onClick={this.onSubmit}>Submit</Button>
    </Form>
  }
```

action creator 

```javascript
export function startCreating(videoFile) {
  return {
    type: 'START_CREATING',
    payload: videoFile
  };
}
```

觸發流程：

- 使用者選擇影片檔案後點擊 Submit Button
- 觸發 `START_CREATING` 的 action
- logic 監聽到此事件，就進入 `logic.js` 的流程
- 先 `validate` 看是否通過
- 通過的話則 action 進入 store/reducer 改變狀態
- 狀態改變後呼叫 `process` 進行後續動作

這樣的流程發送了許多 action event，可以給 reducer 使用比如說 `GET_UPLOAD_TOKEN_REQUEST`, `UPLOAD_CLOUD_STORAGE_REQUEST`, `CREATING_VIDEO_RECORD_REQUEST` 可以用來改變狀態讓 UI 顯示上傳進度

# 測試

`logic.js` 的測試可以分兩種：

- 發送 action 的正確性
- 最終 redux state 或 redux state 變化的正確性



第二點比較簡單，來講如何測試 action 的正確性，正確性包含了：

- 該發送的 action 是否有發 / 順序是否正確
- action 的資料內容是否正確



redux-logic 提供了 `redux-logic-test` 這個 npm lib 來幫助測試，初步我們可以寫這樣一個 mocha test 來測 sunny path (一樣是 psuedo code)

```javascript
describe('[CREATE VIDEO]', () => {
  let store;
  beforeEach(() => {
    store = createMockStore({
      logic: logic
    });
  });

  it('sunny path', () => {
    store.dispatch({ type: START_CREATING, payload: FAKE_VIDEO_FILE});
    return store.whenComplete(() => {
      expect(store.actions).to.be.eql([
        { type: START_CREATING, payload: FAKE_VIDEO_FILE },
        { type: GET_UPLOAD_TOKEN_REQUEST },
        { type: GET_UPLOAD_TOKEN_SUCCESS },
        { type: UPLOAD_CLOUD_STORAGE_REQUEST },
        { type: UPLOAD_CLOUD_STORAGE_SUCCESS },
        { type: CREATING_VIDEO_RECORD_REQUEST },
        { type: CREATING_VIDEO_RECORD_SUCCESS }
      ]);
    });
  });
});
```

對單元測試比較熟的人會發現這樣的單元測試無法執行，因為此時 `createBlob` `getToken` 跟 `postVideoRecord` 都會真正地發出 request。而這邊 redux-logic 提供了一個功能可以瞬間簡化此步驟： dependency injection。

讓我們回到 middleware 設定的地方並且加入一些東西

```javascript
import logic from 'logic';
import {createBlob, getToken, postVideoRecord} from './apis';

const logicDeps = {
  getToken,
  createBlob,
  postVideoRecord
};

//多了一個 deps
const logicMiddleware = createLogicMiddleware([logic], logicDeps);

// 接下來把此 logicMiddleware 塞到 redux middleware 裡面去
...
```



這步驟是把我們會用到的 dependency 先放進 logic 的 scope，於是在 logic.js 就可以改成這樣

```javascript
//logic.js
export const newVideoRecord = createLogic({
  type: 'START_CREATING',
  validate({ getState, action }, allow, reject) {
    const videoFile = action.payload.videoFile;
    
    // 檢查檔案大小
    if (videoFile.size > MAX_VIDEO_SIZE) {
      // TODO: SHOW ERROR
      reject({
        type: 'CREATING_FAILURE',
        payload: 'video too large'
      });
    }
    // ... 其他想要 validate 的事情

    allow(action);
  },

  // getToken, createBlob, postVideoRecord 可以直接從 process 的 input 裡面取出來
  process({getState, action, getToken, createBlob, postVideoRecord}, dispatch, done) {
    // dispatch({type: START_CREATING});
    dispatch({type: 'GET_UPLOAD_TOKEN_REQUEST'});
    const {videoFile} = action.payload;

    // getToken 會跟 api server 要 cloud storage 的 token
    return getToken()
      .catch((err) => {
        dispatch({
          type: 'GET_UPLOAD_TOKEN_FAILURE',
          payload: err
        });
      })
      .then((token) => {
        // 要到 token ，
        // 跟前面一樣

        // 上傳到 cloud storage
        return createBlob(videoFile, token)
          ... // 跟前面一樣
      })
      .then((video) => {
        dispatch({
          type: 'CREATING_VIDEO_RECORD_REQUEST',
          payload: video
        });

        // 跟 api server 建立一筆影片紀錄
        return postVideoRecord(video);
      })
```

然後單元測試就可以變成這樣

```javascript
const injectedDeps = {
  getToken: mockedGetTokenFunction, 
  createBlob: mockedCreateBlobFunction, 
  postVideoRecord: mockedPostVideoRecordFunction
};

describe('[CREATE VIDEO]', () => {
  let store;
  beforeEach(() => {
    store = createMockStore({
      logic: logic,
      injectedDeps // 假的 deps，這樣 logic.js 就會用到這個
    });
  });

  it('sunny path', () => {
    ...
  });
});
```

於是乎我們就可以透過不同的 injectedDeps 來測試不同的  `logic.js` 裡面的流程。單元測試 ok!

# 結論

透過 redux-logic 可以達到商業邏輯分離，好測試，且沒有太高的進入門檻，是 redux-saga 之外另一個好選擇