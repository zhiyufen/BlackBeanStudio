## [RxJava2]操作符之变换操作

用于对Observable发射的数据执行变换操作的各种操作符：

 - `map( )`  — 对序列的每一项都应用一个函数来变换Observable发射的数据序列
 - `flatMap( )`  ,  `concatMap( )`  , and  `flatMapIterable( )`  —
   将`Observable`发射的数据集合变换为`Observables`集合，然后将这些`Observable`发射的数据平坦化的放进一个单独的`Observable`
 - `switchMap( )` —
   将`Observable`发射的数据集合变换为`Observables`集合，然后只发射这些`Observables`最近发射的数据
 - `scan( )`  — 对`Observable`发射的每一项数据应用一个函数，然后按顺序依次发射每一个值
 - `groupBy( ) ` —
   将`Observable`分拆为`Observable`集合，将原始`Observable`发射的数据按`Key`分组，每一个Observable发射一组不同的数据
 - buffer( )  — 它定期从Observable收集数据到一个集合，然后把这些数据集合打包发射，而不是一次发射一个
 - `window( )`  — 定期将来自`Observable`的数据分拆成一些`Observable`窗口，然后发射这些窗口，而不是每次发射一项
 - `cast( )`  — 在发射之前强制将Observable发射的所有数据转换为指定类型

#### 操作符------Buffer
&emsp;&emsp;收集`Observable`的数据放进一个数据包裹，然后发射这些数据包裹，而不是一次发射一个值。`Buffer` 操作符将一个`Observable`变换为另一个，原来的`Observable`正常发射数据，变换产生的`Observable`发射这些数据的缓存集合。
&emsp;&emsp;注意：如果原来的`Observable`发射了一个`onError`通知，`Buffer`会立即传递这个通知，而不是首先发射缓存的数据，即使在这之前缓存中包含了原始`Observable`发射的数据。

在RxJava中有许多 Buffer  的变体：

##### 1.`buffer(count)`
&emsp;&emsp;`buffer(count)`  以列表`(List)`的形式发射非重叠的缓存，每一个缓存至多包含来自原始`Observable`的`count`项数据（最后发射的列表数据可能少于`count`项）

Demo Code:
```java
		static int i =1;//打Log标志位
        //创建一个 Observable 用来发送 0 ～ 22 的23个整数值
        //使用Buffer操作符变换成每次发5个数据的Observable.
        Observable.range(0,23).buffer(5).subscribe(new Observer<List<Integer>>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("yufen","onSubscribe..... And Cureent Thread: " + Thread.currentThread());
            }

            @Override
            public void onNext(@NonNull List<Integer> integers) {
                if (i == 1) {
                    Log.d("yufen", "Next: " + String.valueOf(integers.toString()) + " And Thread: " + Thread.currentThread());
                    i++;
                }else {
                    Log.d("yufen", "Next: " + String.valueOf(integers.toString()));
                }
            }


            @Override
            public void onError(@NonNull Throwable e) {
                Log.d("yufen","Error: " + e.getMessage());
            }

            @Override
            public void onComplete() {
                Log.d("yufen","Sequence complete.");
            }
        });
```
输出Log:
```
10-25 10:52:33.115 6054-6054/blackbean.rxjavademo D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-25 10:52:33.115 6054-6054/blackbean.rxjavademo D/yufen: Next: [0, 1, 2, 3, 4] And Thread: Thread[main,5,main]
10-25 10:52:33.115 6054-6054/blackbean.rxjavademo D/yufen: Next: [5, 6, 7, 8, 9] And Thread: Thread[main,5,main]
10-25 10:52:33.115 6054-6054/blackbean.rxjavademo D/yufen: Next: [10, 11, 12, 13, 14] And Thread: Thread[main,5,main]
10-25 10:52:33.115 6054-6054/blackbean.rxjavademo D/yufen: Next: [15, 16, 17, 18, 19] And Thread: Thread[main,5,main]
10-25 10:52:33.115 6054-6054/blackbean.rxjavademo D/yufen: Next: [20, 21, 22] And Thread: Thread[main,5,main]
10-25 10:52:33.115 6054-6054/blackbean.rxjavademo D/yufen: Sequence complete.
```

##### 2.`Observable<List<T>> buffer(int count, int skip)`
&emsp;&emsp;`buffer(count, skip)`  从原始`Observabl`e的第一项数据开始创建新的缓存，此后每当收到` skip` 项数据，用` count` 项数据填充缓存：开头的一项和后续的` count-1` 项，它以列表(List)的形式发射缓存，取决于` count` 和`skip`的值，这些缓存可能会有重叠部分（比如skip <count时），也可能会有间隙（比如skip > count时）。

###### a. Count > skip时

```java
	//...
	observable.buffer(3,5).subscribe(new Observer<List<Integer>>() {...}
	//....
```
输出Log:
```
10-25 11:25:04.515 3503-3503/blackbean.rxjavademo D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-25 11:25:04.515 3503-3503/blackbean.rxjavademo D/yufen: Next: [0, 1, 2] And Thread: Thread[main,5,main]
10-25 11:25:04.515 3503-3503/blackbean.rxjavademo D/yufen: Next: [5, 6, 7]
10-25 11:25:04.515 3503-3503/blackbean.rxjavademo D/yufen: Next: [10, 11, 12]
10-25 11:25:04.515 3503-3503/blackbean.rxjavademo D/yufen: Next: [15, 16, 17]
10-25 11:25:04.515 3503-3503/blackbean.rxjavademo D/yufen: Next: [20, 21, 22]
10-25 11:25:04.515 3503-3503/blackbean.rxjavademo D/yufen: Sequence complete.
```
&emsp;&emsp;从这里可以看到， 每第n次发射缓存时， 该缓存是:  skip * (n-1), ... skip*(n-1) + count -1; 

###### a. Count < skip时

```java
	//...
	observable.buffer(5,3).subscribe(new Observer<List<Integer>>() {...}
	//....
```
输出Log:
```
10-25 11:13:51.645 25238-25238/blackbean.rxjavademo D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-25 11:13:51.645 25238-25238/blackbean.rxjavademo D/yufen: Next: [0, 1, 2, 3, 4] And Thread: Thread[main,5,main]
10-25 11:13:51.645 25238-25238/blackbean.rxjavademo D/yufen: Next: [3, 4, 5, 6, 7] 
10-25 11:13:51.645 25238-25238/blackbean.rxjavademo D/yufen: Next: [6, 7, 8, 9, 10]
10-25 11:13:51.645 25238-25238/blackbean.rxjavademo D/yufen: Next: [9, 10, 11, 12, 13] 
10-25 11:13:51.645 25238-25238/blackbean.rxjavademo D/yufen: Next: [12, 13, 14, 15, 16]
10-25 11:13:51.645 25238-25238/blackbean.rxjavademo D/yufen: Next: [15, 16, 17, 18, 19] 
10-25 11:13:51.645 25238-25238/blackbean.rxjavademo D/yufen: Next: [18, 19, 20, 21, 22]
10-25 11:13:51.645 25238-25238/blackbean.rxjavademo D/yufen: Next: [21, 22]
10-25 11:13:51.645 25238-25238/blackbean.rxjavademo D/yufen: Sequence complete.
```
##### 3.`buffer(boundary)`
&emsp;&emsp;`buffer(boundary)` 监视一个名叫`boundary`的Observable，每当这个`Observable`发射了一个值，它就创建一个新的`List`开始收集来自原始Observable的数据并发射原来的`List`。

Demo Code:
```java
		Observable.interval(250,TimeUnit.MILLISECONDS)//每隔250毫秒就递增地发送一个整数
                .buffer(Observable.interval(1000,TimeUnit.MILLISECONDS))//每隔1秒就发送一个值
                .subscribe(new Observer<List<Long>>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.d("yufen","onSubscribe..... And Cureent Thread: " + Thread.currentThread());
                    }
                    @Override
                    public void onNext(@NonNull List<Long> integers) {
                        if (i == 1) {
                            Log.d("yufen", "Next: " + String.valueOf(integers.toString()) + " And Thread: " + Thread.currentThread());
                            i++;
                        }else {
                            Log.d("yufen", "Next: " + String.valueOf(integers.toString()));
                        }
                    }
                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.d("yufen","Error: " + e.getMessage());
                    }
                    @Override
                    public void onComplete() {
                        Log.d("yufen","Sequence complete.");
                    }
        });
```


```
10-24 14:09:57.417 25253-25253/blackbean.rxjavademo D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-24 14:09:57.427 25253-25253/blackbean.rxjavademo D/yufen: onCreate End
10-24 14:09:58.427 25253-25621/blackbean.rxjavademo D/yufen: Next: [0, 1, 2, 3] And Thread: Thread[RxComputationThreadPool-1,5,main]
10-24 14:09:59.427 25253-25621/blackbean.rxjavademo D/yufen: Next: [4, 5, 6]
10-24 14:10:00.427 25253-25621/blackbean.rxjavademo D/yufen: Next: [7, 8, 9, 10]
10-24 14:10:01.427 25253-25621/blackbean.rxjavademo D/yufen: Next: [11, 12, 13, 14]
10-24 14:10:02.427 25253-25621/blackbean.rxjavademo D/yufen: Next: [15, 16, 17, 18]
10-24 14:10:03.427 25253-25621/blackbean.rxjavademo D/yufen: Next: [19, 20, 21, 22]
10-24 14:10:04.427 25253-25621/blackbean.rxjavademo D/yufen: Next: [23, 24, 25, 26]
10-24 14:10:05.427 25253-25621/blackbean.rxjavademo D/yufen: Next: [27, 28, 29, 30]
//....
```

##### 4.`buffer(bufferOpenings, bufferClosingSelector) `

```java
	<TOpening, TClosing> Observable<List<T>> buffer(
            ObservableSource<? extends TOpening> openingIndicator,
            Function<? super TOpening, ? extends ObservableSource<? extends TClosing>> closingIndicator)
```
&emsp;&emsp;`buffer(bufferOpenings, bufferClosingSelector)`监视这个叫`bufferOpenings`的`Observable`（它发射`BufferOpening`对象），每当`bufferOpenings`发射了一个数据时，它就创建一个新的`List`开始收集原始`Observable`的数据，并将`bufferOpenings`传递给`closingSelector`函数。这个函数返回一个`Observable`。 `buffer `监视这个`Observable`，当它检测到一个来自这个`Observable`的数据时，就关闭`List`并且发射它自己的数据（之前的那个`List`）。

Demo Code:
```java
        //每隔100毫秒就递增地发送一个整数
        Observable.interval(100,TimeUnit.MILLISECONDS)
                .take(20)//只发送20个数据
                .buffer(Observable.interval(600, TimeUnit.MILLISECONDS, Schedulers.newThread()),//每隔600毫秒发送一个Open信息
                        new Function<Long, ObservableSource<?>>() {
                            @Override
                            public ObservableSource<?> apply(@NonNull Long aLong) throws Exception {
                                return Observable.timer(500, TimeUnit.MILLISECONDS, Schedulers.newThread());//每隔500毫秒发送一个Close信息
                            }
                        }
                ).subscribe(new Observer<List<Long>>() {
                    Disposable disposable;
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.d("yufen","onSubscribe..... And Cureent Thread: " + Thread.currentThread());
                        disposable = d;
                    }
                    @Override
                    public void onNext(@NonNull List<Long> integers) {
                        //发射过来的数据为空时，主动发送完成并停止订阅
                        if (integers.isEmpty()) {
                            onComplete();
                            disposable.dispose();
                            return;
                        }
                        //打Log
                        if (i == 1) {
                            Log.d("yufen", "Next: " + String.valueOf(integers.toString()) + " And Thread: " + Thread.currentThread());
                            i++;
                        }else {
                            Log.d("yufen", "Next: " + String.valueOf(integers.toString()));
                        }
                    }
                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.d("yufen","Error: " + e.getMessage());
                    }
                    @Override
                    public void onComplete() {
                        Log.d("yufen","Sequence complete.");
                    }
        });
```

```
10-25 10:10:26.894 23375-23375/? D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-25 10:10:27.998 23375-23492/blackbean.rxjavademo D/yufen: Next: [5, 6, 7, 8, 9] And Thread: Thread[RxNewThreadScheduler-2,5,main]
10-25 10:10:28.596 23375-23554/blackbean.rxjavademo D/yufen: Next: [11, 12, 13, 14, 15]
10-25 10:10:29.197 23375-23644/blackbean.rxjavademo D/yufen: Next: [17, 18, 19]
10-25 10:10:29.201 23375-23644/blackbean.rxjavademo D/yufen: Sequence complete.
```

##### 4.`buffer(timespan, unit[, scheduler])`
&emsp;&emsp;`buffer(timespan, unit)`定期以`List`的形式发射新的数据，每个时间段，收集来自原始`Observable`的数据（从前面一个数据包裹之后，或者如果是第一个数据包裹，从有观察者订阅原来的`Observale`之后开始）。还有另一个版本的`buffer`接受一个`Scheduler`参数，默认情况下会使用`computation`调度器。

```
Observable<List<T>> buffer(long timespan, TimeUnit unit)
Observable<List<T>> buffer(long timespan, TimeUnit unit, Scheduler scheduler)
```

Demo Code:
```java
        //每隔100毫秒就递增地发送一个整数
        Observable.interval(100,TimeUnit.MILLISECONDS)
                .take(10)//只发送10个数据
                .buffer(300,TimeUnit.MILLISECONDS).subscribe(new Observer<List<Long>>() {...//同上}
        });

```

```
10-25 11:19:56.948 22705-22705/? D/yufen: onCreate Start
10-25 11:19:57.091 22705-22705/? D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-25 11:19:57.093 22705-22705/? D/yufen: onCreate End
10-25 11:19:57.594 22705-22747/blackbean.rxjavademo D/yufen: Next: [0, 1, 2, 3] And Thread: Thread[RxComputationThreadPool-1,5,main]
10-25 11:19:58.094 22705-22747/blackbean.rxjavademo D/yufen: Next: [4, 5, 6, 7, 8]
10-25 11:19:58.098 22705-22748/blackbean.rxjavademo D/yufen: Next: [9]
10-25 11:19:58.098 22705-22748/blackbean.rxjavademo D/yufen: Sequence complete.
```
注意：每个时间窗口发射的数据个数可能是不一样的。在某个时间窗口内，也可能没有数据发射。

##### 5.`buffer(timespan, unit[, scheduler])`
&emsp;&emsp;`buffer(timespan, unit)`定期以`List`的形式发射新的数据，每个时间段，收集来自原始`Observable`的数据（从前面一个数据包裹之后，或者如果是第一个数据包裹，从有观察者订阅原来的Observale之后开始）。还有另一个版本的`buffer`接受一个`Scheduler`参数，默认情况下会使用`computation`调度器。
简单来说：
（1）当达到缓冲的时间200毫秒的就发射数据不管缓冲个数有没有满 
（2）当达到缓冲的个数时就发射数据不管缓冲的时间有没有到

Demo Code:
```java
		//每隔100毫秒就递增地发送一个整数
        Observable.interval(100,TimeUnit.MILLISECONDS)
                .take(10)//只发送10个数据
                .buffer(200,TimeUnit.MILLISECONDS,2).subscribe(new Observer<List<Long>>() {...});

```

```
10-25 13:43:35.216 26518-26518/? D/yufen: onCreate Start
10-25 13:43:35.380 26518-26518/? D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-25 13:43:35.383 26518-26518/? D/yufen: onCreate End
10-25 13:43:35.581 26518-26566/blackbean.rxjavademo D/yufen: Next: [0] And Thread: Thread[RxComputationThreadPool-1,5,main]
10-25 13:43:35.684 26518-26567/blackbean.rxjavademo D/yufen: Next: [1, 2]
10-25 13:43:35.783 26518-26566/blackbean.rxjavademo D/yufen: Next: []
10-25 13:43:35.884 26518-26567/blackbean.rxjavademo D/yufen: Next: [3, 4]
10-25 13:43:35.982 26518-26566/blackbean.rxjavademo D/yufen: Next: []
10-25 13:43:36.084 26518-26567/blackbean.rxjavademo D/yufen: Next: [5, 6]
10-25 13:43:36.182 26518-26566/blackbean.rxjavademo D/yufen: Next: []
10-25 13:43:36.284 26518-26567/blackbean.rxjavademo D/yufen: Next: [7, 8]
10-25 13:43:36.382 26518-26566/blackbean.rxjavademo D/yufen: Next: []
10-25 13:43:36.385 26518-26567/blackbean.rxjavademo D/yufen: Next: [9]
10-25 13:43:36.387 26518-26567/blackbean.rxjavademo D/yufen: Sequence complete.
```

##### 5.`buffer(timespan, timeshift, unit[, scheduler])`
&emsp;&emsp;`buffer(timespan, timeshift, unit)`在每一个`timeshift`时期内都创建一个新的`List`,然后用原始`Observable`发射的每一项数据填充这个列表（在把这个`List` 当做自己的数据发射前，从创建时开始，直到过了 timespan  这么长的时间）
（1）当 timespan > timeshift 的时候，缓冲的数据重叠了 
（2）当 timespan < timeshift 的时候，缓冲的数据有可能丢失 
（3）当 timespan = timeshift 的时候，和前面看到的简单版本一样 

Demo Code:
```java
//每隔100毫秒就递增地发送一个整数
        Observable.interval(100,TimeUnit.MILLISECONDS)
                .take(10)//只发送10个数据
                //每隔300毫秒开启下一个缓冲，每个缓冲时间窗口是 250毫秒。所以两个缓冲之间会有 150毫秒的重叠。
                .buffer(250,300,TimeUnit.MILLISECONDS).subscribe(new Observer<List<Long>>() {...}
```

```
10-26 17:19:45.441 2787-2787/? D/yufen: onCreate Start
10-26 17:19:45.540 2787-2787/? D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-26 17:19:45.542 2787-2787/? D/yufen: onCreate End
10-26 17:19:45.791 2787-2849/? D/yufen: Next: [0, 1] And Thread: Thread[RxComputationThreadPool-1,5,main]
10-26 17:19:46.091 2787-2849/blackbean.rxjavademo D/yufen: Next: [2, 3, 4]
10-26 17:19:46.391 2787-2849/blackbean.rxjavademo D/yufen: Next: [5, 6, 7]
10-26 17:19:46.544 2787-2850/blackbean.rxjavademo D/yufen: Next: [8, 9]
10-26 17:19:46.545 2787-2850/blackbean.rxjavademo D/yufen: Sequence complete.
```

#### 操作符------FlatMap
&emsp;&emsp;FlatMap  将一个发射数据的Observable变换为多个Observables，然后将它们发射的数据合并后放进一个单独的Observable
&emsp;&emsp;FlatMap  操作符使用一个指定的函数对原始Observable发射的每一项数据执行变换操作，这个函数返回一个本身也发射数据的Observable，然后 FlatMap  合并这些Observables发射的数据，最后将合并后的结果当做它自己的数据序列发射。
&emsp;&emsp;这个方法是很有用的，例如，当你有一个这样的Observable：它发射一个数据序列，这些数据本身包含Observable成员或者可以变换为Observable，因此你可以创建一个新的Observable发射这些次级Observable发射的数据的完整集合。
注意： FlatMap  对这些Observables发射的数据做的是合并( merge  )操作，因此它们可能是交错的。有一个操作符不会让变换后的Observables发射的数据交错，它按照严格的顺序发射这些数据，这个操作符通常被叫作 ConcatMap  或者类似的名字。
注意：如果任何一个通过这个 flatMap  操作产生的单独的Observable调用 onError  异常终止了，这个Observable自身会立即调用 onError  并终止。

##### flatMap(Func1))
Demo Code: 

```java
			Observable.interval(100,TimeUnit.MILLISECONDS)
                .take(10)//只发送10个数据
                .flatMap(new Function<Long, ObservableSource<String>>() {
                    @Override
                    public ObservableSource<String> apply(@NonNull Long aLong) throws Exception {
                        return Observable.just(String.valueOf(aLong) + "AAA");//一个发射数据的Observable变换为多个Observables
                    }
                }).subscribe(new Observer<String>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.d("yufen","onSubscribe..... And Cureent Thread: " + Thread.currentThread());w
                    }

                    @Override
                    public void onNext(@NonNull String o) {
                        //打Log
                        if (i == 1) {
                            Log.d("yufen", "Next: " + o + " And Thread: " + Thread.currentThread());
                            i++;
                        }else {
                            Log.d("yufen", "Next: " + o );
                        }
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.d("yufen","Error: " + e.getMessage());
                    }

                    @Override
                    public void onComplete() {
                        Log.d("yufen","Sequence complete.");
                    }
            });
```

```
11-08 17:13:29.779 1788-1788/blackbean.rxjavademo D/yufen: onCreate Start
11-08 17:13:29.866 1788-1788/blackbean.rxjavademo D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
11-08 17:13:29.869 1788-1788/blackbean.rxjavademo D/yufen: onCreate End
11-08 17:13:29.969 1788-1960/blackbean.rxjavademo D/yufen: Next: 0AAA And Thread: Thread[RxComputationThreadPool-1,5,main]
11-08 17:13:30.068 1788-1960/blackbean.rxjavademo D/yufen: Next: 1AAA
11-08 17:13:30.168 1788-1960/blackbean.rxjavademo D/yufen: Next: 2AAA
11-08 17:13:30.268 1788-1960/blackbean.rxjavademo D/yufen: Next: 3AAA
11-08 17:13:30.368 1788-1960/blackbean.rxjavademo D/yufen: Next: 4AAA
11-08 17:13:30.468 1788-1960/blackbean.rxjavademo D/yufen: Next: 5AAA
11-08 17:13:30.568 1788-1960/blackbean.rxjavademo D/yufen: Next: 6AAA
11-08 17:13:30.668 1788-1960/blackbean.rxjavademo D/yufen: Next: 7AAA
11-08 17:13:30.768 1788-1960/blackbean.rxjavademo D/yufen: Next: 8AAA
11-08 17:13:30.868 1788-1960/blackbean.rxjavademo D/yufen: Next: 9AAA //这里不一定是按顺序的
11-08 17:13:30.869 1788-1960/blackbean.rxjavademo D/yufen: Sequence complete.

```
##### flatMap(Func1,int))
&emsp;&emsp;可接受额外的 int 参数的一个变体。这个参数设置 flatMap  从原来的Observable映射Observables的最大同时订阅数。当达到这个限制时，它会等待其中一个终止然后再订阅另一个。

##### flatMap(Func1,Func1,Func0)) /  flatMap(Func1,Func1,Func0,int))
&emsp;&emsp;这版本的 flatMap  为原始Observable的每一项数据和每一个通知创建一个新的Observable（并对数据平坦化）。

##### flatMap(Func1,Func2))/ flatMap(Func1,Func2,int))
&emsp;&emsp;这版本的 flatMap  会使用原始Observable的数据触发的Observable组合这些数据，然后发射这些数据组合。它也有一个接受额外 int  参数的版本。

#### 操作符------flatMapIterable
&emsp;&emsp;flatMapIterable 是flatMap的变体，这变体则是成对的打包数据，然后生成Iterable而不是原始数据和生成的Observables，但是处理方式是相同的。
Demo Code:

```java
Observable.interval(100,TimeUnit.MILLISECONDS)
                .take(10)//只发送10个数据
                .flatMapIterable(new Function<Long, Iterable<String>>() {
                    @Override
                    public Iterable<String> apply(@NonNull Long aLong) throws Exception {
                        ArrayList<String> xx = new ArrayList<>();
                        xx.add("NNNNNNNNN");
                        return xx;
                    }
                }).subscribe(new Observer<String>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.d("yufen","onSubscribe..... And Cureent Thread: " + Thread.currentThread());
                    }

                    @Override
                    public void onNext(@NonNull String o) {
                        //打Log
                        if (i == 1) {
                            Log.d("yufen", "Next: " + o + " And Thread: " + Thread.currentThread());
                            i++;
                        }else {
                            Log.d("yufen", "Next: " + o );
                        }
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.d("yufen","Error: " + e.getMessage());
                    }

                    @Override
                    public void onComplete() {
                        Log.d("yufen","Sequence complete.");
                    }
                });
```

```
11-08 18:10:04.948 6839-6839/? D/yufen: onCreate Start
11-08 18:10:05.016 6839-6839/? D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
11-08 18:10:05.018 6839-6839/? D/yufen: onCreate End
11-08 18:10:05.118 6839-6894/? D/yufen: Next: NNNNNNNNN And Thread: Thread[RxComputationThreadPool-1,5,main]
11-08 18:10:05.218 6839-6894/? D/yufen: Next: NNNNNNNNN
11-08 18:10:05.319 6839-6894/? D/yufen: Next: NNNNNNNNN
11-08 18:10:05.418 6839-6894/? D/yufen: Next: NNNNNNNNN
11-08 18:10:05.518 6839-6894/? D/yufen: Next: NNNNNNNNN
11-08 18:10:05.618 6839-6894/? D/yufen: Next: NNNNNNNNN
11-08 18:10:05.718 6839-6894/blackbean.rxjavademo D/yufen: Next: NNNNNNNNN
11-08 18:10:05.818 6839-6894/blackbean.rxjavademo D/yufen: Next: NNNNNNNNN
11-08 18:10:05.918 6839-6894/blackbean.rxjavademo D/yufen: Next: NNNNNNNNN
11-08 18:10:06.018 6839-6894/blackbean.rxjavademo D/yufen: Next: NNNNNNNNN
11-08 18:10:06.018 6839-6894/blackbean.rxjavademo D/yufen: Sequence complete.
```

#### 操作符------concatMap
&emsp;&emsp;与flatmap的功能一样，唯一不同的是它按次序连接而不是合并那些生成的Observables，然后产生自己的数据序列。

Demo code:
```java
Observable.interval(100,TimeUnit.MILLISECONDS)
                .take(10)//只发送10个数据
                .concatMap(new Function<Long, ObservableSource<String>>() {
                    @Override
                    public ObservableSource<String> apply(@NonNull Long aLong) throws Exception {
                        rreturn Observable.just(String.valueOf(aLong) + "AAA", String.valueOf(aLong) + "BBB", String.valueOf(aLong) + "CCC");
                    }
                }).subscribe(new Observer<String>() {...}
```

```
11-27 16:08:31.577 9111-9111/? D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
11-27 16:08:31.683 9111-9178/blackbean.rxjavademo D/yufen: Next: 0AAA And Thread: Thread[RxComputationThreadPool-1,5,main]
11-27 16:08:31.683 9111-9178/blackbean.rxjavademo D/yufen: Next: 0BBB
11-27 16:08:31.683 9111-9178/blackbean.rxjavademo D/yufen: Next: 0CCC
11-27 16:08:31.782 9111-9178/blackbean.rxjavademo D/yufen: Next: 1AAA
11-27 16:08:31.782 9111-9178/blackbean.rxjavademo D/yufen: Next: 1BBB
11-27 16:08:31.782 9111-9178/blackbean.rxjavademo D/yufen: Next: 1CCC
11-27 16:08:31.882 9111-9178/blackbean.rxjavademo D/yufen: Next: 2AAA
11-27 16:08:31.882 9111-9178/blackbean.rxjavademo D/yufen: Next: 2BBB
11-27 16:08:31.882 9111-9178/blackbean.rxjavademo D/yufen: Next: 2CCC
11-27 16:08:31.982 9111-9178/blackbean.rxjavademo D/yufen: Next: 3AAA
11-27 16:08:31.982 9111-9178/blackbean.rxjavademo D/yufen: Next: 3BBB
11-27 16:08:31.982 9111-9178/blackbean.rxjavademo D/yufen: Next: 3CCC
11-27 16:08:32.082 9111-9178/blackbean.rxjavademo D/yufen: Next: 4AAA
11-27 16:08:32.082 9111-9178/blackbean.rxjavademo D/yufen: Next: 4BBB
11-27 16:08:32.082 9111-9178/blackbean.rxjavademo D/yufen: Next: 4CCC
11-27 16:08:32.182 9111-9178/blackbean.rxjavademo D/yufen: Next: 5AAA
11-27 16:08:32.182 9111-9178/blackbean.rxjavademo D/yufen: Next: 5BBB
11-27 16:08:32.182 9111-9178/blackbean.rxjavademo D/yufen: Next: 5CCC
11-27 16:08:32.282 9111-9178/blackbean.rxjavademo D/yufen: Next: 6AAA
11-27 16:08:32.282 9111-9178/blackbean.rxjavademo D/yufen: Next: 6BBB
11-27 16:08:32.282 9111-9178/blackbean.rxjavademo D/yufen: Next: 6CCC
11-27 16:08:32.382 9111-9178/blackbean.rxjavademo D/yufen: Next: 7AAA
11-27 16:08:32.382 9111-9178/blackbean.rxjavademo D/yufen: Next: 7BBB
11-27 16:08:32.382 9111-9178/blackbean.rxjavademo D/yufen: Next: 7CCC
11-27 16:08:32.482 9111-9178/blackbean.rxjavademo D/yufen: Next: 8AAA
11-27 16:08:32.482 9111-9178/blackbean.rxjavademo D/yufen: Next: 8BBB
11-27 16:08:32.482 9111-9178/blackbean.rxjavademo D/yufen: Next: 8CCC
11-27 16:08:32.582 9111-9178/blackbean.rxjavademo D/yufen: Next: 9AAA
11-27 16:08:32.582 9111-9178/blackbean.rxjavademo D/yufen: Next: 9BBB
11-27 16:08:32.582 9111-9178/blackbean.rxjavademo D/yufen: Next: 9CCC
11-27 16:08:32.582 9111-9178/blackbean.rxjavademo D/yufen: Sequence complete.
```
#### 操作符------switchMap
&emsp;&emsp;它和 flatMap  很像，除了一点：当原始Observable发射一个新的数据（Observable）时，如果之前发射的数据，还没消化完，它也会将取消订阅并停止监视产生执之前那个数据的Observable，直接开始监视当前这一个新的。
例子（略）

####操作符------GroupBy
&emsp;&emsp;将一个Observable分拆为一些Observables集合，它们中的每一个发射原始Observable的一个子序列；
图：
&emsp;&emsp;GroupBy  操作符将原始Observable分拆为一些Observables集合，它们中的每一个发射原始Observable数据序列的一个子序列。哪个数据项由哪一个Observable发射是由一个函数判定的，这个函数给每一项指定一个Key，Key相同的数据会被同一个Observable发射。
&emsp;&emsp;RxJava实现了 groupBy  操作符。它返回Observable的一个特殊子类 GroupedObservable  ，实现了 GroupedObservable  接口的对象有一个额外的方法 getKey  ，这个Key用于将数据分组到指定的Observable。

Demo Code: 
```java
		Observable.interval(100,TimeUnit.MILLISECONDS)
                .take(10)//只发送10个数据
                .groupBy(new Function<Long, String>() {

                    @Override
                    public String apply(Long aLong) throws Exception {
                        return aLong % 2 == 1? "AAA":"BBB";
                    }
                }).subscribe(new Observer<GroupedObservable<String, Long>>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        Log.d("yufen","onSubscribe..... And Cureent Thread: " + Thread.currentThread());
                    }

                    @Override
                    public void onNext(GroupedObservable<String, Long> stringLongGroupedObservable) {
                        Log.d("yufen","onNext is called, Key = " +  stringLongGroupedObservable.getKey());
                        String key = stringLongGroupedObservable.getKey();
                        switch (key) {
                            case "AAA":
                                stringLongGroupedObservable.subscribe(new Consumer<Long>() {
                                    @Override
                                    public void accept(Long aLong) throws Exception {
                                        Log.d("yufen","accept:  " + String.valueOf(aLong) + " [I am Good.]");
                                    }
                                });
                                break;
                            case "BBB":
                                stringLongGroupedObservable.subscribe(new Consumer<Long>() {
                                    @Override
                                    public void accept(Long aLong) throws Exception {
                                        Log.d("yufen","accept:  " + String.valueOf(aLong) + " [I try to best.]");
                                    }
                                });
                                break;
                        }
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.d("yufen","Error: " + e.getMessage());
                    }

                    @Override
                    public void onComplete() {
                        Log.d("yufen","Sequence complete.");
                    }
        });
```

```
11-28 19:48:19.194 20808-20808/blackbean.rxjavademo D/yufen: onCreate Start
11-28 19:48:19.498 20808-20808/blackbean.rxjavademo D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
11-28 19:48:19.602 20808-20856/blackbean.rxjavademo D/yufen: onNext is called
11-28 19:48:19.603 20808-20856/blackbean.rxjavademo D/yufen: accept:  0 [I try to best.]
11-28 19:48:19.699 20808-20856/blackbean.rxjavademo D/yufen: onNext is called
11-28 19:48:19.700 20808-20856/blackbean.rxjavademo D/yufen: accept:  1 [I am Good.]
11-28 19:48:19.799 20808-20856/blackbean.rxjavademo D/yufen: accept:  2 [I try to best.]
11-28 19:48:19.899 20808-20856/blackbean.rxjavademo D/yufen: accept:  3 [I am Good.]
11-28 19:48:19.999 20808-20856/blackbean.rxjavademo D/yufen: accept:  4 [I try to best.]
11-28 19:48:20.099 20808-20856/blackbean.rxjavademo D/yufen: accept:  5 [I am Good.]
11-28 19:48:20.199 20808-20856/blackbean.rxjavademo D/yufen: accept:  6 [I try to best.]
11-28 19:48:20.299 20808-20856/blackbean.rxjavademo D/yufen: accept:  7 [I am Good.]
11-28 19:48:20.399 20808-20856/blackbean.rxjavademo D/yufen: accept:  8 [I try to best.]
11-28 19:48:20.499 20808-20856/blackbean.rxjavademo D/yufen: accept:  9 [I am Good.]
11-28 19:48:20.500 20808-20856/blackbean.rxjavademo D/yufen: Sequence complete.
```

&emsp;&emsp;注意： groupBy  将原始Observable分解为一个发射多个 GroupedObservable  的Observable，一旦有订阅，每个 GroupedObservable  就开始缓存数据。因此，如果你忽略这些 GroupedObservable 中的任何一个，这个缓存可能形成一个潜在的内存泄露。因此，如果你不想观察，也不要忽略 GroupedObservable  。你应该使用像 take(0)  这样会丢弃自己的缓存的操作符。
&emsp;&emsp;如果你取消订阅一个 GroupedObservable  ，那个Observable将会终止。如果之后原始的Observable又发射了一个与这个Observable的Key匹配的数据， groupBy  将会为这个Key创建一个新的 GroupedObservable  。
&emsp;&emsp;groupBy  默认不在任何特定的调度器上执行。


#### 操作符------Map
&emsp;&emsp;对Observable发射的每一项数据应用一个函数，执行变换操作
图：
&emsp;&emsp;Map  操作符对原始Observable发射的每一项数据应用一个你选择的函数，然后返回一个发射这些结果的Observable。
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;
&emsp;&emsp;