---
layout: post
title:  앱의 상태변화 & Delegate Call
date:   2020-02-20
image:  images/2020-02-20/1.png
tags:   [iOS]
---

(본 게시글은 iOS 앱 생명주기와 관련된 게시물 중 두번째인 **앱의 상태변화 & `Delegate Call`**에 대해 다루고 있습니다.)

이번 게시물에서는 앱이 갖는 상태의 종류와 변화하는 과정, 그리고 상태변화 시 호출되는 method들에 대해 정리해보겠습니다.

---

#### App State
1. **Not Running**
   - 앱이 아직 실행되지 않았거나 종료된 상태
2. **Inactive**
   - 앱이 실행중(Foreground 상태)이지만 이벤트를 받고있지 않는 상태
   - Active 상태로 넘어가기 동안 짧게 이 상태에 머무른다.
3. **Active**
   - 앱이 실행중(Foreground 상태)이고 이벤트를 받고있는 상태
   - 앱이 실질적으로 활동하고 있는 상태이며 대부분의 Foreground 상태에 있는 앱들은 이 상태에 머무른다.
4. **Background**
    - 앱이 백그라운드에서 실질적인 동작을 하고 있는 상태
    - Suspended 상태로 넘어가기 전에 잠깐 머무르며, 추가적인 코드 수행이 필요하면 더 머무를 수도 있다.
5. **Suspended**
    - 백그라운드에서 활동을 멈춘 상태
    - 다시 앱을 실행했을 때 빨리 실행되게 하기 위해 메모리에 적재되어 있으나, 실질적인 코드를 수행하진 않는다.
    - 메모리가 부족한 경우 iOS 시스템은 해당 상테에 있는 앱들을 종료함으로써 메모리를 확보한다.

---

#### Delegate Call
- 앱의 상태변화가 일어나면 `UIApplicationDelegate`를 통해 이벤트가 전달됩니다.
- `UIApplicationDelegate`를 참조하는 `AppDelegate` 객체에서 상태 변화에 따른 작업을 정의할 수 있습니다.
- `Delegate Call`은 앱의 실행과 종료를 처리하는 **Process Lifecycle**과 UI의 상태를 처리하는 **UI Lifecycle**로 나누어집니다.

---

##### 1. Process Lifecycle
~~~swift
1. func application(_ application: UIApplication, 
                    willFinishLaunchingWithOptions launchOptions: 
                        [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool
~~~
- 앱이 최초로 실행될 때 호출 (Not Running)
- 주로 앱 실행 시 최초로 실행할 코드를 작성합니다.
- ex. `UIWindow` 객체를 생성하고 `rootViewController`로 지정할 `UIViewController` 인스턴스를 할당

~~~swift
2. func application(_ application: UIApplication, 
                    didFinishLaunchingWithOptions launchOptions: 
                        [UIApplication.LaunchOptionsKey: Any]?) -> Bool
~~~
- 앱이 실행되고 사용자에게 화면이 보이기 직전에 호출 (Not Running → Inactive)
- 주로 앱 실행 후에 최종 초기화 코드를 작성합니다.
  
~~~swift
3. func applicationWillTerminate(_ application: UIApplication)
~~~
- 앱이 종료되기 직전 호출
- 다음과 같은 경우에는 호출되지 않습니다.
  1. 메모리 확보를 위해 iOS 시스템에 의해 앱이 종료된 경우 (Suspended 상태인 경우)
  2. 사용자가 멀티태스킹 UI를 통해 종료한 경우
  3. 오류로 인해 앱이 종료된 경우
  4. 기기를 재부팅한 경우

---

##### 2. UI Lifecycle

~~~swift
1. func applicationDidBecomeActive(_ application: UIApplication)
~~~
- 앱이 Active 상태로 전횐된 직후 호출 (Inactive → Active)

~~~swift  
2. applicationWillResignActive(_ application: UIApplication)
~~~
- 앱이 Inactive 상태로 전환되기 직전 호출 (Active → Inactive)
  
~~~swift
3. applicationDidEnterBackground(_ application: UIApplication)
~~~
- 앱이 Background 상태로 전환된 직후 호출 (Inactive → Background)
- 특별한 처리가 없으면 Suspended 상태로 전환됩니다.
- 주로 앱이 종료되기 전에 중요 데이터를 저장하거나 공유 자원을 해제하는 등을 위한 코드를 작성합니다.

~~~swift
4. applicationWillEnterForeground(_ application: UIApplication)
~~~
- 앱이 Background에서 Foreground로 돌아오기 직전 호출 (Background → Foreground)

---

![app_based_state_transition]({{site.baseurl}}/images/2020-02-20/app_based_state_transition_2.jpeg)

---

##### Reference
- [[iOS] App 생명주기(Life Cycle)](https://velog.io/@cskim/iOS-App-생명주기Life-Cycle)
- [[iOS] 앱의 생명주기(App Life Cycle)와 앱의 구조(App Structure)](https://jinshine.github.io/2018/05/28/iOS/앱의%20생명주기(App%20Life%20Cycle)와%20앱의%20구조(App%20Structure)/)
- [Managing Your App's Life Cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)

---

##### iOS 앱 생명주기
1. [iOS 앱 생명주기]({{site.baseurl}}/2020/02/20/iOS-앱-생명주기)
2. [**앱의 상태변화 & Delegate Call**]({{site.baseurl}}/2020/02/20/앱의-상태변화-&-Delegate-Call)
3. [AppDelegate & SceneDelegate]({{site.baseurl}}/2020/02/20/AppDelegate-&-SceneDelegate)
