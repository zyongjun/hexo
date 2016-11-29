---
title: rx-combining
date: 2016-11-27 11:21:29
categories: Android
keywords: rxJava
tags: [rxJava,rxAndroid,combining]
---
这一篇里学习如何组装Observable数据
<!--more-->

#### CombineLatest
```
private Observable<Integer> createObserver(int index) {
        return Observable.create(new Observable.OnSubscribe<Integer>() {
            @Override
            public void call(Subscriber<? super Integer> subscriber) {
                for (int i = 1; i < 6; i++) {
                    subscriber.onNext(i * index);
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
    }

    public void testCombiningLast() {
        Observable.combineLatest(createObserver(2), createObserver(1), new Func2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer num1, Integer num2) {
                print("left:" + num1 + " right:" + num2);
                return num1 + num2;
            }
        }).subscribe();
    }
```
- call执行的时机:所有的Observable都发射过数据，任意一Observable发射数据时会call所有Observable最新发射的数据
- call中数据的位置对应传入的Observable的位置
- combineLatest也接受list<Observable>

#### join groupJoin
```
Observable.just("left","left1").join(createObserver(1), new Func1<String, Observable<Long>>() {
    @Override
    public Observable<Long> call(String s) {
        return Observable.timer(3000, TimeUnit.MILLISECONDS);
    }
}, new Func1<Integer, Observable<Long>>() {
    @Override
    public Observable<Long> call(Integer integer) {
        return Observable.timer(1000, TimeUnit.MILLISECONDS);
    }
}, new Func2<String, Integer, String>() {
    @Override
    public String call(String s, Integer integer) {
        return s+integer;
    }
}).subscribe(new Action1<String>() {
    @Override
    public void call(String s) {
        print("-------------next"+s);
    }
});
```
基于时间窗口。。
源数据join Observable,Observable每发射一个数据都会依次与源数据组合成结果数据发射出来。
注意:func1接收对应的数据返回的Observable决定对应数据的有效期。
groupJoin用法一样，只是最后组合参数有所不同

#### merge和concat MeregeDelayError
```
Observable.merge(Observable.just(1,2,3),Observable.just(4,5,6))
          .subscribe(new Action1<Integer>() {
              @Override
              public void call(Integer integer) {
                  print("----------"+integer);
              }
          });
```
merge 将多个observable发射的数据整合起来发射，像一个observable一样，但可能是交错的。
如果不想交错可以用concat.

MeregeDelayError
```
Observable.mergeDelayError(Observable.just(1,2,3),Observable.create(new Observable.OnSubscribe<Integer>() {
            @Override
            public void call(Subscriber<? super Integer> subscriber) {
                for (int i= 11;i<16;i++) {
                    if (i == 14) {
                        subscriber.onError(new Throwable("onError"));
                    }
                    subscriber.onNext(i);
                }
            }
        }),Observable.just(6,7,8))
```
如果merge的过程中发生错误，后面的merge就会中断，这里使用mergeDelayError,可以暂时不发射error，当merge完成后，最后发射error
