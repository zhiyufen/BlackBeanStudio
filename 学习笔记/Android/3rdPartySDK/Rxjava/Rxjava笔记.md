# [RxJava]操作符之创建操作

用于创建Observable的操作符

 - Create  — 通过调用观察者的方法从头创建一个Observable
 - Defer  — 在观察者订阅之前不创建这个Observable，为每一个观察者创建一个新的Observable
 - Empty/Never/Throw  — 创建行为受限的特殊Observable
 - From  — 将其它的对象或数据结构转换为Observable
 - Interval  — 创建一个定时发射整数序列的Observable
 - Just  — 将对象或者对象集合转换为一个会发射这些对象的Observable
 - Range  — 创建发射指定范围的整数序列的Observable
 - Repeat  — 创建重复发射特定的数据或数据序列的Observable
 - Start  — 创建发射一个函数的返回值的Observable
 - Timer  — 创建在一个指定的延迟之后发射单个数据的Observable

#### 操作符------Create

&emsp;&emsp;使用 Create  操作符从头开始创建一个Observable，给这个操作符传递一个接受观察者
作为参数的函数，编写这个函数让它的行为表现为一个Observable--恰当的调用观察者的
onSubscribe，onNext，onError和onComplete方法。
&emsp;&emsp;一个形式正确的有限Observable必须尝试调用观察者的onComplete正好一次或者它的
onError正好一次，而且此后不能再调用观察者的任何其它方法。
例子：
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> observer) throws Exception {
                Log.d("blackbean","Subscribe start");
                try {
                    if (!observer.isDisposed()) {
                        for (int i = 1; i < 5; i++) {
                            observer.onNext(i);
                        }
                        observer.onComplete();
                    }
                } catch (Exception e) {
                    observer.onError(e);
                }
            }
        }).subscribe(new Observer<Integer>(){
            @Override
            public void onComplete() {
                Log.d("blackbean","Sequence complete.");
            }

            @Override
            public void onNext(Integer item) {
                Log.d("blackbean","Next: " + item);
            }
            @Override
            public void onError(Throwable error) {
                Log.d("blackbean","Error: " + error.getMessage());
            }

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("blackbean","onSubscribe.....");
            }
        });
```
输出：
```
08-29 14:34:42.126 9610-9610/blackbean.rxjavademo D/blackbean: onSubscribe.....//注意该调用顺序
08-29 14:34:42.126 9610-9610/blackbean.rxjavademo D/blackbean: Subscribe start
08-29 14:34:42.126 9610-9610/blackbean.rxjavademo D/blackbean: Next: 1
08-29 14:34:42.126 9610-9610/blackbean.rxjavademo D/blackbean: Next: 2
08-29 14:34:42.126 9610-9610/blackbean.rxjavademo D/blackbean: Next: 3
08-29 14:34:42.126 9610-9610/blackbean.rxjavademo D/blackbean: Next: 4
08-29 14:34:42.126 9610-9610/blackbean.rxjavademo D/blackbean: Sequence complete.
```

注： create  方法默认不在任何特定的调度器上执行。

####操作符------Defer
&emsp;&emsp;Defer  操作符会一直等待直到有观察者订阅它，然后它使用Observable工厂方法生成一个
Observable。它对每个观察者都这样做，因此尽管每个订阅者都以为自己订阅的是同一个
Observable，事实上每个订阅者获取的是它们自己的单独的数据序列。
&emsp;&emsp;在某些情况下，等待直到最后一分钟（就是知道订阅发生时）才生成Observable可以确保
Observable包含最新的数据。

我们先看看在平常使用无法获取最新的数据的情况：
先定义一下内部数据类：
```
class Data {
        String mString;

        public void setValue(String value) {
            mString = value;
        }
        public String getValue() {
            return mString;
        }
    }
```
```java
        final Data data = new Data();
        data.setValue("Init");
        Observable observable =  Observable.just(data.getValue());
        data.setValue("Hello word!");
        observable.subscribe(new Observer<String>(){
            @Override
            public void onComplete() {
                Log.d("blackbean","Sequence complete.");
            }

            @Override
            public void onNext(String item) {
                Log.d("blackbean","Next: " + item);
            }
            @Override
            public void onError(Throwable error) {
                Log.d("blackbean","Error: " + error.getMessage());
            }

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("blackbean","onSubscribe.....");
            }
        });
```
输出：
```
08-29 17:05:52.066 30886-30886/blackbean.rxjavademo D/blackbean: onSubscribe.....
08-29 17:05:52.066 30886-30886/blackbean.rxjavademo D/blackbean: Next: Init
08-29 17:05:52.066 30886-30886/blackbean.rxjavademo D/blackbean: Sequence complete.
```
我们看到输出的Log，并没有最新的数据“Hello word!", 


```java
		final Data data = new Data();
        data.setValue("Init");
        Observable observable =  Observable.defer(new Callable<ObservableSource<String>>() {
            @Override
            public ObservableSource<String> call() throws Exception {
	            //进行订阅后，才调用返回相应新Observable对象,
	            //因此每个订阅者获取的是它们自己的单独的数据序列。
                Log.d("blackbean","call.....data.getValue() = "+ data.getValue());
                return Observable.just(data.getValue());
            }
        });
        data.setValue("Hello word!");
        observable.subscribe(new Observer<String>(){
            @Override
            public void onComplete() {
                Log.d("blackbean","Sequence complete.");
            }

            @Override
            public void onNext(String item) {
                Log.d("blackbean","Next: " + item);
            }
            @Override
            public void onError(Throwable error) {
                Log.d("blackbean","Error: " + error.getMessage());
            }

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("blackbean","onSubscribe.....");
            }
        });
```

输出：
```
01-19 00:28:15.400 8568-8568/blackbean.rxjavademo D/blackbean: call.....data.getValue() = Hello word!
01-19 00:28:15.400 8568-8568/blackbean.rxjavademo D/blackbean: onSubscribe.....
01-19 00:28:15.400 8568-8568/blackbean.rxjavademo D/blackbean: Next: Hello word!
01-19 00:28:15.400 8568-8568/blackbean.rxjavademo D/blackbean: Sequence complete.
```
可以看到使用Defer订阅时， 才重新获取最新Observable对象，从而获取最新的数据“Hello word!”。当然要实现这个效果， 上面的操作符Creat的实现方法也是一样的。

#### 操作符------Empty/Never/Throw

Empty
创建一个不发射任何数据但是正常终止的Observable
Never
创建一个不发射数据也不终止的Observable
Throw
创建一个不发射数据以一个错误终止的Observable

&emsp;&emsp;这三个操作符生成的Observable行为非常特殊和受限。测试的时候很有用，有时候也用于结合其它的Observables，或者作为其它需要Observable的操作符的参数。
&emsp;&emsp;RxJava将这些操作符实现为  empty  ， never  和 error  。 error  操作符需要一
个 Throwable  参数，你的Observable会以此终止。这些操作符默认不在任何特定的调度器上执行，但是 empty  和 error  有一个可选参数是Scheduler，如果你传递了Scheduler参数，它们会在这个调度器上发送通知。

Demo:
```java
		//TODO: Empty/Never/Throw 操作符
        Observer<Object> xxx =  new Observer<Object>(){
            @Override
            public void onComplete() {
                Log.d("blackbean","Sequence complete.");
            }

            @Override
            public void onNext(@NonNull Object o) {
                Log.d("blackbean","Next: " + o);
            }

            @Override
            public void onError(Throwable error) {
                Log.d("blackbean","Error: " + error.getMessage());
            }

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("blackbean","onSubscribe.....");
            }
        };
        Log.d("blackbean","----------------Empty-------------");
        Observable.empty().subscribe(xxx);
        Log.d("blackbean","----------------Never-------------");
        Observable.never().subscribe(xxx);
        Log.d("blackbean","----------------Throw-------------");
        Observable.error(new Exception("blackbean")).subscribe(xxx);
```

输出：
```
01-19 03:47:37.471 5842-5842/blackbean.rxjavademo D/blackbean: ----------------Empty-------------
01-19 03:47:37.471 5842-5842/blackbean.rxjavademo D/blackbean: onSubscribe.....
01-19 03:47:37.471 5842-5842/blackbean.rxjavademo D/blackbean: Sequence complete.
01-19 03:47:37.471 5842-5842/blackbean.rxjavademo D/blackbean: ----------------Never-------------
01-19 03:47:37.471 5842-5842/blackbean.rxjavademo D/blackbean: onSubscribe.....
01-19 03:47:37.471 5842-5842/blackbean.rxjavademo D/blackbean: ----------------Throw-------------
01-19 03:47:37.481 5842-5842/blackbean.rxjavademo D/blackbean: onSubscribe.....
01-19 03:47:37.481 5842-5842/blackbean.rxjavademo D/blackbean: Error: blackbean
```

#### 操作符------From
&emsp;&emsp;From操作符将其它种类的对象和数据类型转换为Observable
&emsp;&emsp;当你使用Observable时，如果你要处理的数据都可以转换成展现为Observables，而不是需要混合使用Observables和其它类型的数据，会非常方便。这让你在数据流的整个生命周期中，可以使用一组统一的操作符来管理它们。
&emsp;&emsp;例如，Iterable可以看成是同步的Observable；Future，可以看成是总是只发射单个数据的Observable。通过显式地将那些数据转换为Observables，你可以像使用Observable一样与它们交互。
&emsp;&emsp;在RxJava中， from  操作符可以转换Future、Iterable和数组。对于Iterable和数组，产生的Observable会发射Iterable或数组的每一项数据。

#####1. 数组类型
Demo:
```java
        //TODO: From 操作符---数组
        Integer[] items = { 0, 1, 2, 3, 4, 5 };
        Observable myObservable = Observable.fromArray(items);
        myObservable.subscribe(new Observer<Integer>(){
            @Override
            public void onComplete() {
                Log.d("blackbean","Sequence complete.");
            }

            @Override
            public void onNext(@NonNull Integer o) {
                Log.d("blackbean","Next: " + o);
            }

            @Override
            public void onError(Throwable error) {
                Log.d("blackbean","Error: " + error.getMessage());
            }

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("blackbean","onSubscribe.....");
            }
        });

```
输出Log:
```
01-19 04:31:42.871 5949-5949/blackbean.rxjavademo D/blackbean: onSubscribe.....
01-19 04:31:42.871 5949-5949/blackbean.rxjavademo D/blackbean: Next: 0
01-19 04:31:42.871 5949-5949/blackbean.rxjavademo D/blackbean: Next: 1
01-19 04:31:42.871 5949-5949/blackbean.rxjavademo D/blackbean: Next: 2
01-19 04:31:42.871 5949-5949/blackbean.rxjavademo D/blackbean: Next: 3
01-19 04:31:42.871 5949-5949/blackbean.rxjavademo D/blackbean: Next: 4
01-19 04:31:42.871 5949-5949/blackbean.rxjavademo D/blackbean: Next: 5
01-19 04:31:42.871 5949-5949/blackbean.rxjavademo D/blackbean: Sequence complete.
```

#####2. Iterable类型
Demo:
```java
        //TODO: From 操作符---Iterable
        String[] items = { "Zero","One", "Two","Three","Four","Five" };
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            list.add( items[i]);
        }
        Observable myObservable = Observable.fromIterable(list);
        myObservable.subscribe(new Observer<String>(){
            @Override
            public void onComplete() {
                Log.d("blackbean","Sequence complete.");
            }

            @Override
            public void onNext(@NonNull String o) {
                Log.d("blackbean","Next: " + o);
            }

            @Override
            public void onError(Throwable error) {
                Log.d("blackbean","Error: " + error.getMessage());
            }

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("blackbean","onSubscribe.....");
            }
        });
```
输出Log:
```
01-19 05:09:24.791 12872-12872/blackbean.rxjavademo D/blackbean: onSubscribe.....
01-19 05:09:24.791 12872-12872/blackbean.rxjavademo D/blackbean: Next: Zero
01-19 05:09:24.791 12872-12872/blackbean.rxjavademo D/blackbean: Next: One
01-19 05:09:24.791 12872-12872/blackbean.rxjavademo D/blackbean: Next: Two
01-19 05:09:24.791 12872-12872/blackbean.rxjavademo D/blackbean: Next: Three
01-19 05:09:24.791 12872-12872/blackbean.rxjavademo D/blackbean: Next: Four
01-19 05:09:24.791 12872-12872/blackbean.rxjavademo D/blackbean: Sequence complete.
```

##### 3.Callable 类型
Demo:

```java
       //TODO: From 操作符---Iterable
        Observable myObservable = Observable.fromCallable(new Callable() {
            @Override
            public Data call() throws Exception {
                Data data = new Data();
                data.setValue("Hello Word!");
                return data;
            }
        });
        myObservable.subscribe(new Observer<Data>(){
            @Override
            public void onComplete() {
                Log.d("blackbean","Sequence complete.");
            }

            @Override
            public void onNext(@NonNull Data o) {
                Log.d("blackbean","Next: " + o.getValue());
            }

            @Override
            public void onError(Throwable error) {
                Log.d("blackbean","Error: " + error.getMessage());
            }

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("blackbean","onSubscribe.....");
            }
        });

	//自定义数据
    class Data {
        String mString;

        public void setValue(String value) {
            mString = value;
        }
        public String getValue() {
            return mString;
        }
    }
```

输出Log:
```
01-19 05:30:26.501 18230-18230/blackbean.rxjavademo D/blackbean: onSubscribe.....
01-19 05:30:26.501 18230-18230/blackbean.rxjavademo D/blackbean: Next: Hello Word!
01-19 05:30:26.501 18230-18230/blackbean.rxjavademo D/blackbean: Sequence complete.
```

##### 4. Future类型
&emsp;&emsp;对于Future，它会发射Future.get()方法返回的单个数据。 from  方法有一个可接受两个可选参数的版本，分别指定超时时长和时间单位。如果过了指定的时长Future还没有返回一个值，这个Observable会发射错误通知并终止。
注：对于Future不太懂的同学，可参考

&emsp;&emsp;针对这个操作符，RxJava2有相应几个方法：
```
<T> Observable<T> fromFuture(Future<? extends T> future)
<T> Observable<T> fromFuture(Future<? extends T> future, Scheduler scheduler)
<T> Observable<T> fromFuture(Future<? extends T> future, long timeout, TimeUnit unit) 
<T> Observable<T> fromFuture(Future<? extends T> future, long timeout, TimeUnit unit, Scheduler scheduler) 
```
&emsp;&emsp;from  默认不在任何特定的调度器上执行。然而你可以将Scheduler作为可选的第二个参数传递给Observable，它会在那个调度器上管理这个Future。
Demo:
```java
Future<Data> future = new Future<Data>() {
            @Override
            public boolean cancel(boolean mayInterruptIfRunning) {
                return false;
            }

            @Override
            public boolean isCancelled() {
                return false;
            }

            @Override
            public boolean isDone() {
                return false;
            }

            @Override
            public Data get() throws InterruptedException, ExecutionException {
                Log.d("blackbean","call.....");
                Data data = new Data();
                data.setValue("Hello Word!");
                return data;
            }

            @Override
            public Data get(long timeout, @android.support.annotation.NonNull TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException {
                Log.d("blackbean","call.....Have timeout arg");
                Data data = new Data();
                data.setValue("Hello Word!");
                return data;
            }
        };
        //Observable myObservable = Observable.fromFuture(future);
        Observable myObservable = Observable.fromFuture(future,3, TimeUnit.SECONDS, Schedulers.io());
        myObservable.subscribe(new Observer<Data>(){
            @Override
            public void onComplete() {
                Log.d("blackbean","Sequence complete.");
            }

            @Override
            public void onNext(@NonNull Data o) {
                Log.d("blackbean","Next: " + o.getValue());
            }

            @Override
            public void onError(Throwable error) {
                Log.d("blackbean","Error: " + error.getMessage());
            }

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("blackbean","onSubscribe.....");
            }
        });
```
输出的Log:
```
10-14 15:03:16.016 31612-31612/blackbean.rxjavademo D/blackbean: onSubscribe.....
10-14 15:03:16.021 31612-4151/blackbean.rxjavademo D/blackbean: call.....Have timeout arg
10-14 15:03:16.022 31612-4151/blackbean.rxjavademo D/blackbean: Next: Hello Word!
10-14 15:03:16.022 31612-4151/blackbean.rxjavademo D/blackbean: Sequence complete.
```
#### 操作符------Decode
&emsp;&emsp;`StringObservable`  类不是默认RxJava的一部分，包含一个 decode  操作符，这个操作符将一个多字节字符流转换为一个发射字节数组的Observable，这些字节数组按照字符的边界划分。
&emsp;&emsp;要想使用`StringObservable`  类，需要加入rxjava-string.

```
dependencies {
	....
    compile 'io.reactivex:rxjava-string:1.1.1'
}
```
Demo:
```java
		InputStream is = null;
        AssetManager assetManager = getApplicationContext().getAssets();
        Observable<String> stringStringObservable = null;
        try {
            if (assetManager != null ) {
                is = assetManager.open("temp.txt",AssetManager.ACCESS_BUFFER);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        if (is != null) {
            Observable<byte[]> xx = StringObservable.from(is);
            stringStringObservable =  StringObservable.decode(xx,"UTF-8");
        }
        if (stringStringObservable != null ){
            stringStringObservable.subscribe(new rx.Observer<String>() {
                @Override
                public void onCompleted() {
                    Log.d("blackbean","Sequence complete.");
                }

                @Override
                public void onError(Throwable e) {
                    Log.d("blackbean","Error: " + e.getMessage());
                }

                @Override
                public void onNext(String s) {
                    Log.d("blackbean","Next: " + s);
                }
            });
        }
```
#### 操作符------Interval/Range
&emsp;&emsp;Interval  操作符返回一个Observable，它按固定的时间间隔发射一个无限递增的整数序列。
&emsp;&emsp;Interval  默认在 computation  调度器上执行。你也可以传递一个可选的Scheduler参数来指定调度器。
相关方法：

```java
//initialDelay:初始延迟时间；period:发射周期；unit: 发射周期的时间单位；scheduler:特定的调度器
Observable<Long> interval(long initialDelay, long period, TimeUnit unit)
Observable<Long> interval(long period, TimeUnit unit) 
Observable<Long> interval(long period, TimeUnit unit, Scheduler scheduler)
Observable<Long> interval(long initialDelay, long period, TimeUnit unit, Scheduler scheduler)
```

```java
		//// 延迟 1s，间隔 500ms，发送无限增长的 Long 型数列
		Observable.interval(1, 500, TimeUnit.MILLISECONDS).subscribe(new Observer<Long>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("yufen","onSubscribe.....");
            }

            @Override
            public void onNext(@NonNull Long aLong) {
                Log.d("yufen","Next: " + String.valueOf(aLong));
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

输出的Log：

```
10-16 13:35:39.184 10530-10530/blackbean.rxjavademo D/yufen: onSubscribe.....
10-16 13:35:39.190 10530-10569/blackbean.rxjavademo D/yufen: Next: 0
10-16 13:35:39.690 10530-10569/blackbean.rxjavademo D/yufen: Next: 1
10-16 13:35:40.189 10530-10569/blackbean.rxjavademo D/yufen: Next: 2
10-16 13:35:40.690 10530-10569/blackbean.rxjavademo D/yufen: Next: 3
//.....
```
&emsp;&emsp;Range操作符发射一个范围内的有序整数序列，你可以指定范围的起始和长度。
```java
//start:范围起始值；count:范围的根据的数目；initialDelay:初始延迟时间；period:发射周期；unit: 发射周期的时间单位；scheduler:特定的调度器
Observable<Long> intervalRange(long start, long count, long initialDelay, long period, TimeUnit unit)
Observable<Long> intervalRange(long start, long count, long initialDelay, long period, TimeUnit unit, Scheduler scheduler)
```
Demo:
```java
        Observable.intervalRange(100, 5, 1, 500, TimeUnit.MILLISECONDS).subscribe(new Observer<Long>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("yufen","onSubscribe..... And Thread: " + Thread.currentThread());
            }

            @Override
            public void onNext(@NonNull Long aLong) {
                Log.d("yufen","Next: " + String.valueOf(aLong));
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
输出Log：
```
10-16 15:58:06.864 28508-28508/blackbean.rxjavademo D/yufen: onSubscribe..... And Thread: Thread[main,5,main]
10-16 15:58:06.870 28508-28578/blackbean.rxjavademo D/yufen: Next: 100
10-16 15:58:07.370 28508-28578/blackbean.rxjavademo D/yufen: Next: 101
10-16 15:58:07.870 28508-28578/blackbean.rxjavademo D/yufen: Next: 102
10-16 15:58:08.370 28508-28578/blackbean.rxjavademo D/yufen: Next: 103
10-16 15:58:08.870 28508-28578/blackbean.rxjavademo D/yufen: Next: 104
10-16 15:58:08.870 28508-28578/blackbean.rxjavademo D/yufen: Sequence complete.
```
#### 操作符------Just
&emsp;&emsp;Just将单个数据转换为发射那个数据的Observable。
&emsp;&emsp;Just类似于From，但是From会将数组或Iterable的素具取出然后逐个发射，而Just只是简单的原样发射，将数组或Iterable当做单个数据。其实Just的实现 

&emsp;&emsp;注意：如果你传递 null  给Just，它会返回一个发射 null  值的Observable。不要误认为它会返回一个空Observable（完全不发射任何数据的Observable），如果需要空Observable你应该使用Empty操作符。
&emsp;&emsp;RxJava将这个操作符实现为 just  函数，它接受一至九个参数，返回一个按参数列表顺序发射这些数据的Observable。
Demo:
```java
		Observable.just(1, 2, 3).subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("yufen","onSubscribe..... And Thread: " + Thread.currentThread());
            }

            @Override
            public void onNext(@NonNull Integer aLong) {
                Log.d("yufen","Next: " + String.valueOf(aLong));
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
输出Log：
```
10-16 17:01:03.484 16407-16407/blackbean.rxjavademo D/yufen: onSubscribe..... And Thread: Thread[main,5,main]
10-16 17:01:03.484 16407-16407/blackbean.rxjavademo D/yufen: Next: 1
10-16 17:01:03.484 16407-16407/blackbean.rxjavademo D/yufen: Next: 2
10-16 17:01:03.484 16407-16407/blackbean.rxjavademo D/yufen: Next: 3
10-16 17:01:03.484 16407-16407/blackbean.rxjavademo D/yufen: Sequence complete.
```
#### 操作符------Repeat
&emsp;&emsp;创建一个发射特定数据重复多次的Observable.

&emsp;&emsp;Repeat重复地发射数据。某些实现允许你重复的发射某个数据序列，还有一些允许你限制重复的次数。

&emsp;&emsp;RxJava将这个操作符实现为 repeat  方法。它不是创建一个Observable，而是重复发射原始Observable的数据序列，这个序列或者是无限的，或者通过 repeat(n)  指定重复次数。

相关方法：
```java
Observable<T> repeat() //会无限重复发射数据
Observable<T> repeat(long times) //times 次数
```

Demo Code:
```java
        Observable.just(1, 2, 3).repeat(3).subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("yufen","onSubscribe..... And Thread: " + Thread.currentThread());
            }

            @Override
            public void onNext(@NonNull Integer aLong) {
                Log.d("yufen","Next: " + String.valueOf(aLong));
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
输出的Log:
```
10-13 10:04:51.507 31012-31012/blackbean.rxjavademo D/yufen: onSubscribe..... And Thread: Thread[main,5,main] //表示调用在当前
10-13 10:04:51.508 31012-31012/blackbean.rxjavademo D/yufen: Next: 1
10-13 10:04:51.508 31012-31012/blackbean.rxjavademo D/yufen: Next: 2
10-13 10:04:51.508 31012-31012/blackbean.rxjavademo D/yufen: Next: 3
10-13 10:04:51.508 31012-31012/blackbean.rxjavademo D/yufen: Next: 1
10-13 10:04:51.508 31012-31012/blackbean.rxjavademo D/yufen: Next: 2
10-13 10:04:51.508 31012-31012/blackbean.rxjavademo D/yufen: Next: 3
10-13 10:04:51.508 31012-31012/blackbean.rxjavademo D/yufen: Next: 1
10-13 10:04:51.508 31012-31012/blackbean.rxjavademo D/yufen: Next: 2
10-13 10:04:51.508 31012-31012/blackbean.rxjavademo D/yufen: Next: 3
10-13 10:04:51.508 31012-31012/blackbean.rxjavademo D/yufen: Sequence complete.
```
#### 操作符------RepeatWhen
&emsp;&emsp;它不是缓存和重放原始Observable的数据序列，而是有条件的重新订阅和发射原来的Observable。
&emsp;&emsp;将原始Observable的终止通知（完成或错误）当做一个 void  数据传递给一个通知处理器，它以此来决定是否要重新订阅和发射原来的Observable。这个通知处理器就像一个Observable操作符，接受一个发射 void 通知的Observable为输入，返回一个发射 void  数据（意思是，重新订阅和发射原始Observable）或者直接终止（意思是，使用 repeatWhen  终止发射数据）的Observable。
&emsp;&emsp;也就是Function<Observable<Object>, ObservableSource<?>>()的apply()接口方法相当于一个判断接口方法： 进行判断是否需要进行订阅（重复或其它操作）或直接终止
Demo Code:

```java
		static int i = 0;
        Observable.just(1, 2, 3).repeatWhen(new Function<Observable<Object>, ObservableSource<?>>() {
            @Override
            public ObservableSource<?> apply(@NonNull Observable<Object> objectObservable) throws Exception {
                Log.d("yufen","repeatWhen apply.......... i =" + i);
                //return objectObservable;//会无限重复发射数据
				//return objectObservable.timer(3, TimeUnit.SECONDS);//会每隔3秒地无限重复发射数据
                if (i < 6)
                    return objectObservable.timer(3, TimeUnit.SECONDS);//
                else if (i == 6)
                    return Observable.empty();
                else
                    return Observable.error(new Exception("yufen"));
            }
        }).subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d("yufen","onSubscribe..... And Cureent Thread: " + Thread.currentThread());
            }

            @Override
            public void onNext(@NonNull Integer aLong) {
                Log.d("yufen","Next: " + String.valueOf(aLong));// + " And Thread: " + Thread.currentThread());
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
10-18 19:57:09.084 9309-9309/? D/yufen: repeatWhen apply.......... i =2
10-18 19:57:09.089 9309-9309/? D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-18 19:57:09.093 9309-9309/? D/yufen: Next: 1
10-18 19:57:09.093 9309-9309/? D/yufen: Next: 2
10-18 19:57:09.093 9309-9309/? D/yufen: Next: 3
10-18 19:57:09.093 9309-9309/? D/yufen: onCreate End
10-18 19:57:12.093 9309-9352/blackbean.rxjavademo D/yufen: Next: 1
10-18 19:57:12.093 9309-9352/blackbean.rxjavademo D/yufen: Next: 2
10-18 19:57:12.093 9309-9352/blackbean.rxjavademo D/yufen: Next: 3
10-18 19:57:12.095 9309-9352/blackbean.rxjavademo D/yufen: Sequence complete.
---------------------------------------------------------------------

10-18 19:58:05.796 11017-11017/blackbean.rxjavademo D/yufen: repeatWhen apply.......... i =6
10-18 19:58:05.796 11017-11017/blackbean.rxjavademo D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-18 19:58:05.797 11017-11017/blackbean.rxjavademo D/yufen: Sequence complete.
--------------------------------------------------------

10-18 19:58:45.905 12294-12294/? D/yufen: repeatWhen apply.......... i =7
10-18 19:58:45.907 12294-12294/? D/yufen: onSubscribe..... And Cureent Thread: Thread[main,5,main]
10-18 19:58:45.908 12294-12294/? D/yufen: Error: yufen

```
#### 操作符------doWhile/whileDo
&emsp;&emsp;doWhile  在原始序列的每次重复后检查某个条件，如果满足条件才重复发射。
&emsp;&emsp;whileDo  属于可选包 rxjava-computation-expressions  ，不是RxJava标准操作符的一部分。 whileDo  在原始序列的每次重复前检查某个条件，如果满足条件才重复发射。
&emsp;&emsp;不过目前这个Rxjava2似乎不适用。

#### 操作符------Start/toAsync/startFuture/deferFuture/fromAction/fromCallable/fromRunnable/forEachFuture/runAsync
&emsp;&emsp;这些操作符是属于可选的 rxjava-async  模块， 关于functions, futures,actions, callables, runnables 等。这组操作符可以让它们表现得像Observable，因此它们可以在Observables调用链中与其它Observable搭配使用。这里不作详细介绍，后面用到再学习