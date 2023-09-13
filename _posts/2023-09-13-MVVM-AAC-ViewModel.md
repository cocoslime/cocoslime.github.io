---
title:  "[Android] MVVM의 ViewModel과 AAC(Jetpack)의 ViewModel"
excerpt: ""

categories:
  - blog
tags:
  - Android
  - MVVM
  - AAC
  - Jetpack
  - ViewModel
last_modified_at: 2023-09-13T16:00:00-05:00

toc: true
toc_sticky: true
---

Android Architecture Components(AAC)의 ViewModel 과 MVVM 패턴의 ViewModel 의 차이점을 알아봅니다. 몇몇 블로그에서는 둘이 전혀 다른 개념이라고 설명하고 있지만, 이름뿐만 아니라 구현 측면에서는 비슷한 목적을 가지고 있다고도 볼 수 있습니다.

## AAC의 ViewModel
- 주요 목적은 View의 데이터를 저장하여 화면 회전 등 처럼 구성 변경이 일어날 때 데이터를 유지하는 것
- Lifecycle 을 가지고 있으며, 소멸될 때 onCleared 가 호출

> Activity의 화면 회전 시 ViewModel 의 Lifecycle (https://developer.android.com/static/images/topic/libraries/architecture/viewmodel-lifecycle.png)
<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/2023-09-13/ViewModel-Lifecycle.png" alt="qry" >
</p>

- [Android Jetpack](https://developer.android.com/jetpack?hl=ko)의 구성요소로서, Compose 와 Coroutine 를 지원하는 기능
- 화면 단위의 state holder 로 사용하여야 합니다.
	- chip group 이나 form 같이 재사용 될 수 있는 UI 구성요소는 ViewModel을 사용할 수 없습니다.
	- Activity 혹은 Fragment 마다 하나의 ViewModel 을 사용합니다
- Android 공식 문서에서 언급되는 `ViewModel` 은 AAC의 ViewModel 클래스입니다.

## MVVM의 ViewModel

> Microsoft의 MVVM 패턴 도식화 (https://learn.microsoft.com/ko-kr/dotnet/architecture/maui/mvvm)
<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/2023-09-13/MVVM.png" alt="qry" >
</p>

- ViewModel은 View의 데이터와 View의 액션에 의한 메소드를 구현합니다. ViewModel 은 Model 의 데이터를 View 에서 표현 가능하도록 가공하여 전달하는 역할을 합니다.
- View는 ViewModel의 상태를 옵저빙하고, 상태변화에 따라 UI를 변경합니다.
- View 와 ViewModel 은 1:N의 관계이며, 하나의 View 를 여러 개의 영역으로 나누어 관리하도록여러 개의 ViewModel 을 구현할 수 있습니다.

## 공통점
- ViewModel 에서는 View 를 알지 못하며, View 에 대한 Context를 가지고 있지 않습니다.
- View의 데이터를 관리한다는 점이 동일합니다.
- AAC 의 ViewModel 클래스를 통해 MVVM 패턴의 ViewModel 을 구현하는 것도 가능합니다.