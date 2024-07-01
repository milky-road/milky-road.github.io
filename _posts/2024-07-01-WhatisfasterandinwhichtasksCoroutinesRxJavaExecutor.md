---
title: "코루틴, RxJava, Executor 중 어떤 것이 더 빠르고 어떤 작업에 적합한가"
description: ""
coverImage: "/assets/img/2024-07-01-WhatisfasterandinwhichtasksCoroutinesRxJavaExecutor_0.png"
date: 2024-07-01 20:47
ogImage: 
  url: /assets/img/2024-07-01-WhatisfasterandinwhichtasksCoroutinesRxJavaExecutor_0.png
tag: Tech
originalTitle: "What is faster and in which tasks? Coroutines, RxJava, Executor?"
link: "https://medium.com/proandroiddev/what-is-faster-and-in-which-tasks-coroutines-rxjava-executor-952b1ff62506"
---


<img src="/assets/img/2024-07-01-WhatisfasterandinwhichtasksCoroutinesRxJavaExecutor_0.png" />

에버 원더했던 적 있나요? 어떤 멀티스레딩 프레임워크가 빠른지 궁금했던 적이 가끔 있었어요. 어느 날, 운명이 나를 이 질문을 조사하게 이끌었죠. 그래서 여러분도 궁금하다면, 저는 그것들을 테스트하고 비교해본 후 결과를 공유하려고 해요.

# 문제

먼저, 이 일을 왜 했는지 알아보겠어요. 저는 간단한 작업을 마주하게 되었어요: 여러 개의 콜백을 호출하는 시스템이 있었죠. 모든 콜백은 가능한 빨리 완료되어야 했어요.

<div class="content-ad"></div>

Sorry for the confusion but could you please provide the text that needs to be translated into Korean?

<div class="content-ad"></div>

어떤 테스트는 Rx와 Coroutines와 같은 두 기술을 비교하는 간단한 테스트들이었고, 다른 것들은 내 의견으로는 너무 구체적인 테스트 케이스를 가졌어요. 예를 들어, 단순 산술 연산이나 데이터베이스 요청만을 테스트하는 것과 같은 것들이 그랬죠. 

어쨌든, 이에 대해 너무 만족스럽지 않았고, 제가 직접 테스트를 해보기로 결정했어요. 

멀티스레드 프레임워크의 모든 사용 사례를 다 다룰 수 없다는 건 이해해요. 제 업무와 관련된 것들만 테스트할 거예요... 그래서 100% 완벽한 테스트는 아니지만 상당히 철저하게 할 거예요.

## 도구들

<div class="content-ad"></div>

자, 마법사들! 이번에는 측정 도구에 대해 이야기해볼게. 안드로이드 개발자로서 안드로이드 도구를 사용해볼 거야. 이론적으로는 컴퓨터에서 JVM에서 멀티스레딩 테스트를 실행하는 것과 안드로이드 기기에서 실행하는 것은 조금 차이가 있을 수 있어. 하지만 실제로는 큰 차이가 없고 전체 테스트 결과에 영향을 미치지 않아.

그리고, 안드로이드 기기에서 코드 성능을 테스트할 수 있는 젯팩 마이크로벤치마크 도구도 있어.

마이크로벤치마크 측정 테스트는 평범한 인스트루멘테이션 테스트와 비슷한데, 특정한 Rule인 벤치마크룰을 사용한다는 점만 다를 뿐이야.

```kotlin
@get:Rule
val benchmarkRule = BenchmarkRule()

@Test
fun sampleTest() {
   benchmarkRule.measureRepeated {
       // 우리가 이걸 측정할 거야
   }
}
```

<div class="content-ad"></div>

그 결과, 실행에 대한 정보를 포함한 JSON을 얻습니다. 최소 및 최대 시간과 무엇보다 중앙값 시간이 포함되어 있습니다.

```json
    "benchmarks": [
        {
            "name": "sampleTest",
            "params": {},
            "className": "com.test.benchmark.ExampleBenchmark",
            "totalRunTimeNs": 85207217833,
            "metrics": {
                "timeNs": {
                    "minimum": 9.82149833E8,
                    "maximum": 1.019338584E9,
                    "median": 1.004151917E9,
                    "runs": [...]
                },
                "allocationCount": {
                    "minimum": 324.0,
                    "maximum": 324.0,
                    "median": 324.0,
                    "runs": [...]
                }
            },
            "sampledMetrics": {},
            "warmupIterations": 3200,
            "repeatIterations": 5000,
            "thermalThrottleSleepSeconds": 0
        }
    ]
```

이 JSON에는 반복 횟수, 객체 할당 횟수 등 다른 정보가 포함되어 있습니다. 현재로서는 우리에게 중요하지는 않습니다.

# 벤치마크 테스트 케이스

<div class="content-ad"></div>

프레임워크 간의 주요 차이점은:

- 프레임워크가 단일 스레드를 생성하는 데 걸리는 시간;
- 프레임워크가 작업을 효율적으로 빠르게 스레드 간에 분배하는 능력.

이러한 프레임워크 간의 차이를 평가하기 위해 테스트 케이스 결과를 탐색합니다.

이제 우리는 무엇을 테스트할지 결정해야 합니다. 테스트 데이터부터 시작해 보죠.

<div class="content-ad"></div>

# 테스트 데이터

이건 별거 아니에요. 0부터 99까지의 100개 항목을 가진 ArrayList를 만들 거예요.

```kotlin
private fun createTestList(): List<Int> {
   return List(100) { it }
}
```

그리고 각 항목에 대해 몇 가지 작업을 수행할 거예요.

<div class="content-ad"></div>

이제 테스트 케이스를 살펴보겠습니다.

# 단일 스레드

단일 스레드 테스트 케이스부터 시작해봅시다. 이러한 테스트 케이스의 결과를 비교할 때, 각 프레임워크 간의 첫 번째 차이점인 단일 스레드를 생성하는 데 걸리는 시간을 신중하게 살펴보겠습니다. 한 스레드만 있는 경우, 프레임워크가 스레드를 초기화하고 생성하는 데 걸리는 시간을 확인할 수 있습니다.

## 직접 호출

<div class="content-ad"></div>

**첫 번째 테스트 케이스**에서는 메서드를 직접 호출하는 것으로 진행할 거예요. 어떤 프레임워크도 사용하지 않습니다.

```kotlin
@Test
fun directInvoke() {
   val list = createTestList()
   benchmarkRule.measureRepeated {
       list.forEach { action(it) }
   }
}
```

## RxJava

두 번째 테스트 케이스에서는 Rx를 사용할 거예요. **Completable** 내에서 작업을 수행할 거예요. 각 작업마다 별도의 **Completable**이 생성될 거예요. 스케줄러는 **Scheduler.single**이 될 거예요. 이를 통해 모든 작업을 단일 스레드에서 실행할 수 있습니다.

<div class="content-ad"></div>

그리고 완료를 기다려야하기 때문에, 결과 Completable에 blockingAwait를 호출할 것입니다.

```kotlin
@Test
fun rxOne() {
   val list = createTestList()
   val scheduler = Schedulers.single()
   benchmarkRule.measureRepeated {
       val completables = list.map {
           Completable.fromAction {
               action(it)
           }.subscribeOn(scheduler)
       }
       Completable.merge(completables).blockingAwait()
   }
}  
```

## Kotlin 코루틴

젊고 유망한 Kotlin 코루틴 프레임워크 없이 어디에 있었을까요? 코루틴을 실행하기 위해 await를 통해 코루틴의 완료를 기다리려면 async를 통해 동작을 실행할 것입니다. 따라서 각 작업에 대해 별도의 코루틴이 있을 것입니다. 모든 것이 단일 스레드에서 실행되게 하려면, 간단히 runBlocking을 사용하면 됩니다.

<div class="content-ad"></div>

```kotlin
@Test
fun coroutineOne() {
   val list = createTestList()
   benchmarkRule.measureRepeated {
       runBlocking {
           list.map {
               async {
                   action(it)
               }
           }.forEach { it.await() }
       }
   }
}
```

## Kotlin Flow

Let’s also consider Flow in comparison. Directly comparing coroutines and Rx may not be the best approach, as both technologies are used in different ways and employ different concepts, despite their association with multithreading.

We create one flow for each action, combine them all together, and then simply collect them. There will be a separate Flow for each action. Meanwhile, we don’t specify Dispatcher anywhere, so everything happens on the same thread.


<div class="content-ad"></div>

```kotlin
@Test
fun flowOne() {
    val list = createTestList()
    benchmarkRule.measureRepeated {
        runBlocking {
            val flows = list.map {
                flow {
                    val result = action(it)
                    emit(result)
                }
            }
            flows.merge().collect()
        }
    }
}
```

## 스레드의 수는 CPU 스레드의 수와 같다

한 스레드만 사용하는 것은 다중 스레딩을 위한 프레임워크가 있는 상황에서 최선의 선택으로 보이지 않습니다.

스레드 풀의 스레드 수가 CPU 스레드 수와 동일한 상황을 고려해보겠습니다. 이 테스트 케이스의 결과를 비교함으로써 두 번째 차이점인 프레임워크가 작업을 효율적이고 빠르게 스레드 간에 분배할 수 있는 능력을 볼 수 있습니다. 각 작업에 충분한 스레드가 없기 때문에 프레임워크는 어차피 작업을 분배해야 할 것입니다.


<div class="content-ad"></div>

프로세서 코어 간 작업이 고르게 분배될 것이라고 기대해서는 안 됩니다. 운영 체제와 CPU가 결정하는 것은:

- 어떤 작업이 실행될 것인지,
- 언제 실행될 것인지,
- 누가 실행할지입니다.

현대의 "큰" CPU는 SMT/HyperThreading을 사용하여 각 CPU 코어에 여러 논리적 스레드를 만들 수 있습니다. CPU 코어는 독립적으로 명령을 실행할 수 있는 물리적 단위입니다. 반면 스레드는 단일 코어에서 실행할 수 있는 논리적인 소프트웨어 단위입니다. 또한 모바일 CPU 및 최신 Intel CPU는 서로 다른 유형의 CPU 코어를 사용하는 방식을 채택하고 있습니다. 대형 코어는 복잡한 작업에 적합하며, 중간 및 절전 코어는 성능이 크게 다릅니다.

그래서 이 묶음을 적은 수의 스레드를 가진 풀로 생각할 수 있습니다.

<div class="content-ad"></div>

이 CPU 스레드 수를 결정하는 탁월한 방법이 있어요:

```java
Runtime.getRuntime().availableProcessors()
```

저의 테스트 기기에서는 이 값이 여덟인데요.

## RxJava

<div class="content-ad"></div>

우선 Rx부터 시작해봅시다. Rx는 Scheduler.Computation을 사용하여 원하는 동작을 구현합니다. 실제로 코드는 단일 스레드를 사용하는 것과 동일합니다. 달라지는 것은 각 개별 Completable에 대해 Scheduler.computation()을 사용하는 점 뿐입니다.

```kotlin
@Test
fun rxCPU() {
   val list = createTestList()
   val scheduler = Schedulers.computation()
   benchmarkRule.measureRepeated {
       val completables = list.map {
           Completable.fromAction {
               action(it)
           }.subscribeOn(scheduler)
       }
       Completable.merge(completables).blockingAwait()
   }
}
```

## 코틀린 코루틴

이제 코루틴을 살펴보겠습니다. 코루틴은 Dispatchers.Default를 사용합니다. Dispatchers.Default에 대한 설명은 다음과 같습니다:

<div class="content-ad"></div>

그럼, “병렬성”의 최대 레벨은 기본적으로 CPU 코어의 수와 동일합니다. 바로 필요한 것과 일치하죠.

이 코드는 단일 코어 버전과 유사합니다. 이제 작업은 Dispatchers.Default와 함께 withContext 내에서 실행됩니다.

```kotlin
@Test
fun coroutineCPU() {
   val list = createTestList()
   val dispatcher = Dispatchers.Default
   benchmarkRule.measureRepeated {
       runBlocking {
           list.map {
               async {
                   withContext(dispatcher) {
                       action(it)
                   }
               }
           }.forEach { it.await() }
       }
   }
}
```

하지만 중요한 세부 사항이 있어요… Dispatchers.Default를 자세히 살펴보면 이 생성자를 찾을 수 있습니다.

<div class="content-ad"></div>

**DefaultScheduler**이라는 내부 객체가 정의되어 있습니다. 이 객체는 스케줄러 코루틴 디스패처를 만들기 위해 두 개의 상수를 전달한다는 점이 의심스럽습니다: **CORE_POOL_SIZE**, 이 경우 CPU 스레드 수에 해당하며, **MAX_POOL_SIZE**, 그리고 이 값은 두 백만입니다.

더 자세히 살펴보면 이러한 변수들이 **CoroutineScheduler**를 만들기 위해 사용된다는 것을 알 수 있습니다.

```kotlin
private fun createScheduler() = CoroutineScheduler(
  corePoolSize, maxPoolSize, idleWorkerKeepAliveNs, schedulerName
)
```

<div class="content-ad"></div>

요즘 문서에는 이렇게 써 있어요:

Dispatchers.Default는 CPU 스레드의 수에 제한받지 않는다는 걸 보여줍니다.

Dispatchers.Default와 Scheduler.Computation을 직접적으로 비교하는 것은 잘못되어요. 때로는 Dispatchers.Default가 추가적인 스레드를 사용할 수도 있어요. 다행히도 문서에는 두 개를 더 공평하게 비교하는 방법에 대한 정보도 제공되어 있어요. 이를 위해 LimitingDispatcher를 사용하면 됩니다. Dispatchers.Default의 limitedParallelism 메서드를 호출할 때 CPU 스레드 수를 함께 지정해주면 돼요.

```kotlin
@Test
fun coroutineCPULimit() {
   val list = createTestList()
   val threadCount = Runtime.getRuntime().availableProcessors()
   val dispatcher = Dispatchers.Default.limitedParallelism(threadCount)
   benchmarkRule.measureRepeated {
       runBlocking {
           list.map {
               async {
                   withContext(dispatcher) {
                       action(it)
                   }
               }
           }.forEach { it.await() }
       }
   }
}
```

<div class="content-ad"></div>

## 코틀린 Flow

플로우에 대해 같은 작업을 수행해 보겠습니다. Dispatchers.Default를 사용하세요.

```kotlin
@Test
fun flowCPU() {
   val list = createTestList()
   val dispatcher = Dispatchers.Default
   benchmarkRule.measureRepeated {
       runBlocking {
           val flows = list.map {
               flow {
                   val result = action(it)
                   emit(result)
               }.flowOn(dispatcher)
           }
           flows.merge().collect()
       }
   }
}
```

그리고 Dispatchers.Default를 제한하여 사용하세요.

<div class="content-ad"></div>

```kotlin
@Test
fun flowCPULimit() {
   val list = createTestList()
   val threadCount = Runtime.getRuntime().availableProcessors()
   val dispatcher = Dispatchers.Default.limitedParallelism(threadCount)
   benchmarkRule.measureRepeated {
       runBlocking {
           val flows = list.map {
               flow {
                   val result = action(it)
                   emit(result)
               }.flowOn(dispatcher)
           }
           flows.merge().collect()
       }
   }
}
```

## Java Executor

우리는 코루틴의 복잡성을 모두 탐색해야 했기 때문에, 이런 복잡성들을 사용해보는 건 어떨까요? 그래서 우리는 Executor를 사용해보려고 합니다.

우선 Executors.newFixedThreadPool을 사용하여 Executor를 생성해 봅시다. 이것은 단순히 CPU 코어의 수에 의해 제한된 Executor를 생성합니다. 이 경우에는 CPU 코어의 수가 됩니다.


<div class="content-ad"></div>

기존 measureRepeated 전에는 Executor를 생성하므로 Executor를 만드는 것이 간단한 작업이 아닙니다. 그런 다음, 작업과 함께 Executor에서 submit을 호출합니다. 완료를 기다리기 위해 get 메서드를 사용합시다.

```kotlin
@Test
fun executorFixedCPU() {
   val list = createTestList()
   val threadCount = Runtime.getRuntime().availableProcessors()
   val executorService = Executors.newFixedThreadPool(threadCount)
   benchmarkRule.measureRepeated {
       val futures = list.map { executorService.submit { action(it) } }
       futures.forEach { it.get() }
   }
}
```

Executors.newWorkStealingPool 메서드를 사용하여 더 흥미로운 두 번째 유형의 Executor를 만들 것입니다. 실제로 지금은 일정한 수의 스레드를 가진 Executor를 만듭니다. 그러나 메서드 이름에 "Stealing"이라는 단어가 단순한 우연은 아닙니다. 이 Executor의 스레드는 현재 스레드가 비어 있고 다른 스레드에 대기 중인 작업이 있을 때 해당 스레드에서 작업을 훔칠 수 있습니다. 이를 통해 공통 대기열을 정리하는 데 도움이 될 수 있습니다.

코드 측면에서는 모든 것이 이전과 동일합니다. Executor를 만드는 방법만 다를 뿐입니다.

<div class="content-ad"></div>

```kotlin
@Test
fun executorStealCPU() {
   val list = createTestList()
   val threadCount = Runtime.getRuntime().availableProcessors()
   val executorService = Executors.newWorkStealingPool(threadCount)
   benchmarkRule.measureRepeated {
       val futures = list.map { executorService.submit { action(it) } }
       futures.forEach { it.get() }
   }
}
```

# 한 번에 한 쓰레드

이제 무제한의 (사실 무한은 아니고 그냥 많은) 쓰레드를 할당하는 가능성을 탐색해봅시다. 우리에겐 100개의 액션이 있으니, 실제로 100개의 쓰레드를 할당할 수 있습니다. 그 이상은 필요하지 않죠.

## RxJava


<div class="content-ad"></div>

Rx를 우선 살펴보겠습니다. `Scheduler.io`는 이러한 업무를 담당합니다. `Scheduler.io`에는 스레드 캐시가 있습니다. 만약 사용 가능한 빈 스레드가 있는 경우 캐시에서 하나를 가져옵니다. 그렇지 않으면 새 스레드를 생성합니다. 코드는 `Scheduler.computation`와 동일하지만 다른 스케줄러를 사용합니다.

```kotlin
@Test
fun rxIo() {
   val list = createTestList()
   val scheduler = Schedulers.io()
   benchmarkRule.measureRepeated {
       val completables = list.map {
           Completable.fromAction {
               action(it)
           }.subscribeOn(scheduler)
       }
       Completable.merge(completables).blockingAwait()
   }
}
```

## Kotlin 코루틴

코루틴에서 유사한 작업은 `Dispatchers.IO`의 역할이며, 우리는 익숙한 코드에 그냥 삽입하면 됩니다.

<div class="content-ad"></div>


@test
간접 조건을 조정하십시오. 더 많은 정보가 필요한 경우 도와 드리겠습니다 :)


<div class="content-ad"></div>

## Java Executor

포루투니티로 Executor는 아주 멋진 API를 갖고 있지 않아요. 그래서 여기에 백 개의 스레드를 할당해 줄 거에요. 액션이 백 개밖에 없으니까, 그 이상의 스레드는 필요 없을 거에요.


@Test
fun executorFixedIo() {
   val list = createTestList()
   val executorService = Executors.newFixedThreadPool(100)
   benchmarkRule.measureRepeated {
       val futures = list.map { executorService.submit { action(it) } }
       futures.forEach { it.get() }
   }
}


newWorkStealingPool을 위해서도 똑같이 할 거에요.

<div class="content-ad"></div>

이것은 모든 프레임워크의 테스트 케이스 표입니다.

![테스트 케이스 표](/assets/img/2024-07-01-WhatisfasterandinwhichtasksCoroutinesRxJavaExecutor_1.png)

그리고 마지막으로, 테스트를 실행하세요.

<div class="content-ad"></div>

# 테스트

하하, 조금 속였죠. 먼저, 행동의 범주를 고려해 봅시다.

총 다섯 가지 범주가 있습니다:

- 산술 — 간단한 산술 연산;
- 목록조작 — 객체 조작;
- 저장소 — 저장소 작업 시뮬레이션;
- 네트워크 — 네트워크 작업 시뮬레이션;
- 혼합 — 모든 이전 스크립트의 조합, 원본 작업에서 행동이 얼마나 어려울지 모르기 때문에.

<div class="content-ad"></div>

간단하게 시작해봅시다.

# 산술

각 항목에 대해 0부터 99까지의 작업을 수행할 것임을 상기시키고 싶습니다. 이 경우, 연산을 조금 더 복잡하게 만들기 위해 숫자를 실수로 변환하고 해당 숫자의 제곱으로 올려야 합니다. 0의 제곱부터 99의 제곱까지. 물론 어느 시점에서는 실수의 한계에 도달할 수도 있지만, 괜찮습니다.

```kotlin
fun arithmetic(seed: Int): Int {
    return seed.toFloat().pow(seed.toFloat()).toInt()
}
```

<div class="content-ad"></div>

일반적으로, 이 범주로 분류할 수 있는 작업에는 생성자를 사용하여 객체를 만들거나 속성에 액세스하는 것이 포함됩니다. 이러한 작업은 완료하는 데 오랜 시간이 걸리지 않습니다.

결과는 무엇인가요?

![Image](/assets/img/2024-07-01-WhatisfasterandinwhichtasksCoroutinesRxJavaExecutor_2.png)

예상대로, directInvoke가 최고의 성적을 거두었습니다. 가장 가까운 경쟁자보다 70배 빨랐습니다. 이는 이러한 작업이 매우 간단하고 멀티스레딩 프레임워크를 사용함으로써 추가적인 오버헤드가 작업 자체의 비용보다 10배 높을 수 있기 때문에 이해할 만합니다.

<div class="content-ad"></div>

하지만 놀랄 만한 일로 두 번째와 세 번째 자리는 예상치 못한 결과가 나왔어요. 둘 다 스레드 수가 고정된 Executors였답니다. 처음에는 directInvoke가 첫 번째로 나올 것으로 예상하고, 그 다음에는 단일 스레드에서의 테스트 케이스, 그 후에 CPU가 그리고 마지막으로 IO가 일어날 것으로 기대했죠. 그러나 스레드 수가 고정된 Executors는 이런 논리를 완전히 깨버렸어요. 동시에 Executors.newWorkStealingPool를 사용해 만든 Executor는 "무한" 스레드 수로는 성능이 좋지 못하게 나타났어요.

Dispatchers.Default의 시간이 제한과 없는 경우에 각각 다르다는 것도 볼 수 있어요, 그래서 제한이 설정된 거에는 이유가 있어요.

하나의 작업을 처리하는 데 Flow가 가장 큰 오버헤드를 가지고 있다는 것을 주목하는 것도 흥미로운 포인트에요. 다른 모든 경쟁 상대보다 느려요. 그리고 코루틴이 Rx보다 한 스레드에서 더 나을 수 있지만, 스레드 수가 증가할수록 Rx가 선두에 섰다는 점을 알 수 있어요.

요약하자면, 이러한 작업 범주에서는 directInvoke가 가장 효율적이라는 것을 볼 수 있어요. 따라서 이와 같은 작업에는 멀티스레딩 프레임워크를 사용하는 것이 좋지 않아요. 대신 단일 스레드나 소수의 스레드를 사용하는 풀을 사용해야 해요.

<div class="content-ad"></div>

# listsManipulation

지금은 더 복잡한 것, 즉 객체 조작을 살펴 보겠습니다. 객체를 리스트에 추가하면 객체를 다루기가 더 쉬워집니다. 이 작업 범주에는 예를 들어 POJO 매핑이 포함될 수 있습니다.

이 작업 안에서는 매개변수로 받은 숫자와 같은 크기의 새 리스트를 만들고 간단한 연산을 수행한 후, 리스트를 맵으로 변환하고 다시 몇 가지 작업을 수행한 다음 이 컬렉션을 필터링하는 것과 같은 일들을 하게 됩니다. 전반적으로 리스트를 조작합니다.

여기서 복잡성의 기초는 심지어 작업 자체도 아닙니다. 이것은 일반적인 변경 불가능한 리스트이며 시퀀스가 아닙니다. 이는 각 작업 후에 새로운 리스트가 생성된다는 것을 의미합니다.

<div class="content-ad"></div>

```kotlin
fun listsManipulation(seed: Int): Int {
    val result = List(seed) { it }
        .map { it.toFloat() }
        .map { it + 0.3f }
        .associateBy { it.toString() }
        .mapValues { it.value * it.value }
        .filter { it.value > 5f }
    
    return seed
}
```

어떻게 되었나요?

![Image](/assets/img/2024-07-01-WhatisfasterandinwhichtasksCoroutinesRxJavaExecutor_3.png)

directInvoke는 빠르게 선두를 내줬어요. 객체 조작과 같은 기본 작업조차 어려움이 있습니다.




<div class="content-ad"></div>

리드는 Executors.newWorkStealingPool로 생성된 Executor가 차지합니다. 이것은 더 간단한 버전인 Executors.newFixedThreadPool보다 효율적입니다.

Rx는 약간 앞으로 전진합니다. 앞으로는 Scheduler.computation이 있습니다. 이론적으로, 이 작업에는 더 적은 리소스를 사용하기 때문에 이것이 선호되는 선택입니다.

단일 스레드 테스트 케이스는 목록의 끝에 있지만, directInvoke는 여전히 중간에 남아 있습니다.

flowCPU와 coroutineCPU는 목록의 맨 아래에 있습니다. 이 테스트 케이스들은 CPU 코어의 수만큼의 스레드를 가지고 있으며, 어떠한 제한도 없습니다. 테스트를 다시 실행해봤지만 결과는 동일했습니다. 버그가 있을 수 있다고 생각해 coroutine을 업데이트했지만, 변화는 없었습니다.

<div class="content-ad"></div>

Over the course of my research, I've discovered that CPU test cases tend to have a faster execution time compared to IO for tasks at this level of complexity. 

### Storage

When it comes to database or file access, it's crucial to address the blocking operation efficiently. In scenarios like this, where we need to put a thread to sleep within an action, we must tread carefully. The sleep duration typically ranges between 500 and 1,490 microseconds, or 0.5 to 1.5 milliseconds. However, relying on the traditional `Thread.sleep` method can lead to inaccurate outcomes due to the phenomenon known as "busy waiting." To overcome this issue, I recommend using `LockSupport.parkNanos` to properly transition the thread into a sleep state.

```kotlin
private fun storage(seed: Int): Int {
   val timeInMicroseconds = 500 + 10 * seed.toLong()
   LockSupport.parkNanos(TimeUnit.MICROSECONDS.toNanos(timeInMicroseconds))
   return seed
}
```

<div class="content-ad"></div>

이제 결과가 나왔어요.

![WhatisfasterandinwhichtasksCoroutinesRxJavaExecutor](/assets/img/2024-07-01-WhatisfasterandinwhichtasksCoroutinesRxJavaExecutor_4.png)

IO 테스트 케이스를 먼저 실행하고, 그 다음에 CPU를 실행하고, 마지막으로 싱글 스레드를 실행했어요. 교과서에서 본 듯한 내용이죠? "IO 스레드 풀은 IO 작업에 가장 적합하다"라고 말해요.

rxIO 테스트 케이스는 flowsIO와 coroutinesIO보다 빠르네요. 아마도 스레드를 생성하는 데 필요한 자원이 가장 적게 사용된 것 같아요. 하지만... 이런 작업에 그다지 중요한 게 아니에요. 차이는 단지 밀리초에 불과해요. 프레임워크 간의 차이가 거의 없어진 것처럼 보여요.

<div class="content-ad"></div>

# 네트워크

음, 네트워크 케이스를 테스트하지 않을 수 없겠죠? 모든 것이 이전 테스트 때와 똑같습니다. 이번에는 스레드가 잠시 동안 0 밀리초에서 99 밀리초 동안 자고 있는 것이죠. 이 테스트에서는 "바쁜 대기"로 인한 지연이 더 이상 심각한 역할을 하지 않습니다. 그래서 우리는 Thread.sleep을 사용할 것입니다.

```kotlin
private fun network(seed: Int): Int {
   TimeUnit.MILLISECONDS.sleep(seed.toLong())
   return seed
}
```

다음 결과에 주목해 주세요.

<div class="content-ad"></div>

🌟 Here is the Markdown format for the image:

![Whatisfaster](/assets/img/2024-07-01-WhatisfasterandinwhichtasksCoroutinesRxJavaExecutor_5.png)

And now, let's dive into the comparison of different frameworks. The difference between them has become quite faint. Although rxIo is slightly ahead by less than 1%, it falls within the margin of error.

# mixed

The final test holds significant value, especially in relation to both my personal interest and the primary task at hand. We are now navigating tasks with mixed levels of complexity. 🌟

<div class="content-ad"></div>

```kotlin
private fun mixed(seed: Int): Int {
    return when {
        seed % 5 == 0 -> network(seed)
        seed % 3 == 0 -> storage(seed)
        seed % 2 == 0 -> listsManipulation(seed)
        else -> arithmetic(seed)
    }
}
```

우리가 가장 긴 작업을 90밀리초 동안 수행할 것이라는 것을 코드에서 확인할 수 있어요.

그래서 여기 결과들이 있어요.

![Image](/assets/img/2024-07-01-WhatisfasterandinwhichtasksCoroutinesRxJavaExecutor_6.png)


<div class="content-ad"></div>

일반적으로 IO, CPU, One 세 블록으로 나누는 패턴이 동일합니다.

유일한 차이점은 Executors.newWorkStealingPool로 생성된 Executor가 작은 작업을 추가하여 리드를 하는 것입니다. 이는 작업들을 "도둑질"하기 때문으로 보입니다.

**결론**

보시다시피, 파일 시스템과 네트워크 작업 시 멀티스레딩의 역할이 거의 사라지고 있습니다. 대부분의 멀티스레딩은 IO 작업에 사용됩니다. 그러므로 성능을 기반으로 한 멀티스레드 프레임워크 선택은 이상합니다. 편리함과 기타 중요한 요소를 고려하는 것이 더 나은 선택입니다.

<div class="content-ad"></div>

그래도 0.5 밀리초 동안 스레드를 차단하는 것보다 간단한 작업을 할 때는 작은 스레드 풀을 사용하는 것이 더 좋습니다. 예를 들어 CPU 코어 수를 풀의 크기로 사용할 수 있습니다.

응용프로그램 실행자(Executors)와 Rx가 이러한 작업에 가장 적합합니다.

예외는 산술 연산, 객체 생성 등과 같이 매우 간단한 작업에 해당합니다. 이러한 경우 스레드를 전혀 사용하지 않는 것이 좋습니다. 왜냐하면 프레임워크에서 스레드 생성하는 데 걸리는 시간이 모든 작업을 완료하는 데 필요한 시간보다 더 오래 걸릴 수 있기 때문입니다.

제 문제를 해결하기 위해 이러한 단계를 따랐습니다:

<div class="content-ad"></div>

- 나는 모든 간단한 작업을 별도의 작업 풀로 분리하고 directInvoke를 사용하여 실행했어.
- 다른 모든 작업은 동기 및 비동기 코드를 동시에 작성하기 쉽게 하기 때문에 Dispatchers.IO를 사용하여 코루틴에서 실행돼.

이러한 실행 방식에 대해 어떻게 생각하시나요? 현재 어떤 프레임워크를 사용하고 계신가요?