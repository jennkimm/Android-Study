# ViewBinding, DataBinding

### ViewBinding 이란?

- View와 상호작용하는 코드를 쉽게 작성 가능. ViewBinding은 각 XML 레이아웃 파일의 바인딩 클래스를 생성.
- 바인딩 클래스의 인스턴스에는 상응하는 레이아웃에 ID가 있는 모든 뷰의 직접 참조가 포함됨.
- 대부분의 경우 뷰 바인딩이 `findViewById`를 대체.
- DataBinding에서 View에 Data를 Binding 하는 기능을 제외한 View의 접근만 가능한 기능이 바로 ViewBinding.

### **ViewBinding 사용**

**1) ViewBinding 설정**

```kotlin
android {
   ...
   viewBinding {
      enabled = true
   }
}
```

**2) XML 적용**

```xml
<!--result_profile.xml-->
<LinearLayout ... >
   <TextView android:id="@+id/name" />
   <ImageView android:cropToPadding="true" />
   <Button android:id="@+id/button" />
</LinearLayout>
```

**3) Activity에서 ViewBinding**

```kotlin
private lateinit var binding: ResultProfileBinding

override fun onCreate(savedInstanceState: Bundle) {
    super.onCreate(savedInstanceState)
    binding = ResultProfileBinding.inflate(layoutInflater)
    val view = binding.root
    setContentView(view)
}
```

바인딩 클래스 인스턴스 사용하여 뷰 참조

```kotlin
 binding.name.text = viewModel.name
 binding.button.setOnClickListener { viewModel.userClicked() }
```

**4) Fragment에서 ViewBinding**

```kotlin
private var _binding: ResultProfileBinding? = null
private val binding get() = _binding!!

override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    _binding = ResultProfileBinding.inflate(inflater, container, false)
    val view = binding.root
    return view
}

override fun onDestroyView() {
    _binding = null
}
```

### DataBinding 이란?

- 프로그래매틱 방식이 아니라 선언적 형식으로 레이아웃의 UI 구성요소를 앱의 데이터 소스와 결합할 수 있는 지원 라이브러리
- 액티비티에서 많은 UI 프레임 워크 호출을 제거 할 수 있으므로 유지 관리가 더 쉽고 간단
- 앱의 성능을 향상시키고 메모리 누수와 널 포인터 예외를 방지
- Android Studio 1.3 이상에서는 데이터 바인딩 코드를 위한 다양한 코드 편집 기능을 지원
    - 구문 강조표시
    - 식 언어 구문 오류의 플래그 지정
    - XML 코드 완성
    - (선언 탐색 등의) 탐색과 빠른 문서화를 포함한 참조
- 데이터 바인딩 라이브러리는 변경 사항에 대한 데이터를 쉽게 관찰 할 수있는 클래스와 메서드를 제공

기존 findViewById()를 호출하여 TextView 위젯을 찾아 viewModel 변수의 userName 속성에 결합한 방식

```kotlin
findViewById<TextView>(R.id.sample_text).apply {
    text = viewModel.userName
}
```

데이터 바인딩 라이브러리를 사용하여 레이아웃 파일에 직접 위젯에 텍스트를 할당하는 방법

```xml
<TextView
        android:text="@{viewmodel.userName}" />
```

- 표현식 언어로 레이아웃의 뷰와 변수를 연결하는 표현식을 작성할 수 있다. 또한 뷰에 의해 전달된 이벤트를 처리하는 표현식도 작성할 수 있다. 데이터 바인딩 라이브러리는 레이아웃의 뷰를 데이터 객체와 결합하는 데 필요한 클래스를 자동으로 생성해준다. 또한 가져오기, 변수 및 포함과 같이 레이아웃에서 사용할 수 있는 기능을 제공한다.

1) 데이터 바인딩의 레이아웃 파일

layout 의 루트 태그로 시작하고 그 뒤에 data 요소와 view 루트 요소가 나온다.

```kotlin
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
       <data>
           <variable name="user" type="com.example.User"/>
       </data>
       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.firstName}"/>
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.lastName}"/>
       </LinearLayout>
    </layout>

```

2) 데이터 객체

```kotlin
data class User(val firstName: String, val lastName: String)
```

`android:text` 속성에 사용된 `@{user.firstName}` 표현식은 전자 클래스의 `firstName` 필드에 접근한다.

3) 데이터 바인딩

각 레이아웃 파일에 바인딩 클래스가 생성되는데, 클래스 이름은 레이아웃 파일 이름을 기준으로 파스칼 표기법으로 변환하고 Binding 접미사를 추가한다. ex) activity_main.xml , ActivityMainBinding 클래스

이 클래스는 레이아웃 속성(ex: user 변수)에서 레이아웃 뷰까지 모든 바인딩을 보유하며 바인딩 표현식의 값을 할당하는 방법을 알고 있다. 권장되는 바인딩 생성 방법은 다음 예에서와 같이 레이아웃을 확장하는 동안 바인딩을 생성하는 것이다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val binding: ActivityMainBinding = DataBindingUtil.setContentView(
                this, R.layout.activity_main)

        binding.user = User("Test", "User")
    }
```

→ [공식문서 참고](https://developer.android.com/topic/libraries/data-binding/expressions) 

### 응용

- 이벤트 처리

데이터 바인딩을 사용하면 뷰에서 전달되는 표현식 처리 이벤트를 작성할 수 있다. 이벤트 속성 이름은 주로 리스너 메서드의 이름에 따라 결정된다. 

ex) `View.OnClickListener`에는 `onClick()` 메서드가 있으므로 이 이벤트의 속성은 `android:onClick` 이다. 

다음 메커니즘을 사용하여 이벤트를 처리할 수 있다.

1) **메서드 참조**

Activity에 있는 메서드에 android:onClick을 할당하는 것과 비슷한 방법으로 이벤트를 핸들러 메서드에 직접 바인딩 할 수 있다. onClick 속성과 비교할 때 가장 큰 이점은 표현식이 컴파일 시점에 처리된다는 것이다. (Activity에 있는 메서드는 실행 시점에 처리됨) 따라서 메소드가 존재하지 않거나 서명이 올바르지 않으면 컴파일 시 오류가 발생한다.

```kotlin
class MyHandlers {
    fun onClickFriend(view: View) { ... }
}
```

```kotlin
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
       <data>
           <variable name="handlers" type="com.example.MyHandlers"/>
           <variable name="user" type="com.example.User"/>
       </data>
       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.firstName}"
               android:onClick="@{handlers::onClickFriend}"/>
       </LinearLayout>
    </layout>
```

2) **리스너 바인딩**

이벤트 발생 시 실행되는 바인딩 표현식이다.

메서드 참조와의 주요 차이점은 실제 리스너 구현이 이벤트가 트리거될 때가 아니라 데이터가 바인딩될 때 생성된다는 점이다. 이벤트가 발생할 때 표현식을 계산하려면 리스너 결합을 사용해야 한다.

메서드 참조에서는 메서드의 매개변수가 이벤트 리스너의 매개변수와 일치해야한다. 리스너 바인딩에서는 반환 값은 리스너의 예상 반환 값과 일치해야 한다.

```kotlin
 class Presenter {
     fun onSaveClick(task: Task){}
 }

```

```kotlin
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
        <data>
            <variable name="task" type="com.android.example.Task" />
            <variable name="presenter" type="com.android.example.Presenter" />
        </data>
        <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
            <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:onClick="@{() -> presenter.onSaveClick(task)}" />
        </LinearLayout>
    </layout>
```

다음과 같이 click 이벤트를 메서드에 바인딩 할 수 있다. 리스너틑 식의 루트 요소로만 허용되는 람다 식으로 표현된다. 표현식에서 콜백이 사용될 때 데이터 바인딩이 필요한 리스너를 자동으로 생성하고 이벤트에 등록한다. 뷰가 이벤트를 발생시키면 데이터 바인딩이 주어진 식을 계산한다. 정규 바인딩 식에서처럼, 이 리스너 식이 평가되는 동안 여전히 데이터 바인딩의 null 및 스레드 안전이 보장된다.

위의 예시에서는 `onClick(android.view.View)`로 전달되는 `view` 매개변수를 정의하지 않았다. 리스너 바인딩에서는 리스너 매개변수로 두 가지 중에서 선택할 수 있는데, 메서드에 대한 모든 매개변수를 무시하거나 모든 매개변수의 이름을 지정하는 것이다. 매개변수의 이름을 지정하기로 선택하면 식에 매개변수를 사용할 수 있다.

*표현식에서 매개변수를 사용하려는 경우 다음과 같이 작성할 수 있다.

```kotlin
class Presenter {
        fun onSaveClick(view: View, task: Task){}
    }
```

```kotlin
android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```

- 가져오기, 변수 및 포함

→ [공식문서 참고](https://developer.android.com/topic/libraries/data-binding/expressions) 

- `Observable` 데이터 객체

데이터 바인딩에 임의의 POJO(plain old java object)를 사용할 수 있지만, POJO를 수정하더라도 UI가 업데이트 되지는 않는다. 데이터 바인딩을 사용하면 데이터 변경 시 리스너로 다른 객체에 변경사항을 알리는 기능을 데이터 객체에 제공이 가능하다. 

```kotlin
class User {
        val firstName = ObservableField<String>()
        val lastName = ObservableField<String>()
        val age = ObservableInt()
    }
```

```kotlin
    user.firstName = "Google"
    val age = user.age
```

Observable 인터페이스를 구현하는 클래스를 사용하면 식별 가능한 객체의 속성 변경에 관해 알림을 받으려는 리스너를 등록할 수 있다. 

리스너 알림이 전송되는 시점은 개발자가 정해주어야 한다. 한 예로, `BaseObservable`을 구현하는 데이터 클래스는 속성이 변경될 때 알리는 역할을 한다. 이를 위해 `Bindable`주석을 getter에 할당하고 setter에서notifyPropertyChanged() 메서드를 호출하면 된다.

```kotlin
    class User : BaseObservable() {

        @get:Bindable
        var firstName: String = ""
            set(value) {
                field = value
                notifyPropertyChanged(BR.firstName)
            }

        @get:Bindable
        var lastName: String = ""
            set(value) {
                field = value
                notifyPropertyChanged(BR.lastName)
            }
    }

```

`@Bindable` 주석은 컴파일 중에 BR 클래스 파일에 항목을 생성한다. BR 클래스 파일은 모듈 패키지에 생성된다.

- 생성된 바인딩 클래스

→ 다음과 같이 바인딩 객체를 만들 수 있다.

```kotlin
 override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding: MyLayoutBinding = MyLayoutBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
```

- 바인딩 어댑터

→ `setText()` 같은 속성 값을 설정하는 작업, `setOnClickListener()` 메서드를 호출하는 것과 같이 이벤트 리스너를 설정하는 작업

→ 값을 설정하기 위해 호출되는 메서드를 지정하고 고유한 결합 로직을 제공하며 어댑터를 사용함으로써 반환된 객체의 유형을 지정할 수 있다.

→ 다음과 같은 맞춤 어뎁터도 생성 가능.

```kotlin
@BindingAdapter("app:goneUnless")
    fun goneUnless(view: View, visible: Boolean) {
        view.visibility = if (visible) View.VISIBLE else View.GONE
    }
```

**참고 해보기

[데이터 바인딩 샘플](https://github.com/android/databinding-samples)