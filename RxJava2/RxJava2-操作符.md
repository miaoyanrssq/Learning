# RxJava2-操作符



## 1、create
`create` 操作符应该是最常见的操作符了，主要用于产生一个 `Obserable` 被观察者对象，为了方便大家的认知，统一把被观察者 `Observable` 称为发射器（上游事件），观察者 `Observer` 称为接收器（下游事件）。

![] (image/rj1.png)

```
public static void createTest()
    {
        final StringBuffer mRxOperatorsText = new StringBuffer();

        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {

                mRxOperatorsText.append("Observable emit 1" + "\n");
                Log.e(TAG, "Observable emit 1" + "\n");
                e.onNext(1);
                mRxOperatorsText.append("Observable emit 2" + "\n");
                Log.e(TAG, "Observable emit 2" + "\n");
                e.onNext(2);
                mRxOperatorsText.append("Observable emit 3" + "\n");
                Log.e(TAG, "Observable emit 3" + "\n");
                e.onNext(3);
                e.onComplete();
                mRxOperatorsText.append("Observable emit 4" + "\n");
                Log.e(TAG, "Observable emit 4" + "\n" );
                e.onNext(4);

            }
        }).subscribe(new Observer<Integer>() {
            private int i;
            private Disposable mDisposable;
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                mRxOperatorsText.append("onSubscribe : " + d.isDisposed() + "\n");
                Log.e(TAG, "onSubscribe : " + d.isDisposed() + "\n" );
                mDisposable = d;
            }

            @Override
            public void onNext(@NonNull Integer integer) {
                mRxOperatorsText.append("onNext : value : " + integer + "\n");
                Log.e(TAG, "onNext : value : " + integer + "\n" );
                i++;
                if(i == 2)
                {
                    // 在RxJava 2.x 中，新增的Disposable可以做到切断的操作，让Observer观察者不再接收上游事件
                    mDisposable.dispose();
                    mRxOperatorsText.append("onNext : isDisposable : " + mDisposable.isDisposed() + "\n");
                    Log.e(TAG, "onNext : isDisposable : " + mDisposable.isDisposed() + "\n");

                }
            }

            @Override
            public void onError(@NonNull Throwable e) {
                mRxOperatorsText.append("onError : value : " + e.getMessage() + "\n");
                Log.e(TAG, "onError : value : " + e.getMessage() + "\n" );

            }

            @Override
            public void onComplete() {
                mRxOperatorsText.append("onComplete" + "\n");
                Log.e(TAG, "onComplete" + "\n" );
            }
        });
    }
```

输出：

```
03-26 02:01:26.234 4119-4119/com.example.gaoyangzhen.helloworld E/RxjavaTest: onSubscribe : false
03-26 02:01:26.234 4119-4119/com.example.gaoyangzhen.helloworld E/RxjavaTest: Observable emit 1
03-26 02:01:26.234 4119-4119/com.example.gaoyangzhen.helloworld E/RxjavaTest: onNext : value : 1
03-26 02:01:26.234 4119-4119/com.example.gaoyangzhen.helloworld E/RxjavaTest: Observable emit 2
03-26 02:01:26.234 4119-4119/com.example.gaoyangzhen.helloworld E/RxjavaTest: onNext : value : 2
03-26 02:01:26.235 4119-4119/com.example.gaoyangzhen.helloworld E/RxjavaTest: onNext : isDisposable : true
03-26 02:01:26.235 4119-4119/com.example.gaoyangzhen.helloworld E/RxjavaTest: Observable emit 3
03-26 02:01:26.235 4119-4119/com.example.gaoyangzhen.helloworld E/RxjavaTest: Observable emit 4

```

需要注意的几点是：

* 在发射事件中，我们在发射了数值 3 之后，直接调用了 e.onComlete()，虽然无法接收事件，但发送事件还是继续的。

* 另外一个值得注意的点是，在 RxJava 2.x 中，可以看到发射事件方法相比 1.x 多了一个 throws Excetion，意味着我们做一些特定操作再也不用 try-catch 了。

* 并且 2.x 中有一个 Disposable 概念，这个东西可以直接调用切断，可以看到，当它的 isDisposed() 返回为 false 的时候，接收器能正常接收事件，但当其为 true 的时候，接收器停止了接收。所以可以通过此参数动态控制接收事件了。

## 2、map

它的作用是对发射时间发送的每一个事件应用一个函数，是的每一个事件都按照指定的函数去变化

![] (image/rj2.png)

```
public static void mapTest()
    {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
            }
        }).map(new Function<Integer, String>() {
            @Override
            public String apply(@NonNull Integer integer) throws Exception {
                return "the add num is : " + integer;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                Log.e(TAG, "accept:" + s + "\n");
            }
        });
    }
```

输出：

```
03-26 02:45:42.068 11602-11602/com.example.gaoyangzhen.helloworld E/RxjavaTest: accept:the add num is : 1
03-26 02:45:42.069 11602-11602/com.example.gaoyangzhen.helloworld E/RxjavaTest: accept:the add num is : 2
03-26 02:45:42.069 11602-11602/com.example.gaoyangzhen.helloworld E/RxjavaTest: accept:the add num is : 3

```

`map` 基本作用就是将一个 `Observable` 通过某种函数关系，转换为另一种 `Observable`，上面例子中就是把我们的 `Integer` 数据变成了 String 类型。

## 3、zip

zip 专用于合并事件，该合并不是连接（连接操作符后面会说），而是两两配对，也就意味着，最终配对出的 Observable 发射事件数目只和少的那个相同。

```
Observable.zip(getStringObservable(), getIntegerObservable(), new BiFunction<String, Integer, String>() {
            @Override
            public String apply(@NonNull String s, @NonNull Integer integer) throws Exception {
                return s + integer;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                mRxOperatorsText.append("zip : accept : " + s + "\n");
                Log.e(TAG, "zip : accept : " + s + "\n");
            }
        });


private Observable<String> getStringObservable() {
        return Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                if (!e.isDisposed()) {
                    e.onNext("A");
                    mRxOperatorsText.append("String emit : A \n");
                    Log.e(TAG, "String emit : A \n");
                    e.onNext("B");
                    mRxOperatorsText.append("String emit : B \n");
                    Log.e(TAG, "String emit : B \n");
                    e.onNext("C");
                    mRxOperatorsText.append("String emit : C \n");
                    Log.e(TAG, "String emit : C \n");
                }
            }
        });
    }

    private Observable<Integer> getIntegerObservable() {
        return Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                if (!e.isDisposed()) {
                    e.onNext(1);
                    mRxOperatorsText.append("Integer emit : 1 \n");
                    Log.e(TAG, "Integer emit : 1 \n");
                    e.onNext(2);
                    mRxOperatorsText.append("Integer emit : 2 \n");
                    Log.e(TAG, "Integer emit : 2 \n");
                    e.onNext(3);
                    mRxOperatorsText.append("Integer emit : 3 \n");
                    Log.e(TAG, "Integer emit : 3 \n");
                    e.onNext(4);
                    mRxOperatorsText.append("Integer emit : 4 \n");
                    Log.e(TAG, "Integer emit : 4 \n");
                    e.onNext(5);
                    mRxOperatorsText.append("Integer emit : 5 \n");
                    Log.e(TAG, "Integer emit : 5 \n");
                }
            }
        });
    }

```
输出：
```
03-26 02:56:32.206 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: String emit : A 
03-26 02:56:32.206 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: String emit : B 
03-26 02:56:32.207 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: String emit : C 
03-26 02:56:32.207 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: zip : accept : A1
03-26 02:56:32.207 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: Integer emit : 1 
03-26 02:56:32.207 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: zip : accept : B2
03-26 02:56:32.207 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: Integer emit : 2 
03-26 02:56:32.207 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: zip : accept : C3
03-26 02:56:32.207 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: Integer emit : 3 
03-26 02:56:32.207 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: Integer emit : 4 
03-26 02:56:32.207 20873-20873/com.example.gaoyangzhen.helloworld E/RxjavaTest: Integer emit : 5
```

需要注意的是：

* zip 组合事件的过程就是分别从发射器 A 和发射器 B 各取出一个事件来组合，并且一个事件只能被使用一次，组合的顺序是严格按照事件发送的顺序来进行的，所以上面截图中，可以看到，1 永远是和 A 结合的，2 永远是和 B 结合的。

* 最终接收器收到的事件数量是和发送器发送事件最少的那个发送器的发送事件数目相同，所以如截图中，4、5 很孤单，没有人愿意和它交往，孤独终老的单身狗。

## 4、Concat
对于单一的把两个发射器连接成一个发射器，虽然 zip 不能完成，但我们还是可以自力更生，官方提供的 concat 让我们的问题得到了完美解决。

![] (image/rj3.png)

```
public static void concatTest()
    {
        Observable.concat(Observable.just(1,2,3), Observable.just(4,5, 6))
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {

                        Log.e(TAG, "concat :" + integer + "\n");
                    }
                });
    }
```
输出：

```
03-26 06:40:48.353 2914-2914/com.example.gaoyangzhen.helloworld E/RxjavaTest: concat :1
03-26 06:40:48.353 2914-2914/com.example.gaoyangzhen.helloworld E/RxjavaTest: concat :2
03-26 06:40:48.353 2914-2914/com.example.gaoyangzhen.helloworld E/RxjavaTest: concat :3
03-26 06:40:48.353 2914-2914/com.example.gaoyangzhen.helloworld E/RxjavaTest: concat :4
03-26 06:40:48.354 2914-2914/com.example.gaoyangzhen.helloworld E/RxjavaTest: concat :5
03-26 06:40:48.354 2914-2914/com.example.gaoyangzhen.helloworld E/RxjavaTest: concat :6

```
如图，可以看到。发射器 B 把自己的三个孩子送给了发射器 A，让他们组合成了一个新的发射器，非常懂事的孩子，有条不紊的排序接收。

## 5、flatMap
FlatMap 是一个很有趣的东西，我坚信你在实际开发中会经常用到。它可以把一个发射器 Observable 通过某种方法转换为多个 Observables，然后再把这些分散的 Observables装进一个单一的发射器 Observable。但有个需要注意的是，flatMap 并不能保证事件的顺序，如果需要保证，需要用到我们下面要讲的 ConcatMap。

![] (image/rj4.png)

```
public static void flatMapTest()
    {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);

            }
        }).flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(@NonNull Integer integer) throws Exception {

                List<String> list = new ArrayList<>();
                for(int i=0 ; i<3 ; i++)
                {
                    list.add("I am value" + integer);

                }
                int delayTime = (int) (1 + Math.random() * 10);
                return Observable.fromIterable(list).delay(delayTime, TimeUnit.MICROSECONDS);
            }
        }).subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        Log.e(TAG, "flatMap : accept : " + s + "\n");
                    }
                });
    }
```

输出：

```
03-26 07:05:23.880 30103-30103/? E/RxjavaTest: flatMap : accept : I am value1
03-26 07:05:23.880 30103-30103/? E/RxjavaTest: flatMap : accept : I am value2
03-26 07:05:23.880 30103-30103/? E/RxjavaTest: flatMap : accept : I am value2
03-26 07:05:23.880 30103-30103/? E/RxjavaTest: flatMap : accept : I am value2
03-26 07:05:23.880 30103-30103/? E/RxjavaTest: flatMap : accept : I am value1
03-26 07:05:23.881 30103-30103/? E/RxjavaTest: flatMap : accept : I am value1
03-26 07:05:23.881 30103-30103/? E/RxjavaTest: flatMap : accept : I am value3
03-26 07:05:23.881 30103-30103/? E/RxjavaTest: flatMap : accept : I am value3
03-26 07:05:23.881 30103-30103/? E/RxjavaTest: flatMap : accept : I am value3
```
一切都如我们预期中的有意思，为了区分 concatMap（下一个会讲），我在代码中特意动了一点小手脚，我采用一个随机数，生成一个时间，然后通过 delay（后面会讲）操作符，做一个小延时操作，而查看 Log 日志也确认验证了我们上面的说法，它是无序的。

## 6、concatMap

concatMap 与 FlatMap 的唯一区别就是 concatMap 保证了顺序，所以，我们就直接把 flatMap 替换为 concatMap 验证吧。

```
public static void concatMapTest()
    {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);

            }
        }).concatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(@NonNull Integer integer) throws Exception {

                List<String> list = new ArrayList<>();
                for(int i=0 ; i<3 ; i++)
                {
                    list.add("I am value" + integer);

                }
                int delayTime = (int) (1 + Math.random() * 10);
                return Observable.fromIterable(list).delay(delayTime, TimeUnit.MICROSECONDS);
            }
        }).subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        Log.e(TAG, "flatMap : accept : " + s + "\n");
                    }
                });
    }
```
输出：

```
03-26 07:43:00.660 25676-25676/? E/RxjavaTest: flatMap : accept : I am value1
03-26 07:43:00.660 25676-25676/? E/RxjavaTest: flatMap : accept : I am value1
03-26 07:43:00.660 25676-25676/? E/RxjavaTest: flatMap : accept : I am value1
03-26 07:43:00.660 25676-25676/? E/RxjavaTest: flatMap : accept : I am value2
03-26 07:43:00.660 25676-25676/? E/RxjavaTest: flatMap : accept : I am value2
03-26 07:43:00.661 25676-25676/? E/RxjavaTest: flatMap : accept : I am value2
03-26 07:43:00.661 25676-25676/? E/RxjavaTest: flatMap : accept : I am value3
03-26 07:43:00.661 25676-25676/? E/RxjavaTest: flatMap : accept : I am value3
03-26 07:43:00.661 25676-25676/? E/RxjavaTest: flatMap : accept : I am value3
```

## 7、distinct

去重

![] (image/rj5.png)

```
public static void distinctTest()
    {
        Observable.just(1,1,1,2,2,3, 3,4)
                .distinct()
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.e(TAG, "distinct : " + integer + "\n");
                    }
                });
    }
```

```
03-26 08:00:17.206 6559-6559/com.example.gaoyangzhen.helloworld E/RxjavaTest: distinct : 1
03-26 08:00:17.206 6559-6559/com.example.gaoyangzhen.helloworld E/RxjavaTest: distinct : 2
03-26 08:00:17.206 6559-6559/com.example.gaoyangzhen.helloworld E/RxjavaTest: distinct : 3
03-26 08:00:17.206 6559-6559/com.example.gaoyangzhen.helloworld E/RxjavaTest: distinct : 4

```

## 8、filter
过滤器，可以接受一个参数，让其过滤掉不符合条件的值

![] (image/rj6.png)

```
public static void filterTest()
    {
        Observable.just(1,20, 50, -10, 1000)
                .filter(new Predicate<Integer>() {
                    @Override
                    public boolean test(@NonNull Integer integer) throws Exception {
                        return integer >10;
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.e(TAG, "filter : " + integer + "\n");
            }
        });
    }
```

## 9、buffer

buffer 操作符接受两个参数，buffer(count,skip)，作用是将 Observable 中的数据按 skip (步长) 分成最大不超过 count 的 buffer ，然后生成一个  Observable 

![] (image/rj7.png

```
Observable.just(1, 2, 3, 4, 5)
                .buffer(3, 2)
                .subscribe(new Consumer<List<Integer>>() {
                    @Override
                    public void accept(@NonNull List<Integer> integers) throws Exception {
                        mRxOperatorsText.append("buffer size : " + integers.size() + "\n");
                        Log.e(TAG, "buffer size : " + integers.size() + "\n");
                        mRxOperatorsText.append("buffer value : ");
                        Log.e(TAG, "buffer value : " );
                        for (Integer i : integers) {
                            mRxOperatorsText.append(i + "");
                            Log.e(TAG, i + "");
                        }
                        mRxOperatorsText.append("\n");
                        Log.e(TAG, "\n");
                    }
                });

```

我们把 1, 2, 3, 4, 5 依次发射出来，经过 buffer 操作符，其中参数 skip 为 2， count 为 3，而我们的输出 依次是 123，345，5。显而易见，我们 buffer 的第一个参数是 count，代表最大取值，在事件足够的时候，一般都是取 count 个值，然后每次跳过 skip 个事件。

## 10、timer

`timer` 很有意思，相当于一个定时任务。在 1.x 中它还可以执行间隔逻辑，但在 2.x 中此功能被交给了 `interval`，下一个会介绍。但需要注意的是，`timer `和 `interval` 均默认在新线程。

```
public static void timerTest()
    {
        Log.e(TAG, "timer start : " + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(System.currentTimeMillis()) + "\n");
        Observable.timer(2, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Long>() {
                    @Override
                    public void accept(Long aLong) throws Exception {
                        Log.e(TAG, "timer start : " + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(System.currentTimeMillis()) + "\n");
                    }
                });

    }
```

```
03-27 02:12:56.226 1386-1386/com.example.gaoyangzhen.helloworld E/RxjavaTest: timer start : 2018-03-27 02:12:56
03-27 02:12:58.395 1386-1386/com.example.gaoyangzhen.helloworld E/RxjavaTest: timer start : 2018-03-27 02:12:58

```

接收被延迟了 2 秒。

## 11、interval

interval 操作符用于间隔时间执行某个操作，其接受三个参数，分别是第一次发送延迟，间隔时间，时间单位。

```
@Override
   protected void doSomething() {
       mRxOperatorsText.append("interval start : " + TimeUtil.getNowStrTime() + "\n");
       Log.e(TAG, "interval start : " + TimeUtil.getNowStrTime() + "\n");
       mDisposable = Observable.interval(3, 2, TimeUnit.SECONDS)
               .subscribeOn(Schedulers.io())
               .observeOn(AndroidSchedulers.mainThread()) // 由于interval默认在新线程，所以我们应该切回主线程
               .subscribe(new Consumer<Long>() {
                   @Override
                   public void accept(@NonNull Long aLong) throws Exception {
                       mRxOperatorsText.append("interval :" + aLong + " at " + TimeUtil.getNowStrTime() + "\n");
                       Log.e(TAG, "interval :" + aLong + " at " + TimeUtil.getNowStrTime() + "\n");
                   }
               });
   }

   @Override
   protected void onDestroy() {
       super.onDestroy();
       if (mDisposable != null && !mDisposable.isDisposed()) {
           mDisposable.dispose();
       }
   }

```

第一次延迟了 3 秒后接收到，后面每次间隔了 2 秒。
然而，心细的小伙伴可能会发现，由于我们这个是间隔执行，所以当我们的Activity 都销毁的时候，实际上这个操作还依然在进行，所以，我们得花点小心思让我们在不需要它的时候干掉它。查看源码发现，我们subscribe(Cousumer<? super T> onNext)返回的是Disposable，我们可以在这上面做文章。

## 12、doOnNext

其实觉得 doOnNext 应该不算一个操作符，但考虑到其常用性，我们还是咬咬牙将它放在了这里。它的作用是让订阅者在接收到数据之前干点有意思的事情。假如我们在获取到数据之前想先保存一下它，无疑我们可以这样实现。

```
Observable.just(1, 2, 3, 4)
                .doOnNext(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("doOnNext 保存 " + integer + "成功" + "\n");
                        Log.e(TAG, "doOnNext 保存 " + integer + "成功" + "\n");
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                mRxOperatorsText.append("doOnNext :" + integer + "\n");
                Log.e(TAG, "doOnNext :" + integer + "\n");
            }
        });

```

## 13、skip

skip 很有意思，其实作用就和字面意思一样，接受一个 long 型参数 count ，代表跳过 count 个数目开始接收。

![] (image/rj8.png)

```
Observable.just(1,2,3,4,5)
                .skip(2)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("skip : "+integer + "\n");
                        Log.e(TAG, "skip : "+integer + "\n");
                    }
                });

```

## 14、take
take，接受一个 long 型参数 count ，代表至多接收 count 个数据。

![] (image/rj9.png)

```
Flowable.fromArray(1,2,3,4,5)
                .take(2)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("take : "+integer + "\n");
                        Log.e(TAG, "accept: take : "+integer + "\n" );
                    }
                });

```

## 15、just

just，没什么好说的，其实在前面各种例子都说明了，就是一个简单的发射器依次调用 onNext() 方法。

```
Observable.just("1", "2", "3")
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        mRxOperatorsText.append("accept : onNext : " + s + "\n");
                        Log.e(TAG,"accept : onNext : " + s + "\n" );
                    }
                });

```

## 16、Single

顾名思义，Single 只会接收一个参数，而 SingleObserver 只会调用 onError() 或者 onSuccess()。

```
Single.just(new Random().nextInt())
                .subscribe(new SingleObserver<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {

                    }

                    @Override
                    public void onSuccess(@NonNull Integer integer) {
                        mRxOperatorsText.append("single : onSuccess : "+integer+"\n");
                        Log.e(TAG, "single : onSuccess : "+integer+"\n" );
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        mRxOperatorsText.append("single : onError : "+e.getMessage()+"\n");
                        Log.e(TAG, "single : onError : "+e.getMessage()+"\n");
                    }
                });

```

## 17、distinct

![] (image/rj10.png)

```
Observable.just(1, 1, 1, 2, 2, 3, 4, 5)
                .distinct()
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("distinct : " + integer + "\n");
                        Log.e(TAG, "distinct : " + integer + "\n");
                    }
                });

```

输出12345.

## 18、debounce

去除发送频率过快的项

![] (image/rj11.png)

```
public static void debounceTest()
    {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> emitter) throws Exception {
                // send events with simulated time wait
                emitter.onNext(1); // skip
                Thread.sleep(400);
                emitter.onNext(2); // deliver
                Thread.sleep(505);
                emitter.onNext(3); // skip
                Thread.sleep(100);
                emitter.onNext(4); // deliver
                Thread.sleep(605);
                emitter.onNext(5); // deliver
                Thread.sleep(510);
                emitter.onComplete();
            }
        }).debounce(500, TimeUnit.MILLISECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG,"debounce :" + integer + "\n");
                    }
                });

    }
```

输出：

```
03-28 01:35:52.767 12239-12239/com.example.gaoyangzhen.helloworld E/RxjavaTest: debounce :2
03-28 01:35:53.378 12239-12239/com.example.gaoyangzhen.helloworld E/RxjavaTest: debounce :4
03-28 01:35:53.985 12239-12239/com.example.gaoyangzhen.helloworld E/RxjavaTest: debounce :5

```

去除发送间隔时间小于 500 毫秒的发射事件，所以 1 和 3 被去掉了。

## 19、last
last 操作符仅取出可观察到的最后一个值，或者是满足某些条件的最后一项。

![] (image/rj12.png)

```
Observable.just(1, 2, 3)
                .last(4)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("last : " + integer + "\n");
                        Log.e(TAG, "last : " + integer + "\n");
                    }
                });

```

输出： last： 3

## 20、 merge

merge 顾名思义，熟悉版本控制工具的你一定不会不知道 merge 命令，而在 Rx 操作符中，merge 的作用是把多个 Observable 结合起来，接受可变参数，也支持迭代器集合。注意它和 concat 的区别在于，不用等到 发射器 A 发送完所有的事件再进行发射器 B 的发送。

![] (image/rj13.png)

```
Observable.merge(Observable.just(1, 2), Observable.just(3, 4, 5))
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("merge :" + integer + "\n");
                        Log.e(TAG, "accept: merge :" + integer + "\n" );
                    }
                });

```

## 21、 reduce
reduce 操作符每次用一个方法处理一个值，可以有一个 seed 作为初始值。

![] (image/rj14.png)

```
Observable.just(1, 2, 3)
               .reduce(new BiFunction<Integer, Integer, Integer>() {
                   @Override
                   public Integer apply(@NonNull Integer integer, @NonNull Integer integer2) throws Exception {
                       return integer + integer2;
                   }
               }).subscribe(new Consumer<Integer>() {
           @Override
           public void accept(@NonNull Integer integer) throws Exception {
               mRxOperatorsText.append("reduce : " + integer + "\n");
               Log.e(TAG, "accept: reduce : " + integer + "\n");
           }
       });

```
输出： reduce：6

可以看到，代码中，我们中间采用 reduce ，支持一个 function 为两数值相加，所以应该最后的值是：1 + 2 = 3 + 3 = 6

## 22、scan

scan 操作符作用和上面的 reduce 一致，唯一区别是 reduce 只追求结果，而 scan 会始终如一地把每一个步骤都输出。

![] (image/rj15.png)

```
Observable.just(1, 2, 3)
                .scan(new BiFunction<Integer, Integer, Integer>() {
                    @Override
                    public Integer apply(@NonNull Integer integer, @NonNull Integer integer2) throws Exception {
                        return integer + integer2;
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                mRxOperatorsText.append("scan " + integer + "\n");
                Log.e(TAG, "accept: scan " + integer + "\n");
            }
        });

```

输出：

```
scan 1
scan 3
scan 6
```
