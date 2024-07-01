---
title: "Coordinator 패턴을 사용한 네비게이션 방법"
description: ""
coverImage: "/assets/img/2024-07-01-NavigationviaCoordinatorPattern_0.png"
date: 2024-07-01 20:53
ogImage: 
  url: /assets/img/2024-07-01-NavigationviaCoordinatorPattern_0.png
tag: Tech
originalTitle: "Navigation via Coordinator Pattern"
link: "https://medium.com/wereprotein/navigation-via-coordinator-pattern-dd559541be90"
---


각 화면을 포함하는 앱은 사용자가 인터페이스를 탐색하고 다른 기능에 액세스할 수 있는 시스템이 필요합니다. 내장된 네비게이션 컨트롤러와 같은 도구를 통해 또는 사용자 정의 솔루션을 통해 이러한 네비게이션을 구현하는 것은 복잡할 수 있습니다. 잘 설계된 네비게이션 구조는 앱의 다른 부분이 독립적이고 유지보수가 쉽도록 보장하여 뷰 컨트롤러 간에 강하게 결합되지 않고 널리 퍼져 있는 종속성을 피합니다.

![Navigation Structure](/assets/img/2024-07-01-NavigationviaCoordinatorPattern_0.png)

# 문제점은 무엇인가요?

iOS 앱에서 네비게이션을 관리하는 일반적인 방법은 UINavigationController를 사용하는 것입니다. 이 컨트롤러는 뷰 컨트롤러를 푸시하고 팝하여 화면 간의 전환을 원활하고 쉽게 처리합니다. 예를 들어:

<div class="content-ad"></div>

이 접근 방식을 통해 한 뷰 컨트롤러가 다른 뷰 컨트롤러를 알고 만들고 구성하며 표시하는 역할을 맡게 됩니다. 이는 애플리케이션에서 뷰 컨트롤러 사이의 링크가 하드 코딩되므로 응집력이 높습니다. 결과적으로, 동일한 뷰 컨트롤러를 다른 위치에서 표시해야 한다면 구성 코드를 중복해서 작성해야 할 수도 있습니다.

Coordinator 패턴에서는 ViewController가 새로운 ViewController를 만들고 구성하는 역할을 맡은 Coordinator와만 통신합니다. 이 접근 방식은 ViewController들이 서로 분리되어 있으며 이동할 특정 ViewController를 알지 못하도록 보장하기 때문에 더 정리되고 유지보수가 편리한 코드베이스를 유지할 수 있습니다.

# Coordinator 만들기를 시작해봅시다.

<div class="content-ad"></div>

두 개의 프로토콜을 생성해야 합니다. 첫 번째 프로토콜은 모든 코디네이터가 따를 기본 구조를 포함할 것입니다. 두 번째 프로토콜은 메인 코디네이터에 특화됩니다. 이 두 번째 프로토콜은 코디네이터와 뷰 컨트롤러 간의 커뮤니케이션을 원활히 할 것입니다.

Markdown 형식으로 위 코드를 조금 수정해주시면 좋을 것 같네요.

처음에는 모든 코디네이터가 준수해야 할 Coordinator 프로토콜을 생성해야 합니다. 기본적인 구조를 따르는 것이 좋습니다:

- Child Coordinators 속성: 자식 코디네이터를 저장하는 속성을 추가합니다.
- Navigation Controller 속성: 뷰 컨트롤러를 표시하는 데 사용될 네비게이션 컨트롤러를 보유하는 속성을 포함합니다.
- Start 메서드: 코디네이터가 제어를 가져갈 수 있도록 start() 메서드를 정의합니다. 이렇게 하면 필요할 때만 코디네이터를 생성하고 활성화할 수 있습니다.

<div class="content-ad"></div>

이제 ViewController를 만들 수 있는 확장 프로그램을 추가해서 조금 더 쉽게 작업할 수 있게 되었어요.

```js
import UIKit

public enum Storyboard: String {
    case main = "Main"

    var instance: UIStoryboard {
        return UIStoryboard(name: self.rawValue, bundle: Bundle.main)
    }

    func viewController<T: UIViewController>(viewControllerClass: T.Type, function: String = #function, line: Int = #line, file: String = #file) -> T {
        let storyboardID = (viewControllerClass as UIViewController.Type).storyboardID
        guard let scene = instance.instantiateViewController(withIdentifier: storyboardID) as? T else {
            fatalError("ViewController with identifier \(storyboardID), not found in \(self.rawValue) Storyboard.\nFile : \(file) \nLine Number : \(line) \nFunction : \(function)")
        }

        return scene
    }

    static func initialViewController<T: UIViewController>(viewControllerClass: T.Type, function: String = #function, line: Int = #line, file: String = #file) -> T {
         let storyboardID = (viewControllerClass as UIViewController.Type).storyboardID
         guard let scene = UIStoryboard(name: storyboardID, bundle: Bundle.main).instantiateViewController(withIdentifier: storyboardID) as? T else {
             fatalError("ViewController with identifier \(storyboardID), not found in \(storyboardID) Storyboard.\nFile : \(file) \nLine Number : \(line) \nFunction : \(function)")
         }
         return scene
     }
}

extension UIViewController {

    class var storyboardID: String {
        return "\(self)"
    }

    static func instantiate() -> Self {
        return Storyboard.initialViewController(viewControllerClass: self)
    }
}
```

이제 ViewController를 쉽게 만들 수 있는 능력을 갖게 되었어요. 세팅할 때 더 이상 스토리보드에 의존하지 않아도 돼요. 하지만 스토리보드는 여전히 간단한 디자인을 표시하는 데 유용해요. 그래서 저는 프로젝트에서 그것들을 사용하는 것을 선호해요. 우리 자신의 스토리보드를 만들고 그것들을 코디네이터에 통합함으로써, 유연성과 간단함 사이의 균형을 이룰 수 있어요.

메인 스토리보드 참조를 제거했어요.

<div class="content-ad"></div>

## 탐색 관리를 위한 MainCoordinator 작성하기

코디네이터 기반의 탐색의 핵심에는 MainCoordinator 개념이 있습니다. 이 코디네이터는 앱의 탐색 계층 구조로의 진입점 역할을 합니다. 필요한 구성 요소를 초기화하고 앱의 초기 상태를 설정하며 사용자를 다양한 화면과 작업 흐름을 통해 안내합니다. 이를 생성해보겠습니다.

```js
import UIKit

class MainCoordinator: Coordinator {
    var childCoordinators = [Coordinator]()
    var navigationController: UINavigationController
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func start() {
        let vc = InitialViewController.instantiate()
        vc.coordinator = self
        navigationController.pushViewController(vc, animated: false)
    }
}
```

<div class="content-ad"></div>

### 1. Properties:

- `childCoordinators`: An array to keep track of any child coordinators.
- `navigationController`: A navigation controller used for managing view controllers.

### 2. Initializer:

- It initializes the `MainCoordinator` with a navigation controller.

<div class="content-ad"></div>

3. 시작 메서드:

- 이 방법은 조정 프로세스를 시작하는 데 사용됩니다.
- InitialViewController를 인스턴스화하고 앱의 시작점을 설정합니다.
- InitialViewController의 조정자를 자체로 설정합니다.
- InitialViewController를 네비게이션 스택에 푸시합니다.

SceneDelegate.swift에서 조정자 수동으로 초기화하기

애플리케이션을 위해 조정자를 설정한 후에는 앱이 실행될 때 활성화해야 합니다. 일반적으로 이 초기화는 스토리보드에서 처리하지만, 해당 기능을 비활성화했다면, 이제 SceneDelegate.swift 파일 내에서 직접 시작 프로세스를 관리해야 합니다.

<div class="content-ad"></div>

### 메인 코디네이터 초기화:
1. UINavigationController의 인스턴스를 생성합니다.
2. 이 내비게이션 컨트롤러로 MainCoordinator를 초기화합니다.

### 코디네이터 시작:
1. 내비게이션 컨트롤러를 윈도우의 루트 뷰 컨트롤러로 할당합니다.
2. 플로우를 시작하려면 코디네이터의 start() 메서드를 호출합니다.

<div class="content-ad"></div>

여기 세련된 구현 방법이 있습니다:

```js
import UIKit

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    var window: UIWindow?
    var mainCoordinator: MainCoordinator?
    
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene) else { return }
        
        let navController = UINavigationController()
        mainCoordinator = MainCoordinator(navigationController: navController)
        mainCoordinator?.start()
        
        window = UIWindow(windowScene: windowScene)
        window?.rootViewController = navController
        window?.makeKeyAndVisible()
    }
```

간단한 예제로 흐름을 이어보겠습니다;

이를 위해 애플리케이션의 시작점인 InitialViewController에서 이동할 다른 페이지가 필요합니다. 이 간단한 예제에서 RedViewController와 BlueViewController 두 개의 컨트롤러를 만들었습니다. InitialViewController의 두 버튼을 사용하여 이러한 페이지로 이동할 것입니다. 주의해야 할 점은, UIViewController에 대해 작성한 확장을 사용하려면 스토리보드 식별자를 올바르게 지정해야 합니다.

<div class="content-ad"></div>

첫째, 우리 각 뷰 컨트롤러는 그것의 코디네이터와 통신할 방법이 필요합니다. 따라서 ViewController 세 개 모두에 이 속성을 포함하세요.

```swift
weak var coordinator: MainCoordinator?
```

메인 코디네이터에서 작성할 두 가지 메서드를 통해 페이지 간에 이동할 수 있을 것입니다.

<div class="content-ad"></div>


Navigator.py파일에 `navigateToBlue()` 및 `navigateToRed()` 메서드가 정의되어 있습니다. 

이 메서드들은 InitialViewController에서 정의한 액션에서 호출됩니다.

InitialViewController에서는 다음과 같이 호출됩니다.


   @IBAction func navigateToBlueVC(_ sender: Any) {
        coordinator?.navigateToBlue()
    }
    
    @IBAction func navigateToRedVC(_ sender: Any) {
        coordinator?.navigateToRed()
    }


이제 코디네이터가 제어하는 각 뷰 컨트롤러 간에 탐색이 가능한 앱이 작동해야 합니다. 축하합니다!


<div class="content-ad"></div>

# 결론

Coordinator Pattern을 Swift에서 구현하면 iOS 애플리케이션 내에서 네비게이션을 처리하는 구조화되고 유지보수 가능한 방법을 제공합니다. 이 패턴을 사용하면 네비게이션 로직을 뷰 컨트롤러로부터 분리하여 더 깔끔한 코드베이스를 유지할 뿐만 아니라 재사용성을 향상시키고 테스트를 간단하게 만듭니다. 앱이 확장되면 Coordinator Pattern을 통해 네비게이션 아키텍처가 견고하고 적응성이 뛰어나게 유지되어 새로운 기능을 원활하게 통합할 수 있고 개발 프로세스를 더욱 원활하게 만들 수 있습니다. 이 패턴을 받아들이는 것은 더 모듈식이고 조직화된 Swift 애플리케이션으로 가는 한 걸음이며, 이는 최종적으로 더 나은 개발자 경험과 믿을 수 있는 앱을 제공하게 됩니다.