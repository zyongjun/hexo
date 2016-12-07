---
title: rxJava之变换Obserables
date: 2016-10-25 23:59:15
categories: Android
keywords: rxJava,rxAndroid,map,windhike
tags: [rxJava]
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

#### switchMap
```
Observable.just(1,2,3,4,5,6)
                .switchMap(new Func1<Integer, Observable<Integer>>() {
                    @Override
                    public Observable<Integer> call(Integer integer) {
                        return Observable.just(integer).subscribeOn(Schedulers.newThread());
                    }
                })
```
switchMap的用法和flatMap差不多，不过当源数据Observable发射一个数据项的时候，上一个Observable未订阅完成，测取消这个observable,停止监视这个obserable,转而监视新的数据项

#### groupBy
```
Observable.just(1,2,3,4,5,6,7,8,9)
                .groupBy(new Func1<Integer, Integer>() {
                    @Override
                    public Integer call(Integer integer) {
                        return integer%2;
                    }
                })
        .subscribe(new Action1<GroupedObservable<Integer, Integer>>() {
            @Override
            public void call(GroupedObservable<Integer, Integer> result) {
//                print("============="+result.getKey());
                if(result.getKey() == 0) {
                    result.subscribe(new Action1<Integer>() {
                        @Override
                        public void call(Integer integer) {
                            print("=============" + result.getKey() + "---" + integer);
                        }
                    });
                }
            }
        });
```
groupBy根据call返回的key和数据Observable发射的integer打包成GroupedObservable发射

#### map
```
Observable.just(1,2,3,4)
                .map(new Func1<Integer, Integer>() {
                    @Override
                    public Integer call(Integer integer) {
                        return integer*2;
                    }
                });
```
map类似flatMap，直接直接转换对象，而不是通过Observable转换
cast是map的一种实现，直接转换成类型，转换失败会报ClassCastException
```
Observable.just(1,2,3)
                .cast(Integer.class)
```
#### scan
```
Observable.just(1,2,3,4,5,6,7,8)
               .scan(new Func2<Integer, Integer, Integer>() {
                   @Override
                   public Integer call(Integer integer, Integer integer2) {
                       print("----"+integer+"---"+integer2);
                       return integer*integer2;
                   }
               }).subscribe(new Action1<Integer>() {
           @Override
           public void call(Integer integer) {

           }
       });
```
call 中的变换结果作为第一个参数应用到下一个数据传入的运算中

#### window
```
Observable.just(1,2,3,4,5,6)
                .window(2)
                .subscribe(new Action1<Observable<Integer>>() {
                    @Override
                    public void call(Observable<Integer> integerObservable) {
                        print("------call----");
                        integerObservable.subscribe(new Action1<Integer>() {
                            @Override
                            public void call(Integer integer) {
                                print("--------------"+integer);
                            }
                        });
                    }
                });
```
window与buffer类似，发射指定数目，跳过指定数目发射Observable，而不是List.  不仅可以根据数目来分组，也可以根据时间来分组。
