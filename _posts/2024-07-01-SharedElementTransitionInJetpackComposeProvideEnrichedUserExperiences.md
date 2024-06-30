---
title: "Jetpack Compose에서 공유 요소 전환 풍부한 사용자 경험 제공하는 방법"
description: ""
coverImage: "/assets/img/2024-07-01-SharedElementTransitionInJetpackComposeProvideEnrichedUserExperiences_0.png"
date: 2024-07-01 00:06
ogImage: 
  url: /assets/img/2024-07-01-SharedElementTransitionInJetpackComposeProvideEnrichedUserExperiences_0.png
tag: Tech
originalTitle: "Shared Element Transition In Jetpack Compose: Provide Enriched User Experiences"
link: "https://medium.com/proandroiddev/shared-element-transition-in-jetpack-compose-provide-enriched-user-experiences-163d4e435869"
---


![image](https://miro.medium.com/v2/resize:fit:1200/1*XWziJJSgLGdw4aWm9vDCHA.gif)

안녕하세요! 오늘은 공유 요소 전환 또는 컨테이너 변형에 대해 이야기해보려고 해요. 이 애니메이션은 두 UI 요소 사이에 시각적인 연결을 만들어 앱의 미적인 요소와 사용자 경험을 현격하게 향상시킵니다. 화면 간의 전환을 실현하여 두드러지지 않고 통합되는 것은 사용자의 참여와 앱 내 공간 인식을 유지하는 데 도움이 됩니다.

공유 요소 전환 애니메이션을 사용하면 사용자의 주요 요소에 집중할 수 있어 인지 부하와 혼란을 줄이고 전반적인 사용자 경험을 향상시킬 수 있습니다. 이러한 애니메이션을 통해 앱 내비게이션을 더 직관적으로 만들고 동적이고 매력적인 느낌을 주어 상호작용 품질을 현격하게 향상시킬 수 있습니다.

Jetpack Compose에서는 LookaheadScope 또는 Orbital과 같은 라이브러리를 사용하여 공유 요소 전환을 구현할 수 있습니다. 그러나 이러한 애니메이션을 Compose Navigation 라이브러리와 통합하는 것에는 아직 일부 제한 사항이 있습니다.

<div class="content-ad"></div>

행운히도, Compose UI 버전 1.7.0-alpha07에서는 공유 요소 전환을 위한 새로운 API가 도입되었습니다. 이 글에서는 최신 버전의 Compose UI를 사용하여 다양한 상황에서 공유 요소 전환과 컨테이너 변형을 어떻게 원활하게 구현하는지 알아보겠습니다.

# 의존성 구성

새로운 공유 요소 전환 API를 사용하기 위해서는 다음과 같이 Jetpack Compose UI 및 animation의 최신 버전(1.7.0-alpha07 이상)을 사용해야 합니다:

```js
dependencies {
    implementation(androidx.compose.ui:ui:1.7.0-alpha07)
    implementation(androidx.compose.animation:animation:1.7.0-alpha07)
}
```

<div class="content-ad"></div>

# SharedTransitionLayout 및 Modifier.sharedElement

Compose UI 및 애니메이션 버전 1.7.0-alpha07은 공유 요소 전환을 구현할 수 있는 주요 API를 도입했습니다. SharedTransitionLayout 및 Modifier.sharedElement이 그 중 하나입니다.

- SharedTransitionLayout: 이 Composable은 SharedTransitionScope를 제공하는 컨테이너 역할을 합니다. 이를 통해 Modifier.sharedElement을 비롯한 기타 관련 API를 사용할 수 있습니다. 공유 요소 전환의 핵심 기능은 이 Composable 내에서 발생합니다. 내부적으로 SharedTransitionScope는 LookaheadScope API를 활용하여 이러한 전환을 용이하게 합니다. 그러나 LookaheadScope에 대해 상세한 지식은 필요하지 않습니다. 새 API가 이 복잡성을 효과적으로 캡슐화했기 때문입니다.
- Modifier.sharedElement: 이 modifier는 SharedTransitionLayout 내에서 공유 전환을 수행해야 하는 Composable를 식별합니다. 동일한 SharedTransitionScope 내의 다른 Composable과 변환을 수행해야 함을 효과적으로 표시합니다.

이제 두 API를 어떻게 활용할 수 있는지 예제를 살펴보겠습니다:

<div class="content-ad"></div>

자세히 살펴봅시다. 이 예제를 자세히 살펴보겠습니다. Row에는 이미지와 텍스트가 가로로 표시됩니다. Row를 클릭하면 이미지와 텍스트가 세로로 배열된 Column으로 변환됩니다. SharedTransitionLayout 내에서 sharedElement 수정자가 사용된 것을 알 수 있습니다. 다음 세 가지 매개변수를 받습니다:

- state: SharedContentState는 sharedBounds/sharedElement의 속성에 액세스할 수 있도록 설계되었으며 동일한 키의 일치 여부(SharedTransitionScope 내에서 일치하는 것이 발견되었는지 여부 등)를 확인합니다. rememberSharedContentState API를 사용하여 SharedContentState 인스턴스를 만들 수 있습니다. 애니메이션 중에 일치시켜야 할 구성 요소를 식별하는 키를 제공하십시오. 이 키는 전환 발생 시 올바른 구성 요소가 연결되도록 합니다.
- animatedVisibilityScope: 이 매개변수는 animatedVisibilityScope의 대상 상태를 기반으로 공유 요소의 경계를 정의합니다. NavGraphBuilder.composable 함수와 통합할 수 있어서 Compose 네비게이션 라이브러리와 원활하게 작동할 수 있습니다. 나중 섹션에서 자세히 살펴보겠습니다.
- boundsTransform: 이 람다 함수는 FiniteAnimationSpec를 가져오고 반환하며, 공유 요소 전환에 적합한 애니메이션 명세를 적용하는 데 사용됩니다.

위의 코드를 실행한 후 다음과 같은 결과가 나타날 것입니다:

![Click here](https://miro.medium.com/v2/resize:fit:752/1*YACtgRhSLW3hYGgfYHLgSw.gif)

<div class="content-ad"></div>

# 네비게이션과 공유 요소 전환이 함께하는 방법

새로운 공유 요소 전환 API에서는 Compose Navigation 라이브러리와 호환성이 있습니다. 이 향상된 기능을 사용하면 앱 전체에서 더 부드러운 네비게이션 흐름을 가능케 하도록 다른 네비게이션 그래프에 있는 Composable 함수들 간의 공유 요소 전환을 구현할 수 있습니다.

이제 네비게이션 라이브러리와 공유 요소 전환을 통합하는 방법을 알아보겠습니다. 두 가지 간단한 화면을 만들어보겠습니다: 홈 화면(목록 포함)과 상세 화면. 이를 통해 LazyColumn과 같은 방법으로 이 두 개의 다른 네비게이션 그래프 사이에서 요소들을 부드럽게 전환하는 방법을 보여줄 것입니다.

먼저, 아래 예시처럼 빈 Composable 화면으로 NavHost를 설정해야 합니다:

<div class="content-ad"></div>

네비게이션 라이브러리를 사용하여 공유 요소 전환을 구현하려면 NavHost를 SharedTransitionLayout 안에 넣는 것이 중요합니다. 이 설정을 통해 다양한 네비게이션 목적지 간에 공유 요소 전환을 제대로 처리할 수 있습니다.

그런 다음, 이름과 이미지에 대한 속성이 포함된 Pokemon이라는 샘플 데이터 클래스를 정의하세요. 그런 다음, 아래 예시에 설명된 대로 모의 Pokemon 데이터 목록을 만들어보세요:

이제 홈 화면을 구성하는 composable을 구현해봅시다. 각 목록 항목은 이미지와 텍스트가 가로로 나란히 표시되는 행으로 표시됩니다:

본 예시에서는 이미지와 텍스트 컴포넌트의 수정자(modifiers)가 Modifier.sharedElement 함수를 사용한다는 것을 알 수 있습니다. 각 요소에는 고유한 키 값을 할당하여 목록의 여러 항목 중에서 구별할 수 있도록 합니다.

<div class="content-ad"></div>

이제 공유된 요소 전환이 올바르게 작동하도록 하려면 출발 화면의 요소에 할당된 구체적인 키 값이 이동 흐름에서 대응되는 목적지 화면의 요소에 사용된 값과 일치해야 합니다. 다른 화면을 탐색하면서 다양한 컴포저블 간에 원활한 전환을 가능하게 하는 데 이 일치가 중요합니다.

마지막으로 세부 화면을 구현해 봅시다. 이 화면은 간단히 이미지와 텍스트를 표시할 것입니다. 게다가, 화면을 클릭할 때 홈 화면으로 돌아가는 기능도 포함될 것입니다:

따라서 전체 코드는 아래와 같이 될 것입니다:

예시 코드를 실행하면 다음 결과를 관찰할 수 있습니다:

<div class="content-ad"></div>

![image](https://miro.medium.com/v2/resize:fit:866/1*vBg1PonRrhkP_hS5q0i8tw.gif)

If you want to see practical examples, check out the Pokedex-Compose open-source project on GitHub. It showcases the shared element transition APIs in action.

# Modifier.sharedBounds for Container Transform

Now, let's dive into the container transform. Modifier.sharedBounds() is similar to Modifier.sharedElement(), but with a key difference. Modifier.sharedBounds() is best suited for content that looks different visually during transitions. On the other hand, Modifier.sharedElement() is used for content that remains visually consistent, like images. This difference is particularly helpful in scenarios like the container transform pattern.

<div class="content-ad"></div>

이전 예제를 활용하여 컨테이너 변환을 구현하는 것은 간단합니다. Composable 트리의 루트 계층에 Modifier.sharedBounds()를 추가하고 Modifier.sharedElement() 함수를 제거하면 됩니다. 이 수정을 통해 UI 구성 요소 간에 시각적으로 다른 요소들 간의 전환을 가능케 합니다.

이전 섹션의 코드를 아래와 같이 수정해 보겠습니다: 

세부사항을 살펴보면, 홈 컴포저블의 Row와 세부사항 컴포저블의 Column이 모두 위의 예제에서 보여주는 것처럼 Modifier.sharedBounds()를 활용하고 있음을 알 수 있습니다. 이게 전부입니다! 코드를 실행하면 다음과 같은 애니메이션의 결과를 확인할 수 있을 것입니다:

![animation](https://miro.medium.com/v2/resize:fit:926/1*VgYm12rOK7UdNWpM1C4rhQ.gif)

<div class="content-ad"></div>

# 결론

이번 글에서는 다양한 예시를 활용하여 공유 요소 전환과 컨테이너 변형을 구현하는 방법을 배웠습니다. Jetpack Compose가 얼마나 발전했는지 보는 것은 인상적입니다. 이를 통해 복잡한 애니메이션을 쉽게 만들 수 있게 되었습니다. 이 두 종류의 애니메이션은 화면 이동을 직관적이고 동적으로 만들어 사용자 경험을 크게 향상시킬 수 있습니다. 그러나 이러한 애니메이션을 분별적으로 사용하는 것이 중요합니다. 과도하게 사용하는 대신 적절하게 활용함으로써 자연스럽고 매력적인 사용자 경험을 보장할 수 있습니다.

만약 실제 적용 사례를 보고 싶다면, GitHub의 Pokedex-Compose 오픈 소스 프로젝트를 살펴보세요. 이 프로젝트는 공유 요소 전환 API와 여러 Jetpack 라이브러리가 어떻게 작용하는지를 보여줍니다.

이 글에 대한 질문이나 피드백이 있다면, 저자를 Twitter(@github_skydoves) 또는 GitHub에서 찾을 수 있습니다. 또한 Stream을 최신 상태로 유지하고 싶다면, 훌륭한 기술 콘텐츠를 더 보려면 Twitter(@getstream_io)를 팔로우하세요.

<div class="content-ad"></div>

행복한 코딩하세요!

— Jaewoong