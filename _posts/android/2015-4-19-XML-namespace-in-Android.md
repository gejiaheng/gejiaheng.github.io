---
layout: post
title: XML namespace in Android
description: 简单介绍一下Android中XML文件的命名空间是什么以及如何使用
category: android
---

使用Android Studio创建Project时，默认的MainActivity的布局文件activity_main.xml如下：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  tools:context=".MainActivity">
  <TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/hello_world" />
</RelativeLayout>
```

在使用Eclipse的时候，我曾经就有疑问，上面的*xmlns:android="http://schemas.android.com/apk/res/android"*是什么意思。这篇博客就是简单的介绍一下Android XML文件的命名空间是什么以及如何使用。

###XML namespace
xmlns即XML namespace，也就是XML文件的命名空间，这是XML通用的一个特性，并非Android的资源文件特有。XML命名空间使用URI引用来标识，是加在XML元素和元素的属性上的约束。注意，URI是Identifier，所以，虽然通常会有 *http://* 的字符出现，但不一定会像URL一样指向特定的资源。XML的元素和元素的属性，属于不同的命名空间，就可以拥有不同的意义，甚至可以进行自定义，另外这样做也可以防止冲突。

我们可以这样定义XML namespace：
***xmlns:aaa="http://schemas.aaa.com"***
这样就定义了一个*"http://schemas.aaa.com"*的命名空间，它的别名是*"aaa"*

下面是一个简单的同一个XML文件里使用多个命名空间的例子：

```xml
<?xml version="1.0"?>
<bk:book 
  xmlns:bk='urn:loc.gov:books'
  xmlns:isbn='urn:ISBN:0-395-36341-6'>
  <bk:title>Cheaper by the Dozen</bk:title>
  <isbn:number>1568491379</isbn:number>
</bk:book>
```

这样XML的解析器就可以根据不同的命名空间来进行解析工作了。其实，这和Java中类的全限定名有些类似，可以有相同的类名，只要它们处于不同的包中，就还是两个不同的类，JVM是通过类的全限定名来加载类的。

###Android资源文件的命名空间

####xmlns:android="http://schemas.android.com/apk/res/android"
这是Android的Framework层解析XML布局文件默认的命名空间，android是它的别名，所以在View的属性前都会有*android:*的前缀，你也可以取其它的名字，但是这个命名空间的字符串标识符是唯一、不可更改的。

####xmlns:tools="http://schemas.android.com/tools"
Android还有另外一个常用的命名空间，URI是*"http://schemas.android.com/tools"*，它可以让工具提取XML文件的信息，并且在打包APK的时候去除掉此命名空间下的元素和属性，这样就不会影响运行时了。
根据Android Tools的官网，下面举几个常用的例子：
- tools:**ignore**
这个属性可以用于任意XML元素，是以逗号分隔的Lint工具检查项目的ID，它会忽略此元素及其子元素中相应的Lint检查。

```xml
<string name="show_all_apps" tools:ignore="MissingTranslation">All</string>
```

Used by: Lint
- tools:**targetApi**
这个属性和Java类中的*@TargetApi*注解类似，用于指定XML元素运行的API等级，可以使用API等级的整数或者名称表示。

```xml
<GridLayout 
  tools:targetApi="ICE_CREAM_SANDWICH" >
```

Used by: Lint
- tools:**locale**
用于资源文件的根节点，指定资源的语言或者地区。

```xml
<resources 
  xmlns:tools="http://schemas.android.com/tools" 
  tools:locale="es">
```

Used by: Lint, Studio(关闭非英语资源文件的拼写检查)
- tools:**context**
通常用于XML布局文件的根节点，告诉工具这个布局文件是被哪个Activity使用（设计阶段，并非运行时）。这样做可以告诉Layout Editor此布局文件可以使用关联Activity的theme，我们就可以看到对应theme下此布局实时渲染的效果。

```xml
<android.support.v7.widget.GridLayout
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  tools:context=".MainActivity" />
```

Used by: Layout Editor in Studio & Eclipse, Lint
- tools:**layout**
我们通常都是动态使用Fragment类的，其实也可以直接在布局文件中使用fragment标签，此时，可以使用这个属性查看Fragment的对应布局的渲染效果。注意，这只是作为设计阶段的辅助，运行时你可以加载其它任意布局文件。

```xml
<fragment 
  android:name="com.example.master.ItemListFragment"
  tools:layout="@android:layout/list_content" />
```

Used by: Layout Editor in Studio & Eclipse
- tools:**listitem | listheader | listfooter**
这个属性可以用于AdapterView的子类，包括ListView, GridView和ExpandableListView等，告诉工具在设计阶段列表渲染使用的item布局，header和footer的布局。

```xml
<ListView
  android:id="@android:id/list"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  tools:listitem="@android:layout/simple_list_item_2" />
```

Used by: Layout Editor in Studio & Eclipse
- tools:**showIn**
为了方便布局的复用，我们会include某些公用的布局文件。在公用的布局文件的根元素中使用这个属性，可以在合适的上下文，也就是外围布局中渲染这个布局，看到全部的效果。

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  android:text="@string/hello_world"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  tools:showIn="@layout/activity_main" />
```

Used by: Layout Editor in Studio
- tools:**menu**
这个属性告诉工具Action Bar使用的菜单资源文件。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  android:orientation="vertical"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  tools:menu="menu1,menu2" />
```

Used by: Layout Editor in Studio
- tools:**actionBarNavMode**
这个属性告诉工具Action Bar的导航模式，包括*"standard"*,*"list"*,*"tabs"*三种。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  android:orientation="vertical"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  tools:actionBarNavMode="tabs" />
```

Used by: Layout Editor in Studio

####Designtime Layout Attributes
既然我们上面说到的命名空间只是在设计时起作用，我们可以更进一步，覆写Framework提供的View的属性，只是在设计阶段查看界面渲染的效果，打包的时候会被去除，所以不会影响运行时。

要使用这个功能，和上面一样，首先需要在布局的根元素中加入此命名空间的声明。然后，可以把所有原来以 *android:* 前缀开始的View的属性改为 *tools:* 前缀。而且，这两个命名空间下的属性可以同时出现，一个是设计阶段起作用，一个是运行时起作用，互不影响。

```xml
<TextView
  tools="Dummy text"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content" />
```

比如上面这段TextView的声明，由于没有预设的文字，所以没法在工具渲染的时候看到它的样子，此时可以加入 *tools="Dummy text"* ，就可以实时查看这个TextView的样子了，而且不会影响运行时。

再例如，有些View的显示与否是运行时动态控制的，这个时候我们可以这样做：

```xml
<Button
  android:id="@+id/button"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:text="First"
  tools:visibility="invisible" />
<Button
  android:id="@+id/button2"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:text="Second"
  tools:visibility="visible" />
```

这样我们就可以在工具渲染布局的时候动态控制View的可见性，十分方便。

###在Android中自定义命名空间
上面说的都是Android的Framework和开发工具定义的XML命名空间，我们来看看自定义XML命名空间的应用。我们在Android的开发中，偶尔会需要自定义View，自定义的View都是继承自Framework的 *android.view.View* 类或者它的子类。这个时候，如果我们想要除了Framework提供的属性以外的属性，就会用到自定义命名空间。

首先，我们继承 *View* 类，并提供接受 *Context* 和 *AttributeSet* 作为参数的构造方法，这样就可以在XML布局中直接使用自定义View了。

```java
class PieChart extends View {
  public PieChart(Context context, AttributeSet attrs) {
    super(context, attrs);
  } 
} 
```

如果我们想要自定义属性，并且可以在XML布局文件中直接使用，请遵守以下4条：
- 在```<declare-styleable>```资源元素下定义自定义属性
- 在XML布局中声明属性值
- 在运行时获取属性值
- 在View对象上应用属性值

首先，让我们来自定义属性，在 *res/values/* 文件夹下添加 *attrs.xml* 文件，文件的内容如下：

```xml
<resources>
  <declare-styleable name="PieChart">
    <attr name="showText" format="boolean" />
    <attr name="labelPosition" format="enum">
      <enum name="left" value="0"/>
      <enum name="right" value="1"/>
    </attr>
  </declare-styleable>
</resources>
```

上面的代码定义了两个自定义属性， *showText* 和 *labelPosition* ，它们都属于*PieChart*的样式。样式的名称虽然不一定非要和自定义View的类名相同，但是有些代码编辑器是通过名称来进行关联的，所以最好是遵循这个习惯。

下面就可以在XML布局文件中使用自定义属性了，但是和Framework提供的内置属性不同的是，自定义属性不是属于*http://schemas.android.com/apk/res/android*命名空间的，它是属于*http://schemas.android.com/apk/res/[your-package-name]*命名空间的。下面就是使用自定义属性的示例代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
  <com.example.customviews.charting.PieChart
    custom:showText="true"
    custom:labelPosition="left" />
</LinearLayout>
```

上面我们定义了*custom*的命名空间别名，当然你也可以使用其它的别名。

当XML布局中的View对象被创建的时候，它的属性会被封装进AttributeSet对象传进此View的构造方法。虽然我们可以直接从AttributeSet对象读取属性值，但是更好的方法是将AttributeSet对象传给 *obtainStyledAttributes()* 方法，这个方法会返回一个包含了属性数组的*TypedArray*对象。下面就是PieChart类如何读取属性的：

```java
public PieChart(Context context, AttributeSet attrs) {
  super(context, attrs);
  TypedArray a = context.getTheme().
          obtainStyledAttributes(attrs, R.styleable.PieChart, 0, 0); 
  try { 
    mShowText=a.getBoolean(R.styleable.PieChart_showText,false);
    mTextPos = a.getInteger(R.styleable.PieChart_labelPosition, 0);
  } finally { 
    a.recycle();
  } 
}
```

属性可以控制View的样子，但是只有View初始化以后才可以读取它的属性。因此，为了提供动态的行为，需要为每一个自定义属性提供*Getter*和*Setter*方法。下面就是PieChart的showText相关的代码片段：

```java
public boolean isShowText() {
  return mShowText;
}

public void setShowText(boolean showText) {
  mShowText = showText;
  invalidate();
  requestLayout();
}
```

注意，为了保证View处于正确的状态，上面的代码调用了 *invalidate()* 和 *requestLayout()* 两个方法。这样，你就可以在自定义的View类中根据自定义的属性控制View的行为了。 

参考资料：
1. [Namespaces in XML 1.0 (Third Edition)](http://www.w3.org/TR/REC-xml-names/)
2. [Android Tools Project Site - Tools Attributes](http://tools.android.com/tech-docs/tools-attributes#TOC-tools:context>)
3. [Android Tools Project Site - Designtime Layout Attributes](http://tools.android.com/tips/layout-designtime-attributes)
4. [Android Developer - Training - Creating a View Class](http://developer.android.com/training/custom-views/create-view.html)
