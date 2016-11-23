---
title: rxJava之变换Obserables
date: 2016-10-25 23:59:15
categories: Android
keywords: rxJava
tags: [rxJava,rxAndroid,map]
---
这一节来了解如何变换Observable为我们需要的。

<!--more-->

#### buffer
```
Observable.just(1,2,3,4,5,6,7,8,9)
             //.buffer(2)
             .buffer(2,3)
             .subscribe(new Action1<List<Integer>>() {
                 @Override
                 public void call(List<Integer> integers) {
                     print("=========="+integers.toString());
                 }
             });
```
一个参数： 一次缓存指定数据打包为列表发射出来
两个参数: 每隔后者个数据取前者个数据打包成列表发射出来

其实两者调用的是同一个方法，一个参数相当于调用了buffer(count,count)

#### flagmap 与 flatmapiterable
```
Observable.just(1,2,3,4,5,6,7,8,9)
            .flatMap(new Func1<Integer, Observable<Integer>>() {
                @Override
                public Observable<Integer> call(Integer integer) {
                    return Observable.just(integer);
                }
            })
            .subscribe(new Action1<Integer>() {
                @Override
                public void call(Integer integer) {
                    print("------"+integer);
                }
            });
```
flatmap 将目标observable的源数据转化为observable依次发射出来
```
Observable.just(1,2,3,4,5,6,7,8,9)
              .flatMapIterable(new Func1<Integer, Iterable<Integer>>() {
                  @Override
                  public Iterable<Integer> call(Integer integer) {
                      print("----------"+integer);
                      List<Integer> list = new ArrayList<Integer>();
                      list.add(integer);
                  }
              })
              .subscribe(new Action1<Integer>() {
                  @Override
                  public void call(Integer integer) {

                  }
              });
```
flatMapIterable 基本同flatMap，只是将源数据转化为Iterable对象发射出来
