---
title: Android 使用Mockito测试业务逻辑<unittest 之一>
date: 2016-09-12 02:41:46
category: Android
keywords: 单元测试, Mockito, unittest
tags:
	- Android
	- unittest
---

### **前言**
Android中做单元测试一直是永远的痛，最近研究了些项目，发现在这些项目中无论是单元测试还是做ui测试，不管是从代码简洁还是从项目结构上都非常值得学习，这里随便写下自己的总结。
<!--more-->
### **关于单元测试的痛**
很多时候我们做单元测试之所以困难重重，应该大概有几个原因：
1.  待测试类职责不清晰。类里面各个类互相依赖，尤其是如果依赖具体的Android ui我们很难在jvm上做单元测试。
2. 试图测试所有细节。测试本身只关注输入输出，如果在测试用例上不聚焦到类的功能本身，只会事倍而功半。事实上我们只需要关注待测试类所依赖的类是否按功能去正确调用。
3. 违背依赖倒置原则。依赖倒置原则建议高层具体的实现不要依赖高层的实现，而应该依赖于底层抽象

### **Mockito**
Mockito使用起来比较简单，强大之处在于可以测试方法的调用，参数和返回值。对于测试业务的routeline应该是够用的。
下面mockito使用示例来自[这里](http://www.baeldung.com/mockito-verify)

*1 . 验证方法调用：*
```java
List<String> mockedList = mock(MyList.class);
mockedList.size();
verify(mockedList).size();
```
 *2 . 验证调用次数：*
```java
List<String> mockedList = mock(MyList.class);
mockedList.size();
verify(mockedList, times(1)).size();
```
*3 . 验证mock对象的某个方法没有:*
```
List<String> mockedList = mock(MyList.class);
verify(mockedList, times(0)).size();
```
*4 . 验证mock对象没有被调用*

```
List<String> mockedList = mock(MyList.class);
verifyZeroInteractions(mockedList);
```

*5 . 验证之后没有被调用*
```
List<String> mockedList = mock(MyList.class);
mockedList.size();
mockedList.clear();
verify(mockedList).size();
verifyNoMoreInteractions(mockedList);
```
*6 . 验证调用顺序*
```
List<String> mockedList = mock(MyList.class);
mockedList.size();
mockedList.add("a parameter");
mockedList.clear();

InOrder inOrder = Mockito.inOrder(mockedList);
inOrder.verify(mockedList).size();
inOrder.verify(mockedList).add("a parameter");
inOrder.verify(mockedList).clear();
```
*7 . 验证某个方法未被调用*
```
List<String> mockedList = mock(MyList.class);
mockedList.size();
verify(mockedList, never()).clear();
```

*8 .验证方法最少或最多被调用多次：*
```
List<String> mockedList = mock(MyList.class);
mockedList.clear();
mockedList.clear();
mockedList.clear();

verify(mockedList, atLeast(1)).clear();
verify(mockedList, atMost(10)).clear();
```
*9 . 验证方法的参数*
```
List<String> mockedList = mock(MyList.class);
mockedList.add("test");
verify(mockedList).add("test");
```
*10 .任意参数验证*
```
List<String> mockedList = mock(MyList.class);
mockedList.add("test");
verify(mockedList).add(anyString());
verify(mockedList).add(any(String.class));
```
*11 . 验证截获的方法参数*
```
List<String> mockedList = mock(MyList.class);
mockedList.addAll(Lists.<String> newArrayList("someElement"));
ArgumentCaptor<List> argumentCaptor = ArgumentCaptor.forClass(List.class);
verify(mockedList).addAll(argumentCaptor.capture());
List<String> capturedArgument = argumentCaptor.<List<String>> getValue();
assertThat(capturedArgument, hasItem("someElement"));
```
### **如何使类用Mockito方便测试**

针对上面的原因，最关键的一点使我们的测试类保持为pure Java，具体思路就是功能的实现依赖于接口而不要依赖与具体的实现。引用《Android源码设计模式》中的说法：

> 模块间的依赖通过抽象发生，实现类直接不发生直接的依赖关系，其依赖关系通过接口或抽象产生

###**重构示例**
这里以上一篇中动画的类为例来进行重构以便于对其进行测试。

#### **测试什么**
我们对动画的操作封装在AnimatePresenter，用户调用：
```
public void animateImageByFlag(@AnimateState int flag)
```
通过设置不同的flag来验证最终view的属性是不是变成了我们预期的。
开始些测试就要解决个问题：
```
  private ViewPropertyAnimatorCompat mViewCompat;
    public AnimatePresenter(View view){
        mViewCompat = ViewCompat.animate(view);
    }
```
测试类依赖于ViewPropertyAnimatorCompat或者说view，这是具体实现而且是Android的类，最最要命的是ViewPropertyAnimatorCompat是final class. 而Mockito 对于final类、匿名类和Java的基本类型是无法进行mock的.
这里只需要使依赖于一个接口或者抽象即可。

#### **重构**
定义接口，计算出属性值后调用对应方法

```
    interface AnimateCallback{
        void rotationView(float value);
        void translationYView(float value);
        void translationXView(float value);
    }
```
于是实现变成这样：

```
    public void animateImageByFlag(@AnimateState int flag){
        boolean isFilterAfterMode = hasStatus(STATE_MODE_FILAFTER_MASK);
        if(isFilterAfterMode){
            revertRequestFlag(flag);
            animateCallback.rotationView(hasStatus(ROTATION_MASK)?30:0);
            animateCallback.translationXView(hasStatus(TRANSITIONX_MASK)?200:0);
            animateCallback.translationYView(hasStatus(TRANSITIONY_MASK)?200:0);
        }else {
            clearAnimateState();
            animateCallback.rotationView(0);
            animateCallback.translationXView(0);
            animateCallback.translationYView(0);
            if(requestHasState(flag,ROTATION_MASK)){
                animateCallback.rotationView(30);
            }
            if(requestHasState(flag,TRANSITIONY_MASK)){
                animateCallback.translationYView(200);
            }
            if(requestHasState(flag,TRANSITIONX_MASK)){
                animateCallback.translationXView(200);
            }
        }
        mViewCompat.setDuration(500).start();
    }
```
####**测试**
初始化对应的依赖和测试类：


```
    @Mock
    AnimatePresenter.AnimateCallback mCallback;
    private AnimatePresenter mPresenter;

    @Before
    public void setupPresenter(){
        MockitoAnnotations.initMocks(this);
        mPresenter = new AnimatePresenter(any(View.class));
        mPresenter.setAnimateCallback(mCallback);
    }
```
以测试多状态时传入各种组合Flag为例：

```
public void test_multimode_WithFlag(){
        //multimode
        mPresenter.animateWithMode(AnimatePresenter.STATE_MODE_MULTI_MASK);
        mPresenter.animateImageByFlag(AnimatePresenter.ROTATION_MASK);
        ArgumentCaptor<Float> rotaion = ArgumentCaptor.forClass(Float.class);
        verify(mCallback,atLeast(1)).rotationView(rotaion.capture());
        assertEquals((Float) 30.0f,rotaion.getValue());

        mPresenter.animateImageByFlag(AnimatePresenter.TRANSITIONX_MASK);
        ArgumentCaptor<Float> traistionX = ArgumentCaptor.forClass(Float.class);
        verify(mCallback,atLeast(1)).translationXView(traistionX.capture());
        assertEquals((Float) 200f,traistionX.getValue());

        mPresenter.animateImageByFlag(AnimatePresenter.TRANSITIONY_MASK);
        ArgumentCaptor<Float> traistionY = ArgumentCaptor.forClass(Float.class);
        verify(mCallback,atLeast(1)).translationYView(traistionY.capture());
        assertEquals((Float) 200f,traistionY.getValue());
    }
```

运行测试：
![单元测试](http://img.blog.csdn.net/20160527234157884)

### **后话**
571ms，几乎点运行就出来结果，而且测试代码很简单，提前写好测试用来验证程序比在机器上不停的运行效率不可同日而语。而且只要接口没变，即便ui变化，测试代码依然可用。所以单元测试对于开发效率的提升，代码质量保证很有必要的。

以上，谢谢大家。
