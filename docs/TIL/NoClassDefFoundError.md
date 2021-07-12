---
layout: post
title: "NoClassDefFoundError"
author: Twibap
date: '2021-07-12'
category: bug, debug, runtime error, desugaring
---

# NoClassDefFoundError
 - 빌드 시점에는 문제없이 컴파일 하였으나, 런타임 시점에 빌드한 클래스를 찾지 못해 발생하는 버그
   Link: [stackoverflow][https://stackoverflow.com/questions/34413/why-am-i-getting-a-noclassdeffounderror-in-java]
 - Java 8 버전의 람다표현식 관련 패키지를 사용한 부분에서 발생

## 원인
 - Android Gradle 플러그인 4.0.0 이상부터 디슈가링을 지원하지만 사용하고 있지 않았음

## 해결
 - [디슈가링][https://developer.android.com/studio/write/java8-support#library-desugaring] 사용 선언
 ```java
 android {
    defaultConfig {
        // Required when setting minSdkVersion to 20 or lower
        multiDexEnabled true
    }

    compileOptions {
        // Flag to enable support for the new language APIs
        coreLibraryDesugaringEnabled true
        // Sets Java compatibility to Java 8
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:1.1.5'
}
 ```

### 디슈가링이란?
 - 람다표현식 및 람다표현식을 사용하기 위한 패키지가 사용된 부분을 빌드 시점에 구버전 자바에 맞게 일반 표현식으로 표현하여 빌드
 하고, 런타임 시 호출하게 하는 것.