# Data Binding Library 数据绑定库

* This document explains how to use the Data Binding Library to write declarative layouts and minimize the glue code necessary to bind your application logic and layouts
* 本文介绍了如何使用数据绑定库去写声明布局文件和减少绑定你的应用程序的逻辑和布局所必需的粘合代码。

* The Data Binding Library offers both flexibility and broad compatibility — it's a support library, so you can use it with all Android platform versions back to Android 2.1 (API level 7+).
* 数据绑定库是一个支持库，它提供了灵活性和广阔的兼容性，所以你可以在版本Android2.1之后的所有android平台中使用它。

* To use data binding, Android Plugin for Gradle 1.5.0-alpha1 or higher is required. [See how to update the Android Plugin for Gradle.](https://developer.android.com/studio/releases/gradle-plugin.html#updating-plugin)
* 使用数据绑定要求Android中的Gradle插件在1.5.0-alpha1 或者更高的版本。

# Build Environment 构建环境

* To get started with Data Binding, download the library from the Support repository in the Android SDK manager.
* 开始使用数据绑定库前先要在Android SDK 管理者中从支持库中下载该库。

* To configure your app to use data binding, add the dataBinding element to your build.gradle file in the app module.
* 在你的应用程序的模块中的build.gradle文件中添加dataBinding元素去配置你的应用程序去使用数据绑定库的功能。

* Use the following code snippet to configure data binding:
* 使用下面的代码片段去配置使用数据绑定
```
android {
    ....
    dataBinding {
        enabled = true
    }
}
```
* If you have an app module that depends on a library which uses data binding, your app module must configure data binding in its build.gradle file as well.
* 如果你有一个应用程序模块依赖了一个使用数据绑定的库，你的应用程序模块同样需要在build.gradle文件中配置数据绑定。

* Also, make sure you are using a compatible version of Android Studio. Android Studio 1.3 and later provides support for data binding as described in Android Studio Support for Data Binding.
* 此外，请确保你使用了一个兼容版本的Android Studio。Android Studio 1.3 之后的都为数据绑定提供了支持。

# Data Binding Layout Files 数据绑定布局文件

### Writing your first set of data binding expressions

### 写你的第一组数据绑定表达式 
* Data-binding layout files are slightly different and start with a root tag of layout followed by a data element and a view root element. This view element is what your root would be in a non-binding layout file. A sample file looks like this:
* 数据绑定布局文件略有不同，在布局开始的根标签后要跟着一个数据元素和一个根视图元素。你的根视图元素jiang将在一个不具约束力的不惧文件中。看起来就想下面的示例文件：
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
* The user variable within data describes a property that may be used within this layout.
* 使用数据变量描述一个可以在这个布局文件内使用的属性。

```
<variable name="user" type="com.example.User"/>
```
* Expressions within the layout are written in the attribute properties using the "@{}" syntax. Here, the TextView's text is set to the firstName property of user:
* 在布局文件内使用"@{}"语法写属性，在这里，TextView的text设置为user的firstName属性。

```
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName}"/>
```
# Data Object 数据对象

* Let's assume for now that you have a plain-old Java object (POJO) for User:
* 让我们假设现在你有一个简单的User Java对象（POJO）

```
public class User {
   public final String firstName;
   public final String lastName;
   public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
   }
}
```

* This type of object has data that never changes. It is common in applications to have data that is read once and never changes thereafter. It is also possible to use a JavaBeans objects:
* 这种类型的对象有一个永远不会改变的数据。这种数据在应用程序中是常见的，这种数据一旦呗读取一次就永远不会被改变。此外也可以使用JavaBeans对象。

```
public class User {
   private final String firstName;
   private final String lastName;
   public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
   }
   public String getFirstName() {
       return this.firstName;
   }
   public String getLastName() {
       return this.lastName;
   }
}
```

* From the perspective of data binding, these two classes are equivalent. The expression @{user.firstName} used for the TextView's android:text attribute will access the firstName field in the former class and the getFirstName() method in the latter class. Alternatively, it will also be resolved to firstName() if that method exists.
* 从数据绑定的观点来看，这两个类是等价的。TextView的android：text属性通过使用表达式@{user.firsName}将获得模版类中的firsName字段和后面的类中的getFirsName()方法。或者，它也可以被解析为firstName()，如果该方法存在。

# Binding Data 绑定数据

* By default, a Binding class will be generated based on the name of the layout file, converting it to Pascal case and suffixing "Binding" to it. The above layout file was main_activity.xml so the generate class was MainActivityBinding. This class holds all the bindings from the layout properties (e.g. the user variable) to the layout's Views and knows how to assign values for the binding expressions.The easiest means for creating the bindings is to do it while inflating:

* 默认的，将会根据布局文件的名字生成一个绑定类，将其转换为Pascal的实例并在名字后面添加Binding作为后缀.上述的布局文件是main_activity.xml,所以生成类叫做MainActivityBinding.该类控制来自布局的视图中的布局属性中的所有绑定并且知道如何去为绑定表达式分配值。在渲染布局时创建绑定是最简单的方式。

```
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
   User user = new User("Test", "User");
   binding.setUser(user);
}
```
* You're done! Run the application and you'll see Test User in the UI. Alternatively, you can get the view via:
* 你已经完成！运行应用程序你将在UI中看到测试的用户信息。或者，你可以通过视图其获得。

``` 
MainActivityBinding binding = MainActivityBinding.inflate(getLayoutInflater());
```
* If you are using data binding items inside a ListView or RecyclerView adapter, you may prefer to use:

* 如果你正在使用数据绑定一个ListView或者RecyclerView适配器中的项目，你可能更喜欢使用：

```
ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
//or
ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);

```

# Event Handling 事件处理

* Data Binding allows you to write expressions handling events that are dispatched from the views (e.g. onClick). Event attribute names are governed by the name of the listener method with a few exceptions. For example, View.OnLongClickListener has a method onLongClick(), so the attribute for this event is android:onLongClick. There are two ways to handle an event.

 * 数据绑定允许你写表达式去处理来自视图（点击）的转发的事件。通过监听方法宁子管理事件属性名单，有些例外。例如，View.onLongClickListener有一个onLongCkick()方法，所以对于该事件的属性是android:onLongClick.以下是两个方式去处理事件。

1. Method References: In your expressions, you can reference methods that conform to the signature of the listener method. When an expression evaluates to a method reference, Data Binding wraps the method reference and owner object in a listener, and sets that listener on the target view. If the expression evaluates to null, Data Binding does not create a listener and sets a null listener instead.
 * 方法引用：在你的表达式中，你可以引用符合监听方法签名的方法。当一个表达式评估一个方法引用时，数据绑定将方法的引用和监听器中拥有的对象包裹起来并且将监听器设置给目标对象。如果表达式评估是Null，数据绑定不会创建一个监听器而是设置一个值为null的监听器合取代

2. Listener Bindings: These are lambda expressions that are evaluated when the event happens. Data Binding always creates a listener, which it sets on the view. When the event is dispatched, the listener evaluates the lambda expression.

* 监听器绑定:当事件发生时，lambda表达式将被评估。数据绑定总是会创建一个监听者，该监听者会被设置到view上。当事件转发的时候，监听者评估该kambda表达式。

## Method References 方法引用
* Events can be bound to handler methods directly, similar to the way android: onClick can be assigned to a method in an Activity. One major advantage compared to the View#onClick attribute is that the expression is processed at compile time, so if the method does not exist or its signature is not correct, you receive a compile time error.
* 事件可以被直接绑定到处理方法上，与在一个Actuvity中使用android：onClick可以被分配到一个方法上的方式类似。与设置View#onClick属性相比一个主要的优势是该表达式在编译时被处理，如果该方法不存在或者它的签名不正确，你将收到一个编译时错误。

* The major difference between Method References and Listener Bindings is that the actual listener implementation is created when the data is bound, not when the event is triggered. If you prefer to evaluate the expression when the event happens, you should use listener binding.
* 方法的引用和监听者的绑定这两种方式主要的不同是当数据绑定时被监听者的是被真实的创建，而不是在事件触发时。如果你更倾向与在事件发生时去评估该表达式，你应该使用监听者绑定。

* To assign an event to its handler, use a normal binding expression, with the value being the method name to call. For example, if your data object has two methods:

* 使用值是作为方法名的正常的绑定表达式去分配一个事件给处理程序。例如，如果你的数据对象有两个方法。
```
public class MyHandlers {
    public void onClickFriend(View view) { ... }
}
```
* The binding expression can assign the click listener for a View:

* 绑定表达式可以分配给视图的点击监听者:
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.Handlers"/>
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
* Note that the signature of the method in the expression must exactly match the signature of the method in the Listener object.
* 注意在表达式中方法的签名必须准确匹配监听者对象中方法的签名

# Listener Bindings 监听者绑定
* Listener Bindings are binding expressions that run when an event happens. They are similar to method references, but they let you run arbitrary data binding expressions. This feature is available with Android Gradle Plugin for Gradle version 2.0 and later.
* 监听者的绑定是在一个事件发生时运行绑定的绑定表达式。他们类似方法的引用，但它们允许你运行任意的数据绑定表达式，该特性在使用Android Gradle插件且Gradle版本为2.0之后是可用的。

* In method references, the parameters of the method must match the parameters of the event listener. In Listener Bindings, only your return value must match the expected return value of the listener (unless it is expecting void). For example, you can have a presenter class that has the following method:
* 在方法的绑定中，方法的参数必须匹配事件监听者的参数。在监听者绑定中，只有你的返回值必须匹配监听者预期返回的值（除非它的愈切返回值是空）。例如，你可以有一个持久化的类，该类有以下方法：
```
public class Presenter {
    public void onSaveClick(Task task){}
}
```

* Then you can bind the click event to your class as follows:
* 然后你可以使用如下代码绑定点时间到你的类。

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
 * Listeners are represented by lambda expressions that are allowed only as root elements of your expressions. When a callback is used in an expression, Data Binding automatically creates the necessary listener and registers for the event. When the view fires the event, Data Binding evaluates the given expression. As in regular binding expressions, you still get the null and thread safety of Data Binding while these listener expressions are being evaluated.
 * 对于你的表达式，只有作为一个根元素，监听者才允许用lambda表达式表示。

* Note that in the example above, we haven't defined the view parameter that is passed into onClick(android.view.View). Listener bindings provide two choices for listener parameters: you can either ignore all parameters to the method or name all of them. If you prefer to name the parameters, you can use them in your expression. For example, the expression above could be written as:
* 注意在上面的例子中，我们还没有定义传递到onClick（View view）中的视图参数。监听者绑定队友监听参入提供两种选择：你可以忽略所有方法的参数或者他们的名字。如果你更倾向于命名参数，你可以在你的表达式中使用他们。例如，以上的表达式可以被写作：
```
  android:onClick="@{(view) -> presenter.onSaveClick(task)}"

```
* Or if you wanted to use the parameter in the expression, it could work as follows:
* 或者如果你想在你表达式中去使用参数，它可以作为如下方式运行：
```
public class Presenter {
    public void onSaveClick(View view, Task task){}
}
```
```
  android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```
* You can use a lambda expression with more than one parameter:
* 超过一个参数时你可以使用一个lambad表达式

``` 
public class Presenter {
    public void onCompletedChanged(Task task, boolean completed){}
}
```
```
  <CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```
* If the event you are listening to returns a value whose type is not void, your expressions must return the same type of value as well. For example, if you want to listen for the long click event, your expression should return boolean.
* 如果你监听的事件返回了一个类型不是void 的值，你的表达式必须返回一样类型的值。例如，如果你想去监听长点击事件，你的表达式应该返回布尔类型。
```
public class Presenter {
    public boolean onLongClick(View view, Task task){}
}
```
```
 android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
 ``` 
 
 * If the expression cannot be evaluated due to null objects, Data Binding returns the default Java value for that type. For example, null for reference types, 0 for int, false for boolean, etc.
 * 如果表达式因为空对象不能被评估，数不绑定返回一个默认类型的Java值。例如，Null对应引用类型，0对应整型，false对应布尔类型。

* If you need to use an expression with a predicate (e.g. ternary), you can use void as a symbol.
* 如果你需要在表达式中使用断言，你可以使用void作为一个标志。
```
  android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```

# Avoid Complex Listeners 避开复合监听

* Listener expressions are very powerful and can make your code very easy to read. On the other hand, listeners containing complex expressions make your layouts hard to read and unmaintainable. These expressions should be as simple as passing available data from your UI to your callback method. You should implement any business logic inside the callback method that you invoked from the listener expression.

* 监听表达式是非常强大的，它可以提高你的代码的可读性。另一方面，监听者包含的复合表达式增加你的布局的阅读难度和不可维护。对于你的回调方法的表达式应该尽可能简单就像通过你的UI的传递可用的数据给你的回调方法一样。你应该在从监听表达式调用的回调方法中事件所有的业务逻辑。

* Some specialized click event handlers exist and they need an attribute other than android: onClick to avoid a conflict. The following attributes have been created to avoid such conflicts:
* 存在一些专门的单击事件处理程序需要比android: onClick多一个其他的属性以避免冲突。以下的属性被创建去避免冲突：
```

Class	         Listener Setter	                                 Attribute
SearchView	setOnSearchClickListener(View.OnClickListener)	android:onSearchClick
ZoomControls	setOnZoomInClickListener(View.OnClickListener)	android:onZoomIn
ZoomControls	setOnZoomOutClickListener(View.OnClickListener)	android:onZoomOut
```
# Layout Details 布局详情

### Imports 

* Zero or more import elements may be used inside the data element. These allow easy reference to classes inside your layout file, just like in Java.

* 零个或者更多被导入的元素可以在数据元素里面使用。在你的布局文件里可以就像在Java里 面一样很容易的引用类。

```
<data>
    <import type="android.view.View"/>
</data>
```
* Now, View may be used within your binding expression:

* 现在，可以在你的绑定表达式里面使用View
```
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```
* When there are class name conflicts, one of the classes may be renamed to an "alias:"

* 当类名冲突时，一个类可以使用别名。
```
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```

* Now, Vista may be used to reference the com.example.real.estate.View and View may be used to reference android.view.View within the layout file. Imported types may be used as type references in variables and expressions:
* 现在，在布局文件理Vista可以被使用去引用 com.example.real.estate.View并且View也可以被使用去引用com.example.real.estate.View。在变凉和表达式中导入类型可以被作为类型引用去使用。
```
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List&lt;User&gt;"/>
</data>
```
* Note: Android Studio does not yet handle imports so the autocomplete for imported variables may not work in your IDE. Your application will still compile fine and you can work around the IDE issue by using fully qualified names in your variable definitions.
* 注意：Android Studio至今不能处理导入，所以在你的IDE中自动导入变量不能工作。您的应用程序仍然可以很好地编译，你可以通过在变量定义使用完全合格的名称解决此问题的IDE。

```
<TextView
   android:text="@{((User)(user.connection)).lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
 ```
 * Imported types may also be used when referencing static fields and methods in expressions:

* 当用引用静态字段和方法时，也可以在表达式中使用导入的类型。
```
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
…
<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
 ```
 * Just as in Java, java.lang.* is imported automatically.

* 就像在Java中一样，java.lang.*是自动导入的
# Variables
* Any number of variable elements may be used inside the data element. Each variable element describes a property that may be set on the layout to be used in binding expressions within the layout file.
* 任何变量元素的成员都可以在数据元素里面使用。每一个变量元素描述一个属性，该属性可以在布局文件的布局里通过使用绑定表达式设置。
```
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>
```
* The variable types are inspected at compile time, so if a variable implements Observable or is an observable collection, that should be reflected in the type. If the variable is a base class or interface that does not implement the Observable* interface, the variables will not be observed!
* 变量类型在编译时检查，所以如果一个变量实现了Observable或者是一个obsevable集合，在类型中该变量应该被反射。如果该变凉是一个基本类型或者没有实现Observable及其子类接口的接口，该变量就不能被观察！

* When there are different layout files for various configurations (e.g. landscape or portrait), the variables will be combined. There must not be conflicting variable definitions between these layout files.
* 当在不同的布局文件中且各个布局文件的配置也不同时，该变凉将被整合。在那些布局文件之间变量的定义绝对不能有冲突。

* The generated binding class will have a setter and getter for each of the described variables. The variables will take the default Java values until the setter is called — null for reference types, 0 for int, false for boolean, etc.
* 生成的绑定类对于每一个描述的变量将会生成一个setter和getter构造器。在setter方法被调用之前该变量将获得一个默认的Java值－引用是null，整型是0，布尔类型是false

* A special variable named context is generated for use in binding expressions as needed. The value for context is the Context from the root View's getContext(). The context variable will be overridden by an explicit variable declaration with that name.
* 为了使用绑定表达式中，生成一个特殊的被成为上下文的变量是必须的。该上下文的值就是来自于根视图中通过getContext（）方法获得的上下文。该上下文变量将被一个使用名字显示声明的变量覆盖。

# Custom Binding Class Names 自定义绑定类名字
* By default, a Binding class is generated based on the name of the layout file, starting it with upper-case, removing underscores ( _ ) and capitalizing the following letter and then suffixing "Binding". This class will be placed in a databinding package under the module package. For example, the layout file contact_item.xml will generate ContactItemBinding. If the module package is com.example.my.app, then it will be placed in com.example.my.app.databinding.
* 默认的，一个绑定类是基于布局文件的名字进行生成，使用大写开始，移除下划线（_）并且后面跟着的字大写然后添加Binding后缀。该类奖杯放置在模块包下的databinding包中。例如，布局文件contact_item.xml将生成ContactItemBinding.如果模块包是com.example.myapp，那么该类将被放置在com.example.myapp.databinding包中。
* Binding classes may be renamed or placed in different packages by adjusting the class attribute of the data element. For example:

* 绑定类的名字可以通过调整数据元素中类的属性可以重命名。例如：
```
<data class="ContactItem">
    ...
</data>
```
* This generates the binding class as ContactItem in the databinding package in the module package. If the class should be generated in a different package within the module package, it may be prefixed with ".":
* 生成的ContactItem绑定类再模块包下的databinding包中，如果希望该类在模块包下不同的包中，可以使用.前缀。
```
<data class=".ContactItem">
    ...
</data>
```
* In this case, ContactItem is generated in the module package directly. Any package may be used if the full package is provided:
* 在这种情况下，ContactItem直接生成在模块包下，如果提供完整的包，任何包都可以使用它。

```
<data class="com.example.ContactItem">
    ...
</data>

```
# Includes
* Variables may be passed into an included layout's binding from the containing layout by using the application namespace and the variable name in an attribute:
* 来自于包含布局的变量可以在一个属性中通过使用应用程序命名空间和和变量名传入一个被included进来的绑定布局中
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
* Here, there must be a user variable in both the name.xml and contact.xml layout files.

* 这里的name.xml和contact.xml布局文件中都必须有一个user变量
* Data binding does not support include as a direct child of a merge element. For example, the following layout is not supported:

* 数据绑定不支持在作为一个merge元素的直接子元素进行include操作。
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <merge>
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </merge>
</layout>
```
# Expression Language 表达式语言
### Common Features 常用特性
* The expression language looks a lot like a Java expression. These are the same:
* 表达式语言看上去很像java表达式。下面是一些相同的部分

* Mathematical + - / * %    数学运算
* String concatenation +    字符串连接
* Logical && ||				 逻辑运算符
* Binary & | ^					位运算符
* Unary + - ! ~					一元运算符
* Shift >> >>> <<				位移运算
* Comparison == > < >= <=		比较运算
* instanceof						判断实例类型
* Grouping ()						分组
* Literals - character, String, numeric, null
* Cast
* Method calls
* Field access
* Array access []
* Ternary operator ?:
### Examples: 例子

``` 
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age < 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```
# Missing Operations 没有的操作
* A few operations are missing from the expression syntax that you can use in Java.
* 一些可以在Java中使用的操作在表达式语法中是没有的

* this
* super
* new
* Explicit generic invocation
# Null Coalescing Operator Null 的聚合操作
The null coalescing operator (??) chooses the left operand if it is not null or the right if it is null.
null聚合操作(??)如果不为null选择左边的操作数，如果为null，选择右边的操作数

``` 
android:text="@{user.displayName ?? user.lastName}"
```
* This is functionally equivalent to:
* 以下的功能是等价的
```
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```
#Property Reference 属性引用
* The first was already discussed in the Writing your first data binding expressions above: short form JavaBean references. When an expression references a property on a class, it uses the same format for fields, getters, and ObservableFields.
* 在上面有关JavaBean的使用部分中已经讨论了如何写你的第一个数据绑定表达式。当表达式引用了一个类的属性时，他要使用一样的fields，getters，ObservableFields

```
android:text="@{user.lastName}"
```
#Avoiding NullPointerException 避免空指针异常

* Generated data binding code automatically checks for nulls and avoid null pointer exceptions. For example, in the expression @{user.name}, if user is null, user.name will be assigned its default value (null). If you were referencing user.age, where age is an int, then it would default to 0.

* 生成的数据绑定代码会自动检查空引用和空指针异常。例如，在表达式```@{user.name}```,如果user是null，user.name将被配置一个默认的null值。如果你引用user.age,age是一个int类型字段，那么他将默认是0.

#Collections 集合

* Common collections: arrays, lists, sparse lists, and maps, may be accessed using the [] operator for convenience.
```
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String&gt;"/>
    <variable name="sparse" type="SparseArray&lt;String&gt;"/>
    <variable name="map" type="Map&lt;String, String&gt;"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
…
android:text="@{sparse[index]}"
…
android:text="@{map[key]}"
```
#String Literals
* When using single quotes around the attribute value, it is easy to use double quotes in the expression:
```
android:text='@{map["firstName"]}'
```
* It is also possible to use double quotes to surround the attribute value. When doing so, String literals should either use the ' or back quote (`).
```
android:text="@{map[`firstName`}"
android:text="@{map['firstName']}"
```
#Resources

* It is possible to access resources as part of expressions using the normal syntax:
* 使用正常的表达式语法可以获得资源
```
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```
* Format strings and plurals may be evaluated by providing parameters:
* 格式化字符串和复数可以通过提供的参数进行评估：
```
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
```
* When a plural takes multiple parameters, all parameters should be passed:
* 当一个复数采用多个参数是，所有的参数都应该被传入。
```

  Have an orange
  Have %d oranges

android:text="@{@plurals/orange(orangeCount, 
orangeCount)}"
```
* Some resources require explicit type evaluation.
* 一些资源要求明确评估类型
* 

```
Type	           Normal Reference	   Expression Reference
String[]	        @array     	         @stringArray
int[]	            @array	             @intArray
TypedArray          @array           	 @typedArray
Animator	        @animator	         @animator
StateListAnimator   @animator	         @stateListAnimator
color int	        @color	             @color
ColorStateList	    @color	             @colorStateList
```
# Data Objects

* Any plain old Java object (POJO) may be used for data binding, but modifying a POJO will not cause the UI to update. The real power of data binding can be used by giving your data objects the ability to notify when data changes. There are three different data change notification mechanisms, Observable objects, observable fields, and observable collections.
* 数据绑定可以使用所有的Java对象（POJO），但是修改一个POJO不会引起一个UI的更新。通过给予你的数据对象一个在数据发生变化时可以通知的能力来使用数据绑定真正强大的能力。这是三个不同的数据改变通知机制，Observable对象，observable字段，observable集合
* When one of these observable data object is bound to the UI and a property of the data object changes, the UI will be updated automatically.

* 当被绑定到UI中的有一个是observable数据对象时，如果数据对象的属性值改变了，UI也会自动更新。

# Observable Objects

* A class implementing the Observable interface will allow the binding to attach a single listener to a bound object to listen for changes of all properties on that object.
* 一个实现了Observable接口的类将允许给绑定对象绑定关联一个监听器去监听该对象中所有属性的变化情况。
* The Observable interface has a mechanism to add and remove listeners, but notifying is up to the developer. To make development easier, a base class, BaseObservable, was created to implement the listener registration mechanism. The data class implementer is still responsible for notifying when the properties change. This is done by assigning a Bindable annotation to the getter and notifying in the setter.
* Observable接口有一个添加和移除监听器的机制，而已通知是在不断的发展改进的。创建一个基本的BaseObservable类区实现监听器注册机制对于开发来说是很容易的。通过分配一个Bindable注解给数据类的getter和setter，当属性改变时，数据类的视线仍然能响应通知。

```
private static class User extends BaseObservable {
   private String firstName;
   private String lastName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }
   @Bindable
   public String getLastName() {
       return this.lastName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
   public void setLastName(String lastName) {
       this.lastName = lastName;
       notifyPropertyChanged(BR.lastName);
   }
}
```
* The Bindable annotation generates an entry in the BR class file during compilation. The BR class file will be generated in the module package. If the base class for data classes cannot be changed, the Observable interface may be implemented using the convenient PropertyChangeRegistry to store and notify listeners efficiently.

* 在编译的时候，Bindable注解在BR类文件中生成一个条目。在模块包下生成一个BR文件。如果数据类是一个不能改变的类，Observable接口会通过使用适当的PropertyChangeRegister去有效的存储和通知监听者来实现。

#ObservableFields
* A little work is involved in creating Observable classes, so developers who want to save time or have few properties may use ObservableField and its siblings ObservableBoolean, ObservableByte, ObservableChar, ObservableShort, ObservableInt, ObservableLong, ObservableFloat, ObservableDouble, and ObservableParcelable. ObservableFields are self-contained observable objects that have a single field. The primitive versions avoid boxing and unboxing during access operations. To use, create a public final field in the data class:
* 在创建Observable类时还是要做一些工作的，所以一些想去解约事件或者希望有一些可使用的属性的开发着可以使用ObservableField和与之类似的ObservableBoolean，ObservableByte，ObservableChar，ObservableDouble，ObservableLong，ObservableFloat，ObservableDouble，ObservableParcelable。ObservableFields 是一个有单一字段的自包含的observable对象。最初的版本是避免在操作时做装箱和拆箱操作。在数据类中以public final 字段形式创建并使用。

```
private static class User {
   public final ObservableField<String> firstName =
       new ObservableField<>();
   public final ObservableField<String> lastName =
       new ObservableField<>();
   public final ObservableInt age = new ObservableInt();
}
```
* That's it! To access the value, use the set and get accessor methods:
* 就是这样，使用set和get方法去获取值

```
user.firstName.set("Google");
int age = user.age.get();

```
# Observable Collections

* Some applications use more dynamic structures to hold data. Observable collections allow keyed access to these data objects. ObservableArrayMap is useful when the key is a reference type, such as String.

* 一些应用更多地使用动态结构去控制数据。Observable collections 允许键值对的形势去获取数据对象。ObservableArrayMap是很常用的一个，它的key是一个引用类型，比如String。

```
ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
user.put("firstName", "Google");
user.put("lastName", "Inc.");
user.put("age", 17);

```
* In the layout, the map may be accessed through the String keys:

* 在布局中，可以通过String类型的key访问map。

```
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap&lt;String, Object&gt;"/>
</data>
…
<TextView
   android:text='@{user["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user["age"])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```
* ObservableArrayList is useful when the key is an integer:

* ObservableArrayList 也是很常用的，它的key是一个integer：
```
ObservableArrayList<Object> user = new ObservableArrayList<>();
user.add("Google");
user.add("Inc.");
user.add(17);
```
* In the layout, the list may be accessed through the indices:

* 在布局中可以通过 下标访问list

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
# Generated Binding
* The generated binding class links the layout variables with the Views within the layout. As discussed earlier, the name and package of the Binding may be customized. The Generated binding classes all extend ViewDataBinding.

* 使用Views内部的layout可以将生成的绑定类于布局变量链接起来。就像前面讨论的一样，绑定的包名可疑被自定义。所有生成的绑定类都继承ViewDataBinding这个类

### Creating
* The binding should be created soon after inflation to ensure that the View hierarchy is not disturbed prior to binding to the Views with expressions within the layout. There are a few ways to bind to a layout. The most common is to use the static methods on the Binding class.The inflate method inflates the View hierarchy and binds to it all it one step. There is a simpler version that only takes a LayoutInflater and one that takes a ViewGroup as well:
* 在View被inflation后应该尽快创建绑定，这样能确保在layout里用表达式绑定View之前View的层级不被干扰。以下是几种绑定layout的方式。最常用的是在绑定类上使用静态方法inflate方法去Inflates View的层级和绑定它，所有操作只需要一步完成。这是i 个简单版本的方法，里面只使用一噶LayoutInflater和一个ViewGroup：

```
MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflater);
MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflater, viewGroup, false);
```
* If the layout was inflated using a different mechanism, it may be bound separately:

* 如果layout是通过不同的机制进行inflated，那么它可以被单独地绑定。
```
MyLayoutBinding binding = MyLayoutBinding.bind(viewRoot);
```
* Sometimes the binding cannot be known in advance. In such cases, the binding can be created using the DataBindingUtil class:

* 有时，我们并不能预先知道绑定的东西。在这种情况下，可以使用DataBindingUtils类去创建绑定。

```
ViewDataBinding binding = DataBindingUtil.inflate(LayoutInflater, layoutId,
    parent, attachToParent);
ViewDataBinding binding = DataBindingUtil.bindTo(viewRoot, layoutId);
```
# Views With IDs

* A public final field will be generated for each View with an ID in the layout. The binding does a single pass on the View hierarchy, extracting the Views with IDs. This mechanism can be faster than calling findViewById for several Views. For example:

* 在layout中使用View的一个ID为每个View生成一个public final类型的字段。在View的层级中，绑定根据IDs提取View并简单的传入View的层级中。这个机制比通过调用各种Views的findViewById方法获取View更加的快速。例如：

```
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
           android:text="@{user.firstName}"
   android:id="@+id/firstName"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"
  android:id="@+id/lastName"/>
   </LinearLayout>
</layout>
```
* Will generate a binding class with:

* 将会使用如下代码生成一个绑定类：

```
public final TextView firstName;
public final TextView lastName;
```
* IDs are not nearly as necessary as without data binding, but there are still some instances where access to Views are still necessary from code.

* 如果没有数据绑定，IDs并不是必须的，但是仍然有些情况下需要从代码里面访问Views时，IDs仍然是需要的。

# Variables 变量

* Each variable will be given accessor methods.
* 每一个变量都将被给予一个存储器方法

```
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>

```
* will generate setters and getters in the binding:
* 在绑定时将会生成一个setters和一个getters存储器

```
public abstract com.example.User getUser();
public abstract void setUser(com.example.User user);
public abstract Drawable getImage();
public abstract void setImage(Drawable image);
public abstract String getNote();
public abstract void setNote(String note);

```
# ViewStubs

* ViewStubs are a little different from normal Views. They start off invisible and when they either are made visible or are explicitly told to inflate, they replace themselves in the layout by inflating another layout.
* ViewStubs 与一般的Views有些许不同。ViewStubs开始时一般时不可见的，当ViewStubs被置为可见的活着明确调用inflate，ViewStubs将会使用其他的layout替换本身。

* Because the ViewStub essentially disappears from the View hierarchy, the View in the binding object must also disappear to allow collection. Because the Views are final, a ViewStubProxy object takes the place of the ViewStub, giving the developer access to the ViewStub when it exists and also access to the inflated View hierarchy when the ViewStub has been inflated.
* 因为从View的层级中ViewStub本身并不会被渲染出来，绑定对象中的View对于collection也必须允许其消失。因为这个Views是final的，当ViewStub已经被inflated且在ViewStubProxy对象取代ViewStub的位置时，如果获得了View的层级的ViewStub是存的，开发者便能获得该ViewStub。

* When inflating another layout, a binding must be established for the new layout. Therefore, the ViewStubProxy must listen to the ViewStub's ViewStub.OnInflateListener and establish the binding at that time. Since only one can exist, the ViewStubProxy allows the developer to set an OnInflateListener on it that it will call after establishing the binding.
* 在inflating其他的layout时，对于一个新的layout，一个绑定必须是明确的。因此，ViewStubProxy必须监听ViewStubs的ViewStub.OnInflateListener 并且明确这绑定的时间。ViewStubProxy只允许开发者去设置一个OnInflateListener，在明确地绑定时，该监听者将被调用。

# Advanced Binding 高级绑定

### Dynamic Variables 动态变量
* At times, the specific binding class won't be known. For example, a RecyclerView.Adapter operating against arbitrary layouts won't know the specific binding class. It still must assign the binding value during the onBindViewHolder(VH, int).
* 有时，特殊的绑定类不能被了解。例如，一个RecyclerView.Adapter 的操作违反了任意布局不能知道指定的绑定类。当onBindViewHolder（VH，int）回调时它仍然能够被分配到绑定的值。

* In this example, all layouts that the RecyclerView binds to have an "item" variable. The BindingHolder has a getBinding method returning the ViewDataBinding base.
* 比如以下例子，RecyclerView的所有布局绑定一个item变量。BindinngHolder又一个getBinding方法，该方法返回基本的ViewDataBinding。

```
public void onBindViewHolder(BindingHolder holder, int position) {
   final T item = mItems.get(position);
   holder.getBinding().setVariable(BR.item, item);
   holder.getBinding().executePendingBindings();
}
```
###Immediate Binding 立即绑定

* When a variable or observable changes, the binding will be scheduled to change before the next frame. There are times, however, when binding must be executed immediately. To force execution, use the executePendingBindings() method.
* 当一个变量活着observable变化时，在下一帧前绑定操作将被调度去改变。然而，有时绑定必须被立即调用，可以使用executePendingBindings()方法。

### Background Thread 后台线程

* You can change your data model in a background thread as long as it is not a collection. Data binding will localize each variable / field while evaluating to avoid any concurrency issues.
* 你也可以在尽可能长耗时的后台线程去改变你的数据模型，该模型不是一个collection.当产生避免任何并发行问题的评估操作时，数据绑定将使每一个变量／字段局域化。

# Attribute Setters 属性设置

* Whenever a bound value changes, the generated binding class must call a setter method on the View with the binding expression. The data binding framework has ways to customize which method to call to set the value.
* 每当一个绑定值发生改变时，使用绑定表达式的View对应生成的绑定类都必须调用setter方法。数据绑定框架有多种方式去自定义被调用去设置改变的值的方法。

# Automatic Setters 自动设置

* For an attribute, data binding tries to find the method setAttribute. The namespace for the attribute does not matter, only the attribute name itself.
* 对于一个属性，数据绑定尝试着去查找setAttribute的方法。于属性的命名空间没有关系，只与属性本身的名字有关。

* For example, an expression associated with TextView's attribute android:text will look for a setText(String). If the expression returns an int, data binding will search for a setText(int) method. Be careful to have the expression return the correct type, casting if necessary. Note that data binding will work even if no attribute exists with the given name. You can then easily "create" attributes for any setter by using data binding. For example, support DrawerLayout doesn't have any attributes, but plenty of setters. You can use the automatic setters to use one of these.

* 例如，一个与TextView的android : text关联的表达式就会查找一个setText(String) 的方法。如果该表达式返回一个int值，数据绑定会继续搜索一个setText(int)的方法。如果有需要，要特别注意表达式返回正确类型。

```
<android.support.v4.widget.DrawerLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:scrimColor="@{@color/scrim}"
    app:drawerListener="@{fragment.drawerListener}"/>
```

###Renamed Setters 重命名Setters

* Some attributes have setters that don't match by name. For these methods, an attribute may be associated with the setter through BindingMethods annotation. This must be associated with a class and contains BindingMethod annotations, one for each renamed method. For example, the android:tint attribute is really associated with setImageTintList(ColorStateList), not setTint.
* 一些属性的setters不能通过名字进行匹配。对于此种情况，我们可以使用BindingMethods注解将一个属性和setter进行关联。对于每个重命名的方法，该方法必须属于一个类并且包含BindingMethod注解。例如，android:tint 属性实际上是鱼setImageTintList（ColorStateList）方法相关联，而不是setTint方法。

```
@BindingMethods({
       @BindingMethod(type = "android.widget.ImageView",
                      attribute = "android:tint",
                      method = "setImageTintList"),
})
```

* It is unlikely that developers will need to rename setters; the android framework attributes have already been implemented.
* Android框架的属性早已经被实现，开发者去重命名框架的方法是不可能的。

### Custom Setters 自定义setters

* Some attributes need custom binding logic. For example, there is no associated setter for the android:paddingLeft attribute. Instead, setPadding(left, top, right, bottom) exists. A static binding adapter method with the BindingAdapter annotation allows the developer to customize how a setter for an attribute is called.
* 一些属性需要自定义绑定逻辑。例如，对于android：paddingLeft属性是与setter没有联系的。而是与setPadding(left,top,right,bottom)方法相关联。使用BindingAdapter注解的一个静态绑定适配器的方法允许开发者去定义该属性的setter方法被调用时做些什么。

* The android attributes have already had BindingAdapters created. For example, here is the one for paddingLeft:
* android的属性早已被BindingAdaoters创建。例如，这是一个paddingLeft的例子：

```
@BindingAdapter("android:paddingLeft")
public static void setPaddingLeft(View view, int padding) {
   view.setPadding(padding,
                   view.getPaddingTop(),
                   view.getPaddingRight(),
                   view.getPaddingBottom());
}
```

* Binding adapters are useful for other types of customization. For example, a custom loader can be called off-thread to load an image.
* 绑定适配器对于自定义的类型是很有用的。例如，一个自定的一加载器可以调用离线的线程去加载一个图像。

* Developer-created binding adapters will override the data binding default adapters when there is a conflict.
* 当有冲突的时候，被开发者创建的绑定适配器将被默认的绑数据绑定适配器覆盖。

* You can also have adapters that receive multiple parameters.
* 你也可以有一个接收多个参数的适配器

```
@BindingAdapter({"bind:imageUrl", "bind:error"})
public static void loadImage(ImageView view, String url, Drawable error) {
   Picasso.with(view.getContext()).load(url).error(error).into(view);
}
```

```
<ImageView app:imageUrl="@{venue.imageUrl}"
app:error="@{@drawable/venueError}"/>
```

 This adapter will be called if both imageUrl and error are used for an ImageView and imageUrl is a string and error is a drawable.
 
 如果被一个ImageView使用的imageUrl错误或者imageUrl是一个错误的drawable字符串，该适配器都会被调用

* Custom namespaces are ignored during matching.
* 自定义命名空间在匹配的过程中被忽略

* You can also write adapters for android namespace.
* 你也可以对android命名空间写一个适配器

Binding adapter methods may optionally take the old values in their handlers. A method taking old and new values should have all old values for the attributes come first, followed by the new values:

在他们的处理程序中，绑定适配器的方法可以随意的获得旧的值。一个方法获得旧的值和新的值的时候，首先获得的是该属性的所有旧的值，然后才是新的值。

```

@BindingAdapter("android:paddingLeft")
public static void setPaddingLeft(View view, int oldPadding, int newPadding) {
   if (oldPadding != newPadding) {
       view.setPadding(newPadding,
                       view.getPaddingTop(),
                       view.getPaddingRight(),
                       view.getPaddingBottom());
   }
}

```

* Event handlers may only be used with interfaces or abstract classes with one abstract method. For example:
* 事件处理程序仅可以和接口或者抽象类的一个抽象方法一起使用。例如：

```
@BindingAdapter("android:onLayoutChange")
public static void setOnLayoutChangeListener(View view, View.OnLayoutChangeListener oldValue,
       View.OnLayoutChangeListener newValue) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        if (oldValue != null) {
            view.removeOnLayoutChangeListener(oldValue);
        }
        if (newValue != null) {
            view.addOnLayoutChangeListener(newValue);
        }
    }
}

```

* When a listener has multiple methods, it must be split into multiple listeners. For example, View.OnAttachStateChangeListener has two methods: onViewAttachedToWindow() and onViewDetachedFromWindow(). We must then create two interfaces to differentiate the attributes and handlers for them.
* 当一个监听者有多个方法时，它必须被分割成多个简体器。例如，View.OnAttachStateChangeListener 有两个方法：onViewAttachedToWindow（）和onViewDetachdFromWindow().我们必须为该属性创建两个不同的接口并处理它们。

```
@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
public interface OnViewDetachedFromWindow {
    void onViewDetachedFromWindow(View v);
}

@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
public interface OnViewAttachedToWindow {
    void onViewAttachedToWindow(View v);
}

```

* Because changing one listener will also affect the other, we must have three different binding adapters, one for each attribute and one for both, should they both be set.
* 对于每一个应该被设置的属性，我们必须有多个不同的绑定适配器，因为改变一个监听器也会影响另外一个，

```
@BindingAdapter("android:onViewAttachedToWindow")
public static void setListener(View view, OnViewAttachedToWindow attached) {
    setListener(view, null, attached);
}

@BindingAdapter("android:onViewDetachedFromWindow")
public static void setListener(View view, OnViewDetachedFromWindow detached) {
    setListener(view, detached, null);
}

@BindingAdapter({"android:onViewDetachedFromWindow", "android:onViewAttachedToWindow"})
public static void setListener(View view, final OnViewDetachedFromWindow detach,
        final OnViewAttachedToWindow attach) {
    if (VERSION.SDK_INT >= VERSION_CODES.HONEYCOMB_MR1) {
        final OnAttachStateChangeListener newListener;
        if (detach == null && attach == null) {
            newListener = null;
        } else {
            newListener = new OnAttachStateChangeListener() {
                @Override
                public void onViewAttachedToWindow(View v) {
                    if (attach != null) {
                        attach.onViewAttachedToWindow(v);
                    }
                }

                @Override
                public void onViewDetachedFromWindow(View v) {
                    if (detach != null) {
                        detach.onViewDetachedFromWindow(v);
                    }
                }
            };
        }
        final OnAttachStateChangeListener oldListener = ListenerUtil.trackListener(view,
                newListener, R.id.onAttachStateChangeListener);
        if (oldListener != null) {
            view.removeOnAttachStateChangeListener(oldListener);
        }
        if (newListener != null) {
            view.addOnAttachStateChangeListener(newListener);
        }
    }
}

```

* The above example is slightly more complicated than normal because View uses add and remove for the listener instead of a set method for View.OnAttachStateChangeListener. The android.databinding.adapters.ListenerUtil class helps keep track of the previous listeners so that they may be removed in the Binding Adaper.
* 以上的例子比一般的例子要稍微复杂。因为View对监听器使用add和remove去替换一个View。OnAttachStateChangeListener的set方法。android.databinding.adapters.ListenerUtil类帮助跟踪之前的简体器一遍监听器可以在Binding Adapter中移除。

* By annotating the interfaces OnViewDetachedFromWindow and OnViewAttachedToWindow with @TargetApi(VERSION_CODES.HONEYCOMB_MR1), the data binding code generator knows that the listener should only be generated when running on Honeycomb MR1 and new devices, the same version supported by addOnAttachStateChangeListener(View.OnAttachStateChangeListener).
* OnViewDetachedFromWindow和OnViewAttachedToWindsw接口通过使用注解@TaegetApi（VERSION_CODES.HONEYCOMEB_MR1），只有当运行在Honeycomb MR1盒心得设备时，数据绑定代码生产者才会生成监听器。

#Converters 转换器

###Object Conversions 对象转换器

* When an Object is returned from a binding expression, a setter will be chosen from the automatic, renamed, and custom setters. The Object will be cast to a parameter type of the chosen setter.
* 当从一个绑定表达式中返回一个对象，该对象的一个自动的、从命名的河自定义的setters将被选择。该对象的类型将被转换成被选择的setter的参数类型。

* This is a convenience for those using ObservableMaps to hold data. for example:
* 使用ObservableMaps去控制数据的这是很方便的，例如：

```
<TextView
   android:text='@{userMap["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
   
```

* The userMap returns an Object and that Object will be automatically cast to parameter type found in the setter setText(CharSequence). When there may be confusion about the parameter type, the developer will need to cast in the expression.
* userMap返回一个对象并且该对象将被自动转换成在setText（CharSequence）方法中找到的参数类型。当参数类型可能被混淆的时候，开发者奖需要去转换表达式。

###Custom Conversions 自定义转换器

* Sometimes conversions should be automatic between specific types. For example, when setting the background:
* 一些时候的转换器应该自动选择特定的类型。例如，当设置背景时：

```
<View
   android:background="@{isError ? @color/red : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

* Here, the background takes a Drawable, but the color is an integer. Whenever a Drawable is expected and an integer is returned, the int should be converted to a ColorDrawable. This conversion is done using a static method with a BindingConversion annotation:
* 这里，北京获得了一个Drawable，但是颜色是一个整型。无论是一个Drawable异常还是返回一个整型，该整型都应该被转换成一个ColorDrawable。使用一个使用了BindingConversion注解的静态方法完成了转换。

```
@BindingConversion
public static ColorDrawable convertColorToDrawable(int color) {
   return new ColorDrawable(color);
}

```

* Note that conversions only happen at the setter level, so it is not allowed to mix types like this:
* 注意，转换旨在setter方法里触发，所以转换不允许像下面这样的混合类型。

```
<View
   android:background="@{isError ? @drawable/error : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

###Android Studio Support for Data Binding Android Studio对于数据绑定的支持。

Android Studio supports many of the code editing features for data binding code. For example, it supports the following features for data binding expressions:

Android Studio支持很多语句数据绑定代码的代码编辑特性。例如，它支持对于数据绑定表达式的一下特性：

* Syntax highlighting 
* 语法高亮
* Flagging of expression language syntax errors
* XML code completion
* References, including navigation (such as navigate to a declaration) and quick documentation

Note: Arrays and a generic type, such as the Observable class, might display errors when there are no errors.
注意：Arrays和一般的类型，例如Observable类没有错误的时候可能会显示错误。

* The Preview pane displays default values for data binding expressions if provided. In the following example excerpt of an element from a layout XML file, the Preview pane displays the PLACEHOLDER default text value in the TextView.
* 如果支持的话，预览版对于数据绑定表达式会展示默认的值。以下例子类子引用自一个来自于布局xml文件的元素，预览版会显示TextView的
PLACEHOLDER的迷人值

```
<TextView android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:text="@{user.firstName, default=PLACEHOLDER}"/>

```

* If you need to display a default value during the design phase of your project, you can also use tools attributes instead of default expression values, as described in Designtime Layout Attributes.
* 在设计你的项目的时候，如果你需要去现实一个默认值，虽然在设计时的布局属性中描述了，你也可以使用tools属性替换默认的表达式值，






























