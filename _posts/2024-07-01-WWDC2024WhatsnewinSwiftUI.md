---
title: "WWDC 2024, SwiftUI의 최신 업데이트는"
description: ""
coverImage: "/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_0.png"
date: 2024-07-01 00:11
ogImage: 
  url: /assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_0.png
tag: Tech
originalTitle: "WWDC 2024, What’s new in SwiftUI"
link: "https://medium.com/@serhankhan/wwdc-2024-whats-new-in-swiftui-5f2d49380472"
---


## 이 특별한 기사에서는 SwiftUI 및 애플의 새로운 발표에 초점을 맞출 것입니다.

# 주요 기능:

- 새로운 앱: 앱이 새롭게 느껴지도록 새로운 기능을 추가하는 것을 목표로 합니다.
- 플랫폼 활용: 플랫폼마다 개발자로서 앱이 편안하게 느껴질 수 있도록 앱을 미세 조정할 수 있는 도구.
- 프레임워크 기반: 프레임워크의 기초적인 구조 블록에 대한 대규모 개선.
- 경험 제작: 몰입형 경험을 위한 새로운 도구 모음.

새로운 SwiftUI 개선에서, 개발자들은 새로운 탭 보기, 메시 그라데이션 및 반응 잘하는 컨트롤을 통해 앱을 정말로 업데이트할 수 있습니다.

<div class="content-ad"></div>

이번에는 WWDC 2024에서 소개된 새로운 기능들을 이해하기 위해 애플 개발자들에 의해 개발된 Karaoke Planner 앱을 리뷰하겠습니다.

# 신선한 앱

A) SwiftUI를 이용한 신선한 탭바 뷰:

![Fresh Tabbar view for with SwiftUI](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_0.png)

<div class="content-ad"></div>

iOS 18에서 사이드 바가 훨씬 유연해졌어요. 예를 들어, 왼쪽 상단의 버튼을 클릭하면요.

![image1](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_1.png)

그러면 사이드 바의 위치와 UI가 새로운 방식으로 변경돼요 (탭 바 형태로). 그림 3을 참고해 주세요.

![image2](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_2.png)

<div class="content-ad"></div>

그래서 이 새로운 표현을 어떻게 달성할 수 있는지 살펴보겠습니다. 예제 코드 블록으로 들어가 보죠:

```js
import SwiftUI

struct KaraokeTabView: View {
    @State var customization = TabViewCustomization()
    
    var body: some View {
        TabView {
            Tab("Parties", image: "party.popper") {
                PartiesView(parties: Party.all)
            }
            .customizationID("karaoke.tab.parties")
            
            Tab("Planning", image: "pencil.and.list.clipboard") {
                PlanningView()
            }
            .customizationID("karaoke.tab.planning")

            Tab("Attendance", image: "person.3") {
                AttendanceView()
            }
            .customizationID("karaoke.tab.attendance")

            Tab("Song List", image: "music.note.list") {
                SongListView()
            }
            .customizationID("karaoke.tab.songlist")
        }
        .tabViewStyle(.sidebarAdaptable)
        .tabViewCustomization($customization)
    }
}

struct PartiesView: View {
    var parties: [Party]
    var body: some View { Text("PartiesView") }
}

struct PlanningView: View {
    var body: some View { Text("PlanningView") }
}

struct AttendanceView: View {
    var body: some View { Text("AttendanceView") }
}

struct SongListView: View {
    var body: some View { Text("SongListView") }
}

struct Party {
    static var all: [Party] = []
}

#Preview {
    KaraokeTabView()
}
```

사용자 정의 동작을 통해 탭의 재정렬이나 제거가 프로그래밍적으로 완전히 제어 가능하도록 해줍니다.

```js
@State var customization = TabViewCustomization()
```

<div class="content-ad"></div>

The first image shows the new SwiftUI side bar presented at WWDC 2024. This refreshed side bar feature can now also be utilized on tvOS, expanding its usage across multiple platforms like iPadOS, tvOS, and macOS.

The second image displays how the tab bar in macOS can be styled to function as a side bar or segmented control. Exciting possibilities lie ahead!

<div class="content-ad"></div>

![2024-07-01-WWDC2024WhatsnewinSwiftUI_5.png](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_5.png)

B) 시트 프레젠테이션 크기:

플랫폼 간에 통합 및 간소화된 시트 프레젠테이션 크기입니다.

개발자로서 우리는 우리 시트 크기를 조정하기 위해 presentationSizing 수정자를 사용할 수 있습니다. 새로운 SwiftUI에는 (.form), (.page) 또는 (.customSizing) 이 세 가지 대안이 있습니다.

<div class="content-ad"></div>

```swift
struct AllPartiesView: View {
    @State var showAddSheet: Bool = true
    var parties: [Party] = []
    
    var body: some View {
        PartiesGridView(parties: parties, showAddSheet: $showAddSheet)
            .sheet(isPresented: $showAddSheet) {
                AddPartyView()
                    .presentationSizing(.form)
            }
    }
}
```

![Image 1](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_6.png)

Besides, SwiftUI also introduces a new Zoom navigation transition. Now, we can showcase the DetailView with a zoom effect by using the new modifier NavigationTransition(.zoom).

![Image 2](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_7.png)


<div class="content-ad"></div>

여기 당신을 위해 어떻게 보이는지(Let's get the parties started):

```js
import SwiftUI

struct PartyView: View {
    var party: Party
    @Namespace() var namespace
    
    var body: some View {
        NavigationLink {
            PartyDetailView(party: party)
                .navigationTransition(.zoom(
                    sourceID: party.id, in: namespace))
        } label: {
            Text("Party!")
        }
        .matchedTransitionSource(id: party.id, in: namespace)
    }
}

struct PartyDetailView: View {
    var party: Party
    
    var body: some View {
        Text("PartyDetailView")
    }
}

struct Party: Identifiable {
    var id = UUID()
    static var all: [Party] = []
}

#미리보기 {
    @Previewable var party: Party = Party()
    NavigationStack {
        PartyView(party: party)
    }
}
```

Keep in mind, these new Tabview features are available on iPadOS 18 and forward.

<div class="content-ad"></div>

C) 컨트롤 API:

이제 개발자들은 토글이나 버튼과 같은 크기 조절이 가능한 자체 컨트롤을 컨트롤 센터 또는 잠금 화면에 추가할 수 있습니다.

이러한 컨트롤은 액션 버튼에 의해 활성화될 수도 있습니다. 컨트롤은 앱 인텐트로 쉽게 구축할 수 있는 새로운 종류의 위젯입니다!

![이미지](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_9.png)

<div class="content-ad"></div>

아래 코드에서는 새로운 위젯을 제어 센터에 추가하는 방법을 보여줍니다.

```swift
import WidgetKit
import SwiftUI

struct StartPartyControl: ControlWidget {
    var body: some ControlWidgetConfiguration {
        StaticControlConfiguration(
            kind: "com.apple.karaoke_start_party"
        ) {
            ControlWidgetButton(action: StartPartyIntent()) {
                Label("파티 시작!", systemImage: "music.mic")
                Text(PartyManager.shared.nextParty.name)
            }
        }
    }
}

// 모델 코드

class PartyManager {
    static let shared = PartyManager()
    var nextParty: Party = Party(name: "WWDC Karaoke")
}

struct Party {
    var name: String
}

// AppIntent

import AppIntents

struct StartPartyIntent: AppIntent {
    static let title: LocalizedStringResource = "파티 시작"
    
    func perform() async throws -> some IntentResult {
        return .result()
    }
}
```

D) 벡터화 및 함수 플롯

Swift 차트에서의 함수 플로팅은 그래프를 그리는 것을 쉽게 만들어줍니다.

<div class="content-ad"></div>

```swift
import SwiftUI
import Charts

struct AttendanceView: View {
    var body: some View {
        Chart {
          LinePlot(x: "Parties", y: "Guests") { x in
            pow(x, 2)
          }
          .foregroundStyle(.purple)
        }
        .chartXScale(domain: 1...10)
        .chartYScale(domain: 1...100)
    }
}

# Preview {
    AttendanceView()
        .padding(40)
}
```

![WWDC2024](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_10.png)

**E) 테이블 열 동적 생성**

"TableColumnForEach"를 사용하여 동적 테이블 열 수를 가질 수 있습니다.


<div class="content-ad"></div>

```swift
import SwiftUI

struct SongCountsTable: View {

    var body: some View {
        Table(Self.guestData) {
            // A static column for the name
            TableColumn("Name", value: \.name)
            
            TableColumnForEach(Self.partyData) { party in
                TableColumn(party.name) { guest in
                    Text(guest.songsSung[party.id] ?? 0, format: .number)
                }
            }
        }
    }
    
    private static func randSongsSung(low: Bool = false) -> [Int : Int] {
        var songs: [Int : Int] = [:]
        for party in partyData {
            songs[party.id] = low ? Int.random(in: 0...3) : Int.random(in: 3...12)
        }
        return songs
    }
    
    private static let guestData: [GuestData] = [
        GuestData(name: "Sommer", songsSung: randSongsSung()),
        GuestData(name: "Sam", songsSung: randSongsSung()),
        GuestData(name: "Max", songsSung: randSongsSung()),
        GuestData(name: "Kyle", songsSung: randSongsSung(low: true)),
        GuestData(name: "Matt", songsSung: randSongsSung(low: true)),
        GuestData(name: "Apollo", songsSung: randSongsSung()),
        GuestData(name: "Anna", songsSung: randSongsSung()),
        GuestData(name: "Raj", songsSung: randSongsSung()),
        GuestData(name: "John", songsSung: randSongsSung(low: true)),
        GuestData(name: "Harry", songsSung: randSongsSung()),
        GuestData(name: "Luca", songsSung: randSongsSung()),
        GuestData(name: "Curt", songsSung: randSongsSung()),
        GuestData(name: "Betsy", songsSung: randSongsSung())
    ]
    
    private static let partyData: [PartyData] = [
        PartyData(partyNumber: 1, numberGuests: 5),
        PartyData(partyNumber: 2, numberGuests: 6),
        PartyData(partyNumber: 3, numberGuests: 7),
        PartyData(partyNumber: 4, numberGuests: 9),
        PartyData(partyNumber: 5, numberGuests: 9),
        PartyData(partyNumber: 6, numberGuests: 10),
        PartyData(partyNumber: 7, numberGuests: 11),
        PartyData(partyNumber: 8, numberGuests: 12),
        PartyData(partyNumber: 9, numberGuests: 11),
        PartyData(partyNumber: 10, numberGuests: 13),
    ]
    
}

struct GuestData: Identifiable {
    let name: String
    let songsSung: [Int : Int]
    
    let id = UUID()
}

struct PartyData: Identifiable {
    let partyNumber: Int
    let numberGuests: Int
    let symbolSize = 100
    
    var id: Int {
        partyNumber
    }
    
    var name: String {
        "\(partyNumber)"
    }
}

#Preview {
    SongCountsTable()
        .padding(40)
}
```

![Image](https://tarotcards.com/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_11.png)

Mesh Gradients

SwiftUI가 색상이 풍부한 메시 그라데이션을 위한 일등 시민 지원을 추가했습니다. 그리드 색상의 점을 보간하는 방식으로 작동합니다.

<div class="content-ad"></div>

이 특별한 새로운 기능으로 우리 뷰에 그라데이션 배경색을 적용할 수 있게 되었어요.

![Example](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_12.png)

```swift
import SwiftUI

struct MyMesh: View {
    var body: some View {
        MeshGradient(
            width: 3,
            height: 3,
            points: [
                .init(0, 0), .init(0.5, 0), .init(1, 0),
                .init(0, 0.5), .init(0.3, 0.5), .init(1, 0.5),
                .init(0, 1), .init(0.5, 1), .init(1, 1)
            ],
            colors: [
                .red, .purple, .indigo,
                .orange, .cyan, .blue,
                .yellow, .green, .mint
            ]
        )
    }
}

#Preview {
    MyMesh()
        .statusBarHidden()
}
```

G) 출시 씬 문서화 (iPadOS)

<div class="content-ad"></div>

문서 기반 응용 프로그램은 이제 DocumentGroup을 구현하여 새로운 UI를 가질 수 있습니다. DocumentGroup을 사용하면 다음과 같은 UI가 생깁니다:

![image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_13.png)

지금 보시는 것처럼 "Playgrounds title", "Learn to Code" 버튼 및 "New App" 버튼이 디자인에 추가되었습니다.

또한 이 UI 요소 외에도, 우리는 도면 15에서 나타나는 바와 같이 배경 액세서리, 양 쪽 (왼쪽 및 오른쪽)을위한 전경 액세서리를 가지고 있습니다.

<div class="content-ad"></div>

![image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_14.png)

예제 코드 Document Groups를 위해:

```js
DocumentGroupLaunchScene("Your Lyrics") {
    NewDocumentButton()
    Button("New Parody from Existing Song") {
        // Do something!
    }
} background: {
    PinkPurpleGradient()
} backgroundAccessoryView: { geometry in
    MusicNotesAccessoryView(geometry: geometry)
         .symbolEffect(.wiggle(.rotational.continuous()))
} overlayAccessoryView: { geometry in
    MicrophoneAccessoryView(geometry: geometry)
}
```

H) 새로운 심볼 효과:

<div class="content-ad"></div>

이제 심볼들은 새로운 능력을 갖고 있어요. 특정 효과를 받아들이는 우리 어플리케이션들의 사용 예시입니다:

![SF Symbol Example](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_15.png)

다른 새로운 SF 심볼 효과들은 다음과 같아요:

.wiggle effect: 어떤 방향이나 각도로 심볼을 진동시켜서 주의를 끌어요

<div class="content-ad"></div>

**이미지:**
![이미지](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_16.png)

**.breathe 효과:** 심볼을 부드럽게 확대/축소하여 지속적인 활동을 나타냅니다.

**이미지:**
![이미지](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_17.png)

**.rotate 효과:** 심볼의 일부분을 지정된 앵커 포인트 주변으로 회전시킵니다.

<div class="content-ad"></div>

이미지를 Markdown 형식으로 변경해주세요.

일부 기존 프리셋은 새로운 기능으로 개선되었습니다.

예를 들어, 기본 "대체" 애니메이션은 이제 새로운 MagicReplace 동작을 선호합니다.

MagicReplace를 사용하면 기호가 부드럽게 배지와 슬래시를 애니메이션화합니다.

<div class="content-ad"></div>

# 플랫폼 활용하기:

**A) 창 관리**

새로운 윈도우 관리 SwiftUI 수정자들은 macOS의 창에 새로운 스타일과 레벨을 추가하는 데 도움이 됩니다.

```swift
Window("가사 미리보기", id: "lyricPreview") {
    LyricPreview()
}
  .windowStyle(.plain)
  .windowLevel(.floating)
  .defaultWindowPlacement { content, context in
      let displayBounds = context.defaultDisplay.visibleRect
      let contentSize = content.sizeThatFits(.unspecified)
      return topPreviewPlacement(size: contentSize, bounds: displayBounds)
  }
```

<div class="content-ad"></div>

지금은 창에 드래그 제스처를 추가할 수 있어요:

![image1](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_20.png)

```js
Text(currentLyric)
    .background(.thinMaterial, in: .capsule)
    .gesture(WindowDragGesture())
```

<div class="content-ad"></div>

이번에 추가된 새 창 수정자는 visionOS에서도 작동합니다.

새 창 수정자 외에도, 원래 창을 숨기고 새 창을 열 수 있는 푸시 창 작업이 있습니다.

![image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_21.png)

창에 대한 자세한 정보는 다음 링크를 참조하십시오.

<div class="content-ad"></div>

B) 새로운 SwiftUI 입력 방식 (visionOS)

SwiftUI는 각 플랫폼이 제공하는 독특한 입력 방식을 활용할 수 있는 많은 새로운 도구를 제공합니다.

예를 들어 visionOS에서는 뷰가 사람들이 보거나 손가락을 가까이 대거나 포인터를 이동시킬 때 반응하도록 만들 수 있습니다.

![링크](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_22.png)

<div class="content-ad"></div>

![2024-07-01-WWDC2024WhatsnewinSwiftUI_23.png](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_23.png)

So, 이 그림 24와 25를 고려해 보면 프로필이 축소되지 않았고 사용자가 작업을 수행함에 따라 (즉, 보기를 보는 것에 의해), 프로필 이미지가 확장되어 사용자에 대한 더 많은 정보를 볼 수 있었습니다.

새로운 호버 효과 내에서 뷰의 상태를 제어하고 활성 및 비활성 사이의 전환을 조정할 수 있습니다.

```swift
struct ProfileButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .background(.thinMaterial)
            .hoverEffect(.highlight)
            .clipShape(.capsule)
            .hoverEffect { effect, isActive, _ in
                effect.scaleEffect(isActive ? 1.05 : 1.0)
            }
    }
}
```

<div class="content-ad"></div>

C) 수정자 키 대체 옵션들

iPadOS, macOS, 그리고 visionOS 앱들은 키보드 지원을 제공합니다 (바로가기로).

예를 들어, macOS에서 앱에 새로운 바로가기 수정자 키를 추가하고 싶다면 이 몇 줄의 코드로 가능합니다:

```js
Button("창에서 가사 미리보기") {
    // 창에서 미리보기 표시
}
.modifierKeyAlternate(.option) {
    Button("전체 화면에서 가사 미리보기") {
        // 전체 화면에서 미리보기 표시
    }
}
.keyboardShortcut("p", modifiers: [.shift, .command])
```

<div class="content-ad"></div>

맥OS의 메인 메뉴에서, 미리보기 창을 열기 위한 항목이 있다고 가정해봅시다. 위의 코드 조각은 이 사용 예시를 나타내며 아래와 같은 기능이 얻어지게 됩니다:

![preview window](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_24.png)

D) 포인터 상호 작용

포인터 상호 작용은 다양한 기기에서의 중요한 입력 방식 중 하나입니다.

<div class="content-ad"></div>

![image1](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_25.png)

포인터 스타일 API를 사용하면 시스템 포인터의 외관과 가시성을 사용자 정의할 수 있습니다.

![image2](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_26.png)

iPadOS 17.5에서는 Apple Pencil과 Apple Pencil Pro의 기능을 지원하는 SwiftUI가 포함되어 있습니다. 더블 탭과 스퀴즈 기능도 사용 가능합니다.

<div class="content-ad"></div>

**애플 펜슬 스퀴즈 액션**으로 이제 제스처에서 정보를 수집하여 선호하는 조치를 확인할 수 있어요.

![WWDC 2024에서의 SwiftUI 새로운 기능](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_27.png)

E) 위젯 및 실시간 활동

이제 위젯과 실시간 활동이 자동으로 Apple 워치에 표시될 거랍니다.

<div class="content-ad"></div>

개발자로서, watchOS에서 콘텐츠를 조정할 수 있는 새로운 .supplementalActivityFamilies 수정자를 활용할 수 있습니다.

![Image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_28.png)

![Image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_29.png)

그리고 사용자가 실시간 활동을 더욱 향상시킬 수 있도록 두 번 탭을 적용할 수도 있습니다. 이를 위해 .handGestureShortcut 수정자를 활용하세요.

<div class="content-ad"></div>

![Image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_30.png)

# Framework Foundations

A) Custom Containers

A new API on ForEach, `subviewOf`, allows us to iterate over the subviews of a given view. In the example below, each subview is wrapped in its own `CardView`.

<div class="content-ad"></div>

이를 사용하여 SwifUI의 내장 컨테이너인 목록과 동일한 기능을 가진 사용자 정의 컨테이너를 만들 수 있습니다. 이는 정적 및 동적 콘텐츠를 혼합하여 섹션을 지원하고 특정 수정자에 컨테이너를 추가하는 것을 포함합니다.

![image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_31.png)

```js
struct DisplayBoard<Content: View>: View {
  @ViewBuilder var content: Content

  var body: some View {
    DisplayBoardCardLayout {
      ForEach(subviewOf: content) { subview in
        CardView {
          subview
        }
      }
    }.background(BoardBackgroundView())
  }
}

DisplayBoard {
  Text("Scrolling in the Deep")
  Text("Born to Build & Run")
  Text("Some Body Like View")

  ForEach(songsFromSam) { song in
    Text(song.title)
  }
}
```

이는 섹션 또한 지원하기 위해 사용할 수 있습니다.

<div class="content-ad"></div>

**Entry macro:**

![](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_32.png)

![](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_33.png)

B) 편의성 향상신 시사점

<div class="content-ad"></div>

환경키에 대한 완전한 준수 쓰기와 환경 값에 대한 확장을 작성해야 했던 것을 간단한 속성과 Entry 매크로로 대체할 수 있게 되었습니다.

이전: 원하는 구조체에 대한 환경으로 SwiftUI 프로젝트에서 사용할 수 있게끔 하기 위해 enviromentKey에 대한 확장을 작성해야 했습니다.

![image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_34.png)

이후: 현재 @Entry로 표시해서 SwiftUI가 선언을 자동으로 처리하고 SwiftUI 프로젝트 내에서 환경 변수로 사용할 준비를 갖추도록 할 수 있습니다.

<div class="content-ad"></div>

```swift
마법사 파티 컬러: Color = .보라
```

엔트리 매크로는 환경 값과 함께 사용되는 것뿐만 아니라, 엔트리 매크로는 포커스 값, 트랜잭션 및 컨테이너 값에서도 사용할 수 있습니다.

```swift
마법사 노트: String? = nil
```

```swift
애니메이션 파티 아이콘: Bool = false
```

```swift
디스플레이 보드 카드 스타일: DisplayBoardCardStyle = .테두리
```

## 기본 접근성 레이블 보강:


<div class="content-ad"></div>

현재 SwiftUI의 컨트롤에 프레임워크가 제공하는 레이블을 덮어쓰지 않고 추가 정보를 추가할 수 있습니다. 자세한 정보는 링크를 참조해주세요.

![2024-07-01-WWDC2024WhatsnewinSwiftUI_35](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_35.png)

## Xcode 미리보기에 새로운 동적 링킹 아키텍처가 있습니다

이를 통해 프로젝트를 다시 빌드할 필요 없이 미리보기와 빌드 및 실행 간에 전환할 수 있어 반복 속도를 높일 수 있습니다.

<div class="content-ad"></div>

이제는 미리보기 설정도 더 쉬워졌어요. @Previewable 매크로를 사용하여 미리보기에서 상태를 바로 사용할 수 있게 되어, 뷰에 미리보기 내용을 감싸는 부분에서 발생했던 불필요한 작업을 줄일 수 있어요.

![Image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_36.png)

## 텍스트 작업 및 선택 관리를 위한 새로운 방법:

이제 SwiftUI는 텍스트 편집 컨트롤 내의 텍스트 선택에 프로그래밍적 접근과 제어를 제공해요.

<div class="content-ad"></div>

![image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_37.png)

지금은 선택한 범위와 같은 선택 속성을 읽을 수 있습니다.

![image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_38.png)

![image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_39.png)

<div class="content-ad"></div>

이를 사용하여 검사자에서 선택한 단어에 대한 제안된 운율을 보여줄 수 있어요.

![image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_40.png)

## 새로운 .searchFocused로:

우리는 프로그래밍 방식으로 검색 필드의 포커스 상태를 제어할 수 있습니다. 이는 검색 필드가 포커스된 상태인지 확인하고, 프로그래밍 방식으로 검색 필드로 포커스를 이동하거나 검색 필드에서 포커스를 이동할 수 있습니다.

<div class="content-ad"></div>

이제는 어떤  텍스트 필드에 새 텍스트 제안을 추가할 수 있습니다.

몇 줄의 코드만으로 사용자에게 제안된 텍스트를 제공할 수 있습니다. 제안 사항은 드롭다운 메뉴로 나타나며, 옵션을 선택하면 텍스트 필드가 선택한 완료로 업데이트됩니다.

<div class="content-ad"></div>

텍스트를 한국어로 번역해보겠어요!


![이미지](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_42.png)

예시 코드는 다음과 같습니다:

![이미지](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_43.png)

## 그래픽 기능:


<div class="content-ad"></div>

색상 조합: 개발자로서 우리는 색상을 섞어 사용할 수 있습니다. 다른 색상과 주어진 양으로 섞는 색상 혼합기능이 새롭게 추가되었어요.

![Image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_44.png)

사용자 정의 쉐이더 기능:

사용자 정의 쉐이더 기능은 쉐이더가 처음 사용되기 전에 사전 컴파일할 수 있는 기능을 가지고 있어요. 게으른 쉐이더 컴파일로 인한 프레임 드랍을 피할 수 있답니다.

<div class="content-ad"></div>

**스크롤링 향상:**

개발자들에게 세밀한 제어권을 주는 새로운 API가 많이 추가되었습니다.

현재 우리는 onScrollGeometryChanged를 통해 ScrollView의 상태와 깊은 수준의 통합을 가질 수 있습니다. 이를 통해 콘텐츠 오프셋, 콘텐츠 크기 등의 변경 사항에 효율적으로 반응할 수 있습니다. 스크롤 뷰의 콘텐츠 위로 스크롤을 넘어갈 때 나타나는 "초대로 돌아가기" 버튼과 같이요.

<div class="content-ad"></div>

이제 뷰의 가시성이 스크롤로 변경된 것을 감지할 수 있어요!

![image](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_48.png)

<div class="content-ad"></div>

**새로운 스위프트 6 언어 모드:**

화면 이동이나 사운드 자동 재생과 같은 콘텐츠 중심의 멋진 경험을 만들 수 있게 해주는 기능이 추가되었습니다.

컴파일 시간 데이터 경쟁 안전성을 제공하고, SwiftUI는 앱에서 새로운 언어 모드를 쉽게 적용할 수 있도록 API를 개선했습니다.

SwiftUI의 뷰는 항상 메인 액터에서 평가되어 왔고, 이제 뷰 프로토콜에 메인 액터 주석이 표시되어 해당 사항이 반영되었습니다.

<div class="content-ad"></div>

안녕하세요! 이 의미는 모든 View를 준수하는 모든 타입이 기본적으로 main actor에 의해 암시적으로 격리된다는 것을 의미합니다. 따라서 View를 명시적으로 main actor로 표시했다면 그 주석을 제거해도 동작에 아무런 변경이 없습니다.

새로운 Swift 6 언어 모드는 선택 사항이므로 준비가 되면 언제든지 그것을 활용할 수 있습니다. 어떻게 응용 프로그램을 Swift6로 마이그레이션하는지 알아보세요.

<div class="content-ad"></div>

향상된 상호 운용성:

SwiftUI는 새로운 앱을 만드는 데뿐만 아니라 UIKit과 AppKit으로 작성된 기존 앱에 새로운 기능을 추가하는 데도 설계되었습니다.

## 제스처 상호 운용성

이제 내장된 또는 사용자 정의 UIGestureRecognizer를 취해서 Swift SwiftUI 뷰 계층 구조에서 사용할 수 있습니다. 직접 UIKit으로 뒷받침되지 않은 SwiftUI 뷰에서도 작동합니다.

<div class="content-ad"></div>

마크다운 포맷으로 변경해드릴게요.


![Image 1](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_51.png)

## UIKit 및 AppKit에서 SwiftUI 애니메이션

UIKit 및 AppKit에서 SwiftUI 애니메이션의 힘을 활용할 수 있습니다.

![Image 2](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_52.png)


<div class="content-ad"></div>

Experiencing the Art of Craft:

새로운 API로 볼륨 및 몰입형 공간, 그리고 텍스트 효과를 다룰 수 있습니다.

### 사용자 정의 텍스트 렌더러:

예를 들어 단어를 강조하는 데 다음 TextRenderer 프로토콜을 사용할 수 있습니다.

<div class="content-ad"></div>

![2024-07-01-WWDC2024WhatsnewinSwiftUI_53.png](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_53.png)

KaraokeRenderer는 원본 그림 뒤에 복사된 텍스트를 생성하며, 이는 흐리고 색조가 조절된 상태로 표시됩니다.

![2024-07-01-WWDC2024WhatsnewinSwiftUI_54.png](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_54.png)

![2024-07-01-WWDC2024WhatsnewinSwiftUI_55.png](/assets/img/2024-07-01-WWDC2024WhatsnewinSwiftUI_55.png)