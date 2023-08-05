---
title:  "[Android] CountDownTimer - 타이머 동작 구현"
excerpt: "CountDownTimer를 통해 간단히 타이머 동작을 구현해보자"

categories:
  - develop
tags:
  - Android
last_modified_at: 2022-03-01T16:00:00-05:00

toc: false
toc_sticky: false
---

# CountDownTimer

Android의 CountDownTimer는 말그대로 카운트 다운 작업을 쉽게 구현하도록 해줍니다. 카운트 다운이 시작되고 종료될 때까지의 총 시간과 onTick 콜백 함수를 실행할 일정 시간 간격을 설정하여 객체를 생성합니다.

# Code Snippet

```kotlin
const val COUNTDOWN_TIME = 60000L // 60 Seconds
const val ONE_SECOND = 1000L // 1 Second

val timer = object : CountDownTimer (COUNTDOWN_TIME, ONE_SECOND) {
		// 익명 클래스의 객체로 생성
		// 총 카운트 다운 시간은 60초, 1초마다 onTick 함수가 호출된다.
		override fun onTick(millisUntilFinished: Long) {
        // Do Something every 1 seconds
    }

    override fun onFinish() {
				// Do someting when finish
    }
}

timer.start() // 이때부터 60초 이후에 onFinish 호출
// timer.cancel() // maually 취소 가능
```