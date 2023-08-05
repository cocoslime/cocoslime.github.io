---
title:  "[Android] 주기적인 알림 구현 (WorkManager, Notification)"
excerpt: ""

categories:
  - develop
tags:
  - Android
  - Kotlin
  - TIL
  - WorkManager

date: 2022-10-05

toc: true
toc_sticky: true
---

# 주기적인 알림 구현 (WorkManager, Notification)

안드로이드에서 [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager?hl=ko) 를 이용해 주기적으로 정해진 시간 간격마다 푸시 알림을 띄울 수  있습니다. 간단한 예시 프로젝트를 통해 설명드리겠습니다.

먼저 `build.gradle` 에 다음과 같이 의존성을 추가합니다.

```kotlin
dependencies {
		...

    implementation "androidx.work:work-runtime-ktx:2.7.1"
}
```

다음은, 반복적인 작업을 수행할 `Worker` 클래스를 구현합니다. WorkManager 라이브러리의 일부이며, `doWork` 메소드의 작업을 수행합니다. 예시 코드에서는 `Worker` 클래스를 상속하여 `CustomWorker` 를 만듭니다.

```kotlin
class CustomWorker(private val context: Context, params: WorkerParameters) : Worker(context, params){
    override fun doWork(): Result {

		}
}
```

`doWork` 내부에서 Notification 을 띄우는 작업을 수행하려고 합니다. 이를 위해, Notification 채널을 생성하는 메소드와 Notification 객체를 반환하는 메소드를 구현합니다.

```kotlin
private fun createNotificationChannel() {
    val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
    notificationManager.createNotificationChannel(NotificationChannel(
        CHANNEL_ID,
        "Channel Name",
        NotificationManager.IMPORTANCE_DEFAULT
    ))
}

private fun createNotification(title: String, message: String) : Notification {
    val intent = Intent(context, MainActivity:: class.java).apply{
        flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
    }
    val pendingIntent = PendingIntent.getActivity(context, 0, intent, FLAG_IMMUTABLE)
    return NotificationCompat.Builder(context, CHANNEL_ID)
        .setSmallIcon(androidx.appcompat.R.drawable.abc_ic_clear_material)
        .setContentTitle(title)
        .setContentText(message)
        .setContentIntent(pendingIntent)
        .setPriority(NotificationCompat.PRIORITY_DEFAULT)
        .build()
}
```

이 두 메소드를 `doWork` 에서 호출하고, 동작 여부 판단을 위한 로깅을 추가합니다. `inputData` 는 `Worker` 의 프로퍼티입니다. `Worker` 를 생성할 때, 넘겨주는 데이터를 `inputData` 를 통해 가지고 와서 처리가 가능합니다.

```kotlin
override fun doWork(): Result {
	  createNotificationChannel()
	
	  val notification = createNotification(
	      inputData.getString("title") ?: "Empty Title",
	      inputData.getString("message") ?: ""
	  )
	  NotificationManagerCompat.from(context).notify(NOTIFICATION_ID, notification)
	
	  Log.d("CustomWorker", "doWork()")

	  return Result.success()
}
```

`CustomWorker` 의 기본적인 구현은 끝입니다. `WorkManager` 를 통해 작업을 예약하려면, 먼저 `WorkRequest` 객체를 만든 후에 이를 작업 큐에 등록하여야 합니다. 다음 코드에서는 빌더를 사용해 주기적 작업을 위한 `WorkRequest` 를 만듭니다. 자세한 가이드는 [개발자 페이지](https://developer.android.com/topic/libraries/architecture/workmanager/how-to/define-work?hl=ko)를 참고 

```kotlin
val myWorkRequest = PeriodicWorkRequestBuilder<CustomWorker>(15, TimeUnit.MINUTES)
            .setInitialDelay(10, TimeUnit.SECONDS)
            .setInputData(workDataOf(
                "title" to "Reminder",
                "message" to message,
            ))
            .build()
```

주기적인 작업이므로 `PeriodicWorkRequest` 를 사용합니다. Worker는 위에서 구현한 `CustomWorker` 를 가져오고, 15분마다 반복 수행하도록 설정합니다. 최초 지연 시간은 10초이고, `setInputData()` 를 통해 데이터를 넘겨줄 수 있습니다.

> 주기적 작업의 최소 반복 시간은 15분이다. [참고](https://developer.android.com/reference/androidx/work/PeriodicWorkRequest#MIN_PERIODIC_INTERVAL_MILLIS)
> 

위의 WorkRequest를 WorkManager 의 인스턴스에 접근하여 등록합니다.

```kotlin
WorkManager.getInstance(this).enqueueUniquePeriodicWork(
            "name",
            ExistingPeriodicWorkPolicy.REPLACE,
            myWorkRequest
        )
```

이 때, 작업에 대한 고유 이름(여기서는 “name”)을 설정할 수 있으며, 이미 해당하는 작업이 등록되어 있는 경우에 대한 정책도 설정할 수 있습니다(여기서는 REPLACE. 즉 교체). 

샘플 앱의 프로젝트는 다음 repo 에서 확인 가능합니다. [https://github.com/cocoslime/BlogSamples/tree/master/NotificationWorkManager](https://github.com/cocoslime/BlogSamples/tree/master/NotificationWorkManager)