---
layout: post
title:  AppDelegate & SceneDelegate
date:   2020-02-20
image:  images/2020-02-20/2.jpg
tags:   [iOS]
---

(본 게시글은 iOS 생명주기와 관련된 게시물 중 세번째인 **`AppDelegate` & `SceneDelegate`**에 대해 다루고 있습니다.)

Xcode 11 업데이트 이후 프로젝트를 생성하면 `SceneDelegate.swift` 파일이 추가로 생성됩니다. 이번 게시물에서는 왜 `SceneDelegate`가 생겼고, 기존의 `AppDelegate`에서 어떤 부분이 바뀌었는지 정리해보겠습니다.

(게시물을 보시기 전에 관련된 [WWDC2019 영상(15:27)](https://developer.apple.com/videos/play/wwdc2019/258/)을 시청하시길 권장합니다.)

---

#### Multi Window
- 최신 버전의 아이패드부터는 멀티 윈도우라는 개념이 도입되어 하나의 앱을 여러 개 띄울 수 있게 되었습니다.
- 예를 들어, 각각 다른 사이트에 접속된 Safari 앱을 여러 개 띄운다던가, 각종 앱들 옆에 split view로 메모 앱을 각각 띄우는게 가능해졌죠.
- 이때, 각각의 윈도우를 따로 관리해줘야할 필요가 생겼고 따라서 `SceneDelegate`가 도입되었습니다.

---

#### Scene
- 기존의 앱에서는 '하나의 프로세스'는 '하나의 UI'만 가졌기 때문에 `AppDelegate`가 Process Lifecycle과 UI Lifecycle을 모두 담당했습니다. 
- 하지만 iOS 13부터는 하나의 앱 내에서 같은 프로세스를 공유하는 여러 개의 `scene`이 존재할 수 있게 됩니다.
- 따라서 Process Lifecycle은 `AppDelegate`가, UI Lifecycle은 `SceneDelegate`가 나눠갖게 됩니다.

---

#### AppDelegate에서 없어진 것
1. **`window`**
   - 앱 내에서 여러 개의 `window`나 `UISceneSession`을 띄울 수 있기 때문에 `AppDelegate`에서 `window` 개체가 사라졌습니다.
   - 따라서 Xcode 11 이상에서 iOS 12 이하를 지원하는 앱을 만드려면 임의로 `window`를 만들어야 합니다.
2. **UI Lifecycle**
    - 해당 앱이 iOS 13 이상을 지원하고 새로운 Scene Lifecycle을 채택하면 기존에 `AppDelegate`에 있던 UI Lifecycle 관련 method들의 사용이 중지됩니다.
    - `AppDelegate`의 method와 `SceneDelegate`에 새로 생긴 UI Lifecycle method가 1:1로 매치되기 때문에 기존의 사용 방식에서 크게 달라지지는 않았습니다.
     ![match]({{site.baseurl}}/images/2020-02-20/match.jpg)

---

#### AppDelegate에서 추가된 것
- Process Lifecycle 역할을 하는 method 3개가 새로 추가되었습니다

~~~swift
1. func application(_ application: UIApplication, 
                    willFinishLaunchingWithOptions launchOptions: 
                        [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool
~~~
 - 앱이 최초로 실행될 때 호출
 - 기존에는 `UIWindow` 객체를 만들어 `UIViewController` 인스턴스를 할당하는 등의 작업을 했지만, 앱에서 `scene`을 사용하는 경우엔 더이상 그런 작업들을 수행하지 않습니다.

~~~swift
2. func application(_ application: UIApplication, 
                   configurationForConnecting connectingSceneSession: UISceneSession, 
                   options: UIScene.ConnectionOptions) -> UISceneConfiguration
~~~
- 앱에서 새로운 `scene` 또는 `window`를 제공할 때 호출
  
~~~swift
3. func application(_ application: UIApplication, 
                    didDiscardSceneSessions sceneSessions: Set<UISceneSession>)
~~~
- 사용자가 `scene`을 버릴 때 호출
- ex. 멀티태스킹 UI를 통해 앱을 제거한 경우

---

#### SceneDelegate
- UI Lifecycle을 담당하는 역할을 합니다.
- 상태변화가 일어나면 `UIWindowSceneDelegate`를 통해 이벤트가 전달됩니다.

~~~swift
1. func scene(_ scene: UIScene, 
              willConnectTo session: UISceneSession, 
              options connectionOptions: UIScene.ConnectionOptions)
~~~
- 최초 생성될 때 호출
- 새로운 `UIWindow` 객체를 생성하고 `rootViewController`를 설정하며 `scene`을 할당하는 작업 등을 수행
- 연결이 끊어진 `scene`을 복원하는 작업도 수행

~~~swift
2. func sceneDidDisconnect(_ scene: UIScene)
~~~
- scene의 연결이 끊어질 때 호출
- 앱이 종료되었다는 의미는 아니며, 해당 `scene`은 다시 연결하여 사용할 수 있음
- 주로 보관할 필요가 없는 자원을 버리는 작업을 수행

~~~swift
3. func sceneDidBecomeActive(_ scene: UIScene)
~~~
- Active 상태로 전횐된 직후 호출 (Inactive → Active)

~~~swift
4. func sceneWillResignActive(_ scene: UIScene)
~~~
- Inactive 상태로 전환되기 직전 호출 (Active → Inactive)

~~~swift
5. func sceneWillEnterForeground(_ scene: UIScene)
~~~
- 앱이 Background에서 Foreground로 돌아오거나 처음 활성화 되는 경우 호출 (Background → Foreground)

~~~swift
6. func sceneDidEnterBackground(_ scene: UIScene)
~~~
- Background 상태로 전환된 직후 호출 (Inactive → Background)
   

---

![scene_based_state_transition]({{site.baseurl}}/images/2020-02-20/scene_based_state_transition.png)

---

##### Reference
- [Managing Your App's Life Cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)
- [Architecting Your App for Multiple Windows](https://developer.apple.com/videos/play/wwdc2019/258/)
- [AppDelegate vs. SceneDelegate](https://kor45cw.tistory.com/m/297?category=639839)

---

##### iOS 앱 생명주기
1. [iOS 앱 생명주기]({{site.baseurl}}/2020/02/20/iOS-앱-생명주기)
2. [앱의 상태변화 & Delegate Call]({{site.baseurl}}/2020/02/20/앱의-상태변화-&-Delegate-Call)
3. [**AppDelegate & SceneDelegate**]({{site.baseurl}}/2020/02/20/AppDelegate-&-SceneDelegate)