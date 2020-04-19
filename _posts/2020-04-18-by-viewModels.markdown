---
layout: post
title:  "by viewModels()"
date:   2020-04-04 16:55:00 +0900
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
[Delegated Properties]를 이용하면 우리가 공통적으로 사용하는 특정 properties 
예를 들면 lazy properties, observable properties, 그리고 storing properties를 쉽게 구현할 수 있다. 

레퍼런스 문서에 있는 간단한 예를 먼저 보면, 
p라는 [Delegated Properties]를 `Delegate`클래스를 사용해 선언했다. 

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

# viewModels()

따라서, `val model: MyViewModel by viewModel()`의 의미는 
model의 getValue/setValue 동작을 `viewModels`의 getValue/setValue의 동작에 위임하는 것이다. 


[ViewModel]: https://developer.android.com/topic/libraries/architecture/viewmodel
[Delegated Properties]: https://kotlinlang.org/docs/reference/delegated-properties.html