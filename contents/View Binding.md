# View Binding

View Binding은 뷰들과의 상호작용하는 코드를 더 쉽게 작성할 수 있는 기능이다.
모듈에서 View Binding을 사용하도록 설정하면, 해당 모듈에 있는 각 XML 레이아웃 파일에 대한 바인딩 클래스가 생성된다.
바인딩 클래스의 인스턴스에는 해당 레이아웃 안에 ID가 있는 모든 view에 대한 직접 참조가 포함되어 있다.


## 설정 방법
View Binding은 모듈별로 활성화된다. 모듈에서 View Binding을 활성화하려면, 모듈 수준 build.gradle 파일에서 viewBinding 빌드 옵션을 true로 설정하면 된다.
```kotlin
android {
    ...
    buildFeatures {
        viewBinding = true
    }
}
```

바인딩 클래스를 생성하는 것을 원하지 않는 경우, 해당 레이아웃 파일의 루트 뷰에 해당 속성을 추가하면 된다.
```kotlin
<LinearLayout
        ...
        tools:viewBindingIgnore="true" >
    ...
</LinearLayout>
```


## 사용법
모듈에 대해 view binding이 활성화되면, 모듈에 포함된 각 XML 레이아웃 파일에 대해 binding class가 생성된다. 
각 바인딩 클래스에는 root view에 대한 참조와 ID가 있는 모든 view가 포함 된다. ID가 없는 view는 참조에 포함되지 않는다.
바인딩 클래스의 이름은 XML파일 이름을 Pascal 대소문자로 변환하고 끝에 Binding이라는 단어를 추가하여 생성된다. 
ex) result_profile.xml => ResultProfileBinding

모든 바인딩 클래스에는 해당 레이아웃 파일의 root view에 대한 직접 참조를 제공하는 getRoot() 메서드가 포함된다. 

```
<LinearLayout ... >
    <TextView android:id="@+id/name" />
    <ImageView android:cropToPadding="true" />
    <Button android:id="@+id/button"
        android:background="@drawable/rounded_button" />
</LinearLayout>
```
이 예제 코드에서 ResultProfileBinding 클래스의 getRoot() 메서드는 LinearLayout 루트 뷰를 반환한다.


## 액티비티에서 뷰 바인딩 사용
액티비티에서 사용할 바인딩 클래스의 인스턴스를 설정하려면 액티비티의 onCreate() 메서드에서 다음 단계를 수행
1. 생성된 바인딩 클래스에 포함된 정적 inflate() 메서드를 호출, 이렇게 하면 사용할 액티비티에 대한 바인딩 클래스의 인스턴스가 만들어짐.
2. getRoot() 메서드 호출하거나, Kotlin 속성 구문인 root를 사용하여 root view에 대한 참조를 가져옴
3. root view를 setContentView()로 전달하여 화면의 view를 활성화한다.

```kotlin
private lateinit var binding: ResultProfileBinding

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ResultProfileBinding.inflate(layoutInflater)
    val view = binding.root
    setContentView(view)
}
```

이제 바인딩 클래스의 인스턴스를 사용하여 다음 view들 중 하나를 참조할 수 있다.
```kotlin
binding.name.text = viewModel.name
binding.button.setOnClickListener { viewModel.userClicked() }
```


## 프래그먼트에서 view binding 사용법
프래그먼트에서 사용할 바인딩 클래스의 인스턴스를 설정하려면 프래그먼트의 onCreateView() 메서드에서 다음 단계를 수행하여야 한다.
1. 생성된 바인딩 클래스에 포함된 정적 inflate() 메서드를 호출, 이렇게 하면 프래그먼트에서 사용할 바인딩 클래스의 인스턴스가 생성 된다.
2. getRoot() 메서드를 호출하거나, Kotlin 속성 구문인 root를 사용하여 root view에 대한 참조를 가져옴
3. onCreateView() 메서드에서 root view를 반환하여 화면의 view를 활성화한다.

```kotlin
private var _binding: ResultProfileBinding? = null
// This property is only valid between onCreateView and
// onDestroyView.
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
    super.onDestroyView()
    _binding = null
}

```

이제 바인딩 클래스의 인스턴스를 사용하여 다음 view들 중 하나를 참조할 수 있다.
```kotlin
binding.name.text = viewModel.name
binding.button.setOnClickListener { viewModel.userClicked() }
```

프래그먼트는 view 보다 오래살아남는다. 프래그먼트의 onDestroyView() 메서드에서 바인딩 클래스 인스턴스에 대한 참조를 모두 정리해야 한다.


## findViewById와의 차이점
view binding은 findViewById를 사용하는 것 보다 중요한 이점이 있다.
1. Null 안전성: view binding은 view에 대한 직접 참조를 만들기 때문에 잘못된 view id로 인해 null pointer exception이 발생할 위험이 없다. 
2. type 안전성: 각 바인딩 클래스의 필드에는 XML 파일에서 참조하는 뷰와 일치하는 유형이 있다. 이것은 class cass 예외의 위험이 없다는 것을 의미한다.

## data binding과의 비교
view binding과 data binding 모두 view를 직접 참조하는 데 사용할 수 있는 바인딩 클래스를 생성한다. 그러나, view binding은 단순한 케이스를 처리하기 위한 것이며 data binding에 비해 다음과 같은 이점을 제공한다.

1. 빠른 컴파일: view binding은 annotation을 처리할 필요가 없으므로, 컴파일 시간이 더 빠르다.
2. 사용 편의성: 모듈에서 view binding을 활성화하면, 해당 모듈의 모든 레이아웃에 자동으로 적용된다.

반대로, view binding은 data binding과 비교하여 다음과 같은 제한이 있다.

1. view binding은 레이아웃 변수 및 표현식을 지원하지 않으므로, 동적인 UI 콘텐츠 선언이 불가능하다.
2. view binding은 양방향 data binding을 지원하지 않는다. (2 ways data binding은 사용자의 입력값이 곧바로 코드 상의 변수에 바인딩 될 수 있으나, 1 way data binding은 적절한 event를 통해서만 코드 상의 변수에 데이터 값이 담긴다)

이러한 고려사항 때문에 프로젝트에서 view binding과 data binding을 모두 사용하는 것이 좋다. 고급 기능이 필요한 레이아웃에서는 data binding을 사용하고 고급 기능이 필요하지 않은 레이아웃에서는 view binding을 사용할 수 있다.


[출처](https://developer.android.com/topic/libraries/view-binding)
