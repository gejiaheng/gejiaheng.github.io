---
layout:     post
title:      Activity 和 Fragment 的通信
category:   []
tags: [Android]
published: True
date: 2015-08-31
summary: Activity 和 Fragment 通信的方法以及解耦
---

Fragment 让 Android 页面逻辑的开发更加方便，更重要的是，它可以做到代码的可复用。通常来讲，Fragment 是和 Activity 结合起来使用的，在一个 Activity 中，通过不同的 Fragment 可以对页面进行方便的管理。

一般，我们会通过以下两种方式向 Activity 中添加 Fragment：

- 在 XML 布局文件中声明 fragment 节点
- 直接在代码中通过 Activity 的 FragmentManager 动态添加

这里就不详细展开了。对于这种 Fragment 最常见的用法，我们可以把 Fragment 理解为 Activity 内的一块可以展示内容并和用户交互的界面。这里也就有了**内**和**外**的关系，也就是 Activity 包裹着 Fragment，是 Fragment 的宿主。我们在一开始就说要让代码可复用，这是 Fragment API 的核心思想之一，它是作为一个可复用的 UI 模块，可以在相同或者不同的 Activity 中任意使用的。那么，是不是我实现了一个 Fragment 的子类，这部分代码就是像 Google 告诉我们的一样，是可复用的呢？我们先来看一段代码示例，也是我在工作中看到过的。

```java
public class HostActivity extends Activity{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.host_activity);
        initFragment();
    }

    private void initFragment() {
        ConcreteFragment fragment = (ConcreteFragment) getFragmentManager().findFragmentByTag(ConcreteFragment.class.getName());
        if (fragment == null) {
            fragment = ConcreteFragment.newInstance(null);
        }
        FragmentTransaction ft = getFragmentManager().beginTransaction();
        ft.replace(R.id.fragment_container, fragment, ConcreteFragment.class.getName());
    }

    public void doSomething() {}
}
```

```java
public class ConcreteFragment extends Fragment {

    private ConcreteFragment() {}

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.concrete_fragment, null);
    }

    public static ConcreteFragment newInstance(Bundle args) {
        ConcreteFragment fragment = new ConcreteFragment();
        if (args != null) {
            fragment.setArguments(args);
        }
        return fragment;
    }

    private void invokeActivityToDoSomething() {
        Activity activity = getActivity();
        if (activity instanceof HostActivity) {
            ((HostActivity) activity).doSomething();
        }
    }

}
```

我们在 ConcreteFragment 中的 *invokeActivityToDoSomething()* 方法中需要调用宿主 HostActivity 中的 *doSomething()* 方法，上面的代码先获取当前 Fragment 对象 attach 到的宿主 Activity 引用，再判断这个引用是不是 HostActivity 类型，如果是的话，则进行类型强转并调用方法。更糟糕的，我还看到过没有进行 *instanceOf* 类型判断，直接进行强转的。

那么，上面的代码问题在哪里？问题的关键在于，在 Fragment 中，知道了宿主 Activity 的具体类型。而如果要让 Fragment 代码可复用，也就是当我写完了这个类后，我可能在将来将其放到其它任何 Activity 中使用，使用的时候，不得不对之前代码中这种强转进行修改。而要真正做到可复用，Fragment 是不能知道具体宿主 Activity 类型的，它只知道，它是被放入 Activity 中，而不是被放入 XXXActivity 子类中。

不过，我们仍然是需要 Fragment 和 Activity 进行通信的。一个比较好的做法是定义接口。如下所示（重复代码略去）。

```java
public class ConcreteFragment extends Fragment {

    private Iface mIface;

    @Override
public void onAttach(Activity activity) {
    super.onAttach(activity);
    try {
        mIface = (Iface) activity;
    } catch (ClassCastException e) {
        throw new ClassCastException(activity.toString() + " must implement Iface");
    }
}

    private void invokeActivityToDoSomething() {
        Activity activity = getActivity();
        if (activity != null) {
            ((Iface) activity).doSomething();
        }
    }

    public interface Iface {
        void doSomething();
    }

}
```

```java
public class HostActivity extends Activity implements ConcreteFragment.Iface {

    public void doSomething() {
        // Do something
    }
}
```

上面的实现可以概括为以下几点：

1. 在 Fragment 中定义接口，当然也可以定义在单独的 java 文件中，不过如果不是通用的接口，建议还是定义在上下文中；
2. 在 Fragment 的 onAttach() 方法中对宿主 Activity 的类型进行检查，需要强制使用这个 Fragment 的 Activity 实现上面第一步中定义的接口。如果类型错误，则直接抛出异常，这种提前检查抛异常的方式在 SDK 的编写中非常常见，是一种强制性的约束。如果不这么做，则需要在调用接口方法的时候进行类型判断和转型，不推荐这样做；
3. 需要使用这个 Fragment 的 Activity 实现第一歩中定义的接口。

其实过程并不复杂，一开始的时候多写一点代码，后面就可以少写很多不够优雅的代码。更重要的是，不会有运行时异常，代码更加面向对象，更利于维护。

既然文章的题目是 Activity 和 Fragment 的通信，这个通信就是双向的，如果我们要在 Activity 中告诉 Fragment 要做一件事该如何处理？

根据上面提到的内和外的关系，Activity 在外，是宿主，那么 Activity 是知道它使用的 Fragment 具体是什么类型的。因此，可以像下面这样直接找到 Fragment 引用，调用相关的方法即可。

```java
private void invokeFragment() {
    ConcreteFragment fragment = (ConcreteFragment) getFragmentManager().findFragmentByTag(ConcreteFragment.class.getName());
    if (fragment != null) {
        fragment.doAThing();
    }
}
```

还需要提到的一点是，一个 Activity 可能会管理多个 Fragment 实例，那么这些 Fragment 之间的通信又该如何处理？答案是，千万不要直接进行通信，否则代码又和具体类型耦合起来了。Fragment 之间的通信全部通过它们共同的宿主 Activity 进行。有了上面的基础，其实很容易想到，这种情况就是对上面两种情景的综合应用，我也就不再贴出代码了。
