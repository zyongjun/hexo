---
title: rxJava之utility有用的辅助操作符
date: 2016-12-04 22:34:39
categories: Android
keywords: rxJava
tags: [rxJava,rxAndroid]
---
前面按分类介绍了各种操作符，这一节来探究下一些有用的辅助操作符。
<!--more-->
#### delay和delaySubscription
- delay让数据发射延迟
- delaySubscription让注册subscriber延迟注册

```
Observable.just("a","b","c")
//    .delay(3, TimeUnit.SECONDS)
//    .delay(3,TimeUnit.SECONDS,Schedulers.immediate())
      .delay(3,TimeUnit.SECONDS,Schedulers.trampoline())
//    .delaySubscription(2,TimeUnit.SECONDS,Schedulers.immediate())
```
**delay这里有点奇怪还不知道原理，知道的朋友可以在下面评论交流下。
默认是在Schedulers.computation()调度器下，经java测试之在上面两种调度器下面工作才能达到delay的效果，否则subscriber里面的代码都无法执行,Why?**


#### do
doxxx主要用于回调
- doOnEach
- doOnNext
- doOnSubscribe和doOnUnSubscribe则会在Subscriber进行订阅和反订阅的时候触发回调。当一个Observable通过OnError或者OnCompleted结束的时候，会反订阅所有的Subscriber。
- doOnError
- doOnCompleted
- doOnTerminate会在Observable结束前触发回调，无论是正常还是异常终止
- finallyDo(**已过期使用doAfterTerminate替换**)会在Observable结束后触发回调，无论是正常还是异常终止。

#### meterialize和deMeterialize
 meterialize操作符将OnNext/OnError/OnComplete都转化为一个Notification对象并按照原来的顺序发射出来，而deMeterialize则是执行相反的过程。

 ```
 Observable.just(1,2,3)
                .materialize()
                .subscribe(new Action1<Notification<Integer>>() {
                    @Override
                    public void call(Notification<Integer> integerNotification) {

                    }
                });
 ```
 其实就是将源Observable<Integer>转化为了Observable<Notification<Integer>>,**注意onError或者onCompleted也作为Notification数据加在最后面。**

 #### timeInterval和timeStamp

 ```
 Observable.just(1,2,3)
          .timestamp()
          .subscribe(new Action1<Timestamped<Integer>>() {
              @Override
              public void call(Timestamped<Integer> integerTimestamped) {

              }
          });
 ```
- timeStamp则是封装来时间戳和源数据.
 ```
  Observable.just(1,2,3)
    .timeInterval()
    .subscribe(new Action1<TimeInterval<Integer>>() {
        @Override
        public void call(TimeInterval<Integer> interval) {
            print("==="+interval.getValue()+"----"+interval.getIntervalInMilliseconds());
        }
    });
 ```
- timeInterval将Observable<Integer>转化为Observable<TimeInterval<Integer>>,是对源Observable发射数据和注册到发射第一个数据时间间隔或数据发射之间时间间隔的简单封装。
