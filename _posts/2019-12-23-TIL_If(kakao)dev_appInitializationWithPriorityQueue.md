# 19/12/23

## if (kakao) dev 2019

### App initialization with priority queue - Sheldon (이상규)님

영상링크: https://mk-v1.kakaocdn.net/dn/if-kakao/conf2019/conf_video_2019/2_102_04_m1.mp4
발표자료: https://mk.kakaocdn.net/dn/if-kakao/conf2019/발표자료_2019/T07-S04-Sheldon.pdf

이번 세션은 카카오 T가 어떻게 초기화 과정을 거치는지에 대한 소개하지만, 일반적으로 개발자들이 겪을 수 있는 이슈나 사례를 소개하고, 그 것들을 잘 해결 하기 위해서 특정 자료구조를 활용한 방안에 대해서 고민해보고자 함.

#### 초기화를 먼저 생각해보자.

- Power On(스위치 과정)
  1. Filament <-> switchOn()
  2. Discharge occurs <-> dischargeInLight()
  3. Mercury Argon <-> fireOnFilament()
  4. UV-rays <-> mercuryMixArgon()
  5. Fluorescent <-> onUVRays()
  6. Visible Light <-> visibleRay()

#### 카카오 T의 초기화 과정

- App Delegate
  - `applicationDidBecomeActive` : 카카오 T 서비스에서는 이 시점을 entry point로 초기화 과정을 이루고 있음.
  - Application의 시작에 필요한 요소들!
    - 기술적인 초기화
    - 네트워크 상황 체크
    - 위치정보 가져오기
    - 서버의 최신 데이터 가져오기
    - 인증 토큰 갱신
    - 로그인 (kakao, Facebook 등)
    - APNS 토큰 등록
  - 기획적 요구사항
    - 디바이스 인증권한 동의
    - 서비스 이용 약관 동의
    - 전면 배너
    - 권장 업데이트 알림
  - kako T 서비스

    - Active Service List: 택시, 주차, 대리 등의 최신 이용 서비스 리스트

  - 위에서 보다시피, 앱 구동시 해야 할 것이 너무 많음. 그 다음의 고민은?

    - 기능별 연속성을 가진 초기화 과정들

    ```swift
    LocationManager.instance.getCurrent() { poi in
      If KOSession.shared().state == .open {
        API.getActiveServiceList() { serviceList in
          AppConfiguration.poi = poi
          AppConfiguration.serviceList = serviceList
        }
        return
      } else {
        KOSession.shared().open(completionHandler: { success in
          if success {
            API.getActiveServiceList() { serviceList in
              AppConfiguration.poi = poi
              AppConfiguration.serviceList = serviceList
            }
          } else { ... }
        }, authTypes: [NSNumber(value: KOAuthType.talk.rawValue)])
      }
    }
    ```

    - 위 코드에는 문제점이 있음. 예를 들어 카카오 인증을 하지 않는다고 결정 났을 경우(KOSession을 빼야되는 경우), 앞 뒤 관계에 있는 Dependency Callback 구조로 된 Location Manager라던지, Actice service 등의 중간을 매개해주는 리팩토링 작업이 필요하게 됨.

- 위 문제를 Smart하게 풀어보기 위한 한가지 대안

  - **Task 단위로 분리하는 것**
    - 서로간의 연속성을 가진 Dependency를 모두 제거해서, 오로지 자기 자신이 목적에만 집중할 수 있도록 하자.
    - 언제든지 Task에 추가, 삭제가 될 수 있어야 한다.
  - **Priority Queue 구조로 구현!**

#### Priority Queue?

- Queue - First In First Out
- Priority Queue
  - Ex) 장을 보면서 5개의 사과를 담고, 당도가 높은 사과 순으로 먹기.
  - 즉, 들어갈 때는 무작위로 들어가도, 나올 때는 우리가 정해놓은 우선순위 순으로 빠져나오게 되는 구조
  - 구현 방법
    1. Array List
    2. Linked List
    3. Heap: 완전 2진트리로 구성된 자료구조

#### 카카오 T initialization로 돌아가보면..

- 우선순위
  1. 디바이스 인증 권한 동의
  2. 유저의 최신정보 확인
  3. 앱의 최신 정보 확인
  4. 서비스 이용약관 동의
  5. 로그인
  6. 위치정보 권한 확인
  7. 운행중인 서비스의 상태
  8. 전면배너
  9. 권장업데이트

- 코드

  ```swift
  enum CoreTask:Int {
  //Init시에 UI와 관련한 task job의 우선순위별로 기술합니다.
  case rankNetwork
  case rankUseLocation
  case rankCheckMyInformation case rankKakaoLogin
  case rankAppInfo
  case rankActiveService
  case rankOptionUpdate
  case rankDailyTask
  case rankFullAdBanner
  }
  ```
  ```swift
  switch self {
    case .rankNetwork: return TaskNetworkAction() // 해당 Task에 대한 Logic을 struct로 구현
    case .rankUseLocation: return TaskUseLocationAction()
    case .rankCheckMyInformation: return TaskCheckMeAction()
    case .rankKakaoLogin: return TaskKakaoLoginAction()
    case .rankAppInfo: return TaskAppInfoAction()
    ...
  }
  ```

- applicationDidBecome에서는..

  ```swift
  func applicationDidBecomeActive(_ application: UIApplication) {
    TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankNetwork, param:nil))
    TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankUseLocation, param:nil))
    TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankCheckMyInfomation, param:nil))
    TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankKakaoLogin, param: nil)
    TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankAppInfo, param: nil))
    TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankActiveServiceList, param: nil))
    ...
    TaskQueueScheduler.instance.startTask() //Queue Task 의 시작!
  }
  ```

#### But Why Priority Queue?

- Priority Queue를 사용해서, 이런 것들을 만들 필요가 있을까? 라는 의문이 듬

- 꼭 사용해야할까? 라는 장점이 보이질 않음. 하지만, 아래 5가지의 경우를 고려해본다면, 얘기가 달라질 수 있음.

  1. Login Task
     - Task4에서 예외처리가 났을 경우, 결국, Task1로 보내야한다. 물론, 캐싱, 함수를 나누던지 여러 방법이 있지만 복잡해짐.
   - Ex) Get my Information -> Check Kakao Auth*Exception 발생

```swift
  struct TaskKakaoLoginAction: TaskActionable {
    func runTask(param: Any?) {
      if KOSession.shared().state == .open {
        self.nextTask()
        return
      } else { //카카오 인증이 정상적인 경우는 다음 task로...
        KOSession.shared().open(completionHandler: { success in //카카오 인증이 안되어 있는경우는 인증 processs를 태운다
        if success {
          TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankCheckMyInformation, param: nil))
          TaskQueueScheduler.instance.startTask()
        } else { ... }
      }, authTypes: [NSNumber(value: KOAuthType.talk.rawValue)]) }
    }
  }
```

2. Push Notification

  - Push payload: AppDelegate - didReceiveRemoteNotification
  - Initial function: AppDelegate - applicationDidBecomeActive
  - 위 두가지가 충돌날 경우

    ```swift
    func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable: Any]) {
      TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankPushNotification, param:JSON(userInfo)))
    }
    ```

    ```swift
    func applicationDidBecomeActive(_ application: UIApplication) {
      TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankCheckMyInfomation, param: nil))
      TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankKakaoLogin, param: nil))
      TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankUseLocation, param: nil))
      TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankAppInfo, param: nil))
      TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankActiveService, param: nil))
      ...
      TaskQueueScheduler.instance.startTask()
    }
    ```

    - pushNotification의 우선순위는 kakaoLogin보다 낮음

  3. Sceme URL

  ```swift
  func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    TaskQueueScheduler.instance.enQueueTask(action: CoreTaskPack(task: .rankSchemeUrl, param:url))
  }
  ```

  - Push Payload에 URL이 들어온다면?
- 우선순위는 pushNotification과 거의 동일한 우선순위
  
- Ranking by Priority?
    - 그렇다면 우선순위를 어떻게 정할까?
    - **Required or Optional**
    - Optional은 대부분 기획적인 요소가 많을 것임.
  
4. 기획적 요구사항
     - Service Banner 우선순위를 가장 마지막에 해두면 됨.
     - 갑자기, 추가적인 banner를 보여줘야하는 상황에서도 우선순위 조정하여 enqueue만 해주면 된다.
  5. 끊임없는  기획적 요구사항
     - 인 실행시 필수/선택 권한 고지 화면 필요
     - 앱 실행시 바로 띄워야함
     - 위와 같은 경우, 우선순위를 최상위로 해두면 된다.

#### Git commit history

- 실제로 required와 optional을 나누었을 때의  commit message를 보여주심!
