---
title: Dialogflow 在 Android 上的非官方串接實驗記錄
tags:
  - Android
  - DialogFlow
  - ChatBot
date: 2022-04-11
description: 本文記錄在 Android 專案中嘗試使用 Google Cloud Dialogflow Java SDK 串接聊天機器人的測試過程。內容包含初始化、訊息處理的實作方式，以及遇到的相容性問題與安全風險。這不是官方支援的做法，僅供開發實驗與研究參考。
---

> ⚠️ 本文記錄的是 Dialogflow 在 Android 上的測試性串接，並非官方支援或建議做法。若需穩定實作，建議改採 REST API 或透過後端 Proxy 呼叫。

## 前言

最近因為專案上的需求，需要在 Android 端上實作一個簡易的聊天機器人，而後端使用的技術是 Dialogflow API。爬了一堆文之後，卻發現 Android 端的實作說明不多，甚至連官方文件都爬不太到，因此整理一個目前測試下來可以運作的方式。

實作的內容主要參考自 [Medium: Android chatbot with Dialogflow](https://medium.com/@abhi007tyagi/android-chatbot-with-dialogflow-8c0dcc8d8018) 與 [Github: DialogflowChat](https://github.com/abhi007tyagi/DialogflowChat)。

> 本文章主要會說明 Android 端如何串接與實作，
> Dialogflow 的專案建置可以參考
> 這篇 [【技術文章】DialogFlow輕鬆建立屬於你的聊天機器人！](https://www.gaia.net/tc/news_detail/2/33/dialogflow)
> 我覺得寫的滿清楚的~

## 實作 (簡易對話)

### 一、環境建置

#### app's `build.gradle`

```groovy
android {
    packagingOptions {
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/DEPENDENCIES'
        exclude 'META-INF/INDEX.LIST'
    }
}


dependencies {
    // Java V2
    implementation 'com.google.cloud:google-cloud-dialogflow:2.2.0'
    
    // for Remote Procedure Call to avoid 
    // "No functional channel service provider found" error 
    // while creating SessionsClient
    implementation 'io.grpc:grpc-okhttp:1.31.1'
}
```

### 二、初始化

#### 建立 Credentials Inputstream

將從後台下載拿到的 JSON 檔放到任意路徑，我是將它放在 `assets` 資料夾中再讀進來。

那份 JSON 結構應該會是長這樣，

```json
{
  "type": "",
  "project_id": "",
  "private_key_id": "",
  "private_key": "",
  "client_email": "",
  "client_id": "",
  "auth_uri": "",
  "token_uri": "",
  "auth_provider_x509_cert_url": "",
  "client_x509_cert_url": ""
}
```

#### 使用憑證初始化，並建立一個 Session

Session 僅用來識別對話，它不代表連線成功，實際的對話還是要等你送出訊息後才開始。

```kotlin
private var sessionClient: SessionsClient? = null
private var session: SessionName? = null
private val uuid = UUID.randomUUID().toString()


fun initChatBot(inputStream: InputStream) {
    try {
        val credentials = GoogleCredentials.fromStream(inputStream) as ServiceAccountCredentials
        val projectId = credentials.projectId
        val sessionSettings = SessionsSettings.newBuilder()
            .setCredentialsProvider(FixedCredentialsProvider.create(credentials))
            .build()
        sessionClient = SessionsClient.create(sessionSettings)
        session = SessionName.of(projectId, uuid)
    } catch (e: Exception) {
        Log.e(TAG, "init: ", e)
    }
}
```

### 三、發送訊息

```kotlin
suspend fun sendMessage(input: String): DetectIntentResponse? {
    return withContext(Dispatchers.IO) {
        try {
            val textInput =
                TextInput.newBuilder().setText(input).setLanguageCode("zh-tw").build()
            val queryInput =
                QueryInput.newBuilder()
                    .setText(textInput)
                    .build()
            val detectIntentRequest =
                DetectIntentRequest.newBuilder()
                    .setSession(session.toString())
                    .setQueryInput(queryInput)
                    .build()
            sessionClient?.detectIntent(detectIntentRequest)
        } catch (e: Exception) {
            Log.e(TAG, "sendMessage: ", e)
            null
        }
    }
}
```

- `TextInput`: 純文字輸入。
- `QueryInput`: 可以是語音，也可以是純文字。
- `DetectIntentResponse`: 回傳訊息。

### 四、接收訊息

```kotlin
val response = sessionClient?.detectIntent(detectIntentRequest)
response?.let {
    val reply = it.queryResult.fulfillmentText
    Log.d(TAG, "sendMessage: $reply")
}
```

## 外部連結

### Medium/ iThome

- [Medium: How to use Dialogflow v2 java SDK with Kotlin](https://medium.com/@bldd97/how-to-use-dialogflow-v2-java-sdk-with-kotlin-c550363b80c7)
- [iThome: Day 25：「意圖」與「實體」的應用 -「Dialogflow」](https://ithelp.ithome.com.tw/articles/10226833)

### Github

- [googleapis/java-dialogflow](https://github.com/googleapis/java-dialogflow)
  - 部分 Samples 可以從這裡找
- [dialogflow/dialogflow-java-client-v2](https://github.com/dialogflow/dialogflow-java-client-v2)
- 別人整合的 Library [abhi007tyag/Android_Dialogflow_Chatbot_Library](https://github.com/abhi007tyagi/Android_Dialogflow_Chatbot_Library)
- [Issue: how to use in android? the dialogflow api V2](https://github.com/dialogflow/dialogflow-java-client-v2/issues/25)
- [dialogflow/dialogflow-android-client](https://github.com/dialogflow/dialogflow-android-client)
  - v1 舊版
  - 已被棄用，都建議使用下面的 [Google Cloud: Java client library](https://cloud.google.com/dialogflow/es/docs/reference/libraries/java)

### Doc

- [Google Cloud: Java client library](https://cloud.google.com/dialogflow/es/docs/reference/libraries/java)
  - 目前用的 `com.google.cloud:google-cloud-dialogflow:2.2.0` 就是這個
  - 不過文件卻寫 *Caution:The Java client library does not support Android.* :confused: 
  - QuickStart 裡有些資訊可以參考，不過也是要東拼西湊。
- [Google Cloud: Dialogflow API](https://cloud.google.com/dialogflow/es/docs/reference/rest/v2-overview)
- [Google Cloud: API Usage Overview](https://cloud.google.com/dialogflow/es/docs/reference/api-overview)
- [How to Call Google APIs: REST Edition](https://googleapis.github.io/HowToREST)
- [Dialogflow ES documentation](https://cloud.google.com/dialogflow/es/docs)
