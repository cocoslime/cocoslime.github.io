---
title:  "[Android] 안드로이드 앱 서명 (feat. Android App Bundle)"
excerpt: "안드로이드 앱 서명 방식의 변화와 AAB에 대해 알아보자"

categories:
  - Blog
tags:
  - Android
  - ABB
last_modified_at: 2021-12-29T21:00:00-05:00

toc: true
toc_sticky: true
---

<details>
<summary>문서 이력</summary>
<div markdown="1">

- 2021.12.30. 포스팅

</div>
</details>
<br>

안드로이드에서의 앱 서명 방식에 대한 세미나를 준비하여 발표하였고, 이를 간략하게 정리하여 포스팅합니다. 본 포스팅은 2021년 10월 기준 안드로이드 정책을 기반으로 작성되었습니다.

# 안드로이드 앱 서명이란

안드로이드의 모든 APK는 앱 서명 키를 사용하여 서명되어야 설치 될 수 있습니다. 안드로이드는 앱 업데이트가 기기에 설치된 앱의 업데이트인지 확인합니다. 안드로이드는 앱 서명을 사용하여 앱 업데이트를 위한 신규 APK가 기존에 이미 설치된 앱과 같은 곳(개발자, 배포처 등)으로부터 생성되어 진 것인지 확인합니다. 

서명 검증을 통과하는 경우에만 앱 설치를 허용하여, **악성 앱 업데이트 위험**을 줄입니다.

# 전통적인 앱 서명 방식

전통적인 앱 서명 방식은 개발자가 앱 서명 키를 생성하고 앱 설치 검증을 위한 인증서를 직접 생성하여 업로드하는 자체 서명 키 사용 및 관리 방식입니다. 이때의 서명값은 사용자의 단말에 설치될 때 검증을 위해 사용됩니다. 

구글 플레이스토어와 같은 앱 스토어에 개발자는 앱(APK)을 직접 업로드하고 배포하며, 앱 스토어에서는 서명 값을 변경하지 않습니다.

# AAB(Android App Bundle)

## AAB란

Android App Bundle(ABB)은 앱의 모든 컴파일된 코드 및 리소스를 포함하는 게시 형식으로서 앱을 설치하고 실행할 때 사용되는 형식인 APK와는 다르게, 기기에 설치되거나 기기에서 실행될 수 없습니다. 개발자가 직접 APK 생성 및 서명을 하지 않고 Google Play에 맡기는 게시 형식입니다.

Google Play는 AAB를 활용하여 앱 설치를 요청한 각 기기에 맞게 최적화된 APK를 생성하고 제공합니다. 따라서 사용자는 자신의 기기와 관련이 없는 코드, 리소스가 포함되지 않은 더 작고 최적화된 앱을 다운로드하게 되며, 개발자는 다양한 기기에 대한 지원을 최적화하기 위해 여러 종류의 APK를 빌드, 관리할 필요가 없습니다.

AAB는 Google Play 뿐만 아니라 오픈소스이므로 모든 앱 스토어에서 지원 가능합니다.

## Google Play 정책 업데이트

Google Play는 앞으로의 정책 업데이트 일정을 공지하고 있으며, 2021년 8월의 정책 업데이트에서 AAB를 신규 앱의 필수 형식으로 지정하였습니다. 즉, 8월 이후 신규 개발되어 배포되는 앱은 Google Play에 업로드 시에 AAB 형식으로 게시해야 합니다. (8월 이전 배포된 앱은 기존 방식을 그대로 사용 가능 합니다.)

Google Play에서 AAB를 필수 게시 형식으로 지정함에 따라, 전통적인 앱 서명 방식으로 앱 배포를 위한 서명이 불가능해졌습니다. 따라서, AAB 요구사항은 **Play 앱 서명 방식**을 강제합니다.

전통적인 앱 서명 방식에서는 개발자가 APK를 빌드하고 서명한 값이 사용자에 그대로 전달됩니다. 하지만, AAB 형식으로 업로드 한다면, Google Play에서 APK를 빌드하고 서명하므로, 기존의 앱 서명 방식은 적용될 수 없습니다.

# Play 앱 서명 방식

Play 앱 서명은 2017년 출시된 Google Play의 키 관리 서비스 입니다. 전통적인 앱 서명 방식에서는 앱 서명 키를 개발자가 분실하면 더 이상 사용자에게 앱 업데이트를 제공할 수 없고, 키가 유출되면 사용자가 악성 업데이트의 위험에 놓이게 됩니다. Play 앱 서명은 이러한 위험을 없애고 높은 수준의 키 관리 보안 서비스를 제공합니다.

Play 앱 서명은 앱을 서명하는데 필요한 *앱 서명 키*와 업로드를 위한 *업로드 키*라는 두 가지 키를 사용합니다.

## 업로드 키

업로드 키는 개발자가 보관하다가 앱을 Google Play에 업로드하기 위해 서명할 때 사용합니다. Google은 기 등록되어 있는 업로드 인증서를 사용하여 서명을 검증합니다. 최초 업로드 시 서명할 때 사용한 키가 업로드 키가 되며, 차후 업로드에도 이 키를 사용하여야 합니다. 만약, 개발자는 언제든지 업로드 키를 새로 생성하여 새로운 업로드 인증서를 등록할 수 있습니다.

단순히 앱 개발자 입장에서는, 업로드 키 생성 및 서명, 관리는 기존 앱 서명 방식의 앱 서명 키에서의 방식과 동일합니다. Android Studio를 사용한다면, 기존의 앱 빌드 시에 키 생성 및 선택 후 서명하는 과정이 업로드 키의 과정입니다.

## 앱 서명 키

앱 서명 키는 Google에서 안전하게 저장하고 있으며, 배포할 APK를 서명하는 데 사용됩니다. 이 서명을 기기에 설치 및 업데이트 시에 검증합니다. 앱 서명 키는 앱의 전체 기간 동안 변경되지 않습니다. (하지만, 변경될 수 있는 특수 상황이 있습니다.)

앱 서명 키는 3가지의 방법 중 하나를 통해 생성 되거나 등록됩니다. 첫번째는, Google이 Play 앱 서명 구성 시 자동으로 앱 서명 키가 생성됩니다. 이 방법을 선택한다면, 개발자는 Play 앱 서명을 구성하고 앱을 업로드 할 때 추가적인 어떠한 동작도 필요 없습니다. 이 앱 서명 키는 Google이 생성하고 관리하기 때문에, 개발자도 이 키를 획득할 수는 없습니다. 하지만, 이 키로 서명된 인증서 및 apk를 다운로드 가능합니다. 이렇게 서명된 apk를 다른 앱 스토어에도 배포하여 같은 키로 서명된 apk를 여러 배포처를 통해 배포할 수 있습니다.

두 번째 방법은, 개발자가 가지고 있는 앱 서명 키를 업로드하여 대체하는 방식입니다. 이 방법으로, 개발자는 기존에 사용하던 키를 그대로 사용할 수도 있으며, Google에 키 생성을 맡기지 않고 개발자 측에서 생성 및 관리하는 키를 사용할 수 있습니다.

앱 서명 키는 개인키 이므로, 외부로 유출되어서는 안됩니다. 하지만, Play 앱 서명 서비스를 사용하기 위해서는 Google에서 앱 서명 키의 원본을 가지고 있어야 하므로, 개발자는 앱 서명 키를 암호화 하여 업로드하고, Google은 복호화하여 등록된 앱 서명 키를 사용합니다.

세 번째 방법은, 개발자 계정의 배포되는 다른 앱에서 사용중인 앱 서명 키를 사용하는 것입니다.

# 정리

(21년 8월 이후 출시 앱 기준) 아래 그림은 앱 빌드, 서명, 배포 과정을 간략하게 도식화한 그림입니다.

![]({{ site.url }}{{ site.baseurl }}/assets/img/2021-12-30-App-Signing.png)

개발자는 안드로이드 앱 프로젝트를 구현 후 AAB 형식으로 빌드하고 Google Play에 업로드 합니다. Google은 업로드 인증서로 업로드 되는 앱을 검증하고 APK를 생성하여 앱 서명 키로 서명한 후 사용자에게 배포합니다. 사용자 단말은 앱 서명 인증서로 검증한 뒤 앱을 설치합니다.

구글 플레이스토어 정책의 이러한 변화는 크게 두 가지로 구분할 수 있습니다. 게시 형식의 변화, 서명 방식의 변화. 두 변화 모두 관리적인 측면에서 효율적입니다. AAB로의 변화는 앱의 크기를 더 작게 만들 수 있습니다. 즉, 이는 사용자가 앱을 더 빠르게 설치하고, 개발자의 앱이 더 많이 설치 될 수 있음을 의미합니다. 서명 방식의 변화는 키 분실의 위험에서 자유로워 집니다. 구글의 Play 앱 서명 시스템이 앱 서명 키를 관리함으로서 개발자는 키를 분실하여 앱 업데이트를 전송할 수 없는 상황에 빠지지 않습니다. 그리고 자체 서명 키를 사용하지 않는다면, 개발자의 개인 PC에 키를 보관하는 것 보다 구글의 보안 인프라에만 키를 존재하게 하므로 키 유출에 의한 악성 업데이트의 위험에서 보다 안전할 가능성이 높습니다.


<details>
<summary>개인적인 생각</summary>
<div markdown="1">

하지만, 개념적인 보안 측면에서는 생각해 볼 사항이 있습니다. 여기서 개발자는 취미로 개발하는 1인 개발자일수도 있지만, 대형 IT 서비스를 기획하고 개발하는 기업의 형태일 수도 있습니다. 제가 생각하는 이러한 변화의 중요 사항은 앱 서명용 개인 키를 구글 플레이스토어에서 제어할 수 있도록 변경한 것입니다. 구글의 보안 시스템은 아마도 매우 훌륭할 것이므로 키 유출에 대해서는 걱정을 하지 않아도 되지만, 그럼에도 구글이 앱 서명용 개인 키를 가지고 있는 것은 또 다른 이슈입니다. 구글은 안드로이드 생태계를 조금씩 폐쇄적이고 전체 시스템을 구글이 관리, 감독이 가능한 방향으로 가도록 의도하고 있는 듯 합니다. (마치 ios 생태계처럼) 이는 최근 이슈가 되고 있는 인앱 결제 정책과도 연관이 있는 것으로 생각이 됩니다. 안드로이드 생태계에 살짝 발을 담구고 있는 개발자로서, 최근 이러한 변화들도 관심있게 보는 것이 좋겠다고 생각했습니다.

</div>
</details>
<br>
    
    