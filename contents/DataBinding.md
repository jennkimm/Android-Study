# Data Binding

## 개요

데이터 바인딩 라이브러리는 선언적 형식으로 레이아웃의 UI 구성 요소를 app의 데이터 소스와 바인딩할 수 있는 지원 라이브러리다.

레이아웃들은 UI 프레임워크 메소드를 호출하는 코드가 포함된 활동에서 정의되는 경우가 많다. 예로 들어, 아래 코드는 findViewById()를 호출하여 TextView 위젯을 찾아 viewModel 변수의 userName 프로퍼티에 바인딩한다.

```kotlin
findViewById<TextView>(R.id.sample_text).apply {
    text = viewModel.userName
}

```

다음 예제에서는 데이터 바인딩 라이브러리를 사용하여 레이아웃 파일의 위젯에 text를 직접 할당하는 방법을 보여준다. 이렇게 하면 위에 표시된 Kotlin 코드를 사용할 필요가 없다. 할당 표현식(assignment expression)에서 @{} 구문으로 사용하는 것을 유의해야 한다.

```
<TextView
    android:text="@{viewmodel.userName}" />
```

레이아웃 파일의 구성 요소를 바인딩하면 액티비티에서 많은 UI 프레임워크 호출을 제거할 수 있기 때문에 더 간단하고 쉽게 관리할 수 있다. 이렇게 하면 앱의 성능이 향상되고 메모리 부족 및 포인터 예외를 방지할 수 있다. 

<br><br>

## 데이터 바인딩 라이브러리 사용

### 1. 준비 단계(개발 환경 설정 방법)
데이터 바인딩 라이브러리는 유연성과 광범위한 호환성을 모두 제공하기 때문에 Android 4.0(API 레벨 14) 이상을 실행하는 장치와 함께 사용 가능하다.
#### 1) Build 환경
데이터 바인딩을 시작하려면, Android SDK 관리자에서 Support Repository로부터 라이브러리를 다운로드 해야 한다. 그리고 나서, 데이터 바인딩을 사용하도록 앱을 구성하려면 밑의 예제 코드와 같이 앱 모듈의 build.gradle 파일에서 dataBinding 빌드 옵션을 활성화 한다.
```
android {
    ...
    buildFeatures {
        dataBinding true
    }
}
```
#### 2) 데이터 바인딩을 위한 안드로이드 스튜디오 지원
안드로이드 스튜디오는 데이터 바인딩 코드에 대한 많은 편집 기능을 지원한다. 예로 들어, 데이터 바인딩 표현식에 대해 다음 기능들을 지원한다.
- 구문 강조
- 표현식 언어 구문 오류 표시
- XML 코드 완성


<br><br>
### 2. 레이아웃과 바인딩 표현식
표현식 언어를 사용하면 view들에 의해 발생한 이벤트를 처리하는 표현식을 사용할 수 있다. 데이터 바인딩 라이브러리는 레이아웃의 view들을 data object와 바인딩하는 데 필요한 클래스를 자동으로 생성한다.
데이터 바인딩 레이아웃 파일들은 약간 다르며 레이아웃의 루트 태그 다음에 데이터 요소와 view 루트 요소로 시작한다. 이 view 요소는 바인딩되지 않은 레이아웃 파일에 있는 루트이다. 밑의 예제 코드를 보면, data 밑의 LinearLayout이 여기서 말하는 view 루트 요소이다.
```
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
data 내부의 user 변수는 이 레이아웃 내부에서 사용될 수 있는 프로퍼티임을 설명한다.
다음으로, 레이아웃 내부의 식은 @{} 구문을 사용하여 프로퍼티 속성에 지정한다. 위의 코드를 보면, TextView 텍스트는 user 변수의 firstName 프로퍼티로 설정 된다.

#### Data object
위에서 user의 타입인 User 클래스를 살펴 보면, 다음과 같다.
```kotlin
data class User(val firstName: String, val lastName: String)
```
이렇게 정의되어 있기 때문에, firstName과 lastName에 접근할 수 있다.

#### 데이터 바인딩하기
각 레이아웃 파일에 대해 바인딩 클래스가 생성된다. 기본적으로 클래스 이름은 레이아웃 파일 이름에 기반하여 파스칼 대소문자로 변환하고 바인딩 접미사를 추가한다. 예로 들어 레이아웃 파일 이름이 activity_main.xml이라면 생성된 클래스는 ActivityMainBinding이다. 이 바인딩 클래스는 레이아웃 프로퍼티(앞에서의 user 변수)에서 레이아웃 뷰들에 이르는 모든 바인딩을 포함하고 바인딩 식에 값을 할당하는 방법을 알고 있다. 바인딩을 만드는 권장 방법은 밑의 코드와 같이 레이아웃을 inflate하는 동안 만드는 것이다.
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding: ActivityMainBinding = DataBindingUtil.setContentView(
            this, R.layout.activity_main)

    binding.user = User("Test", "User")
}
```
또는 밑의 코드와 같이 LayoutInflater를 사용하여 뷰를 가져올 수 있다.
```kotlin
val binding: ActivityMainBinding = ActivityMainBinding.inflate(getLayoutInflater())
```

Fragment, ListView 또는 RecyclerView 어댑터 내에서 데이터 바인딩 아이템들을 사용하는 경우 밑의 코드에 표시된 것처럼 DataBindingUtil 클래스의 inflate() 메서드를 사용하는 것이 좋다.
```kotlin
val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)

```

#### 이벤트 핸들링
데이터 바인딩을 사용하면 view에서 발생하는 이벤트(예로들어, onClick() 메서드)를 처리하는 표현식을 쓸 수 있다. 이벤트 속성 이름은 몇 가지 예외를 제외하고 리스너 메서드의 이름에 따라 결정 된다. 예로 들어, View.OnClickListener에는 onClick() 메서드가 있으므로 이 이벤트의 속성은 android:onClick이다.

다음 메커니즘을 사용하여 이벤트를 처리할 수 있다.
+ 메서드 참조
+ 리스너 바인딩

<메서드 참조>
이벤트는 android:onClick을 액티비티의 메서드에 할당할 수 있는 방식과 유사하게 핸들러 메서드에 직접 바인딩할 수 있다. 
메서드 참조와 리스너 바인딩의 주요 차이점은 실제 리스너 구현은 이벤트가 트리거될 때가 아니라 데이터가 바인딩 될 때 생성된다는 것이다. 이벤트가 발생할 때 표현식을 계산하려면 리스너 바인딩을 사용해야 한다.
이벤트를 핸들러에 할당하려면 호출할 메서드 이름이 될 값을 사용하여 일반 바인딩 표현식을 사용해야 한다. 예로 들어, 밑의 레이아웃 데이터 객체를 보면
``` kotlin
class MyHandlers {
        fun onClickFriend(view: View) { ... }
    }
```
바인딩 표현식은 다음과 같이 view의 클릭 리스너를 onClickFriend() 메서드에 할당할 수 있다.
```
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

<br>

<리스너 바인딩>
리스너 바인딩은 이벤트가 발생할 때 실행되는 바인딩 표현식이다. 리스너 바인딩은 메서드 참조와 비슷하다. 하지만 리스너 바인딩을 사용하면 임의의 데이터 바인딩 표현식을 실행할 수 있다. 
메서드 참조에서 메서드의 매개변수는 이벤트 리스너의 매개변수와 일치해야 한다. 리스너 바인딩에서는 반환 값만 리스너의 예상 반환 값과 일치하면 된다. 예로 들어 밑의onSaveClick() 메서드가 있는 presenter 클래스가 있고
```kotlin
class Presenter {
        fun onSaveClick(task: Task){}
    }

```
그러면 밑의 코드처럼 클릭 이벤트를 onSaveClick() 메서드에 바인딩할 수 있다.
```
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
표현식에 콜백을 사용하면 데이터 바인딩은 필요한 리스너를 자동으로 생성하여 이벤트에 등록한다. 뷰에서 이벤트가 발생하면 데이터 바인딩은 주어진 표현식을 계산한다. 

위의 예에서는 onClick(View)에 전달되는 view 매개변수가 정의되지 않았다. 매개변수를 정의한 예제는 다음과 같다.
```kotlin
    class Presenter {
        fun onCompletedChanged(task: Task, completed: Boolean){}
    }
```
```
<CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
          android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```

<br>

#### Imports, variables, and includes
import를 사용하면 레이아웃 파일 내의 클래스를 쉽게 참조할 수 있다.
variable을 사용하면 바인딩 표현식에 사용할 수 있는 속성을 선언할 수 있다.
include를 사용하면 앱 전체에서 복잡한 레이아웃을 재사용할 수 있다.

[import]
import를 사용하면 레이아웃 파일 내의 클래스를 쉽게 참조할 수 있다. data 요소 내부에 0개 이상의 import 요소를 사용할 수 있다. 밑의 예제 코드는 View 클래스를 레이아웃 파일로 가져온다.
```
<data>
    <import type="android.view.View"/>
</data>
```
View 클래스를 가져오면 바인딩 표현식에서 참조할 수 있다. 밑의 코드는 View 클래스의 VISIBLE 및 GONE 상수를 참조하는 방법을 보여준다.
```
<TextView
       android:text="@{user.lastName}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```
+ 다른 클래스 가져오기
	가져온 타입은 변수 및 표현식에서 타입 참조로 사용할 수 있다. 밑의 예는 변수 타입으로 사용되는 User 및 List를 보여준다.
    ```
    <data>
        <import type="com.example.User"/>
        <import type="java.util.List"/>
        <variable name="user" type="User"/>
        <variable name="userList" type="List&lt;User>"/>
    </data>
    ```
    
<br>

[Variables]
data 요소 내에서 여러 variable 요소를 사용할 수 있다. 각 variable 요소는 레이아웃 파일 내 바인딩 표현식에 사용될 레이아웃에서 설정할 수 있는 속성을 선언한다. 밑의 코드는 user, image 및 note 변수를 선언한다.
```
<data>
        <import type="android.graphics.drawable.Drawable"/>
        <variable name="user" type="com.example.User"/>
        <variable name="image" type="Drawable"/>
        <variable name="note" type="String"/>
    </data>
```

<br>

[Includes]
속성에 앱 네임스페이스 및 변수 이름을 사용함으로써 포함하는 레이아웃에서 포함된 레이아웃의 바인딩으로 변수를 전달할 수 있다. 밑의 예는 name.xml 및 contact.xml 레이아웃 파일로부터 포함된 user 변수를 보여준다.
```
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:bind="http://schemas.android.com/apk/res-auto">
       <data>
           <variable name="user" type="com.example.User"/>
       </data>
       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
           <include layout="@layout/name"
               bind:user="@{user}"/>
           <include layout="@layout/contact"
               bind:user="@{user}"/>
       </LinearLayout>
    </layout>
    
```

<br><br>

### 3. observable data object로 작업
Observability는 object의 데이터 변경 사항을 다른 이에게 알릴 수 있는 능력을 말한다. 데이터 바인딩 라이브러리를 사용하면 object, 필드 또는 컬렉션을 관찰할 수 있다.
데이터 바인딩을 사용하면 data object의 데이터가 변경될 때 리스너에 알릴 수 있다. observable 클래스에는 object, 필드 그리고 컬렉션의 세가지 유형이 있다. 이러한 observable data object 중 하나가 UI에 바인딩 되어 있고 data object의 프로퍼티가 변경 되면 UI가 자동으로 업데이트 된다.

#### Observable fields
일부 작업은 observable interface를 구현하는 클래스를 만드는 것과 관련이 있다. 클래스에 프로퍼티가 몇 개만 있는 경우에는 그렇게 할 필요는 없다. 이 경우에는 generic observable class와 primitive-specific 클래스를 사용하여 필드를 관찰 가능하도록 설정할 수 있다.
 
 - ObservableBoolean
 - ObservableByte
 - ObservableChar
 - ObservableShort
 - ObservableInt
 - ObservableLong
 - ObservableFloat
 - ObservableDouble
 - ObservableParcelable
 
관찰 가능한 필드는 단일 필드를 가진 자체 포함 observable object이다. 기본 버전은 액세스 작업 중에 boxing과 unboxing을 피한다. 이 메커니즘을 사용하려면, 읽기 전용 프로퍼티를 만들면 된다. 밑의 예제 코드를 보면,

```kotlin
class User {
    val firstName = ObservableField<String>()
    val lastName = ObservableField<String>()
    val age = ObservableInt()
}
```
val 키워드를 사용하여 읽기 전용 프로퍼티로 만든 것을 알수 있다.
필드 값에 액세스하려면 set()과 get() 접근자 메서드를 사용하거나 코틀린 프로퍼티 구문을 사용하면 된다. 밑의 예제 코드를 보면,

```kotlin
user.firstName = "Google"
val age = user.age
```
이렇게 코틀린 속성 구문인 user.firstName, user.age를 사용하여 필드 값을 액세스하는 것을 알 수 있다.

#### Observable collections
일부 앱은 동적 구조를 사용하여 데이터를 보관한다. Observable collection을 사용하면 키를 사용하여 이러한 구조에 액세스할 수 있다. ObservableArrayMap 클래스는 키가 String과 같은 참조 타입일 때 유용하다. 밑의 예제 코드를 보면,

```kotlin
ObservableArrayMap<String, Any>().apply {
    put("firstName", "Google")
    put("lastName", "Inc.")
    put("age", 17)
}
```
키를 참조 타입인 String을 사용한 것을 알 수 있다.
레이아웃에서 밑의 예제 코드와 같이 문자열 키를 사용하여 map을 찾을 수 있다.
```kotlin
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap&lt;String, Object&gt;"/>
</data>
…
<TextView
    android:text="@{user.lastName}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
<TextView
    android:text="@{String.valueOf(1 + (Integer)user.age)}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```
<br> <br>
ObservableArrayList 클래스는 다음과 같이 키가 정수일 때 유용하다. 밑의 예제 코드의 ObservableArrayList는 다음과 같다.

```kotlin
ObservableArrayList<Any>().apply {
    add("Google")
    add("Inc.")
    add(17)
}
```

레이아웃에서 밑의 예제 코드와 같이 인덱스를 통해 리스트에 접근할 수 있다.
```
<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList&lt;Object&gt;"/>
</data>
…
<TextView
    android:text='@{user[Fields.LAST_NAME]}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
<TextView
    android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

<br>

#### Observable objects
Observable interface를 구현하는 클래스를 통해 observable object에 대한 프로퍼티 변경 사항을 알리려는 리스너를 등록할 수 있다.

Observable interface에는 리스너를 추가 및 제거하는 메커니즘이 있지만 알림을 보낼 시기를 결정해야 한다. 개발을 쉽게하기 위해 데이터 바인딩 라이브러리는 리스너 등록 메커니즘을 구현하는 BaseObservable 클래스를 제공한다. BaseObservable을 구현하는 데이터 클래스는 프로퍼티가 변경될 때 이를 알리는 역할을 한다. 이 작업은 getter에 바인딩 가능한 annotation을 할당하고 setter에 notifyPropertyChanged() 메서드를 호출하여 수행된다. 밑의 예제 코드를 보면, 

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
애노테이션과 notifyPropertyChanged() 메서드를 이용한 것을 알 수 있다.


<br>

#### Lifecycle-aware objects
앱의 레이아웃은 데이터 바인딩 소스에 바인딩하여 데이터 변경 사항을 UI에 자동으로 알릴 수 있다. 이렇게 하면 바인딩은 라이프사이클을 인식하며 UI가 화면에 표시될 때만 트리거 된다.

데이터 바인딩은 현재 StateFlow 및 LiveData를 지원한다. 

[StateFlow 사용하기]
앱이 코루틴과 함께 코틀린을 사용하는 경우 StateFlow object를 데이터 바인딩 소스로 사용할 수 있다. 바인딩 클래스와 함께 StateFlow object를 사용하려면 StateFlow object의 범위를 정의할 lifecycle owner를 지정해야 한다. 밑의 예제 코드에서는 바인딩 클래스가 인스턴스화 된 후 lifecycle owner를 지정한다.

```kotlin
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // Inflate view and obtain an instance of the binding class.
        val binding: UserBinding = DataBindingUtil.setContentView(this, R.layout.user)

        // Specify the current activity as the lifecycle owner.
        binding.lifecycleOwner = this
    }
}
```

데이터 바인딩은 Viewmodel object와 원활하게 작동한다. 밑의 예제와 같이 StateFlow와 ViewModel을 함께 사용할 수 있다.

```kotlin
class ScheduleViewModel : ViewModel() {

    private val _username = MutableStateFlow<String>("")
    val username: StateFlow<String> = _username

    init {
        viewModelScope.launch {
            _username.value = Repository.loadUserName()
        }
    }
}
```

레이아웃에서 밑의 예제와 같이 바인딩 식을 사용하여 해당 뷰에 ViewModel object의 프로퍼티와 메서드를 할당한다.

```
<TextView
    android:id="@+id/name"
    android:text="@{viewmodel.username}" />
```
-> UI는 사용자의 이름 값이 변경될 때마다 자동으로 업데이트 된다.

[LiveData를 사용하여 UI에 데이터 변경 알리기]
LiveData object를 데이터 바인딩 소스로 사용하여 데이터 변경 사항에 대해 UI에 자동으로 알릴 수 있다. 
observable field와 같이 관찰 가능한 항목을 구현하는 object와 달리 LiveData object는 데이터 변경을 관찰하는 관찰자의 라이프 사이클을 알고 있다. 
바인딩 클래스와 함께 LiveData object를 사용하려면 LiveData object의 범위를 정의할 lifecycle owner를 지정해야 한다. 밑의 예에서는 바인딩 클래스가 인스턴스화된 후 lifecycle owner를 지정한다.
```kotlin
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // Inflate view and obtain an instance of the binding class.
        val binding: UserBinding = DataBindingUtil.setContentView(this, R.layout.user)

        // Specify the current activity as the lifecycle owner.
        binding.setLifecycleOwner(this)
    }
}
```

ViewModel 구성 요소를 사용하여 UI 관련 데이터를 관리하면 데이터를 레이아웃에 바인딩할 수 있다. ViewModel 구성 요소에서 LiveData object를 사용하여 데이터를 변환하거나 여러 데이터 소스를 병합할 수 있다. 밑의 예제에서는 ViewModel에서 데이터를 변환하는 방법을 보여준다.

```kotlin
class ScheduleViewModel : ViewModel() {
    val userName: LiveData

    init {
        val result = Repository.userName
        userName = Transformations.map(result) { result -> result.value }
    }
}
```
LiveData인 userName을 ViewModel인 ScheduleViewModel에서 변환하고 있는 것을 알 수 있다.

<br><br>

### 4. 생성된 바인딩 클래스
#### 즉시 Binding
변수 또는 observable object가 변경되면 바인딩이 다음 프레임 전에 변경되도록 예약된다. 그러나 바인딩을 즉시 실행해야 하는 경우가 있다. 강제로 실행하려면 executePendingBindings() 메서드를 사용하면 된다.

<br><br>

### 5. Binding adapters
바인딩 어댑터는 값을 설정하는 적절한 프레임워크 호출을 담당한다. setText() 메서드를 호출하는 것과 같은 프로퍼티 값을 설정하는 것이 한 예이다. 또 다른 예는 setOnClickListener() 메서드를 호출하는 것과 같은 이벤트 리스너를 설정하는 것이다.

데이터 바인딩 라이브러리를 사용하면 어댑터를 사용하여 반환되는 object의 타입을 지정하기 위해 호출된 메서드를 지정할 수 있다.

#### 속성 값 설정
바인딩 된 값이 변경될 때 마다 생성된 바인딩 클래스는 바인딩 표현식을 사용하여 뷰에서 setter 메서드를 호출해야 한다. 데이터 바인딩 라이브러리에서 메서드를 자동으로 결정하거나 메서드를 명시적으로 선언하거나 맞춤 로직을 제공해 메서드를 선택하도록 허용할 수 있다.

[자동 메서드 선택]
이름이 example인 속성의 경우 라이브러리는 호환 가능한 타입을 인수로 허용하는 setExample(arg) 메서드를 자동으로 찾으려고 한다. 속성의 네임스페이스는 고려되지 않으며 메서드 검색 시 속성 이름 및 타입만 사용된다.

예를 들어 android:text="@{user.name}" 표현식이 있는 경우 라이브러리는 user.getName()에서 반환한 타입을 허용하는 setText(arg) 메서드를 찾는다. user.getName()의 반환 유형이 String이면 라이브러리는 String 인수를 허용하는 setText() 메서드를 찾는다. 표현식이 int를 대신 반환하면 라이브러리는 int 인수를 허용하는 setText() 메서드를 검색한다. 표현식은 올바른 유형을 반환해야 한다. 필요하다면 반환 값을 변환할 수 있다.

지정된 이름의 속성이 없더라도 데이터 바인딩은 작동한다. 그때는 데이터 바인딩을 사용하여 setter에 필요한 속성을 생성할 수 있다. 예를 들어 밑의 코드를 보면 지원 클래스 DrawerLayout에는 어떤 속성도 없지만 많은 setter가 있다. 다음 레이아웃은 자동으로 setScrimColor(int) 메서드와 setDrawerListener(DrawerListener) 메서드를 각각 app:scrimColor 속성과 app:drawerListener 속성의 setter로 사용한다.
```
<android.support.v4.widget.DrawerLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:scrimColor="@{@color/scrim}"
        app:drawerListener="@{fragment.drawerListener}">
```

<br>
[사용자 지정 메서드 이름 지정] <br>
일부 속성에는 이름이 일치하지 않는 setter가 있다. 이러한 상황에서 속성은 BindingMethods annotation을 사용하여 setter와 연결될 수도 있다. annotation은 클래스와 함께 사용되며 이름이 바뀐 각 메서드에 하나씩 여러 BindingMethod annotation을 포함할 수 있다. BindingMethod는 앱의 어떤 클래스에도 추가할 수 있는 annotation이다. 밑의 예에서는 android:tint 속성은 setTint() 메서드가 아닌 setImageTintList(ColorStateList) 메서드와 연결된다.

```kotlin
@BindingMethods(value = [
        BindingMethod(
            type = android.widget.ImageView::class,
            attribute = "android:tint",
            method = "setImageTintList")])
```
<br>

[사용자 지정 로직 제공]
일부 속성에는 사용자 지정 바인딩 로직이 필요하다. 예를 들어 밑의 코드를 보면 android:paddingLeft 속성에는 연결된 setter가 없다. 대신 setPadding(left, top, right, bottom) 메서드가 제공된다. BindingAdapter annotation이 있는 정적 바인딩 어댑터 메서드를 사용하면 속성의 setter가 호출되는 방식을 사용자 지정 설정할 수 있다.

```kotlin
@BindingAdapter("android:paddingLeft")
    fun setPaddingLeft(view: View, padding: Int) {
        view.setPadding(padding,
                    view.getPaddingTop(),
                    view.getPaddingRight(),
                    view.getPaddingBottom())
    }
```

매개변수 타입은 중요하다. 첫 번째 매개변수는 속성과 연결된 뷰의 타입을 결정한다. 두 번째 매개변수는 지정된 속성의 바인딩 표현식에서 허용되는 타입을 결정한다.
개발자가 정의하는 바인딩 어댑터는 충돌이 발생하면 Android 프레임워크에서 제공하는 기본 어댑터보다 우선 적용된다.

또한 밑의 예에서와 같이 여러 속성을 받는 어댑터도 있을 수 있다.
```kotlin
@BindingAdapter("imageUrl", "error")
    fun loadImage(view: ImageView, url: String, error: Drawable) {
        Picasso.get().load(url).error(error).into(view)
    }
```

밑의 예제 코드와 같이 레이아웃에서 어댑터를 사용할 수 있다.
```
<ImageView app:imageUrl="@{venue.imageUrl}" app:error="@{@drawable/venueError}" />

```
imageUrl과 error가 모두 ImageView 객체에 사용되며 imageUrl은 문자열이고 error는 Drawable이면 어댑터가 호출된다. 따라서, 위의 예제 코드는 이 조건에 부합하므로 어댑터를 호출할 수 있는 것이다. 

속성이 하나라도 설정될 때 어댑터가 호출되도록 하려면 밑의 예에서와 같이 어댑터의 선택적 requireAll 플래그를 false로 설정할 수 있다.
```kotlin
 @BindingAdapter(value = ["imageUrl", "placeholder"], requireAll = false)
    fun setImageUrl(imageView: ImageView, url: String?, placeHolder: Drawable?) {
        if (url == null) {
            imageView.setImageDrawable(placeholder);
        } else {
            MyImageLoader.loadInto(imageView, url, placeholder);
        }
    }
```
이렇게 하면 속성이 하나라도 설정 되면 해당 어댑터가 호출이 된다.

<br><br>

### 6. 양방향 데이터 바인딩
밑의 코드와 같이 단방향 데이터 바인딩을 사용하면 프로퍼티에 값을 설정하고 이 프로퍼티의 변경에 반응하는 리스너를 설정할 수 있다.
```
<CheckBox
        android:id="@+id/rememberMeCheckBox"
        android:checked="@{viewmodel.rememberMe}"
        android:onCheckedChanged="@{viewmodel.rememberMeChanged}"
    />
```
밑의 코드와 같이 양방향 데이터 바인딩은 이 프로세스에 대해서 바로가기를 제공한다.
```
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@={viewmodel.rememberMe}"
/>
```
'=' 기호가 포함된 @={} 표기법은 프로퍼티와 관련된 데이터 변경사항을 받는 동시에 사용자 업데이트를 수신 대기한다.

<br>

#### 사용자 지정 속성을 사용한 양방향 데이터 바인딩
양방향 데이터 바인딩에 사용자 지정 속성을 사용하려면 @InverseBindingAdapter annotation과 @InverseBindingMethod annotation을 사용해야 한다.

예로 들어, MyView라는 사용자 지정 뷰의 time 속성에 양방향 데이터 바인딩을 사용하려면 다음 단계를 완료해야 한다.

1. 초기 값을 설정하고 값이 변경될 때 업데이트하는 메서드에 @BindingAdapter를 사용하여 annotation을 추가한다.
```kotlin
@BindingAdapter("time")
    @JvmStatic fun setTime(view: MyView, newValue: Time) {
        // Important to break potential infinite loops.
        if (view.time != newValue) {
            view.time = newValue
        }
    }
```

2. 뷰에서 값을 읽는 메서드에 @InverseBindingAdapter를 사용하여 annotation을 추가한다.
```kotlin
@InverseBindingAdapter("time")
    @JvmStatic fun getTime(view: MyView) : Time {
        return view.getTime()
    }
```

이 단계만 거치면 속성이 언제 또는 어떻게 변경되는지는 인식하지 못한다. 속성의 변경 시기 또는 방식을 알기 위해서는 뷰에 리스너를 설정해야 한다. 리스너는 사용자 지정 뷰와 연결된 사용자 지정 리스너이거나 포커스 상실 또는 텍스트 변경과 같은 일반 이벤트일 수 있다. 
  
  3. 밑의 예제와 같이 속성 변경과 관련된 리스너를 설정하는 메서드에 @BindingAdapter annotation을 추가한다.
```kotlin
@BindingAdapter("app:timeAttrChanged")
    @JvmStatic fun setListeners(
            view: MyView,
            attrChange: InverseBindingListener
    ) {
        // Set a listener for click, focus, touch, etc.
    }
```
리스너에는 InverseBindingListener가 매개변수로 포함된다. InverseBindingListener를 사용하면 데이터 바인딩 시스템에 속성이 변경되었음을 알릴 수 있다. 그러면 시스템은 @InverseBindingAdapter를 사용하여 annotation이 추가된 메서드 호출을 시작할 수 있다.

<br>

#### 양방향 데이터 바인딩을 사용하는 무한 루프
양방향 데이터 바인딩을 사용할 때 무한 루프가 발생하지 않도록 주의해야 한다. 사용자가 속성을 변경하면 @InverseBindingAdapter를 사용하여 annotation이 추가된 메서드가 호출되고 값이 backing 속성에 할당된다. 그러면 결과적으로 @BindingAdapter를 사용하여 annotation이 추가된 메서드가 호출되고, 이것이 @InverseBindingAdapter를 사용하여 annotation이 추가된 메서드를 또 호출하는 식으로 이어진다. 
따라서 @BindingAdapter를 사용하여 annotation이 추가된 메서드의 새 값과 이전 값을 비교함으로써 발생 가능한 무한 루프를 끊는 것이 중요하다.

<br>

#### 양방향 속성
밑의 표의 속성을 사용할 때 기본적으로 양방향 데이터 바인딩을 지원한다. 

클래스 | 속성 | 바인딩 어댑터
-- | -- | --
AdapterView | android:selectedItemPosition, android:selection | AdapterViewBindingAdapter
CalendarView | android:date | 	CalendarViewBindingAdapter
CompoundButton | android:checked | CompoundButtonBindingAdapter
DatePicker | android:year, android:month, android:day | DatePickerBindingAdapter
NumberPicker | android:value | 	NumberPickerBindingAdapter
RadioButton | android:checkedButton | RadioGroupBindingAdapter
RatingBar | 	android:rating | RatingBarBindingAdapter
SeekBar | 	android:progress | 	SeekBarBindingAdapter
TabHost | 	android:currentTab | 	TabHostBindingAdapter
TextView | 	android:text | 	TextViewBindingAdapter
TimePicker | android:hour, android:minute | 	TimePickerBindingAdapter


<br><br>

[출처 - 안드로이드 공식 문서](https://developer.android.com/topic/libraries/data-binding)
