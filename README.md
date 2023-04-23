# UIKitProjectToSwiftUIProject

새로운 회사에서 UIKit으로 되어있는 기존 Project를 SwiftUI Project로 바꾸며 생겼던 문제와 해결을 기록해 놓아보았습니다.   

## 기본적으로 UIKit에서 SwiftUI로 Project를 호환하는 방법

### [UIHostingController](https://developer.apple.com/documentation/swiftui/uihostingcontroller)

UIKit에서 SwiftUI View로 넘어가기 위해 사용하는 방법입니다.

### [UIViewControllerRepresentable](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable)

SwiftUI에서 UIViewController를 보여주기 위해 사용하는 방법입니다.


   

**대부분의 사용자는 SwiftUI로 넘어가는 시도를 하고 있을 것이라고 생각합니다. ios16 기점으로 새로 편하게 사용할 수 있는 SwiftUI 기능들이 굉장히 많이 생겼고, 생각보다 SwiftUI로 APP을 만들어보니 이제는 SwiftUI로 개발이 가능할 정도로 버그도 많이 사라졌다고 생각합니다. 하지만 아직 아래의 문제를 많이 겪었고 이를 피하기 위한 해결도 담아보았습니다.**


## 문제점

- UIViewControllerRepresentable을 사용하며 느꼈던 가장 큰 문제점은 ViewController의 dismiss 함수를 호출할때, 포함하고 있는 SwiftUI View에서 문제가 생기는 경우가 많은 것입니다. 그냥 UIViewControllerRepresentable에서 포함하고 있는 UIViewController에서 종료하는 함수만 실행했을 뿐인데 포함하고 있는 SwiftUIView를 감싸고 있는 NavigationView가 종료되는 문제도 있었습니다. 가장 큰 문제는 이러한 문제들의 명확한 원인을 찾아 관리가 힘들다는 것입니다.


- 두 번째 두 방식의 Navigation 계층을 같이 관리하기가 힘듭니다. UIKit의 NaviagationController도 이용하고 SwiftUIView의 Navigation View를 같이 이용하면 개념의 혼합이 생겨 관리하기가 힘듭니다. 특히 @EnvironmentObject, @Environment 변수와 navigationbar의 관리에 어려움을 겪었습니다.


## 해결책 

(아직 해보는 중, 모두 진행되면 코드까지 정리해서 보여주려함.)    
ViewController의 dismiss 호출을 피하는 것이 첫번쨰 포인트.   
ViewController의 화면 전환 제어를 모두 SwiftUI 쪽에 넘기는 것이 두번째 포인트.  
화면전환의 핵심 문제들이 해결되면 ViewController의 내부 로직 및 화면의 대체는 오래 걸리는 작업이므로 선택적으로 필요할 때, 할 수 있다는 것이 세번째 포인트.   

- 모든 ViewController를 UIViewControllerRepresentable로 Wrapping합니다.

- 위의 문제점을 해결하기 위해 저는 Navigation, 화면전환의 제어를 오직 SwiftUI에서만  사용할 수 있도록 하였습니다. 이 방법은 한번에 이루어져야합니다. 모든 화면수를 세아려 ViewController의 아래 함수들을 모두 대체해야합니다.

    - navigationController.push...
    - self.performSegue
    - self.present
    - selfdismiss
    - navigationController.pop...   
   
- ViewController의 위 함수들을 모두 SwiftUIView의 함수로 대체하였다면, ViewControllerRepresentable은 단순히 Storyboard, ViewController 코드에 의해 보여주고 내부 로직이 작동만 하게 되고 화면전환을 하지 않게 되어 이 작업만 완료되면 한 화면, 한 화면 필요할때마다 ViewController를 SwiftUIView로 대체해 가면됩니다.

- 내부 로직까지 대체가 완료된 SwiftUI View와 관련되었었던 Legacy들 즉, Wrapping하였던 UIViewControllerRepresentable, ViewController, Storyboard들을 삭제해 나가면 됩니다.
