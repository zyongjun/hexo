---
title: rxjava 之创建Observable
date: 2016-10-25 23:59:15
categories: Android
keywords: rxJava
tags: [rxJava,rxAndroid,map]
---
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

#### 总结
