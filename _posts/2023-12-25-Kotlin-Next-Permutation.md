---
title:  "Kotlin 의 next_permutation (다음 순열)"
excerpt: "C++ 의 next_permutation 에 대응 되는 method 를 kotlin 으로 구현하고, 리스트에 대한 모든 순열을 가져올 수 있다"

categories:
  - blog
tags:
  - kotlin
  - Algorithm
last_modified_at: 2023-12-25T12:00:00-05:00

toc: true
toc_sticky: true
---

C++ 에는 다음과 같은 stdlib method 가 있습니다. 주로 알고리즘 문제를 풀며 사용되는데요.

[next_permutation](https://cplusplus.com/reference/algorithm/next_permutation/)

```cpp
// Rearranges the elements in the range [first,last) into the next lexicographically greater permutation.
template <class BidirectionalIterator>  bool next_permutation (BidirectionalIterator first, BidirectionalIterator last);
```

이름(Permutation)에서도 유추 할 수 있지만, 어떤 요소들의 집합에 대해 다음 순열을 가져오는 데에 사용합니다. 더 이상 다음 순열을 가져올 수 없다면, false 를 반환합니다.
*다음 순열* 이란, 리스트의 요소들을 배치하는 경우의 수(순열)를 사전 순으로 두었을 때, 다음 차례에 오는 순열를 의미합니다.
[0, 1, 2] 인 경우, 다음 순열은 [0, 2, 1] 그 다음은 [1, 0, 2] ... 이런 식으로 진행되어 마지막으로 내림차순으로 정렬된 [2, 1, 0] 에 다다를 수 있습니다.

Kotlin 에서는 다음 순열을 구하는 next_permutation 같은 함수를 제공해주지 않습니다. 따라서, 직접 구현해서 사용해야 하며 이를 위한 여러 가지 구현 방법이 있습니다. 다음의 소스코드는 그 중 하나의 구현 방식 예제입니다.

```kotlin

fun <T : Comparable<T>> nextPermutation(array: Array<T>): Boolean {
    // 1. 뒤쪽부터 탐색하며, 교환 위치(i - 1) 찾기: 처음으로 a[i - 1] < a[i]를 만족하는 위치
    val i = (array.size - 1 downTo 1).find { array[it - 1] < array[it] } ?: return false

    // 2. 뒤쪽부터 탐색하며, 교환 위치(i - 1)와 교환할 큰 수 찾기: 처음으로 a[i - 1] < a[j]를 만족하는 위치
    val j = (array.size - 1 downTo i).find { array[it] > array[i - 1] }!!

    // 3. i - 1 위치값과 j 위치값 교환
    array.swap(i - 1, j)

    // 4. i부터 맨 끝까지 순서 뒤집기(오름차순 정렬)
    array.reverse(i, array.size)

    return true
}

// Swap function for two elements of an array
fun <T> Array<T>.swap(i: Int, j: Int) {
    val temp = this[i]
    this[i] = this[j]
    this[j] = temp
}

// Reverse function for a portion of an array
fun <T> Array<T>.reverse(fromIndex: Int, toIndex: Int) {
    val subList = this.sliceArray(fromIndex until toIndex)
    for (i in fromIndex until toIndex) {
        this[i] = subList[toIndex - i - 1]
    }
}
```

흔하게 사용되는 예제 중 하나로서, 요소들을 배치할 수 있는 모든 경우의 수에 대해 계산을 하고자 할 때, 이 `nextPermutation` 을 사용하게 되는데요.
다음과 같이 사용할 수 있습니다.

```kotlin
val myArray = arrayOf(1, 2, 3, 4)

// 모든 경우의 수를 탐색하기 위해, 배열은 오름차순으로 정렬되어 있어야 합니다. (사전순으로 맨 앞)
myArray.sort() 

do {
    println(calculateSomething(myArray))
} while (nextPermutation(myArray))
```

### 증명

nextPermutation 의 각 단계별로 무슨 일이 일어나는지 살펴보겠습니다:

1. **교환 위치 찾기**: 배열의 끝에서부터 시작하여, 처음으로 `array[i - 1] < array[i]`를 만족하는 위치 `i`를 찾습니다. 이 위치는 '증가하는' 순서가 깨지는 지점이며, 여기서부터 순열을 변형하여 다음 순열을 찾을 수 있습니다. 만약 이러한 위치가 없다면, 배열이 내림차순으로 정렬된 것이며, 이는 가능한 순열 중 마지막 순열임을 의미하므로 false를 반환하고 함수를 종료합니다.

2. **교환할 큰 수 찾기**: 다시 배열의 끝에서 `i`까지 탐색을 하면서 `array[j] > array[i - 1]`을 만족하는 가장 먼저 만나는 `j`를 찾습니다. 이는 `array[i - 1]`과 교환하여 순열에서 다음으로 큰 수를 만들 수 있는 위치입니다.

3. **교환 실행**: `array[i - 1]`과 `array[j]`를 교환합니다. 이 교환은 순열의 '사전순' 다음을 만들기 위한 첫 번째 단계입니다.

4. **순서 뒤집기**: `i`부터 배열의 끝까지의 요소들을 뒤집습니다(오름차순 정렬). 이 단계는 순열의 뒷부분을 가능한 가장 작은 순열로 만듭니다. 교환으로 인해 `i - 1` 위치의 값이 증가했으므로, 남은 부분은 최소화되어야 전체 순열이 사전순으로 다음 순열이 됩니다.
