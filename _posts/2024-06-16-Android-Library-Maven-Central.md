---
title:  "안드로이드 라이브러리 Maven Central 에 배포 하기"
excerpt: "2024년 상반기에 기존 방식인 issues.sonatype.org 가 폐기되었고 새로운 안드로이드 라이브러리 Maven Central 에 배포 방법 설명"

categories:
  - blog
tags:
  - Android
  - Library
  - Maven
last_modified_at: 2024-06-16T12:00:00-05:00

toc: true
toc_sticky: true
---

<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/Pasted image 20240616003610.png" alt="qry" width="600" >
</p>

2024년 상반기에 issues.sonatype.org가 폐기됨을 발표했습니다
[https://central.sonatype.org/faq/what-happened-to-issues-sonatype-org/#what-happened-to-issuessonatypeorg](https://central.sonatype.org/faq/what-happened-to-issues-sonatype-org/#what-happened-to-issuessonatypeorg)

기존의 지라 티켓과 Nexus repository(OSSRH)를 통해 업로드하는 방식은, 기존 사용자는 그대로 이용가능하지만 신규 사용자는 Maven Central portal 을 통해 새로운 방식으로 라이브러리를 업로드 해야합니다.
이 포스팅에서는 Maven Central portal 에 계정을 만들고, 여러 방법 중 본인의 github 계정을 이용해서 라이브러리를 배포하는 방법을 소개합니다.

현재(2024.06.16), Maven Central 에 Central Publishing Portal 을 이용해서 publish 하는 공식 gradle plugin 은 없습니다. [link](https://central.sonatype.org/publish/publish-portal-gradle/). 

대신에, [vanniktech](https://vanniktech.github.io/gradle-maven-publish-plugin/central/) plugin 을 사용합니다.

## Prerequisites
### Maven Central 계정 생성
먼저 계정을 생성합니다.
[https://central.sonatype.com/](https://central.sonatype.com/)
### Namespace 생성 및 검증

<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/Pasted image 20240612200824.png" alt="qry" width="600" >
</p>

Add Namespace 버튼을 클릭하고 다음과 같은 규칙으로 namespace 를 생성합니다.
```
본인의 github 계정이 example 이라면 -> io.github.example
```
그렇게 Unverified Namespace 가 생성이 됩니다.
<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/Pasted image 20240612201012.png" alt="qry" width="600" >
</p>

Verification Key 를 복사하여 자신의 github repository 에 verification key 와 동일한 이름으로 레포지토리를 생성합니다.
```
verification key 가 rdaqyyjnic 이라면, rdaqyyjnic 라는 public repository 를 생성합니다.
즉, github.com/example/rdaqyyjnic 라는 url 이 유효하게 됩니다.
```
이후 verify namespace 버튼을 클릭하여 검증을 완료합니다. verified 후에는 레포지토리를 삭제해도 괜찮습니다.

### Generate Token
id 와 password 대신 사용할 토큰을 생성해야 합니다. Generate User Token 버튼을 클릭하여 토큰을 생성합니다.
<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/Pasted image 20240615141303.png" alt="qry" width="600" >
</p>

다음과 같이 노출되는 토큰의 아이디와 패스워드를 복사하여 둡니다.
<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/Pasted image 20240615141756.png" alt="qry" width="600" >
</p>

### Generate GPG Key
먼저 GPG 키를 생성하기 위한 gnupg 를 설치합니다.
```shell
$ brew install gnupg
$ gpg --full-generate-key --command-fd 0 --status-fd 1 --pinentry-mode loopback
```

만들어진 GPG 키를 확인합니다.
```shell
$ gpg --list-keys --keyid-format LONG

[keyboxd]
---------
pub   rsa4096/F7FAB2CA 2024-05-25 [SC]
      DB506D8D4302948FB17BB5370886B370F7FAB2CA
uid         [ultimate] cocoslime <dongmen94@gmail.com>
sub   rsa4096/CF4E7347 2024-05-25 [E]
```

public key 를 key server 에 업로드합니다.
```
gpg --keyserver keyserver.ubuntu.com --send-keys F7FAB2CA
```
F7FAB2CA 는 Key id 입니다. 자신의 key id 로 바꾸어서 명령어를 입력해 주세요.

### gpg 키 파일 생성
```
gpg --keyring secring.gpg --export-secret-keys > ~/.gnupg/secring.gpg
```

## Publish

### build.gradle.kts
build.gradle.kts 에 [com.vanniktech.maven.publish](https://vanniktech.github.io/gradle-maven-publish-plugin/central/) plugin 를 통해 maven central 에 publish 하는 스크립트를 작성합니다.

먼저 plugin 을 적용합니다.
```groovy
plugins {
    id("com.vanniktech.maven.publish") version "0.28.0"
}
```
다음은 Maven Central 에 게시를 활성화하고, GPG 서명을 활성화합니다.
```groovy
import com.vanniktech.maven.publish.SonatypeHost

mavenPublishing {
  publishToMavenCentral(SonatypeHost.CENTRAL_PORTAL)

  signAllPublications()
}
```
POM 을 configure 합니다. pom 은 프로젝트와 함께 게시되며 프로젝트 좌표와 URL 및 사용된 라이선스와 같은 프로젝트에 대한 일반적인 정보를 나타냅니다.
```groovy
mavenPublishing {
  coordinates("com.example.mylibrary", "mylibrary-runtime", "1.0.3-SNAPSHOT")

  pom {
    name.set("My Library")
    description.set("A description of what my library does.")
    inceptionYear.set("2020")
    url.set("https://github.com/username/mylibrary/")
    licenses {
      license {
        name.set("The Apache License, Version 2.0")
        url.set("http://www.apache.org/licenses/LICENSE-2.0.txt")
        distribution.set("http://www.apache.org/licenses/LICENSE-2.0.txt")
      }
    }
    developers {
      developer {
        id.set("username")
        name.set("User Name")
        url.set("https://github.com/username/")
      }
    }
    scm {
      url.set("https://github.com/username/mylibrary/")
      connection.set("scm:git:git://github.com/username/mylibrary.git")
      developerConnection.set("scm:git:ssh://git@github.com/username/mylibrary.git")
    }
  }
}
```

이렇게 완성된 build.gradle.kts 의 예시입니다.

```groovy
import com.vanniktech.maven.publish.SonatypeHost

plugins {
    id("com.android.library")
    id("org.jetbrains.kotlin.android")
    id("com.vanniktech.maven.publish")
}

android {
   // .. 생략
}

dependencies {
   // ... 생략
}

mavenPublishing {
    publishToMavenCentral(SonatypeHost.CENTRAL_PORTAL)

    signAllPublications()

    coordinates("io.github.cocoslime", "watermark-cover", "0.0.1")

    pom {
        name = "WatermarkCover"
        description = "Cover view/compose with Watermark Text"
        url = "https://github.com/cocoslime/watermark-cover"
        inceptionYear = "2024"

        licenses {
            license {
                name = "The Apache License, Version 2.0"
                url = "http://www.apache.org/licenses/LICENSE-2.0.txt"
            }
        }
        developers {
            developer {
                id = "cocoslime"
                name = "cocoslime"
                url = "https://github.com/cocoslime"
            }
        }

        scm {
            connection = "scm:git:github.com/cocoslime/watermark-cover.git"
            developerConnection.set("scm:git:ssh://github.com:cocoslime/watermark-cover.git")
            url = "https://github.com/cocoslime/watermark-cover/tree/master"
        }
    }
}
```

### gradle.properties
프로젝트 루트의 gradle.properties 혹은 ~/.gradle/gradle.properties 에 다음 내용을 추가합니다. 다음 내용이 공개 레포지토리에 푸시되지 않도록 주의하세요.

```
mavenCentralUsername=TOKENUSERNAME # Maven Central token 의 username 
mavenCentralPassword=TOKENPASSWORD # Maven Central token 의 password 
signing.keyId=F7FAB2CA # key 의 id
signing.password=paswword # key 의 패스워드 
signing.secretKeyRingFile=/Users/guest/.gnupg/secring.gpg # secring.gpg 의 경로
```

### publish task
다음 command 를 통해 maven central 에 publish 합니다.

```
./gradlew publishAllPublicationsToMavenCentralRepository --no-configuration-cache
```

이제, Build 성공 메시지가 나오면 끝입니다!
```
...
BUILD SUCCESSFUL in 23s
```

Maven central 포탈 사이트에서 사이트의 Deployments 를 확인해봅니다.
Drop 버튼과 Publish 버튼 모두 활성화 되어 있다면, Publish 버튼을 눌러서 활성화합니다.

몇 분이내에 배포가 완료되어 포탈에서 검색을 통해 라이브러리 페이지에 접근이 가능하며, 라이브러리 import 가 가능합니다.

<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/Pasted image 20240615132141.png" alt="qry" width="600" >
</p>

## 트러블 슈팅

### Invalid token 에러
publish 태스크에서 다음과 같은 에러가 발생할 수 있습니다.
```
Upload failed: {"error":{"message":"Invalid token"}}
```
포탈의 토큰의 유효기간이 만료되었을 수 있으므로 토큰을 재발급하여 다시 토큰 정보를 기입합니다. 혹은, 토큰 정보가 제대로 기입되어 있는지(gradle.properties) 확인합니다.

### Deployments 에서 Dependency version information is missing 에러
publish task 는 성공 했는데, "Dependency version information is missing" 메시지와 함께 에러가 발생한다면, build.gradle.kts 의 dependencies 블록을 확인합니다.
dependencies 에 version 정보가 명확히 기재되어 있어야 합니다. 저는 compose-bom 을 사용해서 의존성을 관리하고 있었는데, 이런 경우 아래 에러 메시지가 발생했습니다.

<p  align="center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/img/Pasted image 20240615131630.png" alt="qry" width="600" >
</p>

> p.s. [Ian Wagner 님의 comment](http://disq.us/p/3015lnr) 에 의하면, compose-bom 은 google maven repo 에 있어서 maven central 에서 찾지 못하는 문제이므로 compose-bom 을 사용하기 위해선 maven central 에 google maven repo 에 대한 정보를 알려주어야 합니다. 꼭 BOM 에 의한 버전관리가 필요하시다면, pom block 에 다음을 작성해 해결할 수 있습니다. 이 코드는 생성되는 POM 파일에 Google Maven 저장소 정보를 추가합니다.

```groovy

mavenPublishing {
    ...

    pom {
        ...

        withXml {
          def repo = asNode().appendNode('repositories').appendNode('repository')
          repo.appendNode('name', 'Google')
          repo.appendNode('id', 'google')
          repo.appendNode('url', ' https://maven.google.com/')
        }
```

하지만 가능한 모든 의존성을 같은 Maven Central 에서 해결할 수 있도록 하는 것이 더 좋은 방향이라고 생각이 되어, 추천드리는 방식은 아닙니다.
