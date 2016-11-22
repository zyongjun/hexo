---
title: rxjava 之创建Observable
date: 2016-10-25 23:59:15
categories: Android
keywords: rxJava
tags: [rxJava,rxAndroid,map]
---
作为rxJava系列的开始第一篇，这里主要介绍如何创建一个Observable,以及注意的事项。
<!--more-->
#### create
```
Observable.create(new Observable.OnSubscribe<Integer>() {

          @Override
          public void call(Subscriber<? super Integer> subscriber) {
              if (!subscriber.isUnsubscribed()) {
                  subscriber.onNext(1);
//                    subscriber.onError(new Exception("error"));
                  subscriber.onCompleted();
              }

          }
      });
```
这里需要手动处理subscriber的回调

#### range
```
Observable.range(0,10)
```
创建一个0到9的序列
#### just
```
 Observable.just(1,2,3)
```
对象、数组、Iterable转Observable，轮流发射传入的参数
#### defer
```
Observable.defer(new Func0<Observable<Integer>>() {
            @Override
            public Observable<Integer> call() {
                return Observable.just(1,2,3);
            }
        });
```
延迟创建observable,确保创建observable使用数据是最新的
#### from
```
List<Integer>  list = new ArrayList<>();
list.add(1);
list.add(2);
Observable.from(list);
```
对象转Observable,接收Iterable或数组，轮流发射其中内容，just一次发射对象。

#### interval timer repeat
```
Observable.timer(1,TimeUnit.SECONDS) //指定时间后发射0
Observable.interval(3, TimeUnit.SECONDS) //从0开始，指定时间发射一个数字
Observable.just(1).repeat(5);//重复发射指定次数
```

#### 小结
对象转Observable  
- just 一次发射对象
- defer 延迟传教Observable,保证创建数据是最新的
- from 一次发射传入的数组或者iterable
时间相关
- interval 轮询
- timer 延迟任务
- repeat 重复任务
其他:
- create 需手动处理观察者回调
- range 生成数列的obserable

好了，学习rxJava 从学会创建observable开始。
