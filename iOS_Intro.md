# Spec
### iOS 18.0 +
### 인터넷 연결 필요
### Apple ID

# Skills
### UIFramework: UIKit, SwiftUI
### Library: AVFoundation, AVFAudio, StoreKit2, AuthenticationServices, Firebase Auth, Voice Over, Translation
### Local Database: User Defaults
### Remote Database: Firebase Cloud Firestore

# Functions

  <Table>
    <tr>
    <td align = "center">
      <img src="https://github.com/user-attachments/assets/6c6d79b0-0ec7-419c-8379-3106a89c312a" width="300" alt="주행모드" />
    </td>
    <td valign= "top">
      <p>
        <h2> 주행모드 </h2>
        <li> Web Server와 WebSocket으로 통신 방식을 사용합니다. </li>
        <li> 5초간의 동영상 데이터를 수집, HEVC방식 인코딩 후 JSON 데이터 포맷을 만들어 송신합니다. </li>
        <li> 안전하거나 참고정보가 없다면 진동 발생하며, 안내사항은 디바이스 지역설정에 따라 자동으로 번역되어 안내됩니다.</li>
      </p>
    </td>
  </tr>
    <tr>
    <td align = "center">
      <img src="https://github.com/user-attachments/assets/dfeb1273-7bde-4808-ae6f-2bd9efaa40d5" width="300" alt="네트워크 변경 감지 자동연결" />
    </td>
    <td valign= "top">
      <p>
        <h2> 네트워크 변경 감지 자동연결 </h2>
        <li> NWPathMonitor를 활용해 네트워크 변경 감지시 웹소캣 연결 스트림을 새로 생성합니다. </li>
        <li> 기존에 있던 연결 스트림은 앱 특성상 최신정보를 주고받아야 하기 때문에 자동으로 서버측에서 리소스를 해제합니다. </li>
      </p>
    </td>
  </tr>
    <tr>
    <td align = "center">
      <img src="https://github.com/user-attachments/assets/e6e7b471-4526-4a4f-af5d-6b50d2612ec8" width="300" alt="구매" />
    </td>
    <td valign= "top">
      <p>
        <h2> 구매 </h2>
        <li> AppStoreKit2로 구현 </li>
        <li> 영수증 검증과 구매에 대한 검증을 애플 내부에 위임후 구매 상품 정보는 Firebase에서 관리합니다. </li>
      </p>
    </td>
  </tr>
      <tr>
    <td align = "center">
      <img src="https://github.com/user-attachments/assets/0beafd74-b5ae-4f3f-b7b9-af2dc843725b" width="300" alt="로그인" />
    </td>
    <td valign= "top">
      <p>
        <h2> 로그인 </h2>
        <li> iOS가 타겟으로 애플 로그인만 구현되어 있습니다. </li>
      </p>
    </td>
  </tr>
     <tr>
    <td align = "center">
      <img src="https://github.com/user-attachments/assets/cd6aae39-8e13-4fca-b05e-a6a7321dc62e" width="300" alt="내정보 확인(남은 사용량 체크)" />
    </td>
    <td valign= "top">
      <p>
        <h2> 내정보 확인(남은 사용량 체크) </h2>
        <li> 상품구매시 적용된 사용량 정보를 표출합니다. </li>
      </p>
    </td>
  </tr>
  </Table>
  
<section id="Architecture">
</section>

# Architecture
### Clean Architecture + MVVM-C
<img width="1193" alt="image" src="https://github.com/user-attachments/assets/90e64b78-560e-4a8b-b188-86afd489d296" />
의존성 주입 관리는 DIContainer를 활용해 수행했습니다.

예제코드는 다음과 같고 생략된 부분입니다.
```swift
final class AppDIContainer {
    //MAKR: 인프라 의존성 주입
    lazy var appConfigurations = AppConfigurations()
    lazy var guideService: GuideAPIWebRepository = {
        let config = WebSocketAPIConfig(url: appConfigurations.webSocketURL)
        
        return WebSocketRepository(config: config)
    }()
    
    lazy var userInfoService: UserInfoRepository = {
        return FirebaseUserInfoRepository()
    }()
    
    func makeDriveModeSceneDIContainer() -> DriveModeSceneDIContainer {
        let dependencies = DriveModeSceneDIContainer.Dependencies(guideAPIWebRepository: guideService,
                                                                  userInfoRepository: userInfoService)
        
        return DriveModeSceneDIContainer(dependencies:  dependencies)
}

final class DriveModeSceneDIContainer: DriveModeFlowCoordinatorDependencies {
    struct Dependencies {
        let guideAPIWebRepository: GuideAPIWebRepository
        let userInfoRepository: UserInfoRepository
    }
    
    private let dependencies: Dependencies
    
    init(dependencies: Dependencies) {
        self.dependencies = dependencies
    }
    
    // MARK: Utilites
    ...

    // MARK: UseCase
    func makeDriveModeUsecase() -> FetchGuideUseCase {
        FetchGuideUseCase(guideAPIWebRepository: dependencies.guideAPIWebRepository)
    }

    func makefetchUserInfoUseCase() -> FetchUserInfoUseCase {
        FetchUserInfoUseCase(repository: dependencies.userInfoRepository)
    }
    
    // MARK: ViewModel
    func makeDriveModeViewModel(actions: DriveModeViewModelActions) -> DriveModeViewModel {
        let viewModel = DriveModeViewModel(fetchGuideUseCase: makeDriveModeUsecase(),
                                           fetchUserInfoUseCase: makefetchUserInfoUseCase(),
                                           actions: actions)
        
        return viewModel
    }
    
    // MARK: Presentation
    func makeDriveModeViewController(actions: DriveModeViewModelActions) -> DriveModeViewController {
        let vc = DriveModeViewController()
        vc.create(viewModel: makeDriveModeViewModel(actions: actions))
        
        return vc
    }
    
    func makeDriveModeSceneFlowCoordinator(navigationController: UINavigationController, parentCoordinator: DriveModeFlowCoordinatorDelegate) -> DriveModeFlowCoordinator {
        DriveModeFlowCoordinator(navigationController: navigationController, dependencies: self, parentDelegate: parentCoordinator)
    }
```

<section id="problem-solving">
</section>

# 고민 및 문제 해결 과정

### H264 AVC (Advanced Video Coding / MPEG-4 Part 10) vs H265
- **문제:** WebSocket을 통해 데이터를 송수신하는 환경에서 압축 효율이 중요함  
- **해결:** 압축 효율이 뛰어난 H265(HEVC)를 사용하기로 결정  
  > H.265(HEVC)는 AVC보다 약 50% 더 효율적인 압축이 가능합니다.

### 어느 Multimodal AI를 사용해야 할까?
- **고민:** 개발 당시 비용 효율성이 중요한 요소였음  
- **결과:** Gemini Flash가 가장 저렴하여 선택됨
<img width="487" alt="image" src="https://github.com/user-attachments/assets/ce6aa31d-733c-4a94-bd63-cd8fd60f2c90" />


### Gemini API 보안 및 통신 방식 선택
- **문제:** Gemini API 보안을 위해 서버가 필요하며, 클라이언트와 서버 간 통신 방식 선택이 필요함  
- **해결:**  
  - 앱에서 5초 동안 수집되는 이미지 데이터 양을 고려하여, 16KB~64KB 사이의 데이터 송신에 유리한 WebSocket을 채택  
  - HTTP를 사용할 경우 5초마다 요청이 발생하여 오버헤드가 증가하므로 WebSocket 방식을 선택함

### 네트워크 인터페이스 변경에 따른 WebSocket 연결 핸들링
- **문제:** 네트워크 인터페이스 변경(예: Cellular → WiFi → Cellular) 시, 기존 연결이 최신 정보로 업데이트되지 않아 혼란이 발생할 수 있음  
- **해결:**  
  - 인터페이스 변경 시 새로운 WebSocket 연결을 생성하고 기존 연결은 종료하여 최신 정보를 반영  
  - 이전 연결은 4-way handshake가 일어나기 전에 인터페이스 변경으로 인해 제대로 종료되지 않는 문제를, 리버스 프록시 설정으로 idle 상태의 스트림을 주기적으로 제거하여 해결함
