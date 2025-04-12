# DownLink
https://apps.apple.com/kr/app/luminaview/id6737554316

# Spec
### iOS 18.0 +
### 인터넷 연결 필요
### Apple ID

# Skills
### UIFramework: UIKit, SwiftUI
### Library: AVFoundation, AVFAudio, StoreKit2, AuthenticationServices, Firebase Auth, Voice Over, Translation
### Third party: RxSwift
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


// MARK: Child Coordinator 예시
final class DriveModeFlowCoordinator: Coordinator {
    var children: [any Coordinator] = [] 
    private weak var navigationController: UINavigationController?
    private weak var delegate: DriveModeFlowCoordinatorDelegate?
    private let dependencies: DriveModeFlowCoordinatorDependencies
    private var onDismissForViewController: [UIViewController: (() -> Void)] = [:]
    
    init(navigationController: UINavigationController? = nil,
         dependencies: DriveModeFlowCoordinatorDependencies,
         parentDelegate: DriveModeFlowCoordinatorDelegate?) {
        self.navigationController = navigationController
        self.dependencies = dependencies
        self.delegate = parentDelegate
    }
    
    func start(animated: Bool, onDismissed: (() -> Void)?) {
        let actions = DriveModeViewModelActions(
            showCameraPreview: showCameraPreview,
            showPaymentScene: presentPaymentView,
            dismissCameraPreview: dismissCameraPreviewScene,
            dismiss: dismiss,
            presentLoginView: presentLoginView
        )
        let vc = dependencies.makeDriveModeViewController(actions: actions)
        onDismissForViewController[vc] = onDismissed
        navigationController?.pushViewController(vc, animated: animated)
    }
    
    private func dismiss(for viewController: UIViewController) {
        guard let onDismiss = onDismissForViewController[viewController] else { return }
        onDismiss()
        onDismissForViewController[viewController] = nil
    }
    
    func presentLoginView() {
        delegate?.presentLoginScene()
    }
    
    func presentPaymentView() {
        delegate?.presentPaymentScene()
    }
}

// MARK: Parent Coordinator 예시
final class AppFlowCoordinator: Coordinator, DriveModeFlowCoordinatorDelegate, LoginSceneFlowCoordinatorDelegate, PaymentSceneFlowCoordinatorDelegate, OnBoardingSceneFlowCoordinatorDelegate {
    var children: [Coordinator] = []
    var navigationController: UINavigationController
    private let appDIContainer: AppDIContainer
    
    init(navigationController: UINavigationController, appDIContainer: AppDIContainer) {
        self.navigationController = navigationController
        self.appDIContainer = appDIContainer
    }
    
    func start(animated: Bool, onDismissed: (() -> Void)?) {
        let hasLaunchedBefore = UserDefaults.standard.bool(forKey: "hasLaunchedBefore")
        if hasLaunchedBefore {
            presentDriveModeScene()
        } else {
            UserDefaults.standard.set(true, forKey: "hasLaunchedBefore")
            UserDefaults.standard.synchronize() // 즉시 저장
            presentOnboardingScene()
        }
    }
    
    func presentDriveModeScene() {
        navigationController.setViewControllers([], animated: false)
        let driveModeSceneDIContainer = appDIContainer.makeDriveModeSceneDIContainer()
        let coordinator = driveModeSceneDIContainer.makeDriveModeSceneFlowCoordinator(navigationController: navigationController, parentCoordinator: self)
        presentChild(coordinator, animated: false)
    }
    
    func presentLoginScene() {
        let loginSceneDIContainer = appDIContainer.makeLoginSceneDIContainer()
        let coordinator = loginSceneDIContainer.makeLoginSceneFlowCoordinator(navigationController: navigationController, parentCoordinator: self)
        presentChild(coordinator, animated: false)
    }
    
    func presentPaymentScene() {
        let paymentDIContainer = appDIContainer.makePaymentDIContainer()
        let coordinator = paymentDIContainer.makePaymentSceneFlowCoordinator(navigationController: navigationController, parentCoordinator: self)
        presentChild(coordinator, animated: false)
    }
    
    func dismissScene(scene coordinator: Coordinator) {
        removeChild(coordinator)
    }
}
```

<section id="explain_data_stream">
</section>

# 카메라 데이터와 네트워크 전송 흐름
<img width="639" alt="image" src="https://github.com/user-attachments/assets/384bccaf-adbe-49b8-9a10-353a4153bccb" />

## 의도
  - Camera, Network 모듈간 강한결합을 회피
  - 방출하는 데이터 흐름을 단방향 데이터 흐름을 구축
  - ViewModel에서 스트림을 생성해 데이터 스트림 중앙집중관리

## 요약코드
```swift
// 카메라 기능 프로토콜
protocol Recodable {
    func configureCamera()
    func startRecord(subject: PublishSubject<(Data,URL)>)
    func stopRecord()
    func setPreview(view: UIView)
    func recordingStatusStream() -> BehaviorSubject<Bool>
    func cameraAvailabilityStream() -> BehaviorSubject<Bool>
}

final class CameraManger: NSObject, Recodable {
    // 핵심 프로퍼티
    private var captureSession: AVCaptureSession?
    private var movieOutput: AVCaptureMovieFileOutput?
    private var cameraFileSubject: PublishSubject<(Data,URL)>?
    private var cameraRecodingCheckSubject: BehaviorSubject<Bool>
    private var cameraAvailableSubject: BehaviorSubject<Bool>
    
    // 주요 메서드
    func configureCamera() {
        // 카메라 세션, 입출력 설정
        // 오류 발생 시 cameraAvailableSubject.onError() 호출
    }
    
    func startRecord(subject: PublishSubject<(Data,URL)>) {
        cameraFileSubject = subject
        captureSession?.startRunning()
        startFileOutput()
        isRecording()
    }
    
    func stopRecord() {
        captureSession?.stopRunning()
        cameraFileSubject?.onCompleted()
        cameraFileSubject = nil
    }
    
    // AVCaptureFileOutputRecordingDelegate 구현
    func fileOutput(_ output: AVCaptureFileOutput, didFinishRecordingTo outputFileURL: URL, from connections: [AVCaptureConnection], error: Error?) {
        // 파일 데이터를 읽어서 subject로 전달
        let fileData = try Data(contentsOf: outputFileURL)
        cameraFileSubject?.onNext((fileData, outputFileURL))
        startFileOutput() // 다음 녹화 시작
    }
}

// Websocket 모듈
protocol SendableWebSocket: AnyObject {
    func sendToWebSocket(data: Data, fileURL: URL)
}

final class WebSocketRepository: GuideAPIWebRepository, SendableWebSocket {
    // 핵심 프로퍼티
    private var webSocketTask: URLSessionWebSocketTask?
    private var session: URLSession!
    private var messageSubject: PublishSubject<URLSessionWebSocketTask.Message>?
    private var isConnectionLess: Bool = true
    var resultStream: PublishSubject<String>?
    
    // 주요 메서드
    func setupResultStream(resultStream: PublishSubject<String>) {
        self.resultStream = resultStream
    }
    
    func setupAPIConnect(requestStream: PublishSubject<(Data,URL)>, authToken: String) {
        setupWebSocket(authToken: authToken)
        
        // 카메라에서 오는 데이터 구독
        requestStream
            .subscribe(onNext: { [weak self] in
                let fileData = $0.0
                let fileURL = $0.1
                self?.sendToWebSocket(data: fileData, fileURL: fileURL)
            })
            .disposed(by: disposeBag)
    }
    
    private func setupWebSocket(authToken: String) {
        // 웹소켓 설정 및 메시지 구독
        messageSubject = PublishSubject<URLSessionWebSocketTask.Message>()
        messageSubject?
            .subscribe(onNext: { [weak self] message in
                switch message {
                case .string(let text):
                    self?.resultStream?.onNext(text)
                // ...
                }
                self?.listenForMessage()
            })
            .disposed(by: disposeBag)
            
        listenForMessage()
    }
    
    func sendToWebSocket(data: Data, fileURL: URL) {
        // 데이터를 청크로 나눠서 웹소켓으로 전송
        // 전송 완료 후 파일 삭제
    }
}

// ViewModel
struct DriveModeViewModelActions {
    let showCameraPreview: (_ viewModel: DriveModeViewModel) -> Void
    let showPaymentScene: () -> Void
    let dismissCameraPreview: () -> Void
    let presetionLoginView: () -> Void
}

final class DriveModeViewModel: ObservableObject {
    // UI 바인딩 프로퍼티
    @Published var guideContent: String = ""
    @Published var isClear = false
    
    // 주입받은 의존성
    private let fetchGuideUseCase: FetchGuideUseCase // WebSocketRepository 사용
    private let stopGuideUseCase: StopGuideUseCase
    private let cameraManager: Recodable
    private let speakerManager: Speakable
    private var userInfo: UserInfo?
    
    // 주요 메서드
    func startRecordFlow() -> (PublishSubject<String>, Bool) {
        let requestStream = PublishSubject<(Data, URL)>()
        let resultStream = PublishSubject<String>()
        
        // 사용자 로그인/사용량 확인 후
        // 1. 웹소켓 연결 및 결과 관찰 설정
        createResultObserver(stream: resultStream)
        // 2. 가이드 서비스 요청 시작
        fetchGuideUseCase.execute(requestStream: requestStream,
                                  resultStream: resultStream,
                                  authToken: token)
        // 3. 카메라 녹화 시작
        cameraManager.startRecord(subject: requestStream)
        
        return (resultStream, isPossibleStart)
    }
    
    private func createResultObserver(stream: PublishSubject<String>) {
        stream
            .subscribe(onNext: { [weak self] content in
                self?.handleContent(content)
            }, onError: { [weak self] error in
                self?.handleError(error)
            })
            .disposed(by: disposeBag)
    }
    
    private func handleContent(_ content: String) {
        // 서버 응답 처리 및 UI 업데이트
        DispatchQueue.main.async {
            self.guideContent = content
            self.translationVersion += 1
        }
        decreaseUsagage() // 사용량 차감
    }
    
    func stopRecordFlow() {
        // 사용량 업데이트
        // 가이드 서비스 중단
        stopGuideUseCase.execute()
        // 카메라 녹화 중단
        stopRecord()
    }
}

// 데이터 흐름 코드
// 1. 시작: ViewController에서 뷰모델의 메서드 호출
let (resultStream, canStart) = viewModel.startRecordFlow()

// 2. 내부 흐름
// DriveModeViewModel 내부
func startRecordFlow() {
    let requestStream = PublishSubject<(Data, URL)>()
    let resultStream = PublishSubject<String>()
    
    // ViewModel → UseCase → WebSocketRepository
    fetchGuideUseCase.execute(requestStream: requestStream,
                              resultStream: resultStream,
                              authToken: token)
    
    // ViewModel → CameraManager
    cameraManager.startRecord(subject: requestStream)
    
    // 결과 스트림 구독
    createResultObserver(stream: resultStream)
}

// 3. 데이터 전달 (CameraManager → WebSocketRepository)
// CameraManager 내부 (AVCaptureFileOutputRecordingDelegate 콜백)
func fileOutput(_ output: AVCaptureFileOutput, didFinishRecordingTo outputFileURL: URL, ...) {
    let fileData = try Data(contentsOf: outputFileURL)
    cameraFileSubject?.onNext((fileData, outputFileURL))
}

// 4. WebSocketRepository에서 데이터 처리 및 결과 반환
// WebSocketRepository 내부
messageSubject?.subscribe(onNext: { [weak self] message in
    switch message {
    case .string(let text):
        self?.resultStream?.onNext(text) // 결과를 ViewModel로 전달
    }
})

// 5. ViewModel에서 결과 처리
// DriveModeViewModel 내부
private func handleContent(_ content: String) {
    DispatchQueue.main.async {
        self.guideContent = content // UI 업데이트
    }
    decreaseUsagage() // 사용량 차감
}

```
<section id="problem-solving">
</section>

# 고민 및 문제 해결 과정
### H264 AVC (Advanced Video Coding / MPEG-4 Part 10) vs H265
- **문제:** WebSocket을 통해 데이터를 송수신하는 환경에서 압축 효율이 중요함  
- **해결:** 압축 효율이 뛰어난 H265(HEVC)를 사용하기로 결정  
  > H.265(HEVC)는 AVC보다 약 50% 더 효율적인 압축이 가능

### 어느 Multimodal AI를 사용해야 할까?
- **고민:** 개발 당시 비용 효율성이 중요한 요소였음  
- **결과:** Gemini Flash가 가장 저렴하여 선택됨
<img width="487" alt="image" src="https://github.com/user-attachments/assets/ce6aa31d-733c-4a94-bd63-cd8fd60f2c90" />

### Gemini API 보안 및 통신 방식 선택
- **문제:** Gemini API 보안을 위해 서버가 필요하며, 클라이언트와 서버 간 통신 방식 선택이 필요함  
- **해결:**  
  - 앱에서 5초 동안 수집되는 이미지 데이터 양을 고려하여, 16KB~64KB 사이의 데이터 송신에 유리한 WebSocket을 채택  
  - HTTP를 사용할 경우 5초마다 요청이 발생하여 오버헤드가 증가로 WebSocket 방식을 선택함

### 네트워크 인터페이스 변경에 따른 WebSocket 연결 핸들링
- **문제:** 네트워크 인터페이스 변경(예: Cellular → WiFi → Cellular) 시, 기존 연결이 최신 정보로 업데이트되지 않아 혼란이 발생할 수 있음  
- **해결:**  
  - 인터페이스 변경 시 새로운 WebSocket 연결을 생성하고 기존 연결은 종료하여 최신 정보를 반영  
  - 이전 연결은 4-way handshake가 일어나기 전에 인터페이스 변경으로 인해 제대로 종료되지 않는 문제를, 리버스 프록시 설정으로 idle 상태의 스트림을 주기적으로 제거하여 해결

