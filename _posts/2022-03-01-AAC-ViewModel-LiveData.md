---
title:  "[Android] (AAC) ViewModel, LiveData 그리고 Data Binding "
excerpt: "ViewModel과 LiveData 사용법 그리고 둘에 대한 Data Binding"

categories:
  - blog
tags:
  - Android
  - Kotlin
  - TIL
last_modified_at: 2022-03-01T16:00:00-05:00

toc: true
toc_sticky: true
---

이 게시글의 코드 및 설명은 다음을 참고하였습니다: [안드로이드 개발자 사이트](https://developer.android.com/codelabs/kotlin-android-training-live-data?index=..%2F..android-kotlin-fundamentals#7)

# ViewModel 용어

- MVVM 구조 패턴에서의 ViewModel
- AAC(Android Architecture Components)의 ViewModel 클래스
- 두 가지의 뜻이 있지만, 서로 아주 연관이 없는 것은 아니다.

# ViewModel의 역할

- UI controller (프래그먼트나 액티비티)에서 보여질 데이터를 관리
- UI controller 에서 보여질 데이터를 준비하기 위하여 간단한 계산이나 변환 가능

## onCleared()

- ViewModel은 연관된 프래그먼트가 detach되거나, 액티비티가 종료될 경우 destroy된다.
- Destroy 되기 바로 직전에 onCleared() 콜백 함수가 호출 된다.

```kotlin
class GameViewModel : ViewModel() {
    init {
        Log.i("GameViewModel", "GameViewModel created!")
    }

    override fun onCleared() {
        super.onCleared()
        Log.i("GameViewModel", "GameViewModel destroyed!")
    }
}
```

# ViewModelProvider

- ViewModel 과 UI controller 사이의 관계를 위해서, UI controller 내부에서 ViewModel을 참조한다. (ViewModel instance 멤버 변수를 만든다)
- ViewModel 객체를 생성할 때, 직접 생성하지 않고 항상 **ViewModelProvider**를 이용한다.
- ViewModelProvider의 역할
    - (Singleton) 존재하는 ViewModel이 있으면 그것을 반환하고, 없다면 새로 만들어 반환한다.
    - 주어진 scope (Activity or Fragment) 와 ViewModel 객체간의 association을 생성한다.
    - 만들어진 ViewModel은 scope가 살아있을 때 까지 유지된다.

```kotlin
Log.i("GameFragment", "Called ViewModelProvider.get")
val viewModel : GameViewModel = ViewModelProvider(this).get(GameViewModel::class.java)
```

- Fragment가 destroy되고 다시 그려지더라도 ViewModel은 재생성 되지 않는다.
    - example) 화면 rotate, 홈 화면 갔다 오기 등
- Fragment가 destroy되고 detach 되면 ViewModel도 destroy 된다.

# ViewModelFactory

- ViewModel 초기화를 팩토리 메소드 패턴을 이용하여 ViewModel 객체를 생성할 수 있다.
- ViewModelProvider.Factory를 상속

```kotlin
// Factory class. 생성자 인자로 Int를 받음
class ScoreViewModelFactory(private val finalScore : Int) : ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(ScoreViewModel::class.java)) {
            return ScoreViewModel(finalScore) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}

class ScoreViewModel(val score : Int) : ViewModel() {
    init {
        Log.i("ScoreViewModel", "Final score is $score")
    }
}
```

- UI Controller 에서 Factory 객체를 직접 생성하고 Factory의 get 함수를 통해 ViewModel을 가져온다.

```kotlin
// val score : Int
val viewModelFactory = ScoreViewModelFactory(score)
val viewModel = ViewModelProvider(this, viewModelFactory)
                .get(ScoreViewModel::class.java)
```

# LiveData

- LiveData는 Observable하고, lifecycle에 따라 관리되는 data holder class
    - Observable 하다는 것은, LiveData 객체가 변경 되었을 때, 액티비티나 프래그먼트 같은 observer 가 알아차릴 수 있다는 의미
- LiveData를 통해 데이터가 수정되었을 때, 자동적으로 UI 업데이트가 가능하게 해준다.
- LiveData는 어떠한 데이터든 간에 hold할 수 있다.
- lifecycle을 인식하여, STARTED or RESUMED 같은 lifecycle 상태에서만 업데이트 한다.

## ViewModel 에서의 LiveData 사용

- `MutableLiveData` 는 변경 가능한 `LiveData` 클래스이다.
- Encapsulation을 위해 LiveData는 ViewModel 안에서는 editable, 밖에서는 only readable 이어야 한다. 이것을 위해 [Kotlin backing property](https://kotlinlang.org/docs/reference/properties.html#backing-properties)를 사용한다.

```kotlin
class GameViewModel : ViewModel() {
    private val _word = MutableLiveData<String>() // 내부에서 변화할 LiveData
    val word : LiveData<String> // 외부에서 접근할 LiveData
        get() = _word
}
```

## Observable LiveData

LiveData는 observer pattern을 따른다. 패턴에서 “Observable”은 LiveData 객체, “Observer” 들은 UI controller의 함수들이다. LiveData의 데이터가 변화할 때, Observer method들이 수행된다.

Observer 객체들을 LiveData 에 붙이기 위하여, observe() 함수를 사용한다. observe 함수는 LifecycleOwner 타입을 필요로 하는데, 이때 Fragment view의 LifecycleOwner인 viewLifecycleOwner를 사용한다.
    
    Fragment view는 다른 요소로 이동할 때 destroy 되지만, Fragment 자체는 destroy되지 않을 수 있다. 즉, 이것은 두개의 lifecycle을 만들어내는데, fragment’s view의 lifecycle과 fragment의 lifecycle 이다. LiveData의 목적은 View 를 업데이트 하는 것이므로 view의 lifecycle에 맞게 동작하여야 한다.

```kotlin
// In CreateView of Fragment

// Add observer for score
viewModel.score.observe(viewLifecycleOwner, Observer { newScore ->
   binding.scoreText.text = newScore.toString()
})
```

LiveData의 변화만이 Observer 들에게 업데이트를 전송하지만, 한가지 예외는 비활성화 상태에서 활성화 상태로 변화할 때도 Observer 들은 업데이트를 받을 수 있다.

## LiveData Transformation

```Transformations.map()``` 을 사용하여, 원본 LiveData 에서 다른 LiveData 로 변환 할 수 있다.

```kotlin
// currentTime => Long 타입의 정수(seconds)
// val currentTime : LiveData<Long>

val currentTimeString : LiveData<String> = Transformations.map(currentTime) { time ->
   DateUtils.formatElapsedTime(time) // 초 단위의 값을 MM:SS 형태의 String 으로 변환한다.
   // currentTime의 변화에 따라 currentTimeString의 데이터도 자동으로 변한다.
}
```

Observer를 추가하여 currentTimeString의 변화에 대응하여 UI를 업데이트 할 수 있다.

```kotlin
viewModel.currentTimeString.observe(viewLifecycleOwner, Observer<String> { timeStr ->
    binding.timerText.text = timeStr
})
```

# Data Binding

Data binding 라이브러리를 통해 UI Controller의 Listener를 통하지 않고 view 객체와 데이터를 직접 바인딩 할 수 있다.

<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/2022-03-01/AAC-ViewModel-LiveData-arch.png" alt="qry" >
</p>

위와 같은 구조에서 아래와 같은 구조로 변경된다.

<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/2022-03-01/AAC-ViewModel-LiveData-new_arch.png" alt="qry" >
</p>

## ViewModel data binding

ViewModel과 layout을 data binding을 사용해 연결한다.

```xml
<data>
	<variable
		name="gameViewModel"
		type="com.example.android.guesstheword.screens.game.GameViewModel" />
</data>
```

layout의 variable에 ViewModel 객체를 할당한다. 이제 layout은 ViewModel의 함수를 포함한 모든 접근 가능 데이터에 접근할 수 있다.

```kotlin
binding.gameViewModel = viewModel
```

UI Controller 에서 onClickListener를 구현했던 버튼을 layout에서 ViewModel의 함수에 바로 접근하도록 layout 파일을 수정한다.

```xml
<Button
   android:id="@+id/skip_button"
   ...
   android:onClick="@{() -> gameViewModel.onSkip()}"
   ... />
```

~~layout의 모든 버튼에서 ViewModel로 직접 접근하도록 data binding 하는 것은 불가능하다. UI Controller 에서 해당 데이터 변화에 따라 다른 UI component로 이동한다거나 하는 경우는 UI controller의 개입이 필요하다.~~

상태 변화를 위한 LiveData를 만들고 UI controller에서 이를 observe 하다가 변화가 일어나면 다른 Fragment로 이동하는 방식으로 구현은 가능하다.

    View --(Listener)--> ViewModel --(Update)--> LiveData --(Observe)--> Ui Controller

## LiveData data binding

LiveData 객체를 바인딩하여, 데이터의 변화를 UI에 자동적으로 반영하도록 할 수 있다. layout의 view를 ViewModel 내부의 LiveData 객체에 직접 바인딩한다.

```xml
android:text="@{gameViewModel.word}"
```

LiveData의 데이터 바인딩이 작동하기 위해서 UI controller의 binding 변수의 lifecycle owner를 현재 View의 것으로 설정해야 한다.

```kotlin
binding.lifecycleOwner = this.viewLifecycleOwner
```