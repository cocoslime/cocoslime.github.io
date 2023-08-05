---
title:  "[Kotlin] Coroutines Basics"
excerpt: "Kotlin Coroutines 기초 개념 및 사용법"

categories:
  - develop
tags:
  - Coroutines
  - Kotlin
  - TIL
date: 2022-03-08

toc: true
toc_sticky: true
---

<details>
<summary>문서 이력</summary>
<div markdown="1">
- 2022.03.08. 포스팅
- 2022.04.11. 내용 추가
</div>
</details>
<br>

이 글은 다음 링크의 글을 참고하여 한글로 정리한 것입니다.

* [Kotlin 공식 Coroutines 가이드](https://kotlinlang.org/docs/coroutines-guide.html)
* [Kotlin 공식 Coroutines Tutorial](https://play.kotlinlang.org/hands-on/Introduction%20to%20Coroutines%20and%20Channels/01_Introduction)
* [Kotlin World - Coroutines](https://kotlinworld.com/category/Coroutines/Coroutine%20Basics)

## Coroutines Basic

### JVM의 Thread 구조와 기존의 접근 방식

하나의 프로세스에는 여러 스레드가 있을 수 있고, 각 스레드는 독립적으로 작업을 수행할 수 있다. JVM 프로세스는 스레드 중 하나를 Main Thread로 가지며, Main Thread가 종료되면 다른 스레드와 JVM 프로세스도 강제 종료된다. Android 에서는 이 Main Thread는 UI Thread라고도 불리며, 사용자와의 인터펙션을 담당하는 Thread 이다. 만약, 이 Thread 가 부하가 큰 작업을 수행하거나 특정한 이유로 Blocking 되는 경우 앱은 멈춘 것 처럼 보여지며, 일정 시간 이상 지속될 경우 앱은 강제로 종료된다. **(ANR : Application Not Responding. 응답 없음)** 따라서, 안드로이드 개발자는 **비동기 프로그래밍등의 방법을 이용해 메인 쓰레드 에서 많은 부하를 받는 작업을 수행하지 않도록 구현**할 필요가 있다.

기존의 멀티 스레드를 작동시키는 방법은 Runnable 인터페이스 사용, AsyncTask(Deprecated), ExecuterService, RxJava 등이 있습니다. 이 방법들의 한계점은 작업의 단위가 Thread 인 것인데, Thread는 생성 비용 및 스위칭 비용이 비싸며, 한 Thread 가 다른 Thread의 작업을 기다리며 Blocking 될 경우 자원은 낭비됩니다. 

### Coroutine

*Coroutine*은 하나의 중단 가능한 연산의 객체 입니다. 개념적으로는 쓰레드와 비슷합니다. 하나의 Thread 내부에 여러 작은 스레드인 코루틴이 있을 수 있고, 이 중 하나가 중단되더라도 다른 코루틴이 실행되어 Thread 자원의 낭비를 줄입니다.

코루틴은 다른 나머지 코드와 동시에 작동시킬 코드 블록을 필요로 합니다. 그러나, 코루틴은 어떤 특정 쓰레드에 묶여있지 않고, 한 쓰레드에서 실행을 중단 했다가 다른 스레드에서 다시 시작할 수 있습니다.

Light-weight threads 라고도 불리는 Coroutines 에서, 우리는 thread 위에서 돌아가는 코드와 비슷하게 Coroutines에서 코드를 돌아가게 할 수 있습니다. 그러나 Coroutines는 실제 사용 환경에서 thread와의 많은 중요한 차이점을 가지고 있습니다.

다음은 아주 기본적인 코루틴 작동 코드 입니다.

```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { // launch a new coroutine and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello") // main coroutine continues while a previous one is delayed
}

// Output
"Hello"
"World!"
```

`launch`는 코루틴 빌더입니다. 나머지 코드와 동시에 독립적으로 돌아가는 새로운 코루틴을 실행 시킵니다. 그래서 아래에 있는 “Hello”가 먼저 출력되게 됩니다.

`delay`는 특별한 중단 함수 입니다. 이 함수는 코루틴을 특정 시간동안 중단 시킵니다. 즉, 코루틴은 **suspendable (중단 가능)** 합니다. Coroutine이 “중단”된 상태라는 것은, 관련된 연산이 멈추었다는 것이고, 쓰레드에서 제거되어 메모리에 머물러 있는 상태를 뜻합니다. 반면에, 해당 쓰레드는 다른 활동, 작업 등을 처리할 수 있도록 비어지게 됩니다. (UI 쓰레드에서 코루틴으로 작업을 수행하더라도 UI 가 freezing 되지 않고 반응 할 수 있는 이유) 

<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/2022-03-08/SuspensionProcess.gif" alt="qry" >
</p>

연산이 재개될 준비가 되었다면, 다시 쓰레드로 복귀합니다. (꼭 이전의 쓰레드와 같을 필요는 없습니다.) 예를들어, 네트워크 요청 작업의 경우에는 요청을 처리하는 도중에 이 연산은 “suspended” 되어, 현재 돌아가고 있는 thread를 release합니다. 네트워크 요청이 응답 데이터를 가지고 오면, 연산은 재개됩니다.

아래에서 설명할 `runBlocking` 또한 코루틴 빌더입니다. 이 특이한 빌더는 일반 영역인 `fun main()` 과 `runBlocking {...}` 내부의 코루틴 코드를 이어주는 다리 역할을 합니다.

## Extract Function (suspend function)

위의 코드 예제에서 launch 내부의 코드 블럭을 함수로 만들어보려고 합니다. 이때, 우리는 suspend 함수를 만들어야 합니다. 함수 정의에 **`suspend`** 키워드를 사용하여 suspend function을 정의합니다. suspend function은 코루틴 내부에서 일반 함수를 호출 하듯이 호출 가능하고, 코루틴의 실행을 중단 시키는 다른 suspending function(여기서는 delay 함수)를 사용할 수 있습니다.

```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { doWorld() }
    println("Hello")
}

// this is your first suspending function
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```

suspend 함수는 아무데서나 호출될 수 없고, `coroutine` 혹은 다른 suspend 함수 안에서 호출 되어야 합니다. 

## Job Lazy Start

Job을 생성하는 launch 메소드에 CoroutineStart.LAZY 인자를 넘겨 대기 상태의 Job을 생성합니다.
```
val job = CoroutineScope(Disspatchers.Main).launch(start = CoroutineStart.LAZY) {
    println("Job done")
}
```

Lazy 하게 생성된 Job은 `start()` 혹은 `join()` 으로 실행 가능하다. 

* start() : 중단 없이 실행
* join() : Job이 완료될 때까지 일시 중단

## Concurrency (async, Deferred, Dispatchers)

Kotlin의 코루틴은 쓰레드에 비해 매우 저렴합니다. 우리는 새로운 비동기적인 연산이 필요할 때 마다, 새로운 코루틴을 생성할 수 있습니다.

새로운 코루틴을 생성하기 위해 우리는 main “coroutine builders” 중 하나를 사용할 수 있습니다: `launch`, `async`, 그리고 `runBlocking`. 다른 라이브러리들은 추가적인 코루틴 빌더를 정의할 수 있습니다.

async는 새로운 코루틴을 생성하고 `Deferred` 객체를 반환합니다. Deferred (연기됨) 객체는 미래의 어떠한 시점에 결과를 받을 수 있음을 약속하고 그 순간까지 연산 결과 반환이 연기됩니다.

async와 launch의 차이점은 launch는 특정 결과를 반환하는 것이 기대되지 않습니다. launch 함수 자체는 코루틴을 의미하는 `Job` 객체를 반환하는데, 우리는 쓰레드를 사용할 때 처럼 Job.join()을 호출하여 작업이 끝날때 까지 기다릴 수 있습니다.

Deferred는 Job을 확장한 제네릭 타입입니다. async 호출은 넘겨지는 람다 인자에 따라 반환하는 객체 타입이 결정되어 Deferred<타입> 을 반환 합니다. Deferred 객체로 받는 코루틴의 결과를 얻기 위해 우리는 `await()`를 호출할 수 있습니다. 물론, await가 호출되어 결과를 기다리는 코루틴은 중단 됩니다.

```kotlin
fun main() = runBlocking {
    val deferred: Deferred<Int> = async {
        loadData()
    }
    println("waiting...")
    println(deferred.await())
}

suspend fun loadData(): Int {
    println("loading...")
    delay(1000L)
    println("loaded!")
    return 42
}
```

runBlocking 은 일반적인 함수와 suspend 함수 사이에 다리 역할을 해주는 함수입니다. runBlocking은 최상위 레벨의 메인 코루틴을 실행하기 위한 어댑터이며, 테스트나 main 함수에서 주로 사용됩니다.

Deferred 객체의 리스트라면, `awaitAll` 을 호출하여 모든 결과를 전부 기다릴 수 있습니다. 

```kotlin
fun main() = runBlocking {
    val deferreds: List<Deferred<Int>> = (1..3).map {
        async {
            delay(1000L * it)
            println("Loading $it")
            it
        }
    }
    val sum = deferreds.awaitAll().sum()
    println("$sum")
}
```

위의 코드 예제들의 코루틴들은 전부 메인 UI 쓰레드위에서 동작합니다. 공용 쓰레드 풀로부터 온 다른 쓰레드위에서 코루틴을 동작시키기 위해서는 다음과 같이 async를 호출합니다.

```kotlin
async(Dispatchers.Default) { ... }
```

async 함수의 인자 `CoroutineDispatcher` 는 해당 코루틴 작업을 실행할 쓰레드를 결정합니다. 만약 인자로 dispatcher를 넘겨주지 않는다면, async는 outer scope의 dispatcher를 사용합니다.


### Dispatcher

코루틴의 Dispatcher는 자체 스레드 풀의 스레드 상황에 맞춰서 코루틴을 배분하는 역할을 합니다. 안드로이드에는 이미 Dispatcher가 생성되어 있으므로 별도로 생성할 필요가 없이 다음 중 하나의 Dispathcer를 사용하면 됩니다.

* Dispatchers.Main
* Dispatchers.IO
* Dispathcers.Default

```kotlin
CoroutineScope(Dispatchers.Main).launch{
    updateUI()
}
```

Dispatchers.Default 는 JVM의 공용 쓰레드 풀을 의미합니다. 이 쓰레드 풀은 병렬 실행의 수단을 제공합니다. CPU 코어의 수만큼의 쓰레드로 구성되지만, 싱글 코어인 경우에도 쓰레드는 2 개입니다. 메인 UI 쓰레드에서만 코루틴을 동작시키기 위해서는 Distpatchers.Main을 인자로 넘겨줍니다. 만약 메인 쓰레드가 새로운 코루틴을 시작할 때 busy 상태라면, 코루틴은 중단되고, 이 쓰레드에 실행 예약됩니다.

```kotlin
suspend fun getOrgRepos() : List<Repo> // 네트워크에서 Repo 정보를 받아오는 함수
suspend fun getRepoContributors(repo : Repo) : List<User> // 네트워크로 User 정보를 받아오는 함수

suspend fun loadContributorsConcurrent(): List<User> = coroutineScope {
    val repos = getOrgRepos() // List<Repo>
    val deferreds: List<Deferred<List<User>>> = repos.map { repo ->
				// async(Dispatchers.Default) { // Distapchers를 명시
        async() { 
            getRepoContributors(repo) // List<User>
        }
    }
    deferreds.awaitAll().flatten()
}
```

각 end-point에서 명시적으로 dispatcher를 선언하는 것은 좋지 않은 방법입니다. 이보다는 outer scope의 dispatcher를 사용하는 것이 유연한 방법입니다. 이렇게 함으로써, 우리는 이 함수를 어떠한 컨텍스트 위에서도 구동시킬 수 있습니다. 대신 caller-side 에서 dispatcher를 명시합니다.

```kotlin
launch(Dispatchers.Default) {
    val users = loadContributorsConcurrent(service, req)
    withContext(Dispatchers.Main) {
        updateResults(users, startTime)
    }
}
```

위의 샘플 코드에서 updateResults는 UI 업데이트를 위한 함수이므로, 메인 UI 쓰레드에서 호출되어야 합니다. 그래서, Dispatchers.Main의 context와 함께 호출합니다. withContext 는 명시된 코루틴 컨텍스트로 주어진 코드를 호출합니다. 그리고 그것이 완료될 때 까지 중단된 후, 결과를 반환합니다.

### withContext

withContext를 이용하면 비동기 작업을 순차 코드처럼 작성할 수 있게 됩니다. withContext block의 마지막 줄이 withContext의 반환값이 되며 withContext가 끝날 때 까지 해당 코루틴은 일시정지됩니다.

## Coroutine Job의 상태

Job의 상태는 총 6가지: `New, Active, Completing, Completed, Cancelling, Cancelled`

또한 상태를 나타내는 변수는 3가지: `isActive, isCancelled, isComplted`

isCompleted가 false에서 true로 바뀔 때 (즉, 코루틴이 종료 될 때), invokeOnCompletion이 실행된다. 이를 이용해 Job이 끝나고 나서의 처리를 할 수 있습니다.

Lazy 로 생성되지 않은 일반적인 경우의 coroutine은 New 직후 Active 상태가 됩니다. 정상적으로 완료가 되는 경우, 자식 코루틴의 완료를 기다리는 Completing 상태를 지나 Completed 상태가 됩니다.

Active 혹은 Completing 상태에서 실패 혹은 취소 될 경우, Cancelling 상태를 지나 Cancelled 상태가 됩니다.

정리하면 다음과 같습니다.

| State      | isActive | isCancelled | isCompleted |
|------------|----------|-------------|-------------|
| New        | false    | false       | false       |
| Active     | true     | false       | false       |
| Completing | true     | false       | false       |
| Completed  | false    | false       | true        |
| Cancelling | false    | true        | false       |
| Cancelled  | false    | true        | true        |

```
                    (wait children)
New ---> Active ----> Completing -----> Completed
            | cancel/fail |
            |             |
            |     +-------+
            |     |
            V     V  
         Cancelling -----> Cancelled
```

## Coroutine scope 와 Coroutine context

“외부 scope의 dispatcher를 사용한다” 라는 문장보다 사실 “외부 scope의 context의 dispatcher를 사용한다”가 더 맞는 문장입니다. *Coroutine scope* 는 서로 다른 코루틴 사이의 구조와 부모-자식 관계를 담당합니다. 모든 코루틴은 항상 scope 내부에서 시작됩니다. Scope는 Job을 통해 코루틴의 수명을 제어합니다. Scope의 Job이 취소되면, 해당 scope에서 시작된 모든 코루틴은 취소됩니다. 

*Coroutine context* 는 주어진 코루틴을 실행하기 위한 부가적인 technical information 을 저장합니다 (코루틴 이름, 코루틴이 실행되어야 하는 쓰레드가 명시된 dispatcher 등).

launch, async, runBlocking 이 새로운 코루틴 시작을 위해 사용될 때, 알아서 scope를 생성합니다. 그리고 새로운 코루틴은 생성된 이 scope 내부에서 시작됩니다. 이 세 함수들은 모두 인자로 람다 함수를 사용하고, 이 암시적인 리시버의 타입은 *CoroutineScope* 입니다.

```kotlin
launch { /* this: CoroutineScope */
}
```

새로운 코루틴은 scope 안에서만 실행될 수 있습니다. 그러나 runBlocking 으로 시작되는 코루틴은 예외입니다. runBlocking 은 top-level function으로 정의되어 있습니다 (전역 함수). 그래서 runBlocking 은 scope 내부에서 실행되지 않아도 됩니다. (그러나, runBlocking 의 람다 함수 내부는 scope 내부입니다)

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { /* this: CoroutineScope */
    launch { /* ... */ }
    // the same as:    
    this.launch { /* ... */ }
}

// launch impl in library
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
	...
}

// runBlocking impl in library
public fun <T> runBlocking(
		context: CoroutineContext = EmptyCoroutineContext,
		block: suspend CoroutineScope.() -> T
): T {
	...
}
```

위의 샘플에서 내부 중첩 코루틴(launch) 은 외부 코루틴(runBlocking) 의 자식이라고 할 수 있습니다. 이 부모-자식 관계는 scope를 통해 이루어집니다. 즉, 자식 코루틴은 부모 코루틴에 상응하는 scope에서 시작됩니다.

### CoroutineContext 확장

위에서 언급되었던 Dispatcher와 뒤에서 살펴볼 CoroutineExceptionHandler 두 가지 모두 CoroutineContext를 확장하는 구현 클래스입니다. 즉, CoroutineContext가 들어가는 인자의 자리에 들어갈 수 있습니다.

여러 CoroutineContext를 더해서 하나의 Context를 만들 수 있습니다. 방법은 간단합니다.

```kotlin
val newContext = Dispathcers.IO + exceptionHandler // new context

CoroutineScope(newContext).launch{
    ...
}
```

합쳐진 Context에서 합쳐지기 전 각각의 Context에 접근하는 것도 가능합니다.

```kotlin
newContext[exceptionHandler.key] // == exceptionHandler
```

### Scope Builder

다른 빌더들로 부터 제공되는 scope 에 더하여, [coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) 빌더를 이용하여 새로운 코루틴을 시작하지 않고도 새로운 scope를 만들어내는 것은 가능합니다. 이 suspend 함수는 새로운 코루틴 scope를 생성하고 자식들이 완료되기 전까지 종료되지 않습니다. 그리고, 이 scope는 자동적으로 함수를 호출한 외부 scope의 자식이 됩니다.

```kotlin
// Sequentially executes doWorld followed by "Done"
fun main() = runBlocking {
    doWorld()
    println("Done")
}

// Concurrently executes both sections
suspend fun doWorld() = coroutineScope { // this: CoroutineScope
    launch {
        delay(2000L)
        println("World 2")
    }
    launch {
        delay(1000L)
        println("World 1")
    }
    println("Hello")
}
```

### Structured concurrency

코틀린의 구조를 제공하는 메커니즘을 “**structured concurrency(구조화된 동시성)**” 라고 합니다. 이 메커니즘은 다음과 같은 이점을 제공합니다. 

- Scope는 일반적으로 자식 코루틴을 담당합니다. 자식 코루틴의 수명은 scope의 수명에 연결되어 있습니다.
- Scope는 자식 코루틴을 cancel할 수 있습니다.
- Scope는 자동적으로 모든 자식들이 완료될 때까지 기다립니다. 그러므로, 만약 이 scope가 코루틴과 연관되어 있다면, 이 부모 코루틴은 자식 코루틴이 끝나기 전까지 완료되지 않습니다.

반대로, global scope에서 코루틴을 시작할 수 있습니다. 이 코루틴들은 모두 독립적이며, 수명은 전체 어플리케이션에만 연결되어 있습니다 `(GlobalScope.async, GlobalScope.launch)`

구조화된 동시성을 이용하여 새로운 코루틴들을 주어진 scope에서 시작하면, 같은 context로 그들을 모두 실행 시키는 것은 쉬워집니다. 또한, 필요하다면 context를 교체하는 것도 간단합니다. coroutineScope 혹은 coroutine 빌더 중 하나로 만들어진 새로운 scope 는 항상 외부 scope의 context를 상속받습니다.

```kotlin
launch(Dispatchers.Default) {  // outer scope
    val users = loadContributorsConcurrent(service, req)
    // ...
}
```

모든 중첩 코루틴은 상속된 context로 실행됩니다(dispatcher는 context가 가지고 있습니다). 

```kotlin
suspend fun loadContributorsConcurrent(
    service: GitHubService, req: RequestData
): List<User> = coroutineScope {
    // this scope inherits the context from the outer scope 
    // ... 
    async {   // nested coroutine started with the inherited context
        // ...
    }
    // ...
}
```

structured concurrency를 사용하면, 우리는 context에 대한 정보(dispatcher 등) 를 top-level 코루틴을 생성할 때 한번만 설정하더라도, 모든 중첩 코루틴들은 이 context를 상속 받게 됩니다. Android와 같이 UI 어플리케이션에서 코루틴으로 코드를 작성할 때, 최상위 코루틴을 위해 Dispatchers.Main 를 기본으로 사용하고, 다른 쓰레드에서 코드가 실행되어야 할 때, 다른 dispatcher를 설정하여 주는 것이 흔히 구현되는 방식입니다.


## Exception Handling in Coroutine

코루틴에서 Exception을 핸들링 하는 방법 중 주요 2가지는 다음과 같습니다: `invokeOnCompletion, CoroutineExceptionHandler`

### invokeOnCompletion

``` kotlin
CoroutineScope(Dispatchers.Main).launch{
    throw IllegalStateException()
}
.invokeOnCompletion { throwable : Throwable? ->
    println(throwable) // throwable이 null인 경우는 정상 종료.
}
```

### CoroutineExceptionHandler

`CoroutineExceptionHandler`은 에러 처리를 위한 CoroutineContext 이다. 

``` kotlin
val exceptionHandler = CoroutineExceptionHandler {_, exception -> 
    println("Exception : $exception")
}

CoroutineScope(Dispatchers.Main).launch(exceptionHandler){
    throw InterruptedException()
}

// CoroutineExceptionHandler 는 재사용이 가능하다
val job2 = CoroutineScope(Dispatchers.IO).launch(exceptionHandler){
    throw IllegalStateException()
}
```

<p class="callout info">Deferred는 Job을 확장한 제네릭 타입입니다. 즉 Deferred는 Job의 상태 변수 및 Exception Handling 등을 모두 적용할 수 있다. </p>

Deferred의 경우, 예외를 자동으로 전파하지는 않는다. await의 위치를 CoroutineExceptionHandler Context로 설정하면 예외를 받을 수 있다.

```kotlin
val exceptionHandler = CoroutineExceptionHandler {_, exception -> 
    println("Exception : $exception")
}

val deferred = CoroutineScope(Dispatchers.IO).async {
    throw IllegalArgumentException()
}

CoroutineScope(Dispatchers.IO).lauch(exceptionHandler) {
    deferred.await()
}

// Deferred 에서 발생하는 예외는 결과에 첨부되어 결과를 확인할 때 에러가 전파된다.
```

### 부모 - 자식 관계의 코루틴에서의 Exception Handling

자식 코루틴에 Exception이 발생했을 때, 별도의 Exception Handler가 없다면, 부모 코루틴 또한 Exception이 발생되며 취소됩니다. 이 때, 다른 자식 코루틴들 또한 모두 취소됩니다. 이는 심각한 버그를 야기할 수도 있습니다. 이러한 문제를 대응하기 위해 `SupervisorJob`을 사용할 수 있습니다.

`SupervisorJob`은 Exception의 전파 방향을 자식으로 한정 짓습니다. 즉, Exception이 부모 코루틴으로 전파되지 않습니다. 보통 다른 CoroutineContext와 혼합해서 사용됩니다.

``` kotlin
val supervisor = SupervisorJob()
CoroutineScope(Dispatchers.IO).launch {
    val childJob = launch(Dispatchers.IO + supervisor) {
        throw AssertionError("Error!!")
    }
}
```

모든 자식 코루틴에 동일하게 SupervisorJob을 설정하려면, 즉, 특정 블록 내부의 모든 코루틴에 Supervisor Job을 설정하려면, `supervisorScope`를 사용할 수 있다.

``` kotlin
CoroutineScope(Dispatchers.IO).launch {
    supervisorScope {
        val childJob = launch(Dispatchers.IO) {
            throw AssertionError("Error!!")
        }
    }
}
```