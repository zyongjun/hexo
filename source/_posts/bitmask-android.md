---
title: Android多状态组合之位运算（BitMask）
date: 2016-09-15 04:47:39
categories: Android
keywords: BitMask, Android BitMask
tags:
- Android
- BitMask
---

### 熟悉的BitMask

Android中使用位运算来保存状态的地方很多，你一定不会陌生，layout中的类似这种：

```
android:gravity="bottom|right"
```
或者这种：

	intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK|
	Intent.FLAG_ACTIVITY_CLEAR_TASK);

这种一般用于表示多种状态组合后的综合状态。
<!--more-->
View中是否focus,enable,visiblity,systemVisiblity等等各种状态的保存读取都是采用的BitMask的方式， 那么BitMask究竟有什么方便的地方呢，我们通过一个实例来学习下BitMask的使用和一些注解的小技巧。

### 实例
如下图：
![这里写图片描述](http://img.blog.csdn.net/20160525142253613)

假定我们有这样的需求:
1.三种显示动画的模式：singlemode(一种动画结果)，multimode(多种动画混合的结果)，fillaftermode（在上一次动画基础上动画）
2.动画包括三种：rotation水平30度，transitionY,transitionX水平移动200px.

三种模式和三种动画各种可能的组合，效果如图。
分析这样的需求：
     首先显示模式可以用一个变量控制1，2，3种状态，另外应该还要三个变量来记录三个属性的状态，或者从view中拿到三个属性进行对比。另外还需要在不同的模式下组合选中的情况或上一次的情况来执行最终要执行的动画。
当然直接这样些肯定是要收到boss邮件的, 可以对动画模式用多态的方式，对应的模式通过方法返回一个animatior,然后点击切换模式的时候切换状态。

总的来说这样实现也不是很复杂，不过相对类太多代码太多，而这里不过是需要组合几种状态而已。
这里介绍另外一种方式BitMask.  对此比较熟悉的同学可以绕道，模仿总结下View里相应的状态实现即可。
#### **设置状态常量**
   右边三位分别表示三种动画

```
     public static final int ROTATION_MASK = 0x1;//0001
     public static final int TRANSITIONY_MASK =0x2;//0010
     public static final int TRANSITIONX_MASK =0x4;//0100
```

   再往左两位表示模式，其中0x8即00001000，这个1为零时表示multimode, 为1时表示singlemode:

```
        public static final int STATE_MODE_SINGLE_MASK =0x8;//1000
        public static final int STATE_MODE_MULTI_MASK =0x0;//0000
        public static final int STATE_MODE_FILAFTER_MASK=0x10;//10000
```

   定义清除对应状态的常量，这里需要清除动画状态（非fillaftermode）和清除所有状态：

```
   private static final int ANIMATE_CLEAR_MASK =0x7;
	   public static final int STATE_CLEAR_MASK =0x1F;
```

注：1.如果对十六进制转二进制不熟悉的同学自行百度，总的来说使用四合一，一分四的方式，而八进制转二进制采用三合一，一分三。
      2.还有一种方式也比较方便，对0x1左移的办法，如要右边第二位的标识常量（1<< 0x1）,第三位的标识常量(2 << 0x1),以此类推。而如果要通过 ‘取反与运算’清除某些位的状态，只需要这些位都是1其他位是0即可，由于这里是连贯的，要清除animate状态，只需要保证后三位是1.

#### **拆包的运算**
那么对于外面调用时候传过来的标识，有必要知道：
1.如果是一个(组合)状态的设置，如何知道包含哪些状态需要设置：

```
	private boolean requestHasState(@AnimateState int request, @AnimateState int targetState){
            return (request&targetState) != 0;
        }
```

即：

```
		xxxxx
		00010
		-------
		000x0
```

2.下次设置状态之前，我们需要知道我们保存的变量mViewFlag中是否有某个状态：

```
	private boolean hasStatus(@AnimateState int flag){
            return (mViewFlag&flag)!=0;
        }
```

一样的算法，都是&运算解包

#### **对外限制Flag接口**

剩下的还有一个问题，这里只有这些标识位，如果调用的时候随意传入Flag，有可能把其他位置的标识错误的清空或者设置。
View里有这样的代码：

```
@IntDef({VISIBLE, INVISIBLE, GONE})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Visibility {}

    public void setVisibility(@Visibility int visibility) {
        setFlags(visibility, VISIBILITY_MASK);
    }  
```

 RetentionPolicy.Source标识这个注解只在源文件里面存在，编译后.class中并不存在。这样只会约束调用的开发时候的传入值，我们可以放心使用。但是这样还不能传入组合的状态。

```
	@IntDef(flag=true,value=ROTATION_MASK,TRANSITIONY_MASK
		    ,TRANSITIONX_MASK,STATE_MODE_SINGLE_MASK
	        ,STATE_MODE_FILAFTER_MASK,STATE_MODE_MULTI_MASK})
        @Retention(RetentionPolicy.SOURCE)
        public @interface AnimateState{}
```

源码描述：

> /** Defines whether the constants can be used as a flag, or just as an
> enum(the default) */
> 		    boolean flag() default false;


如果作为枚举flag用默认的，表示只能传入对应的变量。如果为true标识当标识使用，可以传入各种组合的Flag.  这里需要传入组合的状态，为true即可。

具体逻辑都在这里：

```

  private void revertRequestFlag(@AnimateState int flag){

        if(hasStatus(flag)){
              mViewFlag&=~flag;
          }else{
              mViewFlag|=flag;
          }
      }

    private void animateImageByFlag(@AnimateState int flag){
        boolean isFilterAfterMode = hasStatus(STATE_MODE_FILAFTER_MASK);
        if(isFilterAfterMode){
            revertRequestFlag(flag);
            mViewCompat.rotation(hasStatus(ROTATION_MASK)?30:0)
                    .translationX(hasStatus(TRANSITIONX_MASK)?200:0)
                    .translationY(hasStatus(TRANSITIONY_MASK)?200:0);
        }else {
            clearAnimateState();
            mViewCompat.rotation(0).translationX(0).translationY(0);
            if(requestHasState(flag,ROTATION_MASK)){
                mViewCompat.rotation(30);
            }
            if(requestHasState(flag,TRANSITIONY_MASK)){
                mViewCompat.translationY(200);
            }

            if(requestHasState(flag,TRANSITIONX_MASK)){
                mViewCompat.translationX(200);
            }
        }
        mViewCompat.setDuration(500).start();
    }
```

轻松实现这么多组合的变化，具体代码请在[github查看](https://github.com/jungledroid/AndroidCookie/blob/master/app/src/main/java/com/example/joez/androidcookie/BitmaskActivity.java)

### **结语 **
BitMask的方便之处在于：
1.一个int的变量可以存储多个状态，而定义多个变量难以控制，使用多态会导致类过多
2.扩展方面，新加入的状态只需要往左移对应的位数来标识即可。

总的来说，如果对位运算设置状态，清除状态，解包比较熟悉的话，使用BitMask还是比较方便。而对于进制转换不熟悉可以使用位移的方式，或者本地有个python解释器也是很方便的。
BitMask位运算可以参考这篇文章：[BitMask 使用参考](http://liaohuqiu.net/cn/posts/bitmasp-and-lipo/?utm_source=tuicool&utm_medium=referral)
