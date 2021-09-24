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
