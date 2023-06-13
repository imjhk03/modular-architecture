<aside>
💡 아래 내용은 Tuist를 가지고 모듈화 아키텍처를 하는 방법을 회사 프로젝트 기준으로 내부적으로 공유한 내용을 공개용으로 옮겼습니다.
</aside>


# Tuist를 활용한 모듈화 아키텍처

작성자: 김주희

<aside>
💡 아주 간단하게 구현된 모습만 작업했습니다. 양해 부탁드립니다.
</aside>

# 모듈화 아키텍처

우선 제가 구상하고 싶은 모습까지는 매우 심플하게 구상했습니다. 제가 목표로 하고 싶은 내용은 아래와 같습니다.

- 공통적으로 작업하는 내용을 모듈화로 만들어서 동일한 코드, 중복 코드 없이 가져다 사용하기
- 각 모듈은 담당하는 기능만 하기
- 기능/화면 단위의 모듈을 진행하여 추후에는 각 화면만 빌드해서 개발하거나 테스트 코드 작성할 수 있도록 하기

아래 스크린샷들은 그림으로 조금 더 명확하게 그렸습니다. 여기서 정리하고 싶은 것은, 비즈니스 로직을 담당하는  Core가 있고 디자인을 그리는 DesignSystem 혹은 UserInterface가 있습니다. 각 모듈에 어떤 것을 모듈화할지는 이야기해서 논의하면 되는 범위라고 생각합니다. 모듈화가 필요한 기능들이 있고, 각 화면 단에서만 필요한 기능들도 있기 때문입니다. 그래도 최소한은 Logger, Network, Foundation Extension은 분리가 필요할 것으로 보입니다.

![모듈 아키텍처_김주희 002](https://github.com/imjhk03/modular-architecture/assets/28954046/0192a3da-a85f-41f3-bbce-cb754a02a34a)
![모듈 아키텍처_김주희 003](https://github.com/imjhk03/modular-architecture/assets/28954046/74b7df06-8b08-4c75-8918-f4b5e8badfe3)
![모듈 아키텍처_김주희 004](https://github.com/imjhk03/modular-architecture/assets/28954046/8573d36e-3747-44be-a9cd-1fe95e68926d)
![모듈 아키텍처_김주희 005](https://github.com/imjhk03/modular-architecture/assets/28954046/87829e55-c4d0-4fb3-87a4-6ea9ae07fd84)
![모듈 아키텍처_김주희 006](https://github.com/imjhk03/modular-architecture/assets/28954046/503de77d-6476-461b-955e-fd26e0ecadde)
![모듈 아키텍처_김주희 007](https://github.com/imjhk03/modular-architecture/assets/28954046/97859322-706c-4179-9697-8d2c5d802b3b)
![모듈 아키텍처_김주희 008](https://github.com/imjhk03/modular-architecture/assets/28954046/5f0c9938-94b1-41a1-bb53-7e1dea25ed4e)
![모듈 아키텍처_김주희 009](https://github.com/imjhk03/modular-architecture/assets/28954046/e0e6f254-496a-444f-b603-b92eb2bf458e)
![모듈 아키텍처_김주희 010](https://github.com/imjhk03/modular-architecture/assets/28954046/548e87a9-f74a-4822-b606-367a983ae703)
![모듈 아키텍처_김주희 012](https://github.com/imjhk03/modular-architecture/assets/28954046/4131d929-bac9-4eae-8907-a7df89fa12f2)

이 아이디어는 카카오뱅크에서 진행하는 [모듈화 아키텍처](https://if.kakao.com/2022/session/88)랑 거의 비슷합니다. 각 기능 단위의 데모앱까지 만들어서 빌드하는게 매력적으로 느껴서 많이 참고했습니다. 지금 우리가 가지고 있는 LoginKit을 조금 더 구체화하고 독립적으로 움직이는 수준까지 가는 것으로 이해하면 될 것 같습니다. 카카오뱅크는 Tuist라는 도구를 이용해서 모듈화 아키텍처를 설계를 진행하고 있습니다.

그래서 Tuist를 알아야하고, Tuist으로 아키텍처 설계하고 구성하고, Tuist으로 Xcode 프로젝트를 생성하는 습관까지 들여야 해서 사전 지식이 필요한 단점이 있습니다. 처음에는 익숙하는데 시간이 필요한데, 사용법을 익숙해지면 xcode 프로젝트를 생성해서 확인하는 시간이 줄어들고, gitignore로 불필요한 파일들을 정리해서 충돌을 해결할 수 있을 것 같습니다.

그러면 Tuist를 가지고 어떻게 xcode 프로젝트를 생성하는지 설명하겠습니다.

# Tuist

모듈화 아키텍처를 설계하기 위해서는 [Tuist](https://github.com/tuist/tuist)를 사용해야 합니다. Tuist란 **Xcode 프로젝트를 생성, 유지 및 상호 작용**하는 데 도움이 되는 명령줄 도구입니다.

> Tuist is a command line tool that helps you **generate**, **maintain** and **interact** with Xcode projects.
> 

이미 예전에 xx님께서 'XcodeGen vs Tuist' 글에서 간단하게 소개해 주셨습니다. 이때는 여러 파일 추가나 삭제가 되면서 MyProject.xcodeproj 파일 충돌에 집중적으로 해서 해소할 수 있는 툴들을 소개했는데, 이 내용을 모듈화 아키텍처 시점으로 설명하겠습니다. 더 자세한 내용은 위 글을 참고하면 됩니다.

## 사용 방법

먼저 tuist를 설치합니다.

```bash
curl -Ls https://install.tuist.io | bash
```

앱을 만들 디렉토리를 만들고

```bash
mkdir MyApp
cd MyApp
```

여기서 tuist 기반 앱 프로젝트를 생성합니다. 최소한의 자원으로 iOS 애플리케이션을 만드는데 `Info.plist` 파일들, `AppDelegate.swift` 파일, 테스트 파일, 그리고 **프로젝트 정의가 담겨져 있는 `Project.swift` 파일**이 생성됩니다.

```bash
tuist init --platform ios

// SwiftUI template으로 생성하고 싶다면 아래 명령어
tuist init --platform ios --template swiftui
```
<img width="419" alt="스크린샷 2023-01-05 오후 1 50 17" src="https://github.com/imjhk03/modular-architecture/assets/28954046/05a91f13-65a0-4577-8687-bc4ff48ad6f7">

모든 폴더에서 설명하지 않고 필요한 파일만 설명하겠습니다. `**Project.swift**` 파일은 프로젝트를 어떻게 구성할지 설정하는 파일입니다. Tuist는 이 파일에 정의된 대로 xcode 프로젝트를 생성합니다. 이와 관련해서 도와주는 파일이 바로 `**Project+Templates.swift**` 파일입니다. Extension으로 분리해서 프로젝트를 구성할 때 필요한 데이터를 function으로 생성할 수 있게 정리되어 있습니다.

각 파일들을 수정하기 위해서는 아래 명령어로 실행하면 됩니다. 명령어를 실행하면 아래와 같이 xcode가 열리게 됩니다. 이때, xcode으로 수정한 내용을 반영하기 위해서는 xcode를 닫고, 터미널에서 ctrl + c를 눌러서 편집 모드를 나가야 합니다.

```bash
tuist edit
```
<img width="1400" alt="스크린샷 2023-01-05 오후 2 37 55" src="https://github.com/imjhk03/modular-architecture/assets/28954046/6c104d2b-496e-4c08-98e0-1c1989215b1b">

### Project.swift

가장 아래에 있는 project가 xcode 프로젝트를 만들기 위한 최종 모습으로 설정하도록 하는 것입니다. 앱의 이름, 플랫폼, 추가로 필요한 타겟들을 추가합니다. 해당 function은 **`Project+Templates.swift`** 파일에 정의되어 있습니다.

```swift
import ProjectDescription
import ProjectDescriptionHelpers
import MyPlugin

/*
                +-------------+
                |             |
                |     App     | Contains MyApp App target and MyApp unit-test target
                |             |
         +------+-------------+-------+
         |         depends on         |
         |                            |
 +----v-----+                   +-----v-----+
 |          |                   |           |
 |   Kit    |                   |     UI    |   Two independent frameworks to share code and start modularising your app
 |          |                   |           |
 +----------+                   +-----------+

 */

// MARK: - Project

// Local plugin loaded
let localHelper = LocalHelper(name: "MyPlugin")

// Creates our project using a helper function defined in ProjectDescriptionHelpers
let project = Project.app(name: "MyApp",
                          platform: .iOS,
                          additionalTargets: ["MyAppKit", "MyAppUI"])
```

### Project+Templates.swift

app function 안에 타겟들을 만들고 최종적으로 Project를 만드는 것을 볼 수 있습니다. InfoPlist에 필요한 데이터를 정리하고 있고 번들 아이디, 각 프로젝트에 필요한 소스들과 리소스들을 정의한 것을 볼 수 있습니다.

```swift
import ProjectDescription

/// Project helpers are functions that simplify the way you define your project.
/// Share code to create targets, settings, dependencies,
/// Create your own conventions, e.g: a func that makes sure all shared targets are "static frameworks"
/// See https://docs.tuist.io/guides/helpers/

extension Project {
    /// Helper function to create the Project for this ExampleApp
    public static func app(name: String, platform: Platform, additionalTargets: [String]) -> Project {
        var targets = makeAppTargets(name: name,
                                     platform: platform,
                                     dependencies: additionalTargets.map { TargetDependency.target(name: $0) })
        targets += additionalTargets.flatMap({ makeFrameworkTargets(name: $0, platform: platform) })
        return Project(name: name,
                       organizationName: "tuist.io",
                       targets: targets)
    }

    // MARK: - Private

    /// Helper function to create a framework target and an associated unit test target
    private static func makeFrameworkTargets(name: String, platform: Platform) -> [Target] {
        let sources = Target(name: name,
                platform: platform,
                product: .framework,
                bundleId: "io.tuist.\(name)",
                infoPlist: .default,
                sources: ["Targets/\(name)/Sources/**"],
                resources: [],
                dependencies: [])
        let tests = Target(name: "\(name)Tests",
                platform: platform,
                product: .unitTests,
                bundleId: "io.tuist.\(name)Tests",
                infoPlist: .default,
                sources: ["Targets/\(name)/Tests/**"],
                resources: [],
                dependencies: [.target(name: name)])
        return [sources, tests]
    }

    /// Helper function to create the application target and the unit test target.
    private static func makeAppTargets(name: String, platform: Platform, dependencies: [TargetDependency]) -> [Target] {
        let platform: Platform = platform
        let infoPlist: [String: InfoPlist.Value] = [
            "CFBundleShortVersionString": "1.0",
            "CFBundleVersion": "1",
            "UIMainStoryboardFile": "",
            "UILaunchStoryboardName": "LaunchScreen"
            ]

        let mainTarget = Target(
            name: name,
            platform: platform,
            product: .app,
            bundleId: "io.tuist.\(name)",
            infoPlist: .extendingDefault(with: infoPlist),
            sources: ["Targets/\(name)/Sources/**"],
            resources: ["Targets/\(name)/Resources/**"],
            dependencies: dependencies
        )

        let testTarget = Target(
            name: "\(name)Tests",
            platform: platform,
            product: .unitTests,
            bundleId: "io.tuist.\(name)Tests",
            infoPlist: .default,
            sources: ["Targets/\(name)/Tests/**"],
            dependencies: [
                .target(name: "\(name)")
        ])
        return [mainTarget, testTarget]
    }
}
```

타겟들은 실제로 앱을 구동하는 Application이 있고, 프레임워크를 추가하는 Target들이 있습니다. 예를 들면 아래와 같습니다.

```swift
MyProject(Application)
ㄴ MyProjectUtility(Framework)
ㄴ DesignSystem(Framework)
```

프레임워크를 역할하는 타겟들은 `makeFrameworkTargets` function으로 정리하고, `makeAppTargets`는 애플리케이션 전체 내용을 정의하고 있습니다.

여기서 번들 아이디와 deploymentTarget을 iOS 13으로 추가해서 프로젝트를 생성하도록 하겠습니다.

```swift
extension Project {
    static let organizationName = "me.kimjh3"

    ...

    let mainTarget = Target(
            name: name,
            platform: platform,
            product: .app,
            bundleId: "\(organizationName).\(name)",
            deploymentTarget: .iOS(targetVersion: "13.0", devices: [.iphone, .ipad]),
            infoPlist: .extendingDefault(with: infoPlist),

    ...
```

여기까지 최소한의 수정을 하고 xcode를 닫은 다음에 터미널에서 ctrl + c를 눌러 편집 모드를 닫습니다. 그리고 `tuist generate` 명령어를 런해서 xcode를 생성합니다.

<img width="550" alt="스크린샷 2023-01-05 오후 2 49 30" src="https://github.com/imjhk03/modular-architecture/assets/28954046/81e07f24-1bce-4e5f-bcc4-b14733d26082">

아래와 같이 xcode가 실행되고, Project.swift 파일에 정의한 내용이 반영되어 있는 것을 확인할 수 있습니다.

<img width="1339" alt="스크린샷 2023-01-05 오후 2 52 33" src="https://github.com/imjhk03/modular-architecture/assets/28954046/5b1ddbe7-d666-4aab-82b5-2deccadd22eb">

## 프레임워크

Tuist으로 새로운 프로젝트를 생성하면 처음부터 Kit 프레임워크랑 UI 프레임워크가 만들어진 것을 확인할 수 있습니다. 제가 생각한 구조랑 비슷합니다. Kit은 Core, UI 부분은 DesignSystem으로 보면 됩니다.

<img width="1207" alt="스크린샷 2023-01-05 오후 2 53 24" src="https://github.com/imjhk03/modular-architecture/assets/28954046/b3c73ddd-31f2-4ab6-911b-7d20dc0e609e">

이렇게 된 구조에서 각 기능 단위의 프로젝트를 더 만들어서 프로젝트를 생성해 봅시다. 폴더 구조를 이해해야 하는데, 아래와 같이 구성되어 있습니다.

<img width="297" alt="스크린샷 2023-01-05 오후 2 56 02" src="https://github.com/imjhk03/modular-architecture/assets/28954046/2e701336-5254-436f-a272-ae4d980c89f9">

Project 폴더 안에는 Core 폴더, Features 폴더, ModularAchitecture 프로젝트, UserInterface 폴더가 있습니다.

Core 폴더 안에는 Utility 프로젝트가 있습니다. 유틸리티 관련 프레임워크로 보면 됩니다.

Features 폴더 안에는 ProductDetail, TagList 프로젝트가 있습니다. 각각을 기능 단위의 데모 앱까지 있는 프로젝트입니다.

참고

[카카오뱅크 iOS 프로젝트의 모듈화 여정: Tuist를 활용한 모듈 아키텍처 설계 사례](https://if.kakao.com/2022/session/88)
