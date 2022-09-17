---
title:  "[Kotlin] Kotlin 에서의 Lower bound 구현"
excerpt: ""

categories:
  - Blog
tags:
  - Kotlin
  - TIL

date: 2022-09-17

toc: true
toc_sticky: true
---

C++ 의 standard library 에서 제공하는 [lower_bound](https://en.cppreference.com/w/cpp/algorithm/lower_bound) 를 kotlin 에 맞게 구현한 코드이다. 

```kotlin
fun <T> lowerBound(elements: Array<Comparable<T>>, low: Int, high: Int, target: T) : Int {
    var currentLow = low
    var currentHigh = high
    var currentMid: Int

    while(currentLow < currentHigh) {
        currentMid = (currentLow + currentHigh) shr 1
        if (currentMid == high) return high

        if (elements[currentMid] < target) currentLow = currentMid + 1
        else currentHigh = currentMid
    }
    return currentLow
}
```