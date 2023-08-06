---
title:  "안드로이드 웹 뷰에서 특정 도메인 url 에 대한 동작 구현"
excerpt: "안드로이드 WebView 특정 도메인의 URL에 대한 동작을 단순히 웹 페이지를 열지 않고, 앱에서 처리해야 하도록 구현"

categories:
  - develop
tags:
  - Android
  - WebView
last_modified_at: 2023-08-06T22:00:00+09:00

toc: true
toc_sticky: true
---

# 안드로이드 웹 뷰에서 특정 도메인 url 에 대한 동작 구현

안드로이드 앱에서 `WebView`를 사용하여 웹 페이지를 표시하는 경우, 특정 도메인의 URL에 대한 동작을 단순히 웹 페이지를 열지 않고, 앱에서 처리해야 하는 경우가 있습니다. 이를 위해 `WebViewClient`의 `shouldOverrideUrlLoading` 메서드를 사용할 수 있습니다. 

### WebViewClient를 상속하는 커스텀 클래스 만들기
먼저, `WebViewClient`를 상속하여 `MyWebViewClient` 클래스를 만듭니다.
```kotlin
class MyWebViewClient : WebViewClient() {
        override fun shouldOverrideUrlLoading(view: WebView, request: WebResourceRequest): Boolean {
            val url = request.url.toString()
            // 특정 도메인의 URL 처리하기
            if (url.startsWith("http://example.com")) {
                // 이곳에 해당 URL을 처리하는 로직을 작성한다.
                // 예를 들면, 특정 화면으로 이동하거나, 다른 앱을 호출하는 등의 동작을 수행할 수 있다.
                return true // URL 로딩을 앱에서 처리했음을 시스템에 알려줌
            }
            return false // 기본적인 웹 페이지 로딩 동작을 따른다.
        }
    }
```

### WebViewActivity 구현
액티비티의 WebView 에 만든 커스텀 WebViewClient 를 추가해줍니다. 

```kotlin
class WebViewActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_webview)

        val webView = binding.webView
        val url = "http://example.com" // 로딩할 웹 페이지 URL

        webView.webViewClient = MyWebViewClient()
    }

    private class MyWebViewClient : WebViewClient() {
        override fun shouldOverrideUrlLoading(view: WebView, request: WebResourceRequest): Boolean {
            val url = request.url.toString()
            // 특정 도메인의 URL 처리하기
            if (url.startsWith("http://example.com")) {
                // 이곳에 해당 URL을 처리하는 로직을 작성한다.
                // 예를 들면, 특정 화면으로 이동하거나, 다른 앱을 호출하는 등의 동작을 수행할 수 있다.
                return true // URL 로딩을 앱에서 처리했음을 시스템에 알려줌
            }
            return false // 기본적인 웹 페이지 로딩 동작을 따른다.
        }
    }
}
```

이제 WebView 에서 "http://example.com" 과 같은 특정 도메인의 URL을 로딩하면 기본적인 웹 페이지 로딩이 아닌 따로 정의한 구현으로 동작합니다. 이 방법을 응용하여 앱에 필요한 웹 페이지 인터페이스를 구현하거나 특정 도메인과의 연동을 수행할 수 있습니다.

```kotlin
webView.loadUrl("http://example.com")
```
