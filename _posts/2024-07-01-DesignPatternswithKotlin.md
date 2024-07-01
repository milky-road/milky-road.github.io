---
title: "코틀린으로 배우는 디자인 패턴 "
description: ""
coverImage: "/assets/img/2024-07-01-DesignPatternswithKotlin_0.png"
date: 2024-07-01 20:46
ogImage: 
  url: /assets/img/2024-07-01-DesignPatternswithKotlin_0.png
tag: Tech
originalTitle: "Design Patterns with Kotlin"
link: "https://medium.com/@narendranathchatterjee/design-patterns-with-kotlin-15bbbaf3f699"
---


# Part 1: 싱글턴 패턴 Kotlin에서

![image](/assets/img/2024-07-01-DesignPatternswithKotlin_0.png)

싱글턴 패턴은 "생성" 패턴입니다.
생성 패턴은 객체를 어떻게 생성하는지에 중점을 둡니다.

싱글턴 패턴은 나쁜 평판을 가지고 있습니다. 그것은 한 때 안티 패턴으로 여겨졌습니다. 뭔가가 어디에나 존재한다면 그것이 좋을 수 없다는 것은 합리적입니다. 그럼에도 불구하고 싱글턴은 어디에서나 나타납니다.

<div class="content-ad"></div>

싱글톤 패턴의 기본 논리는 다음과 같습니다.
1. 시스템 내에서 특정 클래스의 인스턴스는 하나만 존재해야 합니다.
2. 이 인스턴스는 우리의 범위 내 어디에서든 접근 가능해야 합니다. 이 범위는 우리의 프로그램이거나 그 모듈일 수 있습니다.

코틀린은 싱글톤을 매우 쉽게 만들 수 있습니다. 단순히 다음과 같이 사용하면 됩니다.

![이미지1](/assets/img/2024-07-01-DesignPatternswithKotlin_1.png)

![이미지2](/assets/img/2024-07-01-DesignPatternswithKotlin_2.png)

<div class="content-ad"></div>

However, 이것이 잘 맞을 수도 있습니다 considering how 많은 more 단계 is 필요합니다 to get the 같은 functionality in Java or C++.

Shall we look at a Java 예시?

![Java Example](/assets/img/2024-07-01-DesignPatternswithKotlin_3.png)

The steps involved here are
1. Keeping the constructor private
2. Make sure only one instance is created
3. The object is created only when needed and not when the program starts
4. It works properly from multiple threads
5. Doesn’t slow down the program by synchronizing to multiple places.

<div class="content-ad"></div>

그래서 여러분은 이 모든 무거운 작업이 개체 내부적으로 처리된다는 것을 보실 수 있습니다.
Kotlin Object의 주요 차이점은 생성자를 가질 수 없다는 것입니다.
초기화가 필요한 경우(예: 설정)라면 유닛 블록에서 수행할 수 있습니다.
또한 초기화 로직은 액세스할 때까지 비활성화됩니다(지연 초기화)

![Image](https://miro.medium.com/v2/resize:fit:400/0*E3zEnI5Wmxys1sKh.gif)

싱글톤 패턴의 단점은 다음과 같습니다.
1. 과도한 결합
2. 단위 테스트에서의 복잡성

이것은 시리즈 중 첫 번째 블로그입니다. 계속 주목해주세요.
클랩(clap)을 보내주시면 정말 멋질 것 같아요.