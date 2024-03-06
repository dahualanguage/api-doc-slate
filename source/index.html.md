---
title: 大話 API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - graphql

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Dahua API
---

# Getting started

Dahua APIs，目前採用 [GraphQL](https://graphql.org/) 的技術，可以讓 Client 端比較有彈性的選擇需要什麼資料，可以避免資源浪費。
目前有兩種 APIs：

1. 分級測驗 API
2. 文章口語練習 API

## 測試機(dev) endpoint:

`https://7ae6w54a45fhxfmxp7ejsora2m.appsync-api.us-west-2.amazonaws.com/graphql`

## 正式機(prod) endpoint:

`還沒架，預計明年初開始架` 😆

# Authentication

> 請把以下這個放到 GraphQL Query/Mutation 的 Headers 裡面

```graphql
{
  "x-api-key": "yourApiKey"
}
```

Dahua GraphQL API 要求每一個 request 必須要有未過期的 API KEY.

`x-api-key: yourApiKey`

<aside class="notice">
API KEY 每一年需要更新一次 (找 Wilson 拿 API KEY.)
</aside>

# Time

因為用戶可能來自世界各地，所以在這個系統中的所有 `startTime` 都是 [epoch timestamp](https://www.epochconverter.com/) 的形式 (例如 `1698117168`)，Client 端要再根據用戶的時區轉換時間。

# 難度分級 Level

大話的難度分級總共有七級，以下是 Level ID 對照表。

| LevelId         | Level              |
| --------------- | ------------------ |
| `TOCFL1:HSK2`   | Novice             |
| `TOCFL2:HSK3`   | Beginner           |
| `TOCFL3:HSK4`   | Elementary         |
| `TOCFL4:HSK5`   | Intermediate       |
| `TOCFL5:HSK6`   | Upper-intermediate |
| `TOCFL6:HSK7-9` | Advanced           |
| `TOCFL7:HSK7-9` | Challenge          |

# Debugging

所有的 APIs 都會回傳 `requestId`，如果開發過程遇到問題，可以提供 `requestId` 給大話，大話的工程師會根據 `requestId` 去檢查 logs。

```graphql
query exams {
  exams(accountId: "test-accountId", withDetails: true) {
    error
    message
    requestId
    exams {
      accountId
      sk
      startTime
      status
      completedTime
    }
  }
}
```

> 以上的 exams query 會回傳以下 JSON，包含 requestId:

```json
{
  "data": {
    "exams": {
      "error": false,
      "message": "",
      "requestId": "a5b92b87-d574-46d6-a7a8-1225b0b4cf51",
      "exams": [
        {
          "accountId": "test-accountId",
          "sk": "exam:DETERMINE_LEVEL:ts:1699423781",
          "startTime": 1699423781,
          "status": "COMPLETED",
          "completedTime": 1699423820
        }
      ]
    }
  }
}
```

# 分級測驗 API

以下是 "分級測驗 API" 的流程圖。

![分級測驗 API](./images/exam.svg)

## Start Exam - Mutation

```graphql
mutation startExam {
  startExam(accountId: "test-accountId", examType: DETERMINE_LEVEL) {
    error
    message
    exam {
      accountId
      sk
      startTime
      nextQuestionNumber
      nextQuestionId
      status
      type
      remainingNumOfQuestion
      totalNumOfQuestion
      questionDetails
      achievedLevel
    }
  }
}
```

> 以上的 startExam mutation 會回傳以下 JSON:

```json
{
  "data": {
    "startExam": {
      "error": false,
      "message": "",
      "exam": {
        "accountId": "test-accountId",
        "sk": "exam:DETERMINE_LEVEL:ts:1698090652",
        "startTime": 1698090652,
        "nextQuestionNumber": 1,
        "nextQuestionId": "TOCFL1:HSK2:SA|1000000021",
        "status": "IN_PROGRESS",
        "type": "DETERMINE_LEVEL",
        "remainingNumOfQuestion": 3,
        "totalNumOfQuestion": 3,
        "questionDetails": null,
        "achievedLevel": "TOCFL1, HSK2"
      }
    }
  }
}
```

startExam mutation 會開啟一個新的考試，一個學生可以有多個考試。
`examType` 這裡預設為 `DETERMINE_LEVEL`
開啟一個考試後，API 會回傳下一題的題目 ID(`nextQuestionId`)

考試狀態(`status`) 有 3 種(`IN_PROGRESS`, `COMPLETED`, `CANCELED`)

考試類型(`type`) 有 2 種(`DETERMINE_LEVEL`, `ARTICLE_SPEECH_PRACTICE`)

### Mutation Input Parameters

| Parameter            | type   | Description                                                    |
| -------------------- | ------ | -------------------------------------------------------------- |
| accountId (required) | string | 登入帳號 ID                                                    |
| examType (required)  | enum   | 考試種類(目前只有`DETERMINE_LEVEL`跟`ARTICLE_SPEECH_PRACTICE`) |

<aside class="success">
注意 — examType 是 GraphQL enum type
</aside>

### Response

| Parameter              | type    | Description                                                    |
| ---------------------- | ------- | -------------------------------------------------------------- |
| accountId              | string  | 登入帳號 ID                                                    |
| sk                     | string  | 當前測驗的 sort key                                            |
| startTime              | string  | 當前測驗的開始時間                                             |
| nextQuestionNumber     | Int     | 下一題的題號 (1, 2, 3, 4...)                                   |
| nextQuestionId         | string  | 下一題的題目 ID                                                |
| status                 | enum    | 考試狀態(目前只有`IN_PROGRESS`, `COMPLETED`, `CANCELED`)       |
| type                   | enum    | 考試種類(目前只有`DETERMINE_LEVEL`, `ARTICLE_SPEECH_PRACTICE`) |
| remainingNumOfQuestion | Int     | 當前測驗剩下題數                                               |
| totalNumOfQuestion     | Int     | 當前測驗總共的題數                                             |
| questionDetails        | AWSJSON | 當前測驗的考試細節，包含學生選擇的答案與正確答案               |
| achievedLevel          | string  | 學生達到的程度                                                 |

## Questions - Query

```graphql
query questions {
  questions(questionId: "TOCFL1:HSK2:SA|1000000021", isTW: true) {
    questions {
      levelId
      level
      sk
      HSKLevel
      questionId
      TocflLevel
      question
      reference
      source
      type
      options
    }
    error
    message
  }
}
```

> 以上的 questions query 會回傳以下 JSON:

```json
{
  "data": {
    "questions": {
      "questions": [
        {
          "levelId": "TOCFL1:HSK2:SA",
          "level": "TOCFL1, HSK2",
          "sk": "1000000021",
          "HSKLevel": "2",
          "TocflLevel": "1",
          "questionId": "TOCFL1:HSK2:SA|1000000021",
          "question": "这种鱼___年春天最多，男人们会一起搭船去海上找鱼。",
          "reference": "这种鱼___年春天最多，男人们会一起搭船去海上找鱼。",
          "source": "TOCFL考題",
          "type": "SA",
          "options": "{\"A\":\"没有\",\"B\":\"每\",\"C\":\"几\"}"
        }
      ],
      "error": false,
      "message": ""
    }
  }
}
```

Questions query API 可以根據 input parameters 回傳一個或是多個考題，如果有收到`questionId` 的話就只會回傳一個考題。

### Query Input Parameters

| Parameter             | type    | Description                                                       |
| --------------------- | ------- | ----------------------------------------------------------------- |
| questionId (required) | string  | 考題 ID，這是唯一的                                               |
| levelId (optional)    | string  | 程度 ID (必須配合 `sk` 或是 (`skFrom` + `skTo`))                  |
| sk (optional)         | string  | sort key，每一個程度會有多個 sort key，代表每一個程度會有多個考題 |
| skFrom (optional)     | string  | 起始 sort key                                                     |
| skTo (optional)       | string  | 最終 sort key                                                     |
| isTW (optional)       | boolean | 預設為 false(回傳簡體中文)，true 回傳繁體中文                     |

<aside class="warning">目前暫時不用理會 <code>levelId</code>, <code>sk</code>, <code>skFrom</code>, <code>skTo</code> (正在開發中 🛠️ )</aside>

### Response

| Parameter  | type    | Description                                      |
| ---------- | ------- | ------------------------------------------------ |
| levelId    | string  | 考題的程度 Id                                    |
| level      | string  | 考題的程度 (`TOCFL1, HSK2`, `TOCFL2, HSK3`...)   |
| sk         | string  | 考題的 sort key                                  |
| HSKLevel   | string  | HSK 程度                                         |
| TocflLevel | string  | Tocf 程度                                        |
| questionId | string  | 考題 ID，這是唯一的                              |
| question   | string  | 考題題目                                         |
| reference  | string  | 考題的參考(例如: 閱讀測驗的文章)                 |
| source     | string  | 當前測驗的考試細節，包含學生選擇的答案與正確答案 |
| type       | enum    | 考題的總類，目前有(單選題`SA`, 多選題`MA`)       |
| options    | AWSJSON | 這是一個包含所有選項的 JSON                      |

## Answer Question - Mutation

```graphql
mutation answerQuestion {
  answerQuestion(
    accountId: "test-accountId"
    sk: "exam:DETERMINE_LEVEL:ts:1698090652"
    questionId: "TOCFL1:HSK2:SA|1000000021"
    studentAnswers: ["A"]
  ) {
    nextQuestionId
    remainingNumOfQuestion
    nextLevel
    achievedLevel
    examStatus
    message
    error
  }
}
```

> 以上的 answerQuestion mutation 會回傳以下 JSON:

```json
{
  "data": {
    "answerQuestion": {
      "nextQuestionId": "TOCFL1:HSK2:SA|1000000017",
      "remainingNumOfQuestion": 2,
      "nextLevel": "TOCFL1, HSK2",
      "achievedLevel": "TOCFL1, HSK2",
      "examStatus": "IN_PROGRESS",
      "message": "",
      "error": false
    }
  }
}
```

> 測驗結束的 Response 如下：

```json
{
  "data": {
    "answerQuestion": {
      "nextQuestionId": "",
      "remainingNumOfQuestion": 0,
      "nextLevel": "",
      "achievedLevel": "TOCFL1, HSK2",
      "examStatus": "COMPLETED",
      "message": "",
      "error": false
    }
  }
}
```

Answer Question mutation 可以讓學生回答考題。
每一次的 API call 都會回傳下一題的題目 ID (`nextQuestionId`) 與測驗狀態 (`examStatus`)，當 `nextQuestionId` 等於空字串(`""`) 並且 `examStatus` 等於 `COMPLETED`，代表測驗結束，並且最終程度 (`achievedLevel`) 就是學生的程度。

### Mutation Input Parameters

| Parameter      | type            | Description                  |
| -------------- | --------------- | ---------------------------- |
| accountId      | string          | 登入帳號 ID                  |
| sk             | string          | 考試的 sort key              |
| questionId     | string          | 題目 ID                      |
| studentAnswers | Array of String | 學生的答案 (`['A', 'C']`...) |

### Response

| Parameter              | type   | Description      |
| ---------------------- | ------ | ---------------- |
| nextQuestionId         | string | 下一題考題的 ID  |
| remainingNumOfQuestion | Int    | 當前測驗剩下題數 |
| nextLevel              | string | 下一題考題的難度 |
| achievedLevel          | string | 學生達到的程度   |
| examStatus             | enum   | 當前測驗的狀態   |

## Exams - Query

```graphql
query exams {
  exams(accountId: "test-accountId", withDetails: false) {
    error
    message
    exams {
      accountId
      sk
      nextQuestionId
      questionDetails
      remainingNumOfQuestion
      startTime
      completedTime
      status
      totalNumOfQuestion
      achievedLevel
      type
    }
  }
}
```

> 以上的 exams query 會回傳以下 JSON:

```json
{
  "data": {
    "exams": {
      "error": false,
      "message": "",
      "exams": [
        {
          "accountId": "test-accountId",
          "sk": "exam:DETERMINE_LEVEL:ts:1698090545",
          "nextQuestionId": null,
          "questionDetails": null,
          "remainingNumOfQuestion": null,
          "startTime": 1698090545,
          "completedTime": 1699423820,
          "status": "COMPLETED",
          "totalNumOfQuestion": null,
          "achievedLevel": null,
          "type": "DETERMINE_LEVEL"
        },
        {
          "accountId": "test-accountId",
          "sk": "exam:DETERMINE_LEVEL:ts:1698090652",
          "nextQuestionId": null,
          "questionDetails": null,
          "remainingNumOfQuestion": null,
          "startTime": 1698090652,
          "completedTime": 1699423920,
          "status": "COMPLETED",
          "totalNumOfQuestion": null,
          "achievedLevel": null,
          "type": "DETERMINE_LEVEL"
        }
      ]
    }
  }
}
```

> 如果 withDetails 是 true 的話，會回傳以下 JSON：

```json
{
  "data": {
    "exams": {
      "error": false,
      "message": "",
      "exams": [
        {
          "accountId": "test-accountId",
          "sk": "exam:DETERMINE_LEVEL:ts:1698090545",
          "nextQuestionId": "",
          "questionDetails": "{\"1\":{\"sk\":\"1000000002\",\"correctAnswers\":[\"A\"],\"studentAnswers\":[\"A\"],\"level\":\"TOCFL1, HSK2\",\"levelId\":\"TOCFL1:HSK2:SA\",\"isCorrect\":true},\"2\":{\"sk\":\"1000000013\",\"correctAnswers\":[\"B\"],\"studentAnswers\":[\"A\"],\"level\":\"TOCFL1, HSK2\",\"levelId\":\"TOCFL1:HSK2:SA\",\"isCorrect\":false},\"3\":{\"sk\":\"1000000004\",\"correctAnswers\":[\"A\"],\"studentAnswers\":[\"A\"],\"level\":\"TOCFL1, HSK2\",\"levelId\":\"TOCFL1:HSK2:SA\",\"isCorrect\":true},\"consecutiveWrong\":0,\"consecutiveCorrect\":1}",
          "remainingNumOfQuestion": 0,
          "startTime": 1698090545,
          "completedTime": 1699423820,
          "status": "COMPLETED",
          "totalNumOfQuestion": 3,
          "achievedLevel": "TOCFL1, HSK2",
          "type": "DETERMINE_LEVEL"
        },
        {
          "accountId": "test-accountId",
          "sk": "exam:DETERMINE_LEVEL:ts:1698090652",
          "nextQuestionId": "",
          "questionDetails": "{\"1\":{\"sk\":\"1000000021\",\"correctAnswers\":[\"B\"],\"studentAnswers\":[\"A\"],\"level\":\"TOCFL1, HSK2\",\"levelId\":\"TOCFL1:HSK2:SA\",\"isCorrect\":false},\"2\":{\"sk\":\"1000000017\",\"correctAnswers\":[\"A\"],\"studentAnswers\":[\"A\"],\"level\":\"TOCFL1, HSK2\",\"levelId\":\"TOCFL1:HSK2:SA\",\"isCorrect\":true},\"3\":{\"sk\":\"1000000002\",\"correctAnswers\":[\"A\"],\"studentAnswers\":[\"A\"],\"level\":\"TOCFL1, HSK2\",\"levelId\":\"TOCFL1:HSK2:SA\",\"isCorrect\":true},\"consecutiveWrong\":0,\"consecutiveCorrect\":0}",
          "remainingNumOfQuestion": 0,
          "startTime": 1698090652,
          "completedTime": 1699423920,
          "status": "COMPLETED",
          "totalNumOfQuestion": 3,
          "achievedLevel": "TOCFL1, HSK2",
          "type": "DETERMINE_LEVEL"
        }
      ]
    }
  }
}
```

Exam query 可以根據 input parameters 回傳學生的 "一個" 或 "多個" 考題歷史紀錄。

1.  當你只提供 `accountId` 的情況下，Exam API 會回傳這個登錄帳號下 "所有" 的測驗歷史紀錄。
2.  當你提通 `accountId` + `sk` 的情況下，Exam API 會回傳這個登錄帳號下 "一個" 測驗記錄。

比較推薦的做法是，可以先不傳`withDetails`(預設為`false`)，並且拿到學生的多個測驗歷史後，在讓學生點選某一個測驗，這時候在把 `withDetails`設定為`true` 並且傳(`accountId` + `sk`(學生選中的 sk))給 Exam API，這時 Exam API 就會把所有測驗細節回傳給學生。

### Mutation Input Parameters

| Parameter                  | type    | Description                                                    |
| -------------------------- | ------- | -------------------------------------------------------------- |
| accountId (required)       | string  | 登入帳號 ID                                                    |
| sk (optional)              | string  | 測驗的 sort key                                                |
| examType (optional)        | enum    | 考試種類(目前只有`DETERMINE_LEVEL`跟`ARTICLE_SPEECH_PRACTICE`) |
| withDetails (預設為 false) | boolean | 學生的答案                                                     |

<aside class="success">
 注意 - <code>withDetails</code> 預設為 <code>false</code> (API 響應速度較快)，當需要取得學生的作答記錄時，在把 <code>withDetails</code> 設定為 <code>true</code>。
</aside>

### Response

| Parameter              | type    | Description                                      |
| ---------------------- | ------- | ------------------------------------------------ |
| accountId              | string  | 登入帳號 ID                                      |
| sk                     | string  | 測驗的 sort key                                  |
| nextQuestionId         | string  | 下一題考題的 Id                                  |
| questionDetails        | AWSJSON | 當前測驗的考試細節，包含學生選擇的答案與正確答案 |
| remainingNumOfQuestion | Int     | 當前測驗剩下題數                                 |
| startTime              | Int     | 當前測驗的開始時間                               |
| completedTime          | Int     | 當前測驗的完成時間                               |
| status                 | enum    | 當前測驗的狀態                                   |
| totalNumOfQuestion     | Int     | 當前測驗總共的題數態                             |
| achievedLevel          | String  | 學生達到的程度                                   |
| type                   | enum    | 當前測驗的種類                                   |

## Edit Exam - Mutation

```graphql
mutation editExam {
  editExam(
    accountId: "test-accountId"
    sk: "exam:DETERMINE_LEVEL:ts:1698348846"
    status: CANCELED
  ) {
    error
    message
  }
}
```

> 以上的 editExam mutation 會回傳以下 JSON:

```json
{
  "data": {
    "editExam": {
      "error": false,
      "message": "Edited the exam successfully."
    }
  }
}
```

Edit exam mutation 可以修改測驗，目前只允許修改 `status`，未來可能會加上更多可被修改的項目。

學生可以把 `status` 設成 `CANCELED` 取消考試，被取消的考試會在七天後從資料庫中永久刪除。

考試被學生取消的七天內，學生可以把 `status` 從 `CANCELED` 設成 `IN_PROGRESS` 把考試重新激活並繼續作答。

### Mutation Input Parameters

| Parameter            | type   | Description                                                    |
| -------------------- | ------ | -------------------------------------------------------------- |
| accountId (required) | string | 登入帳號 ID                                                    |
| sk (required)        | string | 測驗的 sort key                                                |
| status               | enum   | 想要改成的狀態(目前只有`IN_PROGRESS`, `COMPLETED`, `CANCELED`) |

### Response

| Parameter | type    | Description  |
| --------- | ------- | ------------ |
| error     | boolean | 是否有 Error |
| message   | string  | Error 訊息   |

# 文章口語練習 API

以下是 "文章口語練習 API" 的流程圖，藍色虛線代表異步(async)的在後端執行，冠富只有第三步需要直接上傳 mp3 檔到 AWS S3，其他步驟都只要跟 API 串接即可。

![分級測驗 API](./images/speechPractice.svg)

## Start Exam - Mutation

```graphql
mutation startExam {
  startExam(
    accountId: "test-accountId"
    examType: ARTICLE_SPEECH_PRACTICE
    article: {
      articleId: "69"
      levelId: "TOCFL7:HSK7-9"
      articleContent: "在好萊塢(Hollywood)電影裡的東方角色，總是給人一種神秘的感覺--不是狡猾又危險，就是深藏不露的厲害角色...."
      questions: [
        {
          questionId: "1"
          questionContent: "請問你有沒有自己學習古代哲學的經驗？請說說看。"
          description: "1. 你在什麼情況下知道的？2. 在知道之後，你做了什麼樣的學習？"
        }
      ]
    }
  ) {
    error
    message
    exam {
      accountId
      sk
      startTime
      status
      type
      remainingNumOfQuestion
      totalNumOfQuestion
    }
  }
}
```

> 以上的 startExam mutation 會回傳以下 JSON:

```json
{
  "data": {
    "startExam": {
      "error": false,
      "message": "",
      "requestId": "e17d09f9-12e6-4370-a431-d3523d9c6449",
      "exam": {
        "accountId": "test-accountId",
        "sk": "main:ARTICLE_SPEECH_PRACTICE:ts:1706318115",
        "startTime": 1706318115,
        "status": "IN_PROGRESS",
        "type": "ARTICLE_SPEECH_PRACTICE",
        "remainingNumOfQuestion": 1,
        "totalNumOfQuestion": 1
      }
    }
  }
}
```

startExam mutation 會開啟一個文章口語練習，一個學生可以有多個練習，在這裡 `examType` 必須為 `ARTICLE_SPEECH_PRACTICE`，每次開啟文章口語練習都要傳`article`給 API，返回的口語練習狀態(`status`) 有 3 種(`IN_PROGRESS`, `COMPLETED`, `CANCELED`)

### Mutation Input Parameters

| Parameter                           | type   | Description                                                    |
| ----------------------------------- | ------ | -------------------------------------------------------------- |
| accountId (required)                | string | 登入帳號 ID                                                    |
| examType (required)                 | enum   | 考試種類(目前只有`DETERMINE_LEVEL`跟`ARTICLE_SPEECH_PRACTICE`) |
| article (required)                  | object | 文章內容(包含題目)                                             |
| article.articleId (required)        | string | 文章的 ID                                                      |
| article.levelId (required)          | string | 文章的分級                                                     |
| article.articleContent (required)   | string | 文章內容                                                       |
| article.questions (required)        | array  | 考題(question) Array                                           |
| question (required)                 | object | 考題(question)                                                 |
| question.questionId (required)      | string | 考題的 ID                                                      |
| question.questionContent (required) | string | 題目內容                                                       |
| question.description (optional)     | string | 考題的解釋                                                     |

<aside class="success">
注意 — examType 是 GraphQL enum type
</aside>

### Response

| Parameter              | type   | Description                                              |
| ---------------------- | ------ | -------------------------------------------------------- |
| accountId              | string | 登入帳號 ID                                              |
| sk                     | string | 當前練習的 sort key                                      |
| startTime              | string | 當前練習的開始時間                                       |
| status                 | enum   | 練習狀態(目前只有`IN_PROGRESS`, `COMPLETED`, `CANCELED`) |
| type                   | enum   | 練習種類(`ARTICLE_SPEECH_PRACTICE`)                      |
| remainingNumOfQuestion | Int    | 當前文章練習題的剩下題數                                 |
| totalNumOfQuestion     | Int    | 當前文章口語練習總共的題數                               |

## Upload Voice - Query

```graphql
query getUploadVoiceSignedURL {
  getUploadVoiceSignedURL(
    accountId: "test-accountId"
    sk: "main:ARTICLE_SPEECH_PRACTICE:ts:1706318115"
    articleId: "69"
    articleIdQuestionId: "1"
  ) {
    error
    message
    signedUrl
  }
}
```

> 以上的 getUploadVoiceSignedURL query 會回傳以下 JSON:

```json
{
  "data": {
    "getUploadVoiceSignedURL": {
      "error": false,
      "message": "",
      "signedUrl": "https://dev-commerce-speech.s3.ap-southeast-1.amazonaws.com/ARTICLE_SPEECH_PRACTICE/test-accountId/1706318115/69/1.mp3?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAYQHUHCC46NUU75GE%2F20240127%2Fap-southeast-1%2Fs3%2Faws4_request&X-Amz-Date=20240127T042342Z&X-Amz-Expires=60&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEP3%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLXNvdXRoZWFzdC0xIkcwRQIgVm1ZZUsR1%2BbSruaTat6iIRN5zNdJdQsZzjksGhegUgACIQC%2FbiRzjOjR1%2F193ltTR%2FmwoxaOPoZuiAyCO7BvoMVKYyqTAwi2%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDU4NDYyNzcyMDM3NyIMroF40FQt%2BlZkIV6bKucCZRPmd%2Fxzfu3adiPusT4UaTvFcWvvfzk6gSVieRzlF9aDWK1vwH%2FVHNV6bECdZ3aoa1JjRY8OEdh10GQeJxtX951V06HoDUPJMuWFvrQST6cIMwVAyk18ztzn%2BIoCrVZdg6y0MX24eKVq1cGn8HKVHQUM5aVIOHiBMlrZ4AHL9gDYJQc9sgEqnv2jI%2F7NfnsN%2BHHU1HwXVGPDIywOgDHKucz4c%2BshPUnufkN6zLE2R6FooJ1R7xbjV4mjdhRd25ypMDiKp9tnDRtCPrkMR%2FN1zuSrXtRktark3%2B3NFHX9Vsmlqg440JcpoR4Z8ZU%2B%2BwEJnXzON1dKaVfN22BED6v9baHQFGM3TJgnCQO%2FVfqUsFq%2BZ0EhcQ%2FmmqRyoR6vllviCpGZLswTunocAGEYO5zRJn2ejQsgIk9DMeUikAMOuMz6lPeDNCxooRox9ELHv4c4d5GZzjffEvzF39WU3DaEpXUTGlJl%2BSYwoYrSrQY6nQFcUXN3liJjWZvIW1D09ZTQX89DWLeGfuYJJ%2BY0rJ2afR7KYNIWWqX2vqHsUtTwB2918Gn5ZamwWRkkK3ufMiDpIp7ob%2BN3Mb99CEYY63y1ujF%2BTn02lZmMh%2Fr5qZpaPRXGYZmoRGQaJtwMUU9rauQP%2BDXjyUJUZUHq15hr1Czrq0Psv%2F%2B1sYS0Wun6aZgahjRiQ%2FJ2hNSCi1nOjV4P&X-Amz-Signature=e853aaa06de219a7ee491cbd74deda69317507f6bc3276bcccb09dac7dc5f996&X-Amz-SignedHeaders=content-encoding%3Bhost&x-id=PutObject"
    }
  }
}
```

> 以下是 ChatGPT 給的簡單的 Pseudo code，可以上傳 mp3 檔到 Signed URL

```javascript
// 假設 `signedUrl` 是從後端獲得的 Signed URL
const signedUrl = "YOUR_PRE_SIGNED_URL_HERE";

// `file` 是一個指向你要上傳的 MP3 檔案的參考，可以通過 <input type="file"> 獲得
async function uploadFile(file, signedUrl) {
  try {
    const response = await fetch(signedUrl, {
      method: "PUT",
      // Signed URL 中已包含所有必要的認證頭信息，所以這裡不需要設置 `Authorization` 頭
      headers: {
        "Content-Type": "audio/mpeg",
        "Content-Length": file.size
      },
      body: file, // 將Mp3檔作為請求體
    });

    if (response.ok) {
      console.log("檔案上傳成功");
    } else {
      console.error("檔案上傳失敗", response.statusText);
    }
  } catch (error) {
    console.error("上傳過程中發生錯誤", error);
  }
}

// 假設這是一個簡單的觸發上傳的按鈕事件
document.getElementById("uploadButton").addEventListener("click", function () {
  const fileInput = document.getElementById("fileInput");
  if (fileInput.files.length > 0) {
    const file = fileInput.files[0]; // 獲取用戶選擇的第一個檔案，即 MP3 檔案
    uploadFile(file, signedUrl);
  } else {
    console.log("請選擇一個檔案");
  }
});

// HTML
<input type="file" id="fileInput" accept=".mp3" />
<button id="uploadButton">Upload MP3</button>
```

上傳 mp3 音檔有兩個步驟：

1. 呼叫 `getUploadVoiceSignedURL` API 取得一次性的 Signed URL
2. 透過 Signed URL 直接把音檔上傳到 S3 (不會透過大話 API)

<aside style="background-color: #f9f9f9; border-left: 6px solid #28a745; margin: 20px 0; padding: 15px; box-shadow: 0 2px 4px rgba(0,0,0,.1);">
    <strong>注意：</strong>
    <br>
    <ol>
        <li>音檔的檔名必須是 <code>articleIdQuestionId.mp3</code>（例如：articleIdQuestionId: <code>1</code>, 音檔名必須是 <code>1.mp3</code>）</li>
        <li>Signed URL 會在 60 秒內過期，所以建議學生錄完音，準備上傳的時候再去呼叫 <code>getUploadVoiceSignedURL</code> API</li>
    </ol>
</aside>

### S3 Signed URL 參考資料 (有興趣再讀)

1. [AWS Signed URL 文檔](https://docs.aws.amazon.com/zh_tw/AmazonS3/latest/userguide/PresignedUrlUploadObject.html)
2. [類似的範例](https://medium.com/weekly-webtips/upload-files-directly-to-s3-with-pre-signed-url-31beff41157e)

### Query Input Parameters

| Parameter                      | type   | Description         |
| ------------------------------ | ------ | ------------------- |
| accountId (required)           | string | 登入帳號 ID         |
| sk (required)                  | string | 當前練習的 sort key |
| articleId (required)           | string | 當前文章 ID         |
| articleIdQuestionId (required) | string | 當前文章考題 ID     |

### Response

| Parameter | type    | Description                 |
| --------- | ------- | --------------------------- |
| signedUrl | string  | 上傳音檔用的 Pre-Signed URL |
| error     | boolean | 是否有 Error                |
| message   | string  | Error 訊息                  |

## Update Voice Memo - Mutation

```graphql
mutation updateVoiceMemo {
  updateVoiceMemo(
    accountId: "test-accountId"
    sk: "main:ARTICLE_SPEECH_PRACTICE:ts:1709267875"
    articleId: "69"
    articleIdQuestionId: "1"
    memo: "This is my memo."
  ) {
    error
    message
    requestId
  }
}
```

> 以上的 updateVoiceMemo mutation 會回傳以下 JSON:

```json
{
  "data": {
    "updateVoiceMemo": {
      "error": false,
      "message": null,
      "requestId": "66067a15-9178-45e8-bb08-460cde0e3d86"
    }
  }
}
```

### Query Input Parameters

| Parameter                      | type   | Description         |
| ------------------------------ | ------ | ------------------- |
| accountId (required)           | string | 登入帳號 ID         |
| sk (required)                  | string | 當前練習的 sort key |
| articleId (required)           | string | 當前文章 ID         |
| articleIdQuestionId (required) | string | 當前文章考題 ID     |
| memo (required)                | string | Memo                |

### Response

| Parameter | type    | Description  |
| --------- | ------- | ------------ |
| error     | boolean | 是否有 Error |
| message   | string  | Error 訊息   |

## Get Voice Memo - Query

```graphql
query voiceMemo {
  voiceMemo(
    accountId: "test-accountId"
    sk: "main:ARTICLE_SPEECH_PRACTICE:ts:1709267875"
    articleId: "69"
    articleIdQuestionId: "1"
  ) {
    error
    message
    memo
    requestId
  }
}
```

> 以上的 voiceMemo query 會回傳以下 JSON:

```json
{
  "data": {
    "voiceMemo": {
      "error": false,
      "message": "",
      "memo": "hello~~~~~~",
      "requestId": "409566b5-993f-4ab4-a3e1-14921168a5b0"
    }
  }
}
```

### Query Input Parameters

| Parameter                      | type   | Description         |
| ------------------------------ | ------ | ------------------- |
| accountId (required)           | string | 登入帳號 ID         |
| sk (required)                  | string | 當前練習的 sort key |
| articleId (required)           | string | 當前文章 ID         |
| articleIdQuestionId (required) | string | 當前文章考題 ID     |

### Response

| Parameter | type    | Description  |
| --------- | ------- | ------------ |
| error     | boolean | 是否有 Error |
| message   | string  | Error 訊息   |
| memo      | string  | Memo         |

## Exams - Query

```graphql
query exams {
  exams(
    accountId: "test-accountId"
    sk: "main:ARTICLE_SPEECH_PRACTICE:ts:1706243619"
  ) {
    error
    message
    requestId
    exams {
      questionDetails
      startTime
      status
      completedTime
    }
  }
}
```

> 以上的 exams query 會回傳以下 JSON:

```json
{
  "data": {
    "exams": {
      "error": false,
      "message": "",
      "requestId": "3101f714-e59c-4ca4-8d3b-85b126b72f0a",
      "exams": [
        {
          "questionDetails": "{\"1\":{\"questionId\":\"1\",\"articleId\":\"69\",\"voiceText\":\"我看 Minecraft 的视频的时候,才听说孙子的哲学。 一个玩游戏的人用了孙子的哲学赢一场农场土豆比赛。 我觉得在现代的电脑游戏上用古老的兵法太有趣了。 所以我马上买了一本《孙子兵法》,一天就看完了。 我的叔叔是军事武器的企业家,他给了我《孙子兵法》这本书并强调哲学战略,不但实用于军事的业用在日常生活。\",\"questionContent\":\"請問你有沒有自己學習古代哲學的經驗？請說說看。\",\"sk\":\"q:ARTICLE_SPEECH_PRACTICE:ts:1706243619:69:1\",\"memo\":\"\",\"analyzedResult\":\"這個是AI的口語分析結果.....\"}}",
          "startTime": 1706243619,
          "status": "COMPLETED",
          "completedTime": 1706244545
        }
      ]
    }
  }
}
```

Exam query 可以根據 input parameters 回傳學生的 "一個" 或 "多個" 考試/練習 歷史紀錄。

1.  當你只提供 `accountId` 的情況下，Exam API 會回傳這個登錄帳號下 "所有" 的考試/練習歷史紀錄。
2.  當你提通 `accountId` + `sk` 的情況下，Exam API 會回傳這個登錄帳號下 "一個" 考試/練習記錄。

比較推薦的做法是，可以先不傳 withDetails(預設為 false)，並且拿到學生的多個測驗歷史後，在讓學生點選某一個測驗，這時候在把 withDetails 設定為 true 並且傳(accountId + sk(學生選中的 sk))給 Exam API，這時 Exam API 就會把所有測驗細節回傳給學生。

### Query Input Parameters

| Parameter                   | type    | Description                      |
| --------------------------- | ------- | -------------------------------- |
| accountId (required)        | string  | 登入帳號 ID                      |
| sk (required)               | string  | 當前練習的 sort key              |
| examType (optional)         | string  | 考試種類                         |
| withDetails(預設為 `false`) | boolean | 是否回傳學生的答案 + AI 分析結果 |

### Response

| Parameter                   | type    | Description                                              |
| --------------------------- | ------- | -------------------------------------------------------- |
| error                       | boolean | 是否有 Error                                             |
| message                     | string  | Error 訊息                                               |
| exams                       | array   | exam(考試/練習) array                                    |
| exam                        | object  | exam(考試/練習)                                          |
| exam.startTime              | Int     | 當前練習的開始時間                                       |
| exam.status                 | string  | 考試狀態(目前只有`IN_PROGRESS`, `COMPLETED`, `CANCELED`) |
| exam.completedTime          | Int     | 當前練習的結束時間                                       |
| exam.questionDetails        | AWSJSON | 包含"所有"題目的(語音轉文字內容+AI 分析的結果)           |
| questionDetails[questionId] | obj     | question 包含"單一"題目的(語音轉文字內容+AI 分析的結果)  |
| question.questionId         | string  | 練習題目的 ID                                            |
| question.articleId          | string  | 練習文章的 ID                                            |
| question.voiceText          | string  | 語音轉文字內容                                           |
| question.questionContent    | string  | 練習題題目                                               |
| question.sk                 | string  | 練習題題目 sort key                                      |
| question.memo               | string  | 練習題題目備註                                           |
| question.analyzedResult     | string  | AI 分析的結果                                            |

## Download Voice - Query

```graphql
query getDownloadVoiceSignedURL {
  getUploadVoiceSignedURL(
    accountId: "test-accountId"
    sk: "main:ARTICLE_SPEECH_PRACTICE:ts:1706243619"
    articleId: "69"
    articleIdQuestionId: "1"
  ) {
    error
    message
    signedUrl
  }
}
```

> 以上的 getUploadVoiceSignedURL query 會回傳以下 JSON:

```json
{
  "data": {
    "getUploadVoiceSignedURL": {
      "error": false,
      "message": "",
      "signedUrl": "https://dev-commerce-speech.s3.ap-southeast-1.amazonaws.com/ARTICLE_SPEECH_PRACTICE/test-accountId/1706243619/69/1.mp3?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=ASIAYQHUHCC4ZXLIT3FD%2F20240128%2Fap-southeast-1%2Fs3%2Faws4_request&X-Amz-Date=20240128T060141Z&X-Amz-Expires=60&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBYaDmFwLXNvdXRoZWFzdC0xIkcwRQIgTPC59NGDjRZhbZ82TyoS30QoF0t4KIRvfkD5gmQlnJ8CIQDhfAgF6gywaEGYGtYnwXOCz39%2Bsj20Tp05zCx8rcRqyiqTAwjP%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAAaDDU4NDYyNzcyMDM3NyIMGnm34NCSgUUOwy3xKucCsF6Oo1HlYijF0PKRWyYS7xZtceRLp0dcfdnjv0g%2Fy8v9Y8M9XXLfCPqEncXiM8pB6za9ANhN55Srx6hSvonsME3ZKnR0UZauPoo4bw8UJIqhediLtH%2F98RWiCRWN7ljf727doxPuFAWddPE6BK7wKs8EKC5xDqH1JXN8%2BNlMueJOmFTjyhCnlaK%2FQZlhWQ5O8dIaWRB7u1o2LQzfJg4kA1JQJk3oGVvsXGixdbBUMoiOrAEdtK0laqfuGVYnlAlDY1j4Tbsml9gOLoKC5CtAqTsPWAP7P0Mv4WbuEmJfXrx40OCsAkPUZubk3urGp5z1lb%2Bc%2BGnpJBdcxjLPID6pri6zfn1%2BMqOOnSXq9i965fu3vrzAjIDVBh7kElNtL1fv9Xpfqld9ULZh6egzlP%2FJSOvM9vfsO5ismTGHqrvAATG3sBfYaJJ63Z6hMAnTW9uiy0kL60Jokn7ASVg4r202QUqi7hcOnuAwxNvXrQY6nQG3KE18vQX9I7Oj0kYoqwY9MGEnhejz5M%2FOVMxS9kD6alOfGHmsE8sFy%2FjmNIc4uIDXfx6GV00ziFsoHxEifzBkOsuyiFWoOf8AF2W6hx7R9S8hSrNBhDVkPkQjg3QIOr9IGxR9gv75dfgVH9GG5EH%2FZ2EszXa9RTeXPZOHrcYr3EghW6EljZT0DhbuoeFrPVwlV15Jd7NC%2FYalq2ff&X-Amz-Signature=e421b7c7ce2829dc83254ead59d95cd8fb364338da30e1dc4a273b349c9dc9ce&X-Amz-SignedHeaders=content-encoding%3Bhost&x-id=PutObject"
    }
  }
}
```

> 以下是 ChatGPT 給的簡單的 Pseudo code，可以從 S3 下載 mp3 檔的 Signed URL

```javascript
// singedUrl from getDownloadVoiceSignedURL
const signedUrl = "SIGNED_URL";

// 獲取 audio 標籤
const audioPlayer = document.getElementById("audioPlayer");

// 定義異步函數來下載並播放音頻
async function downloadAndPlayAudio(signedUrl) {
  try {
    const response = await fetch(signedUrl); // 使用 fetch 下載 MP3 文件
    const blob = await response.blob(); // 將響應轉換為 Blob

    // 創建一個指向下載內容的 URL
    const audioUrl = window.URL.createObjectURL(blob);
    audioPlayer.src = audioUrl; // 設置音頻源
    audioPlayer.play(); // 播放音頻
  } catch (error) {
    console.error("Error downloading or playing the audio:", error);
  }
}

// 呼叫函數並傳入 Pre-Signed URL
downloadAndPlayAudio(signedUrl);
```

下載 mp3 音檔有兩個步驟：

1. 呼叫 `getDownloadVoiceSignedURL` API 取得一次性的 Signed URL
2. 透過 Signed URL 直接從 S3 下載音檔 (不會透過大話 API)

### Query Input Parameters

| Parameter                      | type   | Description         |
| ------------------------------ | ------ | ------------------- |
| accountId (required)           | string | 登入帳號 ID         |
| sk (required)                  | string | 當前練習的 sort key |
| articleId (required)           | string | 當前文章 ID         |
| articleIdQuestionId (required) | string | 當前文章考題 ID     |

### Response

| Parameter | type    | Description                 |
| --------- | ------- | --------------------------- |
| signedUrl | string  | 下載音檔用的 Pre-Signed URL |
| error     | boolean | 是否有 Error                |
| message   | string  | Error 訊息                  |

## Edit Exam - Mutation

跟分級測驗一樣
