---
title: rxJava 之filter
date: 2016-11-25 12:38:21
categories: Android
keywords: rxJava
tags: [rxJava,rxAndroid,filter]
---
实际项目中，业务逻辑往往比较复杂，源数据往往不是直接来使用和显示这么简单。除了用上一篇中的变换原始数据，还可以对源数据进行过滤操作。

这一节学习下rxjava的过滤操作符。
<!--more-->
#### debounce和throttleWithTimeout
```
createObserver().throttleWithTimeout(200, TimeUnit.MILLISECONDS);
createObserver().debounce(200, TimeUnit.MILLISECONDS)
```
在一定时间内有发射新数据则放弃上个数据然后继续计时，以此类推
debounce还可接收func1函数：
```
getTimeObservable()
//                .debounce(1, TimeUnit.SECONDS)
              .debounce(new Func1<Integer, Observable<Integer>>() {
                  @Override
                  public Observable<Integer> call(Integer integer) {
                      return Observable.just(integer).delay(1,TimeUnit.SECONDS);
                  }
              })
```
如果func1返回的临时Observable没结束下一个数据发射的时候，取消上一个
#### distinct 和 distinctUtilChanged 去重复
```
Observable.just(1,2,3,4,2,3,4,5)
                .distinct()
                .subscribe(new Action1<Integer>() {
                    @Override
                    public void call(Integer integer) {
                        print("----------"+integer);
                    }
                });
```
distinct去除数据中的重复元素
```
Observable.just(1,2,3,4,2,3,4,5,5,6)
                .distinctUntilChanged()
                .subscribe(new Action1<Integer>() {
                    @Override
                    public void call(Integer integer) {
                        print("-----------"+integer);
                    }
                });
```
distinctUntilChanged去除相连的重复元素
另外，二者都可以接收func1，自定义去重复的规则。原理distinct操作过程中保存了一个Set<U>,这个U即func1的返回，而distinctUntilChanged的原理保存了上一个U和现在的作下对比。比较简单，这里以distinct举例:
如果不想重写Person的equal方法和hashcode方法，可以像下面这样指定属性作为判断是否重复的方式。
```
getPersonList().distinct(new Func1<Person, Integer>() {
                    @Override
                    public Integer call(Person person) {
                        return person.getId();
                    }
                })
```

#### filter和elementAt
```
Observable.just(1,2,3)
                .filter(new Func1<Integer, Boolean>() {
                    @Override
                    public Boolean call(Integer integer) {
                        return integer==2;
                    }
                })
```
filter只发射符合条件的数据
```
Observable.just(1,2,3,4,5)
                .elementAt(2)
                .subscribe(new Action1<Integer>() {
                    @Override
                    public void call(Integer integer) {
                        print("--------------"+integer);
                    }
                })
```
elementAt只发射指定位置的数据

#### first 和 last
```
Observable.just(1,2,3,4,5,6)
//                .first()
        .first(new Func1<Integer, Boolean>() {
            @Override
            public Boolean call(Integer integer) {
                return integer>4;
            }
        })
//                .firstOrDefault(6)
```
first取数据的第一个元素或者符合func1 call返回条件的第一个元素，如果没有符合条件的会抛出异常NoSuchElementException,有需要可以用firstOrDefault来提供一个默认的返回值。
last,lastOrDefault用法差不多，不多赘述。

#### skip skiplast take takeLast
用法比较简单，skip skipLast分别标识忽略最前面指定数目，忽略最后面指定数目
take和takeLast分别表示取最前面指定数目和最后面指定数目。

#### sample throttleFirst
sample定期发射最近发射的一个数据，相当throttleLast
throttleFirst定期发射最开始的一个数据

>注意:与throttleWithTimeout和debounce的区别，这里定期的时间单位里会取数据发射，而throttleWithTimeout和debounce在规定条件下有新的数据会放弃之前的数据重新开始验证。
