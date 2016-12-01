---
title: rx之错误处理
date: 2016-12-01 11:18:26
categories: Android
keywords: rxJava
tags: [rxJava,rxAndroid]
---
rxjava 提供了丰富的错误处理丰富，根据需要可以统一处理错误。
<!--more-->
#### catch
这里先生成一个产生exception的Observable
```
private Observable<Integer> getErrorObservable() {
        return Observable.create(new Observable.OnSubscribe<Integer>() {
            @Override
            public void call(Subscriber<? super Integer> subscriber) {
                for (int i = 1;i<6;i++) {
                    if (i!=2&&i != 4) {
                        subscriber.onNext(i);
                    } else {
                        subscriber.onError(new Throwable("on error happened"));
                    }

                }
            }
        });
    }
```
- onErrorReturn
```
getErrorObservable().onErrorReturn(new Func1<Throwable, Integer>() {
           @Override
           public Integer call(Throwable throwable) {
               return 999;
           }
       })
```
当错误发生的时候，会走onErrorReturn的call，使用返回的数据替换error的这次发射结果,然后马上终止，走onCompleted.

- onErrorResumeNext

```
getErrorObservable()
//                .onErrorResumeNext(Observable.just(1,2,3))
        .onErrorResumeNext(new Func1<Throwable, Observable<? extends Integer>>() {
            @Override
            public Observable<? extends Integer> call(Throwable throwable) {
                return Observable.just(1,2,3);
            }
        })
```
onErrorResumeNext:可以接收一个Observable或Func1,当exception发生的时候，会使用这个Observable替换源Observable继续发射数据。

- onExceptionResumeNext
```
Observable.create(new Observable.OnSubscribe<Integer>() {
            @Override
            public void call(Subscriber<? super Integer> subscriber) {
                for (int i = 1;i<6;i++) {
                    if (i == 3) {
                        subscriber.onError(new Throwable("on error happened"));
                    } else if(i==2){
                        subscriber.onError(new Exception("an exception happend"));
                    }else{
                        subscriber.onNext(i);
                    }

                }
            }
        }).onExceptionResumeNext(Observable.just(10,11))
```
onExceptionResumeNext接收一备用的Observable,与onErrorResumeNext类似，不过只有error类型是Exception的时候才会使用备用Observable，其他错误类型如Throwable则会走subscriber的onError,
