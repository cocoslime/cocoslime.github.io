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