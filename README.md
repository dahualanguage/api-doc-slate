<p align="center">
  <img src="https://raw.githubusercontent.com/slatedocs/img/main/logo-slate.png" alt="Slate: API Documentation Generator" width="226">
  <br>
  <a href="https://github.com/slatedocs/slate/actions?query=workflow%3ABuild+branch%3Amain"><img src="https://github.com/slatedocs/slate/workflows/Build/badge.svg?branch=main" alt="Build Status"></a>
  <a href="https://hub.docker.com/r/slatedocs/slate"><img src="https://img.shields.io/docker/v/slatedocs/slate?sort=semver" alt="Docker Version" /></a>
</p>

<p align="center">Slate helps you create beautiful, intelligent, responsive API documentation.</p>

<p align="center"><img src="https://raw.githubusercontent.com/slatedocs/img/main/screenshot-slate.png" width=700 alt="Screenshot of Example Documentation created with Slate"></p>

<p align="center"><em>The example above was created with Slate. Check it out at <a href="https://slatedocs.github.io/slate">slatedocs.github.io/slate</a>.</em></p>

## Features

- **Clean, intuitive design** — With Slate, the description of your API is on the left side of your documentation, and all the code examples are on the right side. Inspired by [Stripe's](https://stripe.com/docs/api) and [PayPal's](https://developer.paypal.com/webapps/developer/docs/api/) API docs. Slate is responsive, so it looks great on tablets, phones, and even in print.

- **Everything on a single page** — Gone are the days when your users had to search through a million pages to find what they wanted. Slate puts the entire documentation on a single page. We haven't sacrificed linkability, though. As you scroll, your browser's hash will update to the nearest header, so linking to a particular point in the documentation is still natural and easy.

- **Slate is just Markdown** — When you write docs with Slate, you're just writing Markdown, which makes it simple to edit and understand. Everything is written in Markdown — even the code samples are just Markdown code blocks.

- **Write code samples in multiple languages** — If your API has bindings in multiple programming languages, you can easily put in tabs to switch between them. In your document, you'll distinguish different languages by specifying the language name at the top of each code block, just like with GitHub Flavored Markdown.

- **Out-of-the-box syntax highlighting** for [over 100 languages](https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers), no configuration required.

- **Automatic, smoothly scrolling table of contents** on the far left of the page. As you scroll, it displays your current position in the document. It's fast, too. We're using Slate at TripIt to build documentation for our new API, where our table of contents has over 180 entries. We've made sure that the performance remains excellent, even for larger documents.

- **Let your users update your documentation for you** — By default, your Slate-generated documentation is hosted in a public GitHub repository. Not only does this mean you get free hosting for your docs with GitHub Pages, but it also makes it simple for other developers to make pull requests to your docs if they find typos or other problems. Of course, if you don't want to use GitHub, you're also welcome to host your docs elsewhere.

- **RTL Support** Full right-to-left layout for RTL languages such as Arabic, Persian (Farsi), Hebrew etc.

Getting started with Slate is super easy! Simply press the green "use this template" button above and follow the instructions below. Or, if you'd like to check out what Slate is capable of, take a look at the [sample docs](https://slatedocs.github.io/slate/).

## Getting Started with Slate

To get started with Slate, please check out the [Getting Started](https://github.com/slatedocs/slate/wiki#getting-started)
section in our [wiki](https://github.com/slatedocs/slate/wiki).

We support running Slate in three different ways:

- [Natively](https://github.com/slatedocs/slate/wiki/Using-Slate-Natively)
- [Using Vagrant](https://github.com/slatedocs/slate/wiki/Using-Slate-in-Vagrant)
- [Using Docker](https://github.com/slatedocs/slate/wiki/Using-Slate-in-Docker)

## Companies Using Slate

- [NASA](https://api.nasa.gov)
- [Sony](http://developers.cimediacloud.com)
- [Best Buy](https://bestbuyapis.github.io/api-documentation/)
- [Travis-CI](https://docs.travis-ci.com/api/)
- [Greenhouse](https://developers.greenhouse.io/harvest.html)
- [WooCommerce](http://woocommerce.github.io/woocommerce-rest-api-docs/)
- [Dwolla](https://docs.dwolla.com/)
- [Clearbit](https://clearbit.com/docs)
- [Coinbase](https://developers.coinbase.com/api)
- [Parrot Drones](http://developer.parrot.com/docs/bebop/)
- [CoinAPI](https://docs.coinapi.io/)

You can view more in [the list on the wiki](https://github.com/slatedocs/slate/wiki/Slate-in-the-Wild).

## Questions? Need Help? Found a bug?

If you've got questions about setup, deploying, special feature implementation in your fork, or just want to chat with the developer, please feel free to [start a thread in our Discussions tab](https://github.com/slatedocs/slate/discussions)!

Found a bug with upstream Slate? Go ahead and [submit an issue](https://github.com/slatedocs/slate/issues). And, of course, feel free to submit pull requests with bug fixes or changes to the `dev` branch.

## Contributors

Slate was built by [Robert Lord](https://lord.io) while at [TripIt](https://www.tripit.com/). The project is now maintained by [Matthew Peveler](https://github.com/MasterOdin) and [Mike Ralphson](https://github.com/MikeRalphson).

Thanks to the following people who have submitted major pull requests:

- [@chrissrogers](https://github.com/chrissrogers)
- [@bootstraponline](https://github.com/bootstraponline)
- [@realityking](https://github.com/realityking)
- [@cvkef](https://github.com/cvkef)

Also, thanks to [Sauce Labs](http://saucelabs.com) for sponsoring the development of the responsive styles.

# Deploy

```
ssh -T git@github-dahualanguage; eval '$(ssh-agent -s)'; ssh-add ~/.ssh/id_rsa_github_dahua; ./deploy.sh
```

# Push

```
ssh -T git@github-dahualanguage; eval '$(ssh-agent -s)'; ssh-add ~/.ssh/id_rsa_github_dahua; git push
```

# Run locally and visit http://localhost:4567/#introduction

```
bundle exec middleman server
```

PlantText

## 考試 API 流程圖

```
@startuml

title 分級測驗考試流程

participant "冠富" as Client
participant "AWS API" as AWS

note over Client: 1. 開始一個新的測驗
Client -> AWS: startExam(accountId)
note right: 幫accountId創建一個exam
AWS --> Client: accountId, sk, nextQuestionId

note over Client: 2. 取得考題
Client -> AWS: getQuestions(nextQuestionId)
note right: 使用nextQuestionId獲得問題
AWS --> Client: question

note over Client: 3. 回答題目
Client -> AWS: answerQuestion(accountId, sk, questionId, answer)
note right: 判斷答案是否正確，並會傳下一題的ID
AWS --> Client: nextQuestionId, examStatus

note over Client: 重複第2,3步，直到examStatus等於COMPLETED

note over Client: 4. 回顧測驗結果，或是測驗歷史紀錄
Client -> AWS: getExams(accountId, sk)
note right: 回傳測驗結果
AWS --> Client: Exam

@enduml
```

## 文章口語練習 API 流程圖

```
@startuml

title 分級測驗考試流程

participant "冠富" as Client
participant "API" as API
participant "S3" as S3
participant "OpenAI" as OpenAI

note over Client: 1. 開始一個新的口語練習
Client -> API: startExam(article, examType: ARTICLE_SPEECH_PRACTICE)
note right: 創建一個口語練習
API --> Client: accountId, sk

note over Client: 2. 取得signedUrl
Client -> API: getUploadVoiceSignedURL(args...)
API -> S3: args...
note right: S3生成臨時的signedUrl
note left: API傳args給S3
S3 --> API: signedUrl
note right: S3回傳signedUrl給API
API --> Client: signedUrl

note over Client: 3. 上傳音檔
Client -> S3: 直接上傳mp3檔案到S3的signedUrl
S3 --> Client: success

S3 -[#blue]-> OpenAI : mp3
note right: OpenAI轉成文字檔
note left: 觸發S3 streaming
OpenAI -[#blue]-> API: 口說文字檔
note left: 儲存口說文字檔
API -[#blue]-> OpenAI: 口說文字檔
note right: 分析口說文字檔
OpenAI -[#blue]-> API: 分析結果
note left: 儲存分析結果

note over Client: 重複第2,3步，完成所有練習

note over Client: 4. 回顧測驗結果，或是測驗歷史紀錄
Client -> API: exams(accountId, sk)
API --> Client: Exam
note left: 回傳測驗結果

@enduml
```
