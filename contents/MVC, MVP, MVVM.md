# MVC, MVP, MVVM

### 일반적인 아키텍처의 원칙

- 관심사 분리 : 별개의 책임이 있는 여러 클래스로 앱을 나눠야 한다.
- 모델에서 UI 만들기 : 앱의 Views 객체 및 앱 구성요소와 독립되어 있는 모델의 앱 데이터 처리를 담당하는 모델에서 UI를 만들어야 한다.

![Untitled](https://github.com/jennkimm/Android-Study/blob/main/resources/1-1.png)

⇒ 따라서, UI 컨트롤러의 책임과 ViewModel의 책임이 달라야 한다.

**UI 컨트롤러에는 UI및 운영체제 상호작용을 처리하는 로직만 포함되어야 함. UI에 표시할 데이터 관련 로직은 ViewModel에 배치한다.**

### UI 컨트롤러(activity/fragment)

: 화면에 뷰를 그리고 사용자 이벤트나 사용자가 상호작용하는 다른 모든 UI 관련 동작을 캡처하여 UI를 제어한다.

### ViewModel

- 뷰에 표시되는 앱 데이터의 모델. 앱의 UI관련 데이터 처리를 담당하는 아키텍처 구성요소중 하나.
- Android 프레임워크에서 활동이나 프래그먼트가 소멸되고 다시 생성될 때 폐기되지 않는 앱 관련 데이터를 저장.
- 앱에 ViewModel을 구현하려면 아키텍처 구성요소 라이브러리에서 가져온 ViewModel 클래스를 확장하고 이 클래스 내에 앱 데이터를 저장

### ViewModel의 수명 주기

![Untitled](https://github.com/jennkimm/Android-Study/blob/main/resources/1-2.png)

- 프레임워크는 활동이나 프래그먼트의 범위가 유지되는 동안 ViewModel을 유지
- 소유자가 화면 회전과 같은 구성 변경으로 인해 소멸되는 경우에도 소멸되지 않습니다. (소유자의 새 인스턴스는 기존 ViewModel 인스턴스에 다시 연결 )

### 그럼 MVVM, Model-View-ViewModel은 뭐야?!

Activity 나 Fragment 는 View 와 Controller 기능 두가지를 모두 가질 수 있는데, 둘 중 하나를 한쪽으로 빼게 될 경우 View의 데이터바인딩이나 처리에서 중복 코드 작성, 코드 일관성 깨짐 등이 일어날 수 있기 때문에 해당 패턴이 등장했다.

**Model**

- 데이터 소스를 추상화한다. ( DataSource )
- View Model 은 Data Model 과 함께 작동하여 필요한 데이터를 가져오고 저장한다.

**View**

- 사용자 행동(이벤트)에 대한 ViewModel 을 알린다.
- ViewModel 에 의한 observable 변수와 액션에 유연하게 바인딩된다.

**ViewModel**

- View에 관련된 데이터 스트림을 노출한다.
- 뷰가 모델에 이벤트를 전달할 수 있도록 하는 hook을 준비한다.
- ViewModel은 Model을 래핑하고 뷰에 필요한 observerble 데이터를 준비한다.

**장점**

- 안드로이드 데이터 바인딩을 사용하기 때문에, 테스트와 모듈화가 쉽다.
- UI 요구사항이 다시 변경되면 쉽게 바꿀 수 있다.
- View 와 Model 을 연결하기 위해 사용해야 하는 연결 코드를 줄일 수 있다.

**단점**

- Activity와 Fragment의 코드가 많아질수록 유지보수가 어렵다.
- 중첩된 콜백이 많으면 코드를 변경하거나 새로운 기능을 추가하기 어렵고 이해하기가 어렵다.
- Activity와 Fragment에 많은 로직들이 구현되어 있어 유닛 테스팅은 가능하지만 어렵다.

다른 모델로는 MVC, MVP 가 있다.

### MVC, Model-View-Controller

⇒ View와 Controller 둘다 Model 에 의존한다. Controller 가 데이터를 업데이트하고, View 는 데이터를 가져온다.

**Model** 

- 비즈니스 계층 관리. 네트워크 또는 데이터베이스 API를 처리하는 데이터 계층

**View**

- 모델의 데이터를 시각화. 즉, UI Layer.
- 뷰는 하위 모델에 대한 지식이나 상태에 대한 이해가 없고, 사용자가 버튼을 클릭하거나 값을 입력하는 등의 행동을 할 때 무엇을 해야 하는지 모른다.

**Controller** 

- Logic Layer. 즉, 사용자 행동을 통지 받고 필요에 따라 모델을 업데이트함.
- 뷰가 컨트롤러에게 사용자가 버튼을 눌렀다고 알리면, 컨트롤러는 그에 따라 어떻게 모델과 상호작용할지 결정한다.
- 모델에서 데이터가 변화되는 것에 따라 컨트롤러는 뷰의 상태를 적절하게 업데이트하도록 결정할 수 있다.

**장점**

- 코드의 테스트 가능성이 향상된다.
- 확장이 용이하여 새로운 기능을 쉽게 구현할 수 있다.
- Model 클래스는 Android 클래스에 대한 참조가 없으므로 단위 테스트에 가능하다. 마찬가지로 Controller의 단위테스트도 가능하다.
- 모델이 어디에도 종속되지 않으며, 뷰는 유닛 테스트 레벨에서 그다지 테스트할 것이 거의 없어서 쉽게 모델을 테스트할 수 있다. 확장이 용이하여 새로운 기능을 쉽게 구현할 수 있다.

**단점**

- View가 Controller와 Model 모두에 의존한다는 점을 감안할 때, UI Logic의 변경은 여러 클래스의 업데이트가 필요할 수 있으므로 패턴의 유연성이 떨어진다.
- View와 Model이 서로 의존적이다.

### MVP

⇒ MVC 패턴의 단점을 보완하기 위하여 등장한 패턴. ( MVC 패턴은 뷰가 모델에 강력하게 의존하기에, MVP는 각 요소를 명확하게 분리하여 커플링을 낮추기 위해 등장했다. )

*원문 : Since the View and the Presenter work closely together, they need to have a reference to one another. To make the Presenter unit testable with JUnit, the View is abstracted and an interface for it used. The relationship between the Presenter and its corresponding View is defined in a Contract interface class, making the code more readable and the connection between the two easier to understand.

**Model**

- MVC와 동일하며 변화가 없다.

**View**

- 변화 : 액티비티/프래그먼트가 이제 뷰의 일부로 간주된다.

    > 따라서 이들이 서로에게 연관되는 자연스러운 현상을 극복할 필요가 없다.

- 액티비티가 뷰 인터페이스를 구현해서 프리젠터가 코드를 만들 인터페이스를 갖도록 하는 것이 좋다.

    > 특정 뷰와 결합되지 않고 가상 뷰를 구현해서 간단한 유닛 테스트를 실행할 수 있다.

**Presenter**

- 본질적으로는 MVC의 컨트롤러와 같지만, 뷰에 연결되는 것이 아니라 그냥 인터페이스라는 점이 다르다.

    > MVC가 가진 테스트 가능성 문제와 함께 모듈화/유연성 문제 역시 해결.

**장점**

- 매우 훌륭한 분리를 제공한다.

**단점**

- 작은 어플 or 프로토타입 개발 시, 오버헤드처럼 보일 수 있다. (사용되는 인터페이스가 많기 때문에)
- Presenter가 모든 것을 아는 클래스가 된다. 그러나 이는 코드를 더 많이 분해하고, 단위 테스트가 가능한 클래스를 만들면 된다.
