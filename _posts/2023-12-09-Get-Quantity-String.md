---
title:  "Android 에서 단수형, 복수형의 String resource"
excerpt: "단수형, 복수형의 String resource 를 구분해서 가지고 올 수 있다."

categories:
  - blog
tags:
  - Android
last_modified_at: 2023-12-09T12:00:00-05:00

toc: true
toc_sticky: true
---


String resource 를 가지고 올 때, 단수/복수에 따라 다른 String resource 를 가져올 수 있습니다.

먼저, string.xml resource 파일에 plurals 태그를 활용하여 Plurals 리소스를 정의합니다.

```xml
<plurals name="days_count">  
    <item quantity="one">every day</item>  
    <item quantity="other">every %d days</item>  
</plurals>
```

위에 정의한 plurals 리소스를 다음과 같이 `getQuantityString()` 를 사용해서 가져올 수 있습니다.

```kotlin
    // every day  
    resources.getQuantityString(R.plurals.days_count, 1)  
  
    // every 2 days   
	resources.getQuantityString(R.plurals.days_count, 2, 2)
```

getQuantityString 함수는 두 가지의 오버로드가 있습니다.

```java
@NonNull  
public String getQuantityString(@PluralsRes int id, int quantity)

@NonNull  
public String getQuantityString(@PluralsRes int id, int quantity, Object... formatArgs)
```

`quantity` 파라미터는 plurals 의 아이템을 선택하는 역할을 합니다. 따라서, string 의 파라미터에 넣기 위해서는 함수 호출 코드를 봤을 때 quantity 에 해당하는 변수가 두 번 쓰여지는 형태가 될 수 있습니다.
다른 string resource 와 마찬가지로 파라미터(`%1$d, %2$s`)가 있다면, 두번째 오버로드를 사용하여 formatArgs 에 파라미터에 해당하는 값들을 넘겨주어야 합니다. 

## Compose

Compose UI 인 경우, Composable 내에서 다음과 같이 `pluralStringResource` 를 사용하는 것도 가능합니다.

```kotlin
val text: String = pluralStringResource(  
    R.plurals.days_count, daysCount, daysCount  
)
```

pluralStringResource 의 내부 구현도 getQuantityString 을 사용하는 것으로 구현되어 있습니다.

```kotlin
/**
 * Load a plurals resource with provided format arguments.
 *
 * @param id the resource identifier
 * @param count the count
 * @param formatArgs arguments used in the format string
 * @return the pluralized string data associated with the resource
 */
@Composable
@ReadOnlyComposable
fun pluralStringResource(@PluralsRes id: Int, count: Int, vararg formatArgs: Any): String {
    val resources = resources()
    return resources.getQuantityString(id, count, *formatArgs)
}
```