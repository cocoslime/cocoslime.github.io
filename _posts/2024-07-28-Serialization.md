---
title:  "interface Serializable 과 Parcelable 그리고 @Serializable 과 @Parcelize"
excerpt: "interface Serializable 과 interface Parcelable 에 대해 알아본다. 그리고 kotlin 의 anotation @Serializable 과 @Parcelize 에 대해 알아본다."
categories:
  - blog
tags:
  - Android
  - Kotln
  - Serialization
last_modified_at: 2024-07-28T12:00:00-05:00

toc: true
toc_sticky: true
---

Serializable, Parcelable 그리고 @Serializable, @Parcelize 다 비슷한 이름을 가지고 있고, 모두 "직렬화"에 관련된 키워드이지만 각각의 역할 및 세부 동작에 대해 알아보았다.

## 직렬화란?
네 가지 모두 직렬화에 관련된 키워드이다. 직렬화를 간단히 말하면, 객체를 파일 혹은 네트워크 등 외부 환경에 전송/수신 하기 위해 필요한 과정. 전송 가능한 바이트 스트림으로 만들어준다.

## interface Serializable

- 자바(코틀린 X, 안드로이드 X)의 표준 인터페이스이다: java.io.Serializable
- 구현해야할 메소드가 없다. 어떠한 메소드도 가지지 않는 "마커 인터페이스" 이다.
- 이 인터페이스를 선언하기만 해도, JVM 에서 해당 객체를 저장하거나, 다른 서버로 저장할 수 있다. ObjectOutputStream 을 사용해서 객체를 직렬화 할 수 있습니다.

```kotlin
data class Member(
	val id: String
): Serializable

oos.writeObject(Member("ID")) // Serializable 이 없으면 에러
```

- 클래스가 Serializable 을 구현(Implements) 했지만, Serializable 하지 않은 필드가 있는 경우는 직렬화 불가능 하다.
- 상위 클래스가 직렬화 되지 않은 상태에서 서브 클래스가 Serializable 을 구현하더라도 상위 클래스의 필드가 직렬화 되지 않는다.
- 위의 두 조건을 위배되지 않는 Serializable 을 구현한 클래스의 객체는 Java 의 리플렉션을 이용해 처리된다.
	- 런타임에 많은 오버헤드가 발생하고 성능 이슈가 있다.
- Java 의 고유한 바이너리 형식으로 직렬화 된다.

## interface Parcelable

- 안드로이드 SDK 의 인터페이스. 안드로이드만을 위한 직렬화를 지원: android.os.Parcelable
- implements 해야 하는 코드가 많다(보일러플레이트 코드)

```kotlin
import android.os.Parcel
import android.os.Parcelable

data class Member(
    val id: Long,
    val name: String,
    val friends: List<Member>
) : Parcelable {
    constructor(parcel: Parcel) : this(
        parcel.readLong(),
        parcel.readString() ?: "",
        parcel.createTypedArrayList(CREATOR) ?: emptyList()
    )

    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeLong(id)
        parcel.writeString(name)
        parcel.writeTypedList(friends)
    }

    override fun describeContents(): Int {
        return 0
    }

    companion object CREATOR : Parcelable.Creator<Member> {
        override fun createFromParcel(parcel: Parcel): Member {
            return Member(parcel)
        }

        override fun newArray(size: Int): Array<Member?> {
            return arrayOfNulls(size)
        }
    }
}

```

- Parcelable 을 구현하여, 안드로이드 프레임워크의 API 중 Parcelable 을 필요로 하는 곳에 이 클래스를 인자로 넘길수 있게 된다. 주로 Intent, Bundle 을 이용해서 서로 다른 컴포넌트(ex. Activity) 간의 데이터 전달에 사용된다.
- Android에서 성능을 위해 최적화된 방식으로, Serializable 보다 더 나은 성능을 제공한다.

## Annotation @Serializable

- https://kotlinlang.org/docs/serialization.html
- kotlin 이 제공하는 직렬화 라이브러리: kotlinx.serialization.Serializable
- JSON 혹은 protocol buffer 같은 일반적인 데이터 직렬화 방식으로 만들어준다.
	- 주로 JSON 직렬화를 위해 사용된다.
- `Json.encodeToString, Json.decodeFromString<T>` 메소드를 통해 직렬화/역직렬화가 가능하다.

```kotlin
import kotlinx.serialization.Serializable
import kotlinx.serialization.json.Json
import kotlinx.serialization.encodeToString

@Serializable
data class Data(val a: Int, val b: String)

fun main() {
   val json = Json.encodeToString(Data(42, "str"))
}
```

- Gson, Moshi, Jackson 등 비슷한 역할을 하는 라이브러리가 있다.

## Annotation @Parcelize

- https://developer.android.com/kotlin/parcelize
- kotlin-parcelize 플러그인이 @Parcelize 어노테이션이 붙은 클래스에 대해 위에서 설명한 Parcelable 의 메소드를 **알아서 구현**해준다. 

```groovy
plugins {
    id("kotlin-parcelize")
}
```

- 컴파일 타임에 바이트 코드 변조를 통해 이루어지기 때문에 메소드를 추가해야 하거나 런타임 시 오버헤드 비용이 발생하지 않는다.
- Parcelable interface 를 구현하는 클래스 정의에 `@Parcelize` 어노테이션을 추가한다

```kotlin
import kotlinx.parcelize.Parcelize

@Parcelize
class User(val firstName: String, val lastName: String, val age: Int): Parcelable
```

- Parcelize를 사용하려면 모든 직렬화된 프로퍼티가 기본 생성자에 선언되어야 한다. 또한 기본 생성자 매개변수 중 일부가 프로퍼티가 아닌 경우 @Parcelize를 적용할 수 없다.
- 직렬화하는 코드를 커스텀하고 싶다면, companion class 에 다음을 추가하면 된다.

```kotlin
@Parcelize
data class User(val firstName: String, val lastName: String, val age: Int) : Parcelable {
    private companion object : Parceler<User> {
        override fun User.write(parcel: Parcel, flags: Int) {
            // Custom write implementation
        }

        override fun create(parcel: Parcel): User {
            // Custom read implementation
        }
    }
}
```