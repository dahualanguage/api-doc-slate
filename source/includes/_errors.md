# Errors

<aside class="notice">
Dahua GraphQL API 並不會回傳 4XX 或 5XX error.
</aside>

### Response

| Parameter | type    | Description  |
| --------- | ------- | ------------ |
| message   | string  | Error 訊息   |
| error     | boolean | 是否有 Error |

> 如果有 Error 會回傳以下 JSON，並顯示原因，

```json
{
  "data": {
    "yourQueryOrMutation": {
      "message": "Unexpected error... the reason is ....",
      "error": true
    }
  }
}
```

> 以下是個範例 (answerQuestion API 的 Error)：

```json
{
  "data": {
    "answerQuestion": {
      "nextQuestionId": null,
      "remainingNumOfQuestion": null,
      "nextLevel": null,
      "achievedLevel": null,
      "examStatus": null,
      "message": "Exam status is COMPLETED, user can only answer the question for an IN_PROGRESS exam.",
      "error": true
    }
  }
}
```
