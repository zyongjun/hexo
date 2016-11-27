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
