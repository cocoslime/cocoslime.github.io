---
title:  "Kotlin Flow 의 flatMapMerge"
excerpt: "병렬로 데이터 변환을 처리하는 flatMapMerge 에 대해 알아보자"

categories:
  - blog
tags:
  - Kotlin
  - Flow
last_modified_at: 2023-02-09T15:00:00+09:00

toc: true
toc_sticky: true
---

Kotlin Flow 에는 세 가지의 flatMap 이 있다.

flatMapConcat, flatMapLatest, flatMapMerge

이 중에 flatMapMerge는 데이터 변환 동작을 병렬적으로 처리하는 하나의 Flow를 반환한다. 

```kotlin
@FlowPreview
fun <T, R> Flow<T>.flatMapMerge(concurrency: Int = DEFAULT_CONCURRENCY, transform: suspend (T) -> Flow<R>): Flow<R>
```

이 함수는 발행되는 데이터에 대해 순차적으로 `transform` 을 호출하고, 최대 `concurrency` 만큼 병렬적으로 수집되는 flow 를 병합한다. `concurrency` 는 동시에 collect 될 수 있는 flow의 수를 제어하는데, 보통의 경우는 `DEFAULT_CONCURRENCY` 로 사용하며, 16으로 세팅되어 있다고 한다. (JVM에 의해 변경될 수 있음)

여기서 `concurrency`가 1이면, flatMapConcat 과 동일하다.

이 연산은 의외로 자주 사용되지는 않는다. 보통의 경우는 map 에서 suspend function 을 동작하는 것으로도 충분하고, 선형적으로 변형하는 것이 이해하기 더 쉽기 때문이다.

<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/2023-02-09-flatMapMerge.png" alt="qry" >
</p>


`flatMapMerge`의 동작은 내부적으로 다음 두 함수를 호출 한 것과 동일한 동작을 한다.

```kotlin
map(transform).flattenMerge(concurrency)
```

flattenMerge 는 병렬적으로 수집되는 여러 개의 flow 로 부터 하나의 flow 를 만든다.

```kotlin
@FlowPreview
fun <T> Flow<Flow<T>>.flattenMerge(concurrency: Int = DEFAULT_CONCURRENCY): Flow<T>
```

참고 페이지

[flatMapMerge](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-merge.html)
[flattenMerge](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flatten-merge.html)