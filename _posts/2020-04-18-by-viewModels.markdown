---
layout: post
title:  "by viewModels()"
date:   2020-04-18 16:55:00 +0900
categories: Android
---

Android Architecture Component 중에 [ViewModel] 관련 예제를 보던중 특이한 코드가 있었다.

{% highlight kotlin %}
class MyActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        // Create a ViewModel the first time the system calls an activity's onCreate() method.
        // Re-created activities receive the same MyViewModel instance created by the first activity.

        // Use the 'by viewModels()' Kotlin property delegate
        // from the activity-ktx artifact
        val model: MyViewModel by viewModels()
        model.getUsers().observe(this, Observer<List<User>>{ users ->
            // update UI
        })
    }
}
{% endhighlight %}

여기서 ViewModel을 생성하는 부분의 코드가 전혀 이해되지 않았다.
{% highlight kotlin %}
val model: MyViewModel by viewModels()
{% endhighlight %}

# Delegated Properties
우리가 공통적으로 사용하는 lazy properties, observable properties, 그리고 storing properties를 쉽게 구현하기 위해서,
Kotlin에서는 [Delegated Properties]를 지원한다.

레퍼런스 문서에 있는 간단한 예를 살펴 보면, 
`p`라는 [Delegated Properties]를 `Delegate`클래스를 사용해 선언했다. 

{% highlight kotlin %}
class Example {
    var p: String by Delegate()
}
{% endhighlight %}

{% highlight kotlin %}
import kotlin.reflect.KProperty

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
 
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
{% endhighlight %}

`p`를 읽을때는 `Delegate` 클래스의 `getValue()`가 호출되고,
첫번째 파라미터는 `p`를 읽은 객체이고 두번째 파라미터는 `p`에 대한 `KProperty`를 전달 받는다. 

{% highlight kotlin %}
val e = Example()
println(e.p)
{% endhighlight %}

이를 실행하면, 아래와 같이 출력된다.

> Example@33a17727, thank you for delegating ‘p’ to me!

마찬가지로 `setValue()`의 경우에도 마지막 파라미터로 value가 전달되는 것을 제외하면 동일하다
{% highlight kotlin %}
e.p = "NEW"
{% endhighlight %}

> NEW has been assigned to ‘p’ in Example@33a17727.

# Lazy
[lazy()]는 lambda를 받아서 `Lazy<T>`를 리턴하는 함수로, lazy property를 구현할 수 있다.
첫번째 `get()`에서는 전달된 lambda가 실행되고 그 리턴값을 저장한다. 다음 호출부터는 저장된 결과가 리턴된다.

{% highlight kotlin %}
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main() {
    println(lazyValue)
    println(lazyValue)
}
{% endhighlight %}
> computed!  
> Hello  
> Hello

참고로, [Lazy.kt]를 보면 `getValue`가 extension으로 정의되있고, T의 인스턴스인 `value`를 리턴하게 되있다.
{% highlight kotlin %}
@kotlin.internal.InlineOnly
public inline operator fun <T> Lazy<T>.getValue(thisRef: Any?, property: KProperty<*>): T = value
{% endhighlight %}


# viewModels()

따라서, `val model: MyViewModel by viewModel()`의 의미는 
model의 getValue/setValue 동작을 `viewModels`의 getValue/setValue의 동작에 위임하는 것이다. 

아래 코드는 [ActivityViewModelLazy.kt]이다.

{% highlight kotlin %}
@MainThread
inline fun <reified VM : ViewModel> ComponentActivity.viewModels(
    factory: ViewModelProvider.Factory? = null
): Lazy<VM> = ActivityViewModelLazy(this, VM::class, factory)

/**
 * An implementation of [Lazy] used by [ComponentActivity.viewModels] tied to the given [activity],
 * [viewModelClass], [factory]
 */
class ActivityViewModelLazy<VM : ViewModel>(
    private val activity: ComponentActivity,
    private val viewModelClass: KClass<VM>,
    private val factory: ViewModelProvider.Factory?
) : Lazy<VM> {
    private var cached: VM? = null
    override val value: VM
        get() {
            var viewModel = cached
            if (viewModel == null) {
                val application = activity.application
                        ?: throw IllegalArgumentException("ViewModel can be accessed " +
                                "only when Activity is attached")
                val resolvedFactory = factory ?: AndroidViewModelFactory.getInstance(application)
                viewModel = ViewModelProvider(activity, resolvedFactory).get(viewModelClass.java)
                cached = viewModel
            }
            return viewModel
        }
    override fun isInitialized() = cached != null
}
{% endhighlight %}

소스를 보면 `model`을 접근하면 `ActivityViewModelLazy`의 `get`이 호출되고, 
`ViewModelProvider`에서 해당 `MyViewModel`의 ViewModel을 가져오게 된다. 

즉, Java 예제와 비교하면 lazy인 것을 제외하면 비슷한 것이다. 
{% highlight java %}
public class MyActivity extends AppCompatActivity {
    public void onCreate(Bundle savedInstanceState) {
        MyViewModel model = new ViewModelProvider(this).get(MyViewModel.class);
        model.getUsers().observe(this, users -> {
            // update UI
        });
    }
}
{% endhighlight %}

[ViewModel]: https://developer.android.com/topic/libraries/architecture/viewmodel
[Delegated Properties]: https://kotlinlang.org/docs/reference/delegated-properties.html
[lazy()]: https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html
[Lazy.kt]: https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Lazy.kt
[ActivityViewModelLazy.kt]: https://android.googlesource.com/platform/frameworks/support/+/0699f8f5b5aa7d79ba48d57a3710989ae2f50ee3/activity/ktx/src/main/java/androidx/activity/ActivityViewModelLazy.kt