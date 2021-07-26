## Glide 使用笔记

[TOC]

Glide 是比较流行的图片加载库，官方文档： http://bumptech.github.io/glide/

### 1. Glide V4的配置

Gradle:

```groovy
dependencies {
    compile 'com.github.bumptech.glide:glide:4.11.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
}
```

权限：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="your.package.name"

    <uses-permission android:name="android.permission.INTERNET"/>
    <!--
    Allows Glide to monitor connectivity status and restart failed requests if users go from a
    a disconnected to a connected network state.
    -->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

    <application>
      ...
    </application>
</manifest>
```

更多配置请参考官网[Setup](http://bumptech.github.io/glide/doc/download-setup.html),  如SDK要求，混淆，Kotlin支持等；

### 2. 基本使用

#### 基本用法

加载图片：

```java
Glide.with(fragment)
    .load(myUrl)
    .into(imageView);
```

取消加载：

```java
Glide.with(fragment).clear(imageView);
```

尽管及时取消不必要的加载是很好的实践，但这并不是必须的操作。实际上，当 [`Glide.with()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/Glide.html#with-android.app.Fragment-) 中传入的 Activity 或 Fragment 实例销毁时，Glide 会自动取消加载并回收资源。

#### 在 Application 模块中的使用

在 Application 模块中，可创建一个添加有 `@GlideModule` 注解，继承自 `AppGlideModule` 的类。此类可生成出一个流式 API，内联了多种选项，和集成库中自定义的选项：

```java
package com.example.myapp;

import com.bumptech.glide.annotation.GlideModule;
import com.bumptech.glide.module.AppGlideModule;

@GlideModule
public final class MyAppGlideModule extends AppGlideModule {}
```

生成的 API 默认名为 `GlideApp` ，与 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 的子类包名相同。在 Application 模块中将 `Glide.with()` 替换为 `GlideApp.with()`，即可使用该 API 去完成加载工作。

```java
GlideApp.with(fragment)
   .load(myUrl)
   .placeholder(placeholder)
   .fitCenter()
   .into(imageView);
```

更多Generated API，请参考Generated API章节；

#### 定制请求

Glide 提供了许多可应用于单一请求的选项，包括变换、过渡、缓存选项等。默认选项可以直接应用于请求上：

```java
Glide.with(fragment)
  .load(myUrl)
  .placeholder(placeholder)
  .fitCenter()
  .into(imageView);
```

选项也可以通过 `RequestOptions` 类来在多个请求之间共享：

```java
RequestOptions sharedOptions = 
    new RequestOptions()
      .placeholder(placeholder)
      .fitCenter();

Glide.with(fragment)
  .load(myUrl)
  .apply(sharedOptions)
  .into(imageView1);

Glide.with(fragment)
  .load(myUrl)
  .apply(sharedOptions)
  .into(imageView2);
```

#### 在 ListView 和 RecyclerView 中的使用

在 ListView 或 RecyclerView 中加载图片的代码和在单独的 View 中加载完全一样。Glide 已经自动处理了 View 的复用和请求的取消：

```java
@Override
public void onBindViewHolder(ViewHolder holder, int position) {
    String url = urls.get(position);
    Glide.with(fragment)
        .load(url)
        .into(holder.imageView);
}
```

对 url 进行 null 检验并不是必须的，如果 url 为 null，Glide 会清空 View 的内容，或者显示 [placeholder Drawable](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#placeholder-int-) 或 [fallback Drawable](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#fallback-int-) 的内容。

Glide 唯一的要求是，对于任何可复用的 `View` 或 [`Target`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/target/Target.html) ，如果它们在之前的位置上，用 Glide 进行过加载操作，那么在新的位置上要去执行一个新的加载操作或调用 [`clear()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-) API 停止 Glide 的工作。

```java
@Override
public void onBindViewHolder(ViewHolder holder, int position) {
    if (isImagePosition(position)) {
        String url = urls.get(position);
        Glide.with(fragment)
            .load(url)
            .into(holder.imageView);
    } else {
        Glide.with(fragment).clear(holder.imageView);
        holder.imageView.setImageDrawable(specialDrawable);
    }
}
```

对 `View` 调用 [`clear()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-) 或 `into(View)`，表明在此之前的加载操作会被取消，并且在方法调用完成后，但Glide 不会改变 view 的内容。

你为一个 view 设置好了一个 `Drawable`，但该 view 在之前的位置上使用 Glide 进行过加载图片的操作； 如果你忘记调用 [`clear()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)，而又没有开启新的加载操作，那么就会出现这种情况：Glide 加载完毕后可能会将这个 view 改回成原来的内容。

#### 非 View 目标

除了将 `Bitmap` 和 `Drawable` 加载到 `View` 之外，你也可以开始异步加载到你的自定义 `Target` 中：

```java
Glide.with(context
  .load(url)
  .into(new CustomTarget<Drawable>() {
    @Override
    public void onResourceReady(Drawable resource, Transition<Drawable> transition) {
      // Do something with the Drawable here.
    }

    @Override
    public void onLoadCleared(@Nullable Drawable placeholder) {
      // Remove the Drawable provided in onResourceReady from any Views and ensure 
      // no references to it remain.
    }
  });
```

使用自定义 `Target` 有一些陷阱，所以请务必阅读 [目标文档页](https://muyangmin.github.io/glide-docs-cn/doc/targets.html) 的详细内容。

#### 后台线程

在后台线程加载图片也是直接使用 [`submit(int, int)`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestBuilder.html#submit-int-int-)：

```java
FutureTarget<Bitmap> futureTarget =
  Glide.with(context)
    .asBitmap()
    .load(url)
    .submit(width, height);

Bitmap bitmap = futureTarget.get();

// Do something with the Bitmap and then when you're done with it:
Glide.with(context).clear(futureTarget);
```

如果你不想让 `Bitmap` 和 `Drawable` 自身在后台线程中，你也可以使用和前台线程一样的方式来开始异步加载：

```java
Glide.with(context)
  .asBitmap()
  .load(url)
  .into(new Target<Bitmap>() {
    ...
  });
```

### 3. Generated API

Glide v4 使用 [注解处理器 (Annotation Processor)](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/Processor.html) 来生成出一个 API，它允许应用扩展 Glide 的 API并包含各种集成库提供的组件。

Generated API 模式的设计出于以下两个目的：

1. 集成库可以为 Generated API 扩展自定义选项。
2. 在 Application 模块中可将常用的选项组打包成一个选项在 Generated API 中使用

虽然以上所说的工作均可以通过手动创建 [`RequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html) 子类的方式来完成，但想将它用好更具有挑战，并且降低了 API 使用的流畅性。

更多使用及配置注意事项，请参考：https://muyangmin.github.io/glide-docs-cn/doc/generatedapi.html

#### 使用 Generated API

Generated API 默认名为 `GlideApp` ，与 Application 模块中 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)的子类包名相同。在 Application 模块中将 `Glide.with()` 替换为 `GlideApp.with()`，即可使用该 API 去完成加载工作：

```java
GlideApp.with(fragment)
   .load(myUrl)
   .placeholder(R.drawable.placeholder)
   .fitCenter()
   .into(imageView);
```

与 `Glide.with()` 不同，诸如 `fitCenter()` 和 `placeholder()` 等选项在 Builder 中直接可用，并不需要再传入单独的 [`RequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html) 对象。

#### GlideExtension

Glide Generated API 可在 Application 和 Library 中被扩展。扩展使用被注解的静态方法来添加新的选项、修改现有选项、甚至添加额外的类型支持。

[`@GlideExtension`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html) 注解用于标识一个扩展 Glide API 的类。任何扩展 Glide API 的类都必须使用这个注解来标记，否则其中被注解的方法就会被忽略。

被 [`@GlideExtension`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html) 注解的类应以工具类的思维编写。这种类应该有一个私有的、空的构造方法，应为 final 类型，并且仅包含静态方法。被注解的类可以含有静态变量，可以引用其他的类或对象。

在 Application 模块中可以根据需求实现任意多个被 [`@GlideExtension`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html) 注解的类，在 Library 模块中同样如此。当 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 被发现时，所有有效的 [Glide 扩展类](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html) 会被合并，所有的选项在 API 中均可以被调用。合并冲突会导致 Glide 的 Annotation Processor 抛出编译错误。

被 `@GlideExtention` 注解的类有两种扩展方式：

1. [`GlideOption`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideOption.html) - 为 [`RequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html) 添加一个自定义的选项。
2. [`GlideType`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideType.html) - 添加对新的资源类型的支持(GIF，SVG 等等)。

#### GlideOption

用 [`@GlideOption`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideOption.html) 注解的静态方法用于扩展 [`RequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html) 。`GlideOption` 可以：

1. 定义一个在 Application 模块中频繁使用的选项集合。
2. 创建新的选项，通常与 Glide 的 [`Option`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Option.html) 类一起使用。

要定义一个选项集合，你可以这么写：

```java
@GlideExtension
public class MyAppExtension {
  // Size of mini thumb in pixels.
  private static final int MINI_THUMB_SIZE = 100;

  private MyAppExtension() { } // utility class

  @NonNull
  @GlideOption
  public static BaseRequestOptions<?> miniThumb(BaseRequestOptions<?> options) {
    return options
      .fitCenter()
      .override(MINI_THUMB_SIZE);
  }
```

这将会在 [`RequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html) 的子类中生成一个方法，类似这样：

```java
public class GlideOptions extends RequestOptions {
  
  public GlideOptions miniThumb() {
    return (GlideOptions) MyAppExtension.miniThumb(this);
  }

  ...
}
```

你可以为方法任意添加参数，但要保证第一个参数为 [`RequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html)。

```java
@GlideOption
public static BaseRequestOptions<?> miniThumb(BaseRequestOptions<?> options, int size) {
  return options
    .fitCenter()
    .override(size);
}
```

在自动生成的方法中新添的参数同样被加了进来：

```java
public GlideOptions miniThumb(int size) {
  return (GlideOptions) MyAppExtension.miniThumb(this);
}
```

之后你就可以使用生成的 `GlideApp` 类调用你的自定义方法：

```java
GlideApp.with(fragment)
   .load(url)
   .miniThumb(thumbnailSize)
   .into(imageView);
```

使用 `@GlideOption` 标记的方法应该为静态方法，并且返回值为 `BaseRequestOptions`。请注意，这些生成的方法在标准的 `Glide` 和 `RequestOptions` 类里不可用，只存在于生成的等效类中。

#### GlideType

被 [`@GlideType`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideType.html) 注解的静态方法用于扩展 [`RequestManager`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestManager.html) 。被 `@GlideType` 注解的方法允许你添加对新的资源类型的支持，包括指定默认选项。

例如，为添加对 GIF 的支持，你可以添加一个被 `@GlideType` 注解的方法：

```java
@GlideExtension
public class MyAppExtension {
  private static final RequestOptions DECODE_TYPE_GIF = decodeTypeOf(GifDrawable.class).lock();

  @NonNull
  @GlideType(GifDrawable.class)
  public static RequestBuilder<GifDrwable> asGif(RequestBuilder<GifDrawable> requestBuilder) {
    return requestBuilder
      .transition(new DrawableTransitionOptions())
      .apply(DECODE_TYPE_GIF);
  }
}
```

这样会生成一个包含对应方法的 [`RequestManager`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestManager.html) ：

```java
public class GlideRequests extends RequesetManager {

  public GlideRequest<GifDrawable> asGif() {
    return (GlideRequest<GifDrawable> MyAppExtension.asGif(this.as(GifDrawable.class));
  }
  
  ...
}
```

之后你可以使用生成的 `GlideApp` 类调用你的自定义类型：

```java
GlideApp.with(fragment)
  .asGif()
  .load(url)
  .into(imageView);
```

被 `@GlideType` 标记的方法必须使用 [`RequestBuilder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html) 作为其第一个参数，这里的泛型  对应 [`@GlideType`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideType.html) 注解中传入的类。该方法应为静态方法，且返回值为 RequestBuilder 。方法必须定义在一个被 [`@GlideExtension`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html) 注解标记的类中。

### 4. 占位符

Glide允许用户指定三种不同类型的占位符，分别在三种不同场景使用：

- [placeholder](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#placeholder-int-)
- [error](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#error-int-)
- [fallback](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#fallback-int-)

#### 占位符(Placeholder)

占位符是当请求正在执行时被展示的 Drawable 。当请求成功完成时，占位符会被请求到的资源替换。如果被请求的资源是从内存中加载出来的，那么占位符可能根本不会被显示。如果请求失败并且没有设置 `error Drawable` ，则占位符将被持续展示。类似地，如果请求的url/model为 `null` ，并且 `error Drawable` 和 `fallback` 都没有设置，那么占位符也会继续显示。

```java
Glide.with(fragment)
  .load(url)
  .placeholder(R.drawable.placeholder)
  .into(view);
```

Or:

```java
Glide.with(fragment)
  .load(url)
  .placeholder(new ColorDrawable(Color.BLACK))
  .into(view);
```

#### 错误符(Error)

`error Drawable` 在请求永久性失败时展示。`error Drawable` 同样也在请求的url/model为 `null` ，且并没有设置 `fallback Drawable` 时展示。

```java
Glide.with(fragment)
  .load(url)
  .error(R.drawable.error)
  .into(view);
```

Or:

```java
Glide.with(fragment)
  .load(url)
  .error(new ColorDrawable(Color.RED))
  .into(view);
```

#### 后备回调符(Fallback)

`fallback Drawable` 在请求的url/model为 `null` 时展示。设计 `fallback Drawable` 的主要目的是允许用户指示 `null` 是否为可接受的正常情况。例如，一个 `null` 的个人资料 url 可能暗示这个用户没有设置头像，因此应该使用默认头像。然而，`null` 也可能表明这个元数据根本就是不合法的，或者取不到。 默认情况下Glide将 `null` 作为错误处理，所以可以接受 `null` 的应用应当显式地设置一个 `fallback Drawable` 。

使用 [generated API][4]：

```java
Glide.with(fragment)
  .load(url)
  .fallback(R.drawable.fallback)
  .into(view);
```

Or:

```java
Glide.with(fragment)
  .load(url)
  .fallback(new ColorDrawable(Color.GREY))
  .into(view);
```

### 5. 选项

#### 请求选项

Glide中的大部分设置项都可以直接应用在 `Glide.with()` 返回的 [`RequestBuilder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html) 对象上。

可用的选项包括（但不限于）：

- 占位符(`Placeholders`)
- 转换(`Transformations`)
- 缓存策略(`Caching Strategies`)
- 组件特有的设置项，例如编码质量，或`Bitmap`的解码配置等。

例如，要应用一个 [`CenterCrop`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/CenterCrop.html) [转换](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Transformation.html)，你可以使用以下代码：

```java
Glide.with(fragment)
    .load(url)
    .centerCrop()
    .into(imageView);
```

#### RequestOptions

如果你想让你的应用的不同部分之间共享相同的加载选项，你也可以初始化一个新的 `RequestOptions` 对象，并在每次加载时通过 [`apply()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#apply-com.bumptech.glide.request.RequestOptions-) 方法传入这个对象：

```java
RequestOptions cropOptions = new RequestOptions().centerCrop(context);
...
Glide.with(fragment)
    .load(url)
    .apply(cropOptions)
    .into(imageView);
```

[`apply()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#apply-com.bumptech.glide.request.RequestOptions-) 方法可以被调用多次，因此 `RequestOption` 可以被组合使用。如果 `RequestOptions` 对象之间存在相互冲突的设置，那么只有最后一个被应用的 `RequestOptions` 会生效。

#### 过渡选项

[TransitionOptions](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/TransitionOptions.html) 用于决定你的加载完成时会发生什么。

使用 `TransitionOption` 可以应用以下变换：

- View淡入
- 与占位符交叉淡入
- 或者什么都不发生

如果不使用变换，你的图像将会“跳入”其显示位置，直接替换掉之前的图像。为了避免这种突然的改变，你可以淡入view，或者让多个Drawable交叉淡入，而这些都需要使用`TransitionOptions`完成。

例如，要应用一个交叉淡入变换：

```java
import static com.bumptech.glide.load.resource.drawable.DrawableTransitionOptions.withCrossFade;

Glide.with(fragment)
    .load(url)
    .transition(withCrossFade())
    .into(view);
```

不同于`RequestOptions`，`TransitionOptions`是特定资源类型独有的，你能使用的变换取决于你让Glide加载哪种类型的资源。

这样的结果是，假如你请求加载一个 `Bitmap` ，你需要使用 [BitmapTransitionOptions](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/BitmapTransitionOptions.html) ，而不是 [DrawableTransitionOptions](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/drawable/DrawableTransitionOptions.html) 。同样，当你请求加载 `Bitmap` 时，你只需要做简单的淡入，而不需要做复杂的交叉淡入。

#### RequestBuilder

[RequestBuilder](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html) 是Glide中请求的骨架，负责携带请求的url和你的设置项来开始一个新的加载过程。

使用 `RequestBuilder` 可以指定：

- 你想加载的资源类型(Bitmap, Drawable, 或其他)
- 你要加载的资源地址(url/model)
- 你想最终加载到的View
- 任何你想应用的（一个或多个）`RequestOption` 对象
- 任何你想应用的（一个或多个）`TransitionOption` 对象
- 任何你想加载的缩略图 [`thumbnail()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)

要构造一个 `RequestBuilder` 对象，你可以通过先调用 `Glide.with()`然后再调用某一个 `as` 方法来完成：

```java
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).asDrawable();
```

或先调用 `Glide.with()` 然后 `load()`：

```java
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).load(url);
```

##### 选择资源类型

`RequestBuilders` 是特定于它们将要加载的资源类型的。默认情况下你会得到一个 Drawable RequestBuilder ，但你可以使用 `as...` 系列方法来改变请求类型。例如，如果你调用了 `asBitmap()` ，你就将获得一个 `Bitmap``RequestBuilder` 对象，而不是默认的 Drawable RequestBuilder。

```java
RequestBuilder<Bitmap> requestBuilder = Glide.with(fragment).asBitmap();
```

##### 应用 RequestOptions

前面提到，可以使用 [`apply()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#apply-com.bumptech.glide.request.RequestOptions-) 方法应用 `RequestOptions` ，使用 [`transition()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#transition-com.bumptech.glide.TransitionOptions-) 方法应用 `TransitionOptions` 。

```java
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).asDrawable();
requestBuilder.apply(requestOptions);
requestBuilder.transition(transitionOptions);
```

RequestBuilder 也可以被复用于开始多个请求：

```java
RequestBuilder<Drawable> requestBuilder =
        Glide.with(fragment)
            .asDrawable()
            .apply(requestOptions);

for (int i = 0; i < numViews; i++) {
   ImageView view = viewGroup.getChildAt(i);
   String url = urls.get(i);
   requestBuilder.load(url).into(view);
}
```

##### 缩略图 (Thumbnail) 请求

Glide 的 [`thumbnail()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-) API 允许你指定一个 [`RequestBuilder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html) 以与你的主请求并行启动。[`thumbnail()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-) 会在主请求加载过程中展示。如果主请求在缩略图请求之前完成，则缩略图请求中的图像将不会被展示。[`thumbnail()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-) API 允许你简单快速地加载图像的低分辨率版本，并且同时加载图像的无损版本，这可以减少用户盯着加载指示器 *【例如进度条–译者注】* 的时间。

[`thumbnail()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-) API 对本地和远程图片都适用，尤其是当低分辨率缩略图存在于 Glide 的磁盘缓存时，它们将很快被加载出来。

[`thumbnail()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-) API 使用起来相对简单：

```java
Glide.with(fragment)
  .load(url)
  .thumbnail(Glide.with(fragment)
    .load(thumbnailUrl))
  .into(imageView);
```

只要你的 `thumbnailUrl` 指向的图片比你的主 `url` 的分辨率更低，它将会很好地工作。相当数量的加载 API 提供了不同的指定图片尺寸的方法，它们尤其适用于 [`thumbnail()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-) API。

如果你仅仅想加载一个本地图像，或者你只有一个单独的远程 URL， 你仍然可以从缩略图 API 受益。请使用 Glide 的 [`override`](https://muyangmin.github.io/glide-docs-cn/javadocs/430/com/bumptech/glide/request/RequestOptions.html#override-int-int-) 或 [`sizeMultiplier`](https://muyangmin.github.io/glide-docs-cn/javadocs/420/com/bumptech/glide/request/RequestOptions.html#sizeMultiplier-float-) API 来强制 Glide 在缩略图请求中加载一个低分辨率图像：

```java
int thumbnailSize = ...;
Glide.with(fragment)
  .load(localUri)
  .thumbnail(Glide.with(fragment)
    .load(localUri)
    .override(thumbnailSize))
  .into(view);
```

[`thumbnail()`](https://muyangmin.github.io/glide-docs-cn/javadocs/430/com/bumptech/glide/RequestBuilder.html#thumbnail-float-) 方法有一个简化版本，它只需要一个 `sizeMultiplier` 参数。如果你只是想为你的加载相同的图片，但尺寸为 `View` 或 `Target` 的某个百分比的话特别有用：

```java
Glide.with(fragment)
  .load(localUri)
  .thumbnail(/*sizeMultiplier=*/ 0.25f)
  .into(imageView);
```

##### 在失败时开始新的请求

从 Glide 4.3.0 开始，你现在可以使用 [`error`](https://muyangmin.github.io/glide-docs-cn/javadocs/430/com/bumptech/glide/RequestBuilder.html#error-com.bumptech.glide.RequestBuilder-) API 来指定一个 [`RequestBuilder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html)，以在主请求失败时开始一次新的加载。例如，在请求 `primaryUrl` 失败后加载 `fallbackUrl`：

```java
Glide.with(fragment)
  .load(primaryUrl)
  .error(Glide.with(fragment)
      .load(fallbackUrl))
  .into(imageView);
```

如果主请求成功完成，这个[error](https://muyangmin.github.io/glide-docs-cn/javadocs/430/com/bumptech/glide/RequestBuilder.html#error-com.bumptech.glide.RequestBuilder-) [`RequestBuilder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html) 将不会被启动。如果你同时指定了一个 [`thumbnail()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-) 和一个 [`error()`](https://muyangmin.github.io/glide-docs-cn/javadocs/430/com/bumptech/glide/RequestBuilder.html#error-com.bumptech.glide.RequestBuilder-) [`RequestBuilder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html)，则这个后备的 [`RequestBuilder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html) 将在主请求失败时启动，即使缩略图请求成功也是如此。

#### 组件选项

[`Option`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Option.html) 类是给Glide的组件添加参数的通用办法，包括 [`ModelLoaders`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/model/ModelLoader.html) , [`ResourceDecoders`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/ResourceDecoder.html) , [`ResourceEncoders`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/ResourceEncoder.html) , [`Encoders`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Encoder.html) 等等。一些Glide的内置组件提供了设置项，自定义的组件也可以添加设置项。

`Option` 通过 `RequestOptions` 类应用到请求上：

```java
Glide.with(context)
  .load(url)
  .option(MyCustomModelLoader.TIMEOUT_MS, 1000L)
  .into(imageView);
```

你也可以创建一个全新的 RequestOptions 对象：

```java
RequestOptions options = new RequestOptions()
  .set(MyCustomModelLoader.TIMEOUT_MS, 1000L);

Glide.with(context)
  .load(url)
  .apply(options)
  .into(imageView);
```

### 6. 变换

在Glide中，[Transformations](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Transformation.html) 可以获取资源并修改它，然后返回被修改后的资源。通常变换操作是用来完成剪裁或对位图应用过滤器，但它也可以用于转换GIF动画，甚至自定义的资源类型。

#### 内置类型

Glide 提供了很多内置的变换，包括：

- [CenterCrop](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/CenterCrop.html)
- [FitCenter](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/FitCenter.html)
- [CircleCrop](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/CircleCrop.html)

#### 应用

通过 [RequestOptions](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html) 类可以应用变换：

##### 默认变换

```java
Glide.with(fragment)
  .load(url)
  .fitCenter()
  .into(imageView);
```

或使用 `RequestOptions` ：

```java
RequestOptions options = new RequestOptions();
options.centerCrop();

Glide.with(fragment)
    .load(url)
    .apply(options)
    .into(imageView);
```

#### 多重变换

默认情况下，每个 [`transform()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#transform-java.lang.Class-com.bumptech.glide.load.Transformation-) 调用，或任何特定转换方法(`fitCenter()`, `centerCrop()`, `bitmapTransform()` etc)的调用都会替换掉之前的变换。

如果你想在单次加载中应用多个变换，请使用 [`MultiTransformation`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/MultiTransformation.html) 类，或其快捷方法 [`.transforms()`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/request/RequestOptions.html#transforms-com.bumptech.glide.load.Transformation...-) 。

```java
Glide.with(fragment)
  .load(url)
  .transform(new MultiTransformation(new FitCenter(), new YourCustomTransformation())
  .into(imageView);
```

或使用快捷方法：

```java
Glide.with(fragment)
  .load(url)
  .transform(new FitCenter(), new YourCustomTransformation())
  .into(imageView);
```

请注意，你向 [`MultiTransformation`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/MultiTransformation.html) 的构造器传入变换参数的顺序，决定了这些变换的应用顺序。

#### 定制变换

尽管 Glide 提供了各种各样的内置 [`Transformation`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Transformation.html) 实现，如果你需要额外的功能，你也可以实现你自己的 [`Transformation`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/FitCenter.html)。

如果你只需要变换 `Bitmap`，最好是从继承 [`BitmapTransformation`](https://muyangmin.github.io/glide-docs-cn/javadocs/440/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html) 开始。`BitmapTransformation` 为我们处理了一些基础的东西，例如，如果你的变换返回了一个新修改的 Bitmap ，`BitmapTransformation`将负责提取和回收原始的 Bitmap。

一个简单的实现看起来可能像这样：

```java
public class FillSpace extends BitmapTransformation {
    private static final String ID = "com.bumptech.glide.transformations.FillSpace";
    private static final String ID_BYTES = ID.getBytes(STRING_CHARSET_NAME);

    @Override
    public Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        if (toTransform.getWidth() == outWidth && toTransform.getHeight() == outHeight) {
            return toTransform;
        }

        return Bitmap.createScaledBitmap(toTransform, outWidth, outHeight, /*filter=*/ true);
    }

    @Override
    public void equals(Object o) {
      return o instanceof FillSpace;
    }

    @Override
    public int hashCode() {
      return ID.hashCode();
    }

    @Override
    public void updateDiskCacheKey(MessageDigest messageDigest)
        throws UnsupportedEncodingException {
      messageDigest.update(ID_BYTES);
    }
}
```

尽管你的 `Transformation` 将几乎确定比这个示例更复杂，但它应该包含了相同的基本元素和复写方法。

##### 必需的方法

请特别注意，对于任何 `Transformation` 子类，包括 `BitmapTransformation`，你都有三个方法你 **必须** 实现它们，以使得磁盘和内存缓存正确地工作：

1. `equals()`
2. `hashCode()`
3. `updateDiskCacheKey`

如果你的 [`Transformation`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Transformation.html) 没有参数，通常使用一个包含完整包限定名的 `static` `final` `String` 来作为一个 ID，它可以构成 `hashCode()` 的基础，并可用于更新 `updateDiskCacheKey()` 传入的 `MessageDigest`。如果你的 [`Transformation`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Transformation.html) 需要参数而且它会影响到 `Bitmap` 被变换的方式，它们也必须被包含到这三个方法中。

例如，Glide 的 [`RoundedCorners`](https://muyangmin.github.io/glide-docs-cn/javadocs/440/com/bumptech/glide/load/resource/bitmap/RoundedCorners.html) 变换接受一个 `int`，它决定了圆角的弧度。它的`equals()`, `hashCode()` 和 `updateDiskCacheKey` 实现看起来像这样：

```java
  @Override
  public boolean equals(Object o) {
    if (o instanceof RoundedCorners) {
      RoundedCorners other = (RoundedCorners) o;
      return roundingRadius == other.roundingRadius;
    }
    return false;
  }

  @Override
  public int hashCode() {
    return Util.hashCode(ID.hashCode(),
        Util.hashCode(roundingRadius));
  }

  @Override
  public void updateDiskCacheKey(MessageDigest messageDigest) {
    messageDigest.update(ID_BYTES);

    byte[] radiusData = ByteBuffer.allocate(4).putInt(roundingRadius).array();
    messageDigest.update(radiusData);
  }
```

原来的 `String` 仍然保留，但 `roundingRadius` 被包含到了三个方法中。这里，`updateDiskCacheKey` 方法还演示了你可以如何使用 `ByteBuffer` 来包含基本参数到你的 `updateDiskCacheKey` 实现中。

##### 不要忘记 equals() / hashCode()!

值得重申的一点是，为了让内存缓存正常地工作你是否必须实现 `equals()` 和 `hashCode()` 方法。很不幸，即使你没有复写这两个方法，`BitmapTransformation` 和 `Transformation` 也能通过编译，但这并不意味着它们能正常工作。我们正在探索一些方案，以使在 Glide 的未来版本中，使用默认的 `equals()` 和 `hashCode` 方法将抛出一个编译时错误。

#### Glide中的特殊行为

##### 重用变换

`Transformation` 的设计初衷是无状态的。因此，在多个加载中复用 `Transformation` 应当总是安全的。创建一次 `Transformation` 并在多个加载中使用它，通常是很好的实践。

##### ImageView的自动变换

在Glide中，当你为一个 [ImageView](http://developer.android.com/reference/android/widget/ImageView.html) 开始加载时，Glide可能会自动应用 [FitCenter](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/FitCenter.html) 或 [CenterCrop](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/CenterCrop.html) ，这取决于view的 [ScaleType](http://developer.android.com/reference/android/widget/ImageView.ScaleType.html) 。如果 `scaleType` 是 `CENTER_CROP` , Glide 将会自动应用 `CenterCrop` 变换。如果 `scaleType` 为 `FIT_CENTER` 或 `CENTER_INSIDE` ，Glide会自动使用 `FitCenter` 变换。

当然，你总有权利覆写默认的变换，只需要一个带有 `Transformation` 集合的 [RequestOptions](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html) 即可。另外，你也可以通过使用 [`dontTransform()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#dontTransform--) 确保不会自动应用任何变换。

##### 自定义资源

因为 Glide 4.0 允许你指定你将解码的资源的父类型，你可能无法确切地知道将会应用何种变换。例如，当你使用 [`asDrawable()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestManager.html#asDrawable--) (或就是普通的 `with()` ，因为 `asDrawable()` 是默认情形)来加载 Drawable 资源时，你可能会得到 [`BitmapDrawable`](http://developer.android.com/reference/android/graphics/drawable/BitmapDrawable.html) 子类，也有可能得到 [`GifDrawable`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/gif/GifDrawable.html) 子类。

为了确保你添加到 `RequestOptions` 中的任何变换都会被使用，Glide将 `Transformation` 添加到一个Map中保存，其Key为你提供变换的资源类型。当资源被成功解码时，Glide使用这个Map来取回对应的 `Transformation` 。

Glide可以将 `Bitmap` `Transformation`应用到 `BitmapDrawable` , `GifDrawable` , 以及 `Bitmap` 资源上，因此通常你只需要编写和应用 `Bitmap` `Transformation` 。然而，如果你添加了额外的资源类型，你可能需要考虑派生 [`RequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html) 类，并且，在内置的这些 `Bitmap` `Transformations` 之外，你还需要为你的自定义资源类型提供一个 `Transformation` 。

### 7. 目标

在Glide中，[`Target`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/target/Target.html) 是介于请求和请求者之间的中介者的角色。Target 负责展示占位符，加载资源，并为每个请求决定合适的尺寸。被使用得最频繁的是 [`ImageViewTargets`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/target/ImageViewTarget.html) ，它用于在 ImageView 上展示占位符、Drawable 和 Bitmap 。用户还可以实现自己的 Target ，或者从任何可用的基类派生子类。

#### 指定目标

[`into(Target)`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#into-Y-) 方法不仅仅用于启动每个请求，它同时也指定了接收请求结果的 Target：

```java
Target<Drawable> target = 
  Glide.with(fragment)
    .load(url)
    .into(new Target<Drawable>() {
      ...
    });
```

Glide 提供了一个辅助方法 [`into(ImageView)`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#into-android.widget.ImageView-) ，它接受一个 `ImageView` 参数并为其请求的资源类型包装了一个合适的 [`ImageViewTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/target/ImageViewTarget.html)：

```java
Target<Drawable> target = 
  Glide.with(fragment)
    .load(url)
    .into(imageView);
```

#### 取消和重用

你可以注意到 [`into(Target)`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#into-Y-) 和 [`into(ImageView)`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#into-android.widget.ImageView-) 都返回了一个 `Target` 实例。如果你重用这个 `Target` 来在将来开始一个新的加载，则之前开始的任何请求都会被取消，它们使用的资源将被释放：

```java
Target<Drawable> target = 
  Glide.with(fragment)
    .load(url)
    .into(new Target<Drawable>() {
      ...
    });
... 
// Some time in the future:
Glide.with(fragment)
  .load(newUrl)
  .into(target);
```

你也可以使用返回的 `Target` 来 [`clear()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-) 之前的加载，这将在不需要开始新的加载的情况下释放掉任何相关资源：

```java
Target<Drawable> target = 
  Glide.with(fragment)
    .load(url)
    .into(new Target<Drawable>() {
      ...
    });
... 
// Some time in the future:
Glide.with(fragment).clear(target);
```

Glide 的 [ViewTarget](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html) 子类使用了 Android Framework 的 [`getTag()`](https://developer.android.com/reference/android/view/View.html#getTag()) 和 [`setTag()`](https://developer.android.com/reference/android/view/View.html#setTag(java.lang.Object)) 方法来存储每个请求的相关信息，因此如果你在使用 [`ViewTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html) 或在往 `ImageView` 中加载图片，你可以直接重用或清理这个 `View`:

```java
Glide.with(fragment)
  .load(url)
  .into(imageView);

// Some time in the future:
Glide.with(fragment).clear(imageView);

// Or:
Glide.with(fragment)
  .load(newUrl)
  .into(imageView);
```

此外，**仅对[`ViewTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html)而言**，你可以在每次加载或清理调用时都传入一个新的实例，而 Glide 仍然可以从 `View` 的 tag 中取回之前一次加载的信息：

```java
Glide.with(fragment)
  .load(url)
  .into(new DrawableImageViewTarget(imageView));

// Some time in the future:
Glide.with(fragment)
  .load(newUrl)
  .into(new DrawableImageViewTarget(imageView));
```

注意，除非你的 `Target` 继承自 [`ViewTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html)，或实现了 [`setRequest()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html#setRequest-com.bumptech.glide.request.Request-) 和 [`getRequest()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html#getRequest--)并允许你从新的 `Target` 实例中取回上一次加载的信息，否则这种使用方法将**不**奏效。

#### 清理

当你完成了对资源（`Bitmap`，`Drawable` 等）的使用时，及时清理（[`clear`](https://muyangmin.github.io/glide-docs-cn/doc/14)）你创建的这些 `Target` 是一个好的实践。即使你认为你的请求已经完成了，也应该使用 [`clear()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-) 以使 Glide 可以重用被这次加载使用的任何资源 (特别是 Bitmap )。未调用 [`clear()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-) 会浪费 CPU 和内存，阻塞更重要的加载，甚至如果你在同一个 surface (View, Notification, RPC 等) 上有两个 `Target`，可能会引发图片显示错误。对于像 [`SimpleTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/SimpleTarget.html)这种无法从一个新实例里跟踪前一个请求的 `Target` 来说，及时清理尤为重要。

#### 尺寸 (Sizes and dimensions)

默认情况下，Glide 使用目标通过 [`getSize`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/target/Target.html#getSize-com.bumptech.glide.request.target.SizeReadyCallback-) 方法提供的尺寸来作为请求的目标尺寸。这允许 Glide 选取合适的 URL，下采样，裁剪和变换合适的图片以减少内存占用，并确保加载尽可能快地完成。

##### View 目标

[`ViewTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html) 通过检查 View 的属性和/或使用一个 [`OnPreDrawListener`](https://developer.android.com/reference/android/view/ViewTreeObserver.OnPreDrawListener.html) 在 View 绘制之前直接测量尺寸来实现 [`getSize()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/target/Target.html#getSize-com.bumptech.glide.request.target.SizeReadyCallback-) 方法。因此， Glide 可以自动调整大部分图片以匹配目标 `View`。加载更小的图片可使 Glide 更快地完成加载 (在缓存到磁盘以后)，并使用更少的内存，在图片尺寸一致时还可以增加 Glide 的 BitmapPool 的命中率。

`ViewTarget` 使用以下逻辑：

1. 如果 `View` 的布局参数尺寸 > 0 且 > padding，则使用该布局参数；
2. 如果 `View` 尺寸 > 0 且 > padding，使用该实际尺寸；
3. 如果 `View` 布局参数为 `wrap_content` 且至少已发生一次 layout ，则打印一行警告日志，建议使用 `Target.SIZE_ORIGINAL` 或通过 `override()` 指定其他固定尺寸，并使用屏幕尺寸为该请求尺寸；
4. 其他情况下（布局参数为 `match_parent`， `0`， 或 `wrap_content` 且没有发生过 layout ），则等待布局完成，然后回溯到步骤1。

有时在使用 `RecyclerView`时，`View` 可能被重用且保持了前一个位置的尺寸，但在当前位置会发生改变。为了处理这种场景，你可以创建一个新的 [`ViewTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html) 并为 `waitForLayout()` 方法传入 true：

```java
@Override
public void onBindViewHolder(VH holder, int position) {
  Glide.with(fragment)
    .load(urls.get(position))
    .into(new DrawableImageViewTarget(holder.imageView, /*waitForLayout=*/ true));
```

##### 强大的尺寸管理

通常 Glide 在显式地为加载的 View 设置了 dp 尺寸时提供了最快且最可预测的结果。如果无法达到这一点，Glide 也通过 `onPreDrawListener` 提供了为 `layout_weight`，`match_parent` 和其他相对尺寸的完备鲁棒的支持。最后，如果这些都无法达成，Glide 应该也为 `wrap_content` 提供了合理的行为。

##### 后备方案

在任何情况下，如果 Glide 看起来获取了错误的 View 尺寸，你都可以手动覆盖来纠正它。你可以选择扩展 [`ViewTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/target/ViewTarget.html) 实现你自己的逻辑，或者使用 `RequestOption` 里的 [`override()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#override-int-int-)方法。

#### 定制目标

如果你正在使用一个 `Target` 且你将要加载的不是可以允许你派生 [`ViewTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html) 的 View, 你讲需要实现 [`getSize()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/target/Target.html#getSize-com.bumptech.glide.request.target.SizeReadyCallback-) 方法。

实现 `getSize` 可能最简单的方案是直接调用回调：

```java
@Override
public void getSize(SizeReadyCallback cb) {
  cb.onSizeReady(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL);
}
```

使用 `Target.SIZE_ORIGINAL` 可能非常低效，或如果你的图片足够大可能引发 OOM 。作为替代方案，你也可以为你的 `Target` 的构造器传入一个尺寸，并把这些尺寸提供给回调：

```java
public class CustomTarget<T> implements Target<T> {
  private final int width;
  private final int height;
 
  public CustomTarget(int width, int height) {
    this.width = width;
    this.height = height;
  }

  ...

  @Override
  public void getSize(SizeReadyCallback cb) {
    cb.onSizeReady(width, height);
  }
}
```

如果你的应用内使用一致的图片尺寸，或你确切地知道你需要的尺寸，你也可以传入一个特定尺寸的集合。如果你不知道所需的具体尺寸，但可以异步地得出结果，你也可以使用列表持有在 [`getSize()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/target/Target.html#getSize-com.bumptech.glide.request.target.SizeReadyCallback-) 中给出你的任何回调，然后执行你的异步过程并稍后在你得出尺寸之后通知你持有的这些回调。

如果你持有了这些SizeReadyCallback回调，请确保同时实现 [`removeCallback`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html#removeCallback-com.bumptech.glide.request.target.SizeReadyCallback-) 以避免内存泄露。

#### 动画资源和定制目标

如果你只是要加载 `GifDrawable`，或任何其他资源类型到一个 `View`，你应该总是尽可能地使用 `into(ImageView)`。除了优雅的处理或新发起请求之外，Glide的大部分 `ViewTarget` 实现已经为您处理了 `Drawable` 动画。如果你确实必须使用定制的 `ViewTarget`，请确保继承自 `ViewTarget` 或在新请求开始之前和展示资源结束之后严格地清理从 `into(Target)` 返回的 `Target`。

如果你并非往 `View` 中加载图片，而直接使用 `ViewTarget` 或使用了定制的 `Target` 比如 [`SimpleTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/SimpleTarget.html) 且你正在加载一个动画资源例如 [`GifDrawable`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/resource/gif/GifDrawable.html)，你需要确保在 `onResourceReady` 中调用 [`start()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/resource/gif/GifDrawable.html#start--) 来启动这个动画：

```java
Glide.with(fragment)
  .asGif()
  .load(url)
  .into(new SimpleTarget<>() {
    @Override
    public void onResourceReady(GifDrawable resource, Transition<GifDrawable> transition) {
      resource.start();
      // Set the resource wherever you need to use it.
    }
  });
```

如果你加载的是 `Bitmap` 或 `GifDrawable`，你可以判断这个可绘制对象是否实现了 [`Animatable`](https://developer.android.com/reference/android/graphics/drawable/Animatable.html)：

```java
Glide.with(fragment)
  .load(url)
  .into(new SimpleTarget<>() {
    @Override
    public void onResourceReady(Drawable resource, Transition<GifDrawable> transition) {
      if (resource instanceof Animatable) {
        resource.start();
      }
      // Set the resource wherever you need to use it.
    }
  });
```

### 8.过渡 

在 Glide 中，[`Transitions`(直译为”过渡”)](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/transition/Transition.html) 允许你定义 Glide 如何从占位符到新加载的图片，或从缩略图到全尺寸图像过渡。Transition 在单一请求的上下文中工作，而不会跨多个请求。因此，[`Transitions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/transition/Transition.html) 并不能让你定义从一个请求到另一个请求的动画（比如，交叉淡入效果）。

#### 默认过渡

不同于 Glide v3，Glide v4 将**不会**默认应用交叉淡入或任何其他的过渡效果。每个请求必须手动应用过渡。

#### 指定过渡

你可以查看 [Options documentation](https://muyangmin.github.io/glide/doc/options.html#transitionoptions) 以获取概览和示例代码。

[`TransitionOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/TransitionOptions.html) 用于给一个特定的请求指定过渡。 每个请求可以使用 [`RequestBuilder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html) 中的 [`transition()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#transition-com.bumptech.glide.TransitionOptions-) 方法来设定 [`TransitionOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/TransitionOptions.html) 。还可以通过使用 [`BitmapTransitionOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/bitmap/BitmapTransitionOptions.html) 或 [`DrawableTransitionOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/drawable/DrawableTransitionOptions.html) 来指定类型特定的过渡动画。对于 Bitmap 和 Drawable 之外的资源类型，可以使用 [`GenericTransitionOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/GenericTransitionOptions.html)。

#### 性能提示

Android中的动画代价是比较大的，尤其是同时开始大量动画的时候。 交叉淡入和其他涉及 alpha 变化的动画显得尤其昂贵。 此外，动画通常比图片解码本身还要耗时。在列表和网格中滥用动画可能会让图像的加载显得缓慢而卡顿。为了提升性能，请在使用 Glide 向 ListView , GridView, 或 RecyclerView 加载图片时考虑避免使用动画，尤其是大多数情况下，你希望图片被尽快缓存和加载的时候。作为替代方案，请考虑预加载，这样当用户滑动到具体的 item 的时候，图片已经在内存中了。

#### 常见错误

##### 对占位符和透明图片交叉淡入

Glide 的默认交叉淡入(cross fade)效果使用了 [`TransitionDrawable`](https://developer.android.com/reference/android/graphics/drawable/TransitionDrawable.html) 。它提供两种动画模式，由 [`setCrossFadeEnabled()`](https://developer.android.com/reference/android/graphics/drawable/TransitionDrawable.html#setCrossFadeEnabled(boolean)) 控制。当交叉淡入被禁用时，正在过渡的图片会在原先显示的图像上面淡入。当交叉淡入被启用时，原先显示的图片会从不透明过渡到透明，而正在过渡的图片则会从透明变为不透明。

在 Glide 中，我们默认禁用了交叉淡入，这样通常看起来要好看一些。实际的交叉淡入，如上所述对两个图片同时改变 alpha 值，通常会在过渡的中间造成一个短暂的白色闪屏，这个时候两个图片都是部分不透明的。

不幸的是，虽然禁用交叉淡入通常是一个比较好的默认行为，当待加载的图片包含透明像素时仍然可能造成问题。当占位符比实际加载的图片要大，或者图片部分为透明时，禁用交叉淡入会导致动画完成后占位符在图片后面仍然可见。如果你在加载透明图片时使用了占位符，你可以启用交叉淡入，具体办法是调整 [`DrawableCrossFadeFactory`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/transition/DrawableCrossFadeFactory.html) 里的参数并将结果传到 [`transition()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/TransitionOptions.html#transition-com.bumptech.glide.request.transition.TransitionFactory-) 中：

```
DrawableCrossFadeFactory factory =
        new DrawableCrossFadeFactory.Builder().setCrossFadeEnabled(true).build();

GlideApp.with(context)
        .load(url)
        .transition(withCrossFade(factory))
        .diskCacheStrategy(DiskCacheStrategy.ALL)
        .placeholder(R.color.placeholder)
        .into(imageView);
```

更多信息参见 [Issue #2017](https://github.com/bumptech/glide/issues/2017)

##### 在多个请求间交叉淡入

[`Transitions`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/transition/Transition.html) 并不能让你在不同请求中加载的两个图像之间做过渡。当新的加载被应用到 View 或 Target (查看 [Target的文档](https://muyangmin.github.io/glide/doc/targets.html#targets-and-automatic-cancellation) )上时，Glide 默认会取消任何已经存在的请求。因此，如果你想加载连个个不同的图片并在它们之间做动画，你无法直接通过 Glide 来完成。等待第一个加载完成并在 View 外持有这个 Bitmap 或 Drawable ，然后开始新的加载并手动在这两者之间做动画，诸如此类的策略看起来有效，但是实际上不安全，并可能导致程序崩溃或图像错误。

相反，最简单的办法是使用包含两个 [`ImageView`](https://developer.android.com/reference/android/widget/ImageView.html) 的 [`ViewSwitcher`](https://developer.android.com/reference/android/widget/ViewSwitcher.html) 来完成。将第一张图片加载到 [`getNextView()`](https://developer.android.com/reference/android/widget/ViewSwitcher.html#getNextView()) 的返回值里面，然后将第二张图片加载到 [`getNextView()`](https://developer.android.com/reference/android/widget/ViewSwitcher.html#getNextView()) 的下一个返回值中，并使用一个 [`RequestListener`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestListener.html) 在第二张图片加载完成时调用 [`showNext()`](https://developer.android.com/reference/android/widget/ViewAnimator.html#showNext()) 。为了更好地控制，你也可以使用 [Android开发者文档](https://developer.android.com/training/animation/crossfade.html) 指出的策略。但要记住与 [`ViewSwitcher`](https://developer.android.com/reference/android/widget/ViewSwitcher.html) 一样，仅在第二次图像加载完成后才开始交叉淡入淡出。

#### 定制过渡

如果要定义一个自定义的过渡动画，你需要完成以下两个步骤：

1. 实现 [`TransitionFactory`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/transition/TransitionFactory.html) .
2. 使用 [`DrawableTransitionOptions#with`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/drawable/DrawableTransitionOptions.html#with-com.bumptech.glide.request.transition.TransitionFactory-) 来将你自定义的 `TransitionFactory` 应用到加载中。

如果要改变你的 transition 的默认行为，以更好地控制它在不同的加载源（内存缓存，磁盘缓存，或uri）下是否被应用，你可以检查一下你的 [`TransitionFactory`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/transition/TransitionFactory.html) 中传递给 [`build()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/transition/TransitionFactory.html#build-com.bumptech.glide.load.DataSource-boolean-) 方法的那个 [`DataSource`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/DataSource.html) 。

如需示例代码，请查看 [`DrawableCrossFadeFactory`](https://github.com/bumptech/glide/blob/8f22bd9b82349bf748e335b4a31e70c9383fb15a/library/src/main/java/com/bumptech/glide/request/transition/DrawableCrossFadeFactory.java#L35).

### 9. 配置

从 Glide 4.9.0 开始，在某些情形下必须完成必要的设置 (`setup`)。

对于应用程序（application），仅当以下情形时才需要做设置：

- 使用一个或更多集成库
- 修改 Glide 的配置(`configuration`)（磁盘缓存大小/位置，内存缓存大小等）
- 扩展 Glide 的API。

对于库（library），仅当库需要注册一个或多个组件时才需要做设置。

#### 应用程序

应用程序(Applications)如果希望使用集成库和/或 Glide 的 API 扩展，则需要：

1. 恰当地添加一个 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 实现。
2. (可选)添加一个或多个 [`LibraryGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 实现。
3. 给上述两种实现添加 [`@GlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideModule.html) 注解。
4. 添加对 Glide 的注解解析器的依赖。
5. 在 proguard 中，添加对 [`AppGlideModules`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 的 keep 。

在 Glide 的 [Flickr 示例应用](https://github.com/bumptech/glide/blob/master/samples/flickr/src/main/java/com/bumptech/glide/samples/flickr/FlickrGlideModule.java) 中，有一个 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 的示例实现：

```java
@GlideModule
public class FlickrGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Registry registry) {
    registry.append(Photo.class, InputStream.class, new FlickrModelLoader.Factory());
  }
}
```

请注意添加对 Glide 的注解和注解解析器的依赖：

```groovy
compile 'com.github.bumptech.glide:annotations:4.11.0'
annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
```

最后，你应该在你的 `proguard.cfg` 中 keep 住你的 AppGlideModule 实现：

```xml
-keep public class  extends com.bumptech.glide.module.AppGlideModule
-keep class com.bumptech.glide.GeneratedAppGlideModuleImpl
```

#### 程序库 (Libraries)

程序库若不需要注册定制组件，则不需要做任何配置步骤，可以完全跳过这个章节。

程序库如果需要注册定制组件，例如 `ModelLoader`，可按以下步骤执行：

1. 添加一个或多个 [`LibraryGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 实现，以注册新的组件。
2. 为每个 [`LibraryGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 实现，添加 [`@GlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideModule.html) 注解。
3. 添加 Glide 的注解处理器的依赖。

一个 [`LibraryGlideModule`] 的例子，在 Glide 的[OkHttp 集成库](https://github.com/bumptech/glide/blob/master/integration/okhttp3/src/main/java/com/bumptech/glide/integration/okhttp3/OkHttpLibraryGlideModule.java) 中：

```java
@GlideModule
public final class OkHttpLibraryGlideModule extends LibraryGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory());
  }
}
```

使用 [`GlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideModule.html) 注解需要使用 Glide 注解的依赖：

```groovy
//只参考编译，但不会编译该库进包；
compile 'com.github.bumptech.glide:annotations:4.11.0'
```

##### 避免在程序库中使用 AppGlideModule

程序库一定 **不要** 包含 `AppGlideModule` 实现。这么做将会阻止依赖该库的任何应用程序管理它们的依赖，或配置诸如 Glide 缓存大小和位置之类的选项。

此外，如果两个程序库都包含 `AppGlideModule`，应用程序将无法在同时依赖两个库的情况下通过编译，而不得不在二者之中做出取舍。

这确实意味着程序库将无法使用 Glide 的 generated API，但是使标准的 `RequestBuilder` 和 `RequestOptions` 加载仍然有效（可以在 [选项](https://muyangmin.github.io/glide-docs-cn/doc/options.html) 页找到例子）。

#### 应用程序选项

Glide 允许应用通过 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 实现来完全控制 Glide 的内存和磁盘缓存使用。Glide 试图提供对大部分应用程序合理的默认选项，但对于部分应用，可能就需要定制这些值。在你做任何改变时，请注意测量其结果，避免出现性能的倒退。

##### 内存缓存

默认情况下，Glide使用 [`LruResourceCache`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/LruResourceCache.html) ，这是 [`MemoryCache`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/MemoryCache.html) 接口的一个缺省实现，使用固定大小的内存和 LRU 算法。[`LruResourceCache`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/LruResourceCache.html) 的大小由 Glide 的 [`MemorySizeCalculator`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/MemorySizeCalculator.html) 类来决定，这个类主要关注设备的内存类型，设备 RAM 大小，以及屏幕分辨率。

应用程序可以自定义 [`MemoryCache`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/MemoryCache.html) 的大小，具体是在它们的 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 中使用 [`applyOptions(Context, GlideBuilder)`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html#applyOptions-android.content.Context-com.bumptech.glide.GlideBuilder-) 方法配置 [`MemorySizeCalculator`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/MemorySizeCalculator.html) ：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    MemorySizeCalculator calculator = new MemorySizeCalculator.Builder(context)
        .setMemoryCacheScreens(2)
        .build();
    builder.setMemoryCache(new LruResourceCache(calculator.getMemoryCacheSize()));
  }
}
```

也可以直接覆写缓存大小：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    int memoryCacheSizeBytes = 1024 * 1024 * 20; // 20mb
    builder.setMemoryCache(new LruResourceCache(memoryCacheSizeBytes));
  }
}
```

甚至可以提供自己的 [`MemoryCache`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/MemoryCache.html) 实现：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setMemoryCache(new YourAppMemoryCacheImpl());
  }
}
```

##### Bitmap 池

Glide 使用 [`LruBitmapPool`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/LruBitmapPool.html) 作为默认的 [`BitmapPool`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html) 。[`LruBitmapPool`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/LruBitmapPool.html) 是一个内存中的固定大小的 `BitmapPool`，使用 LRU 算法清理。默认大小基于设备的分辨率和密度，同时也考虑内存类和 [`isLowRamDevice`](https://developer.android.com/reference/android/app/ActivityManager.html#isLowRamDevice()) 的返回值。具体的计算通过 Glide 的 [`MemorySizeCalculator`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/MemorySizeCalculator.html) 来完成，与 Glide 的 [`MemoryCache`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/MemoryCache.html) 的大小检测方法相似。

应用可以在它们的 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 中定制 [`BitmapPool`] 的尺寸，使用 [`applyOptions(Context, GlideBuilder)`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html#applyOptions-android.content.Context-com.bumptech.glide.GlideBuilder-) 方法并配置 [`MemorySizeCalculator`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/MemorySizeCalculator.html):

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    MemorySizeCalculator calculator = new MemorySizeCalculator.Builder(context)
        .setBitmapPoolScreens(3)
        .build();
    builder.setBitmapPool(new LruBitmapPool(calculator.getBitmapPoolSize()));
  }
}
```

应用也可以直接复写这个池的大小：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    int bitmapPoolSizeBytes = 1024 * 1024 * 30; // 30mb
    builder.setBitmapPool(new LruBitmapPool(bitmapPoolSizeBytes));
  }
}
```

甚至可以提供 [`BitmapPool`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html) 的完全自定义实现：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setBitmapPool(new YourAppBitmapPoolImpl());
  }
}
```

##### 磁盘缓存

Glide 使用 [`DiskLruCacheWrapper`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/DiskLruCacheWrapper.html) 作为默认的 [`磁盘缓存`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.html) 。 [`DiskLruCacheWrapper`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/DiskLruCacheWrapper.html) 是一个使用 LRU 算法的固定大小的磁盘缓存。默认磁盘大小为 [250 MB](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.Factory.html#DEFAULT_DISK_CACHE_SIZE) ，位置是在应用的 [缓存文件夹](https://developer.android.com/reference/android/content/Context.html#getCacheDir()) 中的一个 [特定目录](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.Factory.html#DEFAULT_DISK_CACHE_DIR) 。

假如应用程序展示的媒体内容是公开的（例如从无授权机制的网站或搜索引擎上加载），那么应用可以将这个缓存位置改到外部存储：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setDiskCache(new ExternalCacheDiskCacheFactory(context));
  }
}
```

无论使用内部或外部磁盘缓存，应用程序都可以改变磁盘缓存的大小：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    int diskCacheSizeBytes = 1024  1024  100;  100 MB
    builder.setDiskCache(new InternalCacheDiskCacheFactory(context, diskCacheSizeBytes));
  }
}
```

应用程序还可以改变缓存文件夹在外存或内存上的名字：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    int diskCacheSizeBytes = 1024  1024  100;  100 MB
    builder.setDiskCache(
        new InternalCacheDiskCacheFactory(context, cacheFolderName, diskCacheSizeBytes));
  }
}
```

应用程序还可以自行选择 [`DiskCache`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.html) 接口的实现，并提供自己的 [`DiskCache.Factory`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.Factory.html) 来创建缓存。Glide 使用一个工厂接口来在后台线程中打开 [`磁盘缓存`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.html) ，这样方便缓存做诸如检查路径存在性等的IO操作而不用触发 [严格模式](https://developer.android.com/reference/android/os/StrictMode.html) 。

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setDiskCache(new DiskCache.Factory() {
        @Override
        public DiskCache build() {
          return new YourAppCustomDiskCache();
        }
    });
  }
}
```

#### 默认请求选项

虽然 [`请求选项`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/request/RequestOptions.html) 通常由每个请求单独指定，你也可以通过 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 应用一个 [`请求选项`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/request/RequestOptions.html) 的集合以作用于你应用中启动的每个加载：

```java
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setDefaultRequestOptions(
        new RequestOptions()
          .format(DecodeFormat.RGB_565)
          .disallowHardwareBitmaps());
  }
}
```

一旦你创建了新的请求，这些选项将通过 `GlideBuilder` 中的 `setDefaultRequestOptions` 被应用上。因此，任何单独请求里应用的选项将覆盖 `GlideBuilder` 里设置的冲突选项。

类似地，[`RequestManagers`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/RequestManager.html) 允许你为这个特定的 [`RequestManager`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/RequestManager.html) 启动的所有加载请求设置默认的 [`请求选项`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/request/RequestOptions.html)。 因为每个 `Activity` 和 `Fragment` 都拥有自己的 [`RequestManager`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/RequestManager.html)，你可以使用 [`RequestManager`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/RequestManager.html) 的 [`applyDefaultRequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/RequestManager.html#applyDefaultRequestOptions-com.bumptech.glide.request.RequestOptions-) 方法来设置默认的 [`RequestOption`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/request/RequestOptions.html)，并仅作用于一个特定的 `Activity` 或 `Fragment`：

```java
Glide.with(fragment)
  .applyDefaultRequestOptions(
      new RequestOptions()
          .format(DecodeFormat.RGB_565)
          .disallowHardwareBitmaps());
```

[`RequestManager`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/RequestManager.html) 还有一个 [`setDefaultRequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/RequestManager.html#setDefaultRequestOptions-com.bumptech.glide.request.RequestOptions-) 方法，可以完全替换掉之前设置的任意的默认 [`请求选项`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/request/RequestOptions.html)，无论它是通过 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 的 [`GlideBuilder`] 还是 [`RequestManager`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/RequestManager.html)。使用 [`setDefaultRequestOptions`]要小心，因为很容易意外覆盖掉你其他地方设置的重要默认选项。 通常 [`applyDefaultRequestOptions`](https://muyangmin.github.io/glide-docs-cn/javadocs/410/com/bumptech/glide/RequestManager.html#applyDefaultRequestOptions-com.bumptech.glide.request.RequestOptions-)更安全，使用起来更直观。

#### 未捕获异常策略 (UncaughtThrowableStrategy)

在加载图片时假如发生了一个异常 (例如, OOM), Glide 将会使用一个 `GlideExecutor.UncaughtThrowableStrategy` 。

默认策略是将异常打印到设备的 LogCat 中。 这个策略从 Glide 4.2.0 起将可被定制。 你可以传入一个磁盘执行器和/或一个 resize 执行器：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    final UncaughtThrowableStrategy myUncaughtThrowableStrategy = new ...
    builder.setDiskCacheExecutor(newDiskCacheExecutor(myUncaughtThrowableStrategy));
    builder.setResizeExecutor(newSourceExecutor(myUncaughtThrowableStrategy));
  }
}
```

##### 日志级别

你可以使用 [`setLogLevel`](https://muyangmin.github.io/glide-docs-cn/javadocs/420/com/bumptech/glide/GlideBuilder.html#setLogLevel-int-) (结合 Android 的 [`Log`](https://developer.android.com/reference/android/util/Log.html) 定义的值) 来获取格式化日志的子集，包括请求失败时的日志行。通常来说 `Log.VERBOSE` 将使日志变得更冗杂，`Log.ERROR` 会让日志更趋向静默，详细可见 [javadoc](https://muyangmin.github.io/glide-docs-cn/javadocs/420/com/bumptech/glide/GlideBuilder.html#setLogLevel-int-) 。

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setLogLevel(Log.DEBUG);
  }
}
```

#### 注册组件

应用程序和库都可以注册很多组件来扩展 Glide 的功能。可用的组件包括：

- [`ModelLoader`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/model/ModelLoaderFactory.html), 用于加载自定义的 Model(Url, Uri,任意的 POJO )和 Data(InputStreams, FileDescriptors)。
- [`ResourceDecoder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/ResourceDecoder.html), 用于对新的 Resources(Drawables, Bitmaps)或新的 Data 类型(InputStreams, FileDescriptors)进行解码。
- [`Encoder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Encoder.html), 用于向 Glide 的磁盘缓存写 Data (InputStreams, FileDesciptors)。
- [`ResourceTranscoder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/resource/transcode/ResourceTranscoder.html)，用于在不同的资源类型之间做转换，例如，从 BitmapResource 转换为 DrawableResource 。
- [`ResourceEncoder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/ResourceEncoder.html)，用于向 Glide 的磁盘缓存写 Resources(BitmapResource, DrawableResource)。

组件通过 [`Registry`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/Registry.html) 类来注册。例如，添加一个 `ModelLoader` ，使其能从自定义的Model对象中创建一个 InputStream ：

```java
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Registry registry) {
    registry.append(Photo.class, InputStream.class, new CustomModelLoader.Factory());
  }
}
```

在一个 `GlideModule` 里可以注册很多组件。[`ModelLoader`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/model/ModelLoaderFactory.html) 和 [`ResourceDecoder`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/ResourceDecoder.html) 对于同样的参数类型还可以有多种实现。

#### 剖析(Anatomy)一个请求

被注册组件的集合（包括默认被 Glide 注册的和在 Module 中被注册的），会被用于定义一个加载路径集合。每个加载路径都是从提供给 [`load`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#load-java.lang.Object-) 方法的数据模型到 [`as`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestManager.html#as-java.lang.Class-) 方法指定的资源类型的一个逐步演进的过程。一个加载路径（粗略地）由下列步骤组成:

1. 模型(Model) -> 数据(Data) (由`模型加载器(ModelLoader)`处理)
2. 数据(Data) -> 资源(Resource) (由`资源解析器(ResourceDecoder)`处理)
3. 资源(Resource) -> 转码后的资源(Transcoded Resource) (可选；由`资源转码器(ResourceTranscoder)`处理)

`编码器(Encoder)`可以在步骤2之前往Glide的磁盘缓存中写入数据。 `资源编码器(ResourceEncoder)`可以在步骤3之前网Glide的磁盘缓存写入资源。

当一个请求开始后，Glide将尝试所有从数据模型到请求的资源类型的可用路径。如果任何一个加载路径成功，这个请求就将成功。只有所有可用加载路径都失败时，这个请求才会失败。

#### 排序组件

在 [`Registry`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/Registry.html) 类中定义了 `prepend()` , `append()` 和 `replace()` 方法，它们可以用于设置 Glide 尝试每个 `ModelLoader` 和 `ResourceDecoder` 之间的顺序。对组件进行排序允许你注册一些只处理特定树模型的子集的组件（即只处理特定类型的Uri，或仅仅特定类型的图像格式），并可以在后面追加一个捕获所有类型的组件以处理其他情况。

##### prepend()

假如你的 `ModelLoader` 或者 `ResourceDecoder` 在某个地方失败了，这时候你想将已有的数据交由 Glide 的默认行为来处理，可以使用 `prepend()`。 `prepend()` 将确保你的 `ModelLoader` 或 `ResourceDecoder` 先于之前注册的其他组件并被首先执行。如果你的 `ModelLoader` 或者 `ResourceDecoder` 从其 `handles()` 方法中返回了一个 `false` 或失败，所有其他的 `ModelLoader` 或 `ResourceDecoder` 将以它们被注册的顺序执行，一次一个，作为一种回退方案。

##### append()

要处理新的数据类型或提供一个到 Glide 默认行为的回退，使用 `append()`。`append()` 将确保你的 `ModelLoader` 或 `ResourceDecoder` 仅在 Glide 的默认组件被尝试后才会被调用。 如果你正在尝试处理 Glide 的默认组件能处理的某些子类型 (例如一些特定的 Uri 授权或子类型)，你可能需要使用 `prepend()` 来确保 Glide 的默认组件不会在你的定制组件之前加载。

##### replace()

要完全替换 Glide 的默认行为并确保它绝不运行，请使用 `replace()`。 `replace()` 将移除所有处理给定模型和数据类的 `ModelLoaders`，并添加你的 `ModelLoader` 来代替。 `replace()` 在使用库(例如 OkHttp 或 Volley)替换掉 Glide 的网络逻辑时尤其有用，这种时候你会希望确保仅 OkHttp 或 Volley 被调用。

#### 添加一个 ModelLoader

举个例子，添加一个 `ModelLoader`，它从一个新的自定义 Model 对象中建立一个 InputStream：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.append(Photo.class, InputStream.class, new CustomModelLoader.Factory());
  }
}
```

在这里，`append()`可以被安全地使用，因为 Photo.class 是一个你的应用定制的模型对象，所以你知道 Glide 的默认行为中并没有你需要替换的东西。

相反，如果要处理一种新的 [`BaseGlideUrlLoader`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/model/stream/BaseGlideUrlLoader.html) 中的 String Url类型，你应该使用 `prepend()` 以使你的 `ModelLoader` 在 Glide 对 `Strings` 的默认 `ModelLoaders` 之前运行：

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.prepend(String.class, InputStream.class, new CustomUrlModelLoader.Factory());
  }
}
```

最后，如果要完全移除和替换 Glide 对某种特定类型的默认处理，例如一个网络库，你应该使用 `replace()`:

```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory());
  }
}
```

#### 模块类和注解

Glide v4 依赖于两种类，[`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 与 [`LibraryGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) ，以配置 Glide 单例。这两种类都允许用于注册额外的组件，例如 [`ModelLoaders`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/model/ModelLoader.html) , [`ResourceDecoders`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/ResourceDecoder.html) 等。但只有 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 被允许配置应用特定的设置项，比如缓存实现和缓存大小。

##### AppGlideModule

如果应用希望实现 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 的任意方法或使用集成库，则可以添加一个 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 实现。 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 实现是一个信号，它会让 Glide 的注解解析器生成一个单一的所有已发现的 [`LibraryGlideModules`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 的联合类。

对于一个特定的应用，只能存在一个 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 实现（超过一个会在编译时报错）。因此，程序库不能提供 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 实现。

##### @GlideModule

为了让 Glide 正确地发现 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 和 [`LibraryGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 的实现类，它们的所有实现都必须使用 [`@GlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/GlideModule.html) 注解来标记。这个注解将允许 Glide 的 [注解解析器](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/compiler/GlideAnnotationProcessor.html) 在编译时去发现所有的实现类。

##### 注解处理器

另外，为了发现 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 和 [`LibraryGlideModules`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)，所有的库和应用还必须包含一个Glide的注解解析器的依赖。

#### 冲突

应用程序可能依赖多个程序库，而它们每一个都可能包含一个或更多的 [`LibraryGlideModules`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 。在极端情况下，这些 [`LibraryGlideModules`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 可能定义了相互冲突的选项，或者包含了应用程序希望避免的行为。应用程序可以通过给他们的 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 添加一个 [`@Excludes`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/Excludes.html) 注解来解决这种冲突，或避免不需要的依赖。

例如，如果你依赖了一个库，它有一个 [`LibraryGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 叫做`com.example.unwanted.GlideModule`，而你不想要它：

```java
@Excludes(com.example.unwanted.GlideModule)
@GlideModule
public final class MyAppGlideModule extends AppGlideModule { }
```

你也可以排除多个模块：

```java
@Excludes({com.example.unwanted.GlideModule, com.example.conflicing.GlideModule})
@GlideModule
public final class MyAppGlideModule extends AppGlideModule { }
```

[`@Excludes`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/annotation/Excludes.html) 注解不仅可用于排除 [`LibraryGlideModules`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) 。如果你还在从 Glide v3 到新版本的迁移过程中，你还可以用它来排除旧的，废弃的 [`GlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/GlideModule.html) 实现。

#### 清单解析

为了维持对 Glide v3 的 [`GlideModules`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/GlideModule.html) 的向后兼容性，Glide 仍然会解析应用程序和所有被包含的库中的 `AndroidManifest.xml` 文件，并包含在这些清单中列出的旧 [`GlideModules`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/GlideModule.html) 模块类。

如果你已经迁移到 Glide v4 的 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 和 [`LibraryGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html) ，你可以完全禁用清单解析。这样可以改善 Glide 的初始启动时间，并避免尝试解析元数据时的一些潜在问题。要禁用清单解析，请在你的 [`AppGlideModule`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html) 实现中复写 [`isManifestParsingEnabled()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/module/AppGlideModule.html#isManifestParsingEnabled--) 方法：

```java
@GlideModule
public final class MyAppGlideModule extends AppGlideModule {
  @Override
  public boolean isManifestParsingEnabled() {
    return false;
  }
}
```

### 10. 缓存

默认情况下，Glide 会在开始一个新的图片请求之前检查以下多级的缓存：

1. 活动资源 (Active Resources) - 现在是否有另一个 View 正在展示这张图片？
2. 内存缓存 (Memory cache) - 该图片是否最近被加载过并仍存在于内存中？
3. 资源类型（Resource） - 该图片是否之前曾被解码、转换并写入过磁盘缓存？
4. 数据来源 (Data) - 构建这个图片的资源是否之前曾被写入过文件缓存？

前两步检查图片是否在内存中，如果是则直接返回图片。后两步则检查图片是否在磁盘上，以便快速但异步地返回图片。

如果四个步骤都未能找到图片，则Glide会返回到原始资源以取回数据（原始文件，Uri, Url等）。

关于 Glide 缓存的默认大小与它们在磁盘上的位置的更多细节，或如何配置这些参数，请查看 [配置](https://muyangmin.github.io/glide-docs-cn/doc/configuration.html#disk-cache) 页面。

#### 缓存键(Cache Keys)

在 Glide v4 里，所有缓存键都包含至少两个元素：

1. 请求加载的 model（File, Url, Url）。如果你使用自定义的 model, 它需要正确地实现 `hashCode()` 和 `equals()`
2. 一个可选的 [`签名`(Signature)](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#signature-com.bumptech.glide.load.Key-)

另外，步骤1-3(活动资源，内存缓存，资源磁盘缓存)的缓存键还包含一些其他数据，包括：

1. 宽度和高度
2. 可选的`变换（Transformation）`
3. 额外添加的任何 [`选项(Options)`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Option.html)
4. 请求的数据类型 (Bitmap, GIF, 或其他)

活动资源和内存缓存使用的键还和磁盘资源缓存略有不同，以适应内存 [`选项(Options)`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Option.html)，比如影响 Bitmap 配置的选项或其他解码时才会用到的参数。

为了生成磁盘缓存上的缓存键名称，以上的每个元素会被哈希化以创建一个单独的字符串键名，并在随后作为磁盘缓存上的文件名使用。

#### 配置缓存

Glide 提供一系列的选项，以允许你选择加载请求与 Glide 缓存如何交互。

##### 磁盘缓存策略（Disk Cache Strategy）

[`DiskCacheStrategy`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/DiskCacheStrategy.html) 可被 [`diskCacheStrategy`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#diskCacheStrategy-com.bumptech.glide.load.engine.DiskCacheStrategy-) 方法应用到每一个单独的请求。 目前支持的策略允许你阻止加载过程使用或写入磁盘缓存，选择性地仅缓存无修改的原生数据，或仅缓存变换过的缩略图，或是兼而有之。

默认的策略叫做 [`AUTOMATIC`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/DiskCacheStrategy.html#AUTOMATIC) ，它会尝试对本地和远程图片使用最佳的策略。当你加载远程数据（比如，从URL下载）时，`AUTOMATIC` 策略仅会存储未被你的加载过程修改过(比如，变换，裁剪–译者注)的原始数据，因为下载远程数据相比调整磁盘上已经存在的数据要昂贵得多。对于本地数据，`AUTOMATIC` 策略则会仅存储变换过的缩略图，因为即使你需要再次生成另一个尺寸或类型的图片，取回原始数据也很容易。

指定 [`DiskCacheStrategy`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/DiskCacheStrategy.html) 非常容易:

```
Glide.with(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.ALL)
  .into(imageView);
```

##### 仅从缓存加载图片

某些情形下，你可能希望只要图片不在缓存中则加载直接失败（*比如省流量模式？–译者注*）。如果要完成这个目标，你可以在单个请求的基础上使用 [`onlyRetrieveFromCache`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#onlyRetrieveFromCache-boolean-) 方法：

```
Glide.with(fragment)
  .load(url)
  .onlyRetrieveFromCache(true)
  .into(imageView);
```

如果图片在内存缓存或在磁盘缓存中，它会被展示出来。否则只要这个选项被设置为 true ，这次加载会视同失败。

##### 跳过缓存

如果你想确保一个特定的请求跳过磁盘和/或内存缓存（*比如，图片验证码 –译者注*），Glide 也提供了一些替代方案。

仅跳过内存缓存，请使用 [`skipMemoryCache()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#skipMemoryCache-boolean-) :

```
Glide.with(fragment)
  .load(url)
  .skipMemoryCache(true)
  .into(view);
```

仅跳过磁盘缓存，请使用 [`DiskCacheStrategy.NONE`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/DiskCacheStrategy.html#NONE) :

```
Glide.with(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.NONE)
  .into(view);
```

这两个选项可以同时使用:

```
Glide.with(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.NONE)
  .skipMemoryCache(true)
  .into(view);
```

虽然提供了这些办法让你跳过缓存，但你通常应该不会想这么做。从缓存中加载一个图片，要比拉取-解码-转换成一张新图片的完整流程快得多。

如果你只是想更新缓存中的某个条目，请继续阅读下面关于 [`invalidation`](https://muyangmin.github.io/glide-docs-cn/doc/caching.html#cache-invalidation) 一节的介绍。

##### 实现

如果内置的选项不满足你的需求，你也可以编写你自己的 [`DiskCache`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.html) 实现。请查看 [配置](https://muyangmin.github.io/glide-docs-cn/doc/configuration.html#disk-cache) 页获得更多信息。

#### 缓存的刷新

因为磁盘缓存使用的是哈希键，所以**并没有**一个比较好的方式来简单地删除某个特定url或文件路径对应的所有缓存文件。如果你只允许加载或缓存原始图片的话，问题可能会变得更简单，但因为Glide还会缓存缩略图和提供多种变换(`transformation`)，它们中的任何一个都会导致在缓存中创建一个新的文件，而要跟踪和删除一个图片的所有版本无疑是困难的。

在实践中，使缓存文件无效的最佳方式是在内容发生变化时（url，uri，文件路径等）更改你的标识符。

##### 定制缓存刷新策略

因为通常改变标识符比较困难或者根本不可能，所以Glide也提供了 [`签名`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#signature-com.bumptech.glide.load.Key-) API 来混合（你可以控制的）额外数据到你的缓存键中。签名(`signature`)适用于媒体内容，也适用于你可以自行维护的一些版本元数据。

- MediaStore 内容 - 对于媒体存储内容，你可以使用Glide的 [`MediaStoreSignature`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/signature/MediaStoreSignature.html) 类作为你的签名。`MediaStoreSignature` 允许你混入修改时间、MIME类型，以及item的方向到缓存键中。这三个属性能够可靠地捕获对图片的编辑和更新，这可以允许你缓存媒体存储的缩略图。
- 文件 - 你可以使用 [`ObjectKey`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/signature/ObjectKey.html) 来混入文件的修改日期。
- Url - 尽管最好的让 url 失效的办法是让 server 保证在内容变更时对URL做出改变，你仍然可以使用 [`ObjectKey`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/signature/ObjectKey.html) 来混入任意数据（比如版本号）。

将签名传入加载请求很简单：

```
Glide.with(yourFragment)
    .load(yourFileDataModel)
    .signature(new ObjectKey(yourVersionMetadata))
    .into(yourImageView);
```

媒体存储签名对于 MediaStore 数据来说也很直接：

```
Glide.with(fragment)
    .load(mediaStoreUri)
    .signature(new MediaStoreSignature(mimeType, dateModified, orientation))
    .into(view);
```

你还可以定义你自己的签名，只要实现 [`Key`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/Key.html) 接口就好。请确保正确地实现 `equals()`, `hashCode()` 和 `updateDiskCacheKey()` 方法:

```
public class IntegerVersionSignature implements Key {
    private int currentVersion;

    public IntegerVersionSignature(int currentVersion) {
         this.currentVersion = currentVersion;
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof IntegerVersionSignature) {
            IntegerVersionSignature other = (IntegerVersionSignature) o;
            return currentVersion == other.currentVersion;
        }
        return false;
    }

    @Override
    public int hashCode() {
        return currentVersion;
    }

    @Override
    public void updateDiskCacheKey(MessageDigest md) {
        messageDigest.update(ByteBuffer.allocate(Integer.SIZE).putInt(signature).array());
    }
}
```

请记住，为了避免降低性能，您将需要在后台批量加载任何版本元数据，以便在要加载图像时即已处于可用状态。

如果这些努力都无法奏效，您不能更改标识符，也不能跟踪任何合理的版本元数据的情况下，也可以使用 [`diskCacheStrategy()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestOptions.html#diskCacheStrategy-com.bumptech.glide.load.engine.DiskCacheStrategy-) 和 [`DiskCacheStrategy.NONE`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/DiskCacheStrategy.html#NONE) 来完全禁用磁盘缓存。

#### 资源管理

Glide 的磁盘和内存缓存都是 LRU ，这意味着在达到使用限制或持续接近限制值之前，它们将占用持续增加的内存或磁盘空间。为了增加额外的灵活性，Glide 提供了一些额外的方式来让你可以管理你的应用使用的资源。

请记住，更大的内存缓存、位图池和磁盘缓存通常能提供更好的性能，或者至少在同等级别。如果你改变了缓存的大小， 你应该小心地测量一下你改动之前和之后的性能对比，以确保你的修改带来的性价比是可以接受的。

#### 内存缓存

默认情况下 Glide 的内存缓存和 [`BitmapPool`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html) 会响应 [`ComponentCallback2`](http://d.android.com/reference/android/content/ComponentCallbacks2.html?is-external=true) ，并根据 Android framework 提供的级别自动清理内容。 因此你通常不需要尝试动态监视或清理你的缓存或 [`BitmapPool`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)。然而，如果必要的话，Glide 确实提供了几个手动选项。

##### 永久尺寸调整

要改变你应用中 Glide 的可用 RAM 大小，请查看 [配置](https://muyangmin.github.io/glide-docs-cn/doc/configuration.html#memory-cache)。

##### 暂时尺寸调整

要在你应用的特定部分暂时允许 Glide 使用更多或更少的内存，你可以使用 [`setMemoryCategory`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/Glide.html#setMemoryCategory-com.bumptech.glide.MemoryCategory-):

```
// This method must be called on the main thread.
Glide.get(context).clearMemory();
```

清理所有内存并非特别经济，并且应该尽可能避免，以避免出现抖动和增加加载时间。

##### 磁盘缓存

Glide 在运行时仅提供对磁盘缓存的有限控制，但是其大小和配置可以在 `AppGlideModule` 中改变。

##### 永久尺寸修改

要改变你应用中 Glide 可用的 sdcard 可用空间，请查看 [配置](https://muyangmin.github.io/glide-docs-cn/doc/configuration.html#memory-cache)。

##### 清理磁盘缓存

要尝试清理所有磁盘缓存条目，你可以使用 [`clearDiskCache`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/Glide.html#clearDiskCache--)。

```
new AsyncTask<Void, Void, Void> {
  @Override
  protected Void doInBackground(Void... params) {
    // This method must be called on a background thread.
    Glide.get(applicationContext).clearDiskCache();
    return null;
  }
}
```

### 11. 资源重用 

Glide 中的资源包含很多东西，例如 `Bitmap`，`byte[]` 数组， `int[]` 数组，以及大量的 POJO 。无论什么时候，Glide 都会尝试重用这些资源，以限制你应用中的内存抖动数量。

任何尺寸的对象的过多分配都会显著增加你应用中的垃圾回收 (GC)。虽然 Android 较新的 ART 运行时的 GC 惩罚比 Dalvik 运行时要低，但无论你使用什么设备，过多内存分配都会降低应用的性能。

#### Dalvik

Dalvik 设备 (Lollipop 之前)在过多分配时将不得不面对特别大的代价，值得在这里讨论一下。

Dalvik 有两种基本的 GC 模式， GC_CONCURRENT 和 GC_FOR_ALLOC ，这两种你都可以在 logcat 中看到。

- GC_CONCURRENT 对于每次收集将阻塞主线程大约 5ms 。因为每个操作都比一帧(16ms)要小，GC_CONCURRENT 通常不会造成你的应用丢帧。
- GC_FOR_ALLOC 是一种 stop-the-world 收集，可能会阻塞主线程达到 125ms 以上。GC_FOR_ALLOC 几乎每次都会造成你的应用丢失多个帧，导致视觉卡顿，特别是在滑动的时候。

不幸的是，Dalvik 似乎甚至连适度的分配（例如一个 16kb 的缓冲区）都处理得不是很好。重复的中等分配，或即使单次大的分配（比如说一个 Bitmap ），将会导致 GC_FOR_ALLOC 。因此，你分配的内存越多，就会招来越多 stop-the-world 的 GC，而你的应用将有更多的丢帧。

通过复用中到大尺寸的资源， Glide 可以帮你尽可能地减少这种 GC，以保持应用的流畅。

#### Glide 如何追踪和重用资源

Glide 采用较为宽容的办法来处理资源重用。Glide 会在它相信某个资源可以安全地复用时才这么做，但它并不要求调用者在每次请求之后都回收资源。除非某个调用者显式地表示它已经用完了某个资源（见下文），资源将不会被回收或重用。

##### 引用计数

为决定某个资源是否正在被使用，以及什么时候可以安全地被重用，Glide 为每个资源保持了一个引用计数。

##### 增加引用计数

每次调用 [`into()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestBuilder.html#into-Y-) 来加载一个资源，这个资源的引用计数会被加一。如果相同的资源被加载到两个不同的 [`Target`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html)，则在两个加载都完成后，它的引用计数将会为二。

##### 减少引用计数

引用计数仅在调用者通过以下方式表示它们用完资源后会减少：

1. 在加载资源的 [`View`](http://d.android.com/reference/android/view/View.html?is-external=true) 或 [`Target`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html) 上调用 [`clear()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-) 。
2. 在这个[`View`](http://d.android.com/reference/android/view/View.html?is-external=true) 或 [`Target`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html) 上调用对另一个资源请求的 [`into`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestBuilder.html#into-Y-) 方法。

##### 释放资源

当引用计数到达 0 时，这个资源会被释放并被返回给 Glide 以重用。当资源被返回给 Glide 以重用以后，继续使用它是不安全的，因此以下行为是 **不安全的**：

1. 使用 `getImageDrawable` 来取回 `ImageView` 中加载的 `Bitmap` 或 `Drawable`，并使用某种方式展示它( `setImageDrawable`，动画，或 `TransitionDrawable` 或其他任何方式 )。
2. 使用 [`SimpleTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/SimpleTarget.html) 来将一个资源加载到 [`View`](http://d.android.com/reference/android/view/View.html?is-external=true)，但没有实现 [`onLoadCleared()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html#onLoadCleared-android.graphics.drawable.Drawable-) 方法并在其中将资源从 [`View`](http://d.android.com/reference/android/view/View.html?is-external=true) 中移除。
3. 对 Glide 加载的任何 `Bitmap` 调用 [`recycle()`](https://developer.android.com/reference/android/graphics/Bitmap.html#recycle())。

在清理对应的 [`View`](http://d.android.com/reference/android/view/View.html?is-external=true) 或 [`Target`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html) 之后还保持对资源的引用是不安全的，因为这个资源可能已经被销毁，或被重用于展示一个不同的图片，这可能导致未定义行为，图形损坏，或甚至导致继续使用该资源的应用崩溃。例如，在被释放回 Glide 之后， `Bitmap` 可能会被存储在一个 `BitmapPool` 中，并在未来的某个时刻被用重用于保存一张新图片的字节数据，或者它们已经被调用了 [`recycle()`]。在这两种情况下继续引用这个 `Bitmap` 并期待它们保持原始图像都是不安全的。

##### 池化 (Pooling)

尽管 Glide 的大部分回收逻辑主要针对 Bitmap，但所有的 [`Resource`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/Resource.html) 实现均可实现 [`recycle()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/Resource.html#recycle--) 方法并将它们包含的任意可重用的数据池化。 [`ResourceDecoder`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/ResourceDecoder.html) 可以返回开发者希望的任意 [`Resource`] API，因此开发者可以定制或提供额外的池化规则，只需要实现它们自己的 [`Resource`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/Resource.html) 和 [`ResourceDecoder`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/ResourceDecoder.html)。

特别地，对于 `Bitmap`，Glide 提供了一个 [`BitmapPool`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html) 接口，以允许 [`Resource`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/Resource.html) 获取和重用 [`Bitmap`] 对象。 Glide 的 [`BitmapPool`] 可以从任意的 `Context` 中使用 Glide 的单例获取到：

```
Glide.get(context).getBitmapPool();
```

类似地，希望为 `Bitmap` 池化施加更多控制的用户可以直接实现他们自己的 [`BitmapPool`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)，然后可以通过 `GlideModule` 的方式提供给 Glide。参见[配置页](https://muyangmin.github.io/glide-docs-cn/doc/configuration.html#bitmap-pool).

#### 常见错误

然而，允许池化让保证用户不会误用资源或`Bitmap`变得很困难。 Glide 会在可能的地方尝试添加一些断言，但是因为我们并不持有底层的 `Bitmap`，我们无法保证调用者在告诉我们 [`clear()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-) 或一个新请求之后，会立即停用这些资源。

#### 资源重用错误的征兆

有多种迹象可能暗示 `Bitmap` 或其他在 Glide 中被池化的资源出了问题。我们列出了一些最常见的现象，但这不是一个完备的列表。

##### Cannot draw a recycled Bitmap

Glide 的 `BitmapPool` 是固定大小的。当 `Bitmap` 从中被踢出而没有被重用时，Glide 将会调用 [`recycle()`](https://developer.android.com/reference/android/graphics/Bitmap.html#recycle())。如果应用在向 Glide 指出可以安全地回收之后 “不经意间” 继续持有 `Bitmap`，则应用可能尝试绘制这个 `Bitmap`，进而在 `onDraw` 方法中造成崩溃。

一种可能的情况是，一个目标被用于两个`ImageView`，而其中一个在 `Bitmap` 被放到 `BitmapPool` 中后仍然试图访问被回收后的 `Bitmap`。基于以下因素，要复现这种复用错误可能很困难：1）Bitmap 何时被放入池中，2）Bitmap 何时被回收，3）何种尺寸的 `BitmapPool` 和内存缓存会导致 `Bitmap` 的回收。可以在你的 `GlideModule` 中加入下面的代码片段，以使这个问题更容易复现：

```
@Override
public void applyOptions(Context context, GlideBuilder builder) {
    int bitmapPoolSizeBytes = 1024 * 1024 * 0; // 0mb
    int memoryCacheSizeBytes = 1024 * 1024 * 0; // 0mb
    builder.setMemoryCache(new LruResourceCache(memoryCacheSizeBytes));
    builder.setBitmapPool(new LruBitmapPool(bitmapPoolSizeBytes));
}
```

上面的代码确保没有内存缓存，且 `BitmapPool` 的尺寸为0；因此 `Bitmap` 如果恰好没有被使用，它将立刻被回收。这是为了调试目的让它更快出现。

##### Can’t call reconfigure() on a recycled bitmap

资源将在它们不再被使用时被返回到 Glide 的 `BitmapPool` 中。这里的内部实现基于 `Request`(它控制着[`Resource`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/engine/Resource.html)) 的生命周期管理。如果在这些 Bitmap 上调用了 [`recycle()`](https://developer.android.com/reference/android/graphics/Bitmap.html#recycle())，但它们仍然在池中，就会使 Glide 无法重用它们而导致你的应用崩溃并抛出这个信息。这里的一个关键点是，这个崩溃很可能发生在未来的某个点，而不在这个违例代码的执行处！

##### View 在图片之间闪烁或相同的图像在多个 View 中展示

如果一个 `Bitmap` 被多次返回到 `BitmapPool` 中，或它已被返回到池中单仍然被一个 [`View`](http://d.android.com/reference/android/view/View.html?is-external=true) 持有，另一个图片可能会被解码到这个 `Bitmap` 对象中。如果这种情况发生，就会使得 `Bitmap` 的内容会被替换为新的图片。 在这个过程中，`View` 可能仍然试图绘制这个 `Bitmap`，而这将导致原始的 `View` 展示一张新的图片。

#### 重用错误的原因

一些常见的重用错误原因已被列在下面。就像上面的征兆一样，要列出全面的列表是很困难的，但是在尝试调试应用程序中的重用错误时，这些是您应该考虑的一些事情。

##### 尝试往相同的 Target 加载两个不同的资源

在 Glide 中没有安全的办法来加载多个资源到单一的 Target 中。用户可以使用 [`thumbnail()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-) API 来加载一系列资源到一个 [`Target`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html)，但也仅仅在下一个 [`onResourceReady()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html#onResourceReady-R-com.bumptech.glide.request.transition.Transition-) 调用之前才可以安全地引用早前的一个资源。

通常一个更好的答案是使用第二个 [`View`](http://d.android.com/reference/android/view/View.html?is-external=true) 并将第二章图片加载到这第二个 View 上。 [`ViewSwitcher`](https://developer.android.com/reference/android/widget/ViewSwitcher.html) 可以很好地允许你在两个单独请求的不同图片之间做交叉淡入效果 (cross fade)。你可以仅添加一个 `ViewSwitcher` 在你的布局中，使用两个 `ImageView` 作为其子控件，然后使用两次 [`into(ImageView)`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestBuilder.html#into-android.widget.ImageView-)方法，每次一个子控件，来加载两张图片。

对于绝对要求将多个资源加载到相同 [`View`](http://d.android.com/reference/android/view/View.html?is-external=true) 的用户，可以使用两个单独的 [`Target`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html)。为确保每个加载都不会取消另一个，用户还需要避免使用 [`ViewTarget`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html) 子类，或使用一个自定义的 [`ViewTarget`] 子类并复写(override)其 [`setRequest()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html#setRequest-com.bumptech.glide.request.Request-) 和 [`getRequest()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html#getRequest--) 以使得它们不使用 [`View`](http://d.android.com/reference/android/view/View.html?is-external=true) 的 tag 来存储 [`Request`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/Request.html)。这属于高级用法，一般不推荐。

##### 往Target中加载资源，清除或重用Target，并继续引用该资源

最简单的比较这个错误的办法是确保所有对资源的引用都在 [`onLoadCleared()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/request/target/Target.html#onLoadCleared-android.graphics.drawable.Drawable-) 调用时置空。通常，加载一个 `Bitmap` 然后对 `Target` 解引用，并且不要再次调用 [`into()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestBuilder.html#into-Y-) 或 [`clear()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)，这样是安全的。然而，加载了一个 `Bitmap`，清除这个 `Target`，并在之后继续持有 `Bitmap` 引用是不安全的。类似地，加载资源到一个 `View` 上然后从 View 中获取这个资源 (通过 `getImageDrawable()` 或任何其他手段)，并在其他某个地方继续引用它，也是不安全的。

##### 在 `Transformation` 中回收原始Bitmap

正如在 [`变换`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/Transformation.html) 的 JavaDoc 中所说，传入 [`transform()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html#transform-com.bumptech.glide.load.engine.bitmap_recycle.BitmapPool-android.graphics.Bitmap-int-int-) 的原始 `Bitmap` 将会自动被回收，只要这个 `Transformation` 返回的 `Bitmap` 和原始传入 [`transoform()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html#transform-com.bumptech.glide.load.engine.bitmap_recycle.BitmapPool-android.graphics.Bitmap-int-int-) 的不是同一个实例。这是和其他加载库很重要的一个不同，例如 Picasso。 [`BitmapTransformation`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html) 提供了 Glide 的资源创建的模板，但它的回收是在内部完成的，所以不管是 `Transformation` 还是 `BitmapTransformation` 都不要回收传入的 `Bitmap` 或 `Resource`。

另外值得注意的是，任何定制的 `BitmapTransformation` 从 `BitmapPool` 中创建、但没有从 [`transform()`](https://muyangmin.github.io/glide-docs-cn/javadocs/431/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html#transform-com.bumptech.glide.load.engine.bitmap_recycle.BitmapPool-android.graphics.Bitmap-int-int-) 返回的中间 `Bitmap`，都会被返回到 `BitmapPool` 或被回收，但不会两种情况同时发生。你永远都不应该 [`recycle()`](https://developer.android.com/reference/android/graphics/Bitmap.html#recycle()) 从 Glide 中创建的 `Bitmap`。

### 12. 调试

#### 请求错误

最高级别和最容易理解的日志都通过 `Glide` 标签打印。Glide 标签将记录成功和失败的请求以及不同级别的详细信息，具体取决于日志级别。VERBOSE 会被用于记录成功的请求，DEBUG则会打印出详细的错误信息。

你也可以通过手动调用 [`setLogLevel(int)`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/GlideBuilder.html#setLogLevel-int-) 方法控制Glide标签的冗余度。`setLogLevel` 允许你–举个栗子–在开发构建(developer builds)时启用更加冗余的日志，而在发布(release builds)构建时则关闭它们。

#### 请求监听器与定制日志

如果你想使用编程的办法跟踪成功和失败信息、跟踪应用中的整体缓存命中率，或增加对本地日志的控制，你可以使用 [`RequestListener`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/request/RequestListener.html) 接口。 `RequestListener` 可以通过 [`RequestBuilder#listener()`](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/RequestBuilder.html#listener-com.bumptech.glide.request.RequestListener-) 方法来添加到单独的加载请求中。下面是一个使用示例：

```java
Glide.with(fragment)
   .load(url)
   .listener(new RequestListener() {
       @Override
       boolean onLoadFailed(@Nullable GlideException e, Object model,
           Target<R> target, boolean isFirstResource) {
         // Log the GlideException here (locally or with a remote logging framework):
         Log.e(TAG, "Load failed", e);

         // You can also log the individual causes:
         for (Throwable t : e.getRootCauses()) {
           Log.e(TAG, "Caused by", t);
         }
         // Or, to log all root causes locally, you can use the built in helper method:
         e.logRootCauses(TAG);

         return false; // Allow calling onLoadFailed on the Target.
       }

       @Override
       boolean onResourceReady(R resource, Object model, Target<R> target,
           DataSource dataSource, boolean isFirstResource) {
         // Log successes here or use DataSource to keep track of cache hits and misses.

         return false; // Allow calling onResourceReady on the Target.
       }
    })
    .into(imageView);
```

请注意，每个 GlideException 都有多个 `Throwable` root cause。在 Glide 中可能有有任意多的方法使得 注册组件(`ModelLoader`, `ResourceDecoder`, `Encoder` 等)作用于从给定的模型（URL, File 等）加载给定的资源 (`Bitmap`, `GifDrawable` 等)。每个 `Throwable` root cause 描述了一个特定的 Glide 组件组合为什么失败。理解某个特定请求为何失败可能需要检查所有的 root cause。

然而，你也可能会发现某个单一的 root cause 比其他的要重要一些。例如你正在加载 URL 并试图找出特定的 HttpException (它意味着你的加载是由于一个网络错误而失败)，你可以遍历所有的 root cause 并使用 `instanceof` 来检查其类型：

```
for (Throwable t : e.getRootCauses()) {
  if (t instanceof HttpException) {
    Log.e(TAG, "Request failed due to HttpException!", t);
    break;
  }
}
```

当然你也可以使用类似的迭代过程和 `instanceof` 操作符来检查 Http 错误之外其他你关心的异常类型。

为减少对象分配起见，你可以为多个加载重用相同的`RequestListener`。

更多请参考：https://muyangmin.github.io/glide-docs-cn/doc/debugging.html

### 13. 从v3迁移到v4

https://muyangmin.github.io/glide-docs-cn/doc/migrating.html



https://muyangmin.github.io/glide-docs-cn/doc/caching.html





### 其它

#### 1. V3.8 版本设置圆角图片

```java
Glide.with(CommonContextHolder.getContext())
            .load(imageItem.getUrl())
            .placeholder(R.drawable.default_cover)
            .crossFade()
            .transform(new GlideRoundTransform(context, 9))
            .into(view);
```

占位图圆角：

```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/info_flow_default_cover_color"/>
    <corners android:radius="@dimen/info_flow_default_cover_border_radius"/>
</shape>
```

加载图片圆角：

```java
/**
 * Class for transform bitmap to round bitmap.
 */
public class GlideRoundTransform extends BitmapTransformation {
    private static float mRadius = 0f;
    public GlideRoundTransform(Context context, int dp) {
        super(context);
        mRadius = context.getResources().getDisplayMetrics().density * dp;
    }

    @Override protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        return roundCrop(pool, toTransform);
    }

    private static Bitmap roundCrop(BitmapPool pool, Bitmap source) {
        if (source == null) return null;

        Bitmap result = pool.get(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
        if (result == null) {
            result = Bitmap.createBitmap(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
        }

        Canvas canvas = new Canvas(result);
        Paint paint = new Paint();
        paint.setShader(new BitmapShader(source, BitmapShader.TileMode.CLAMP, BitmapShader.TileMode.CLAMP));
        paint.setAntiAlias(true);
        RectF rectF = new RectF(0f, 0f, source.getWidth(), source.getHeight());
        canvas.drawRoundRect(rectF, mRadius, mRadius, paint);
        return result;
    }

    @Override public String getId() {
        return getClass().getName() + Math.round(mRadius);
    }
}
```



#### 2. V3 调试

为了在异常发生时可以看到它们，你可以打开Glide中处理所有媒体加载响应的类GenericRequest的log开关。很简单，在命令行运行下面的指令即可：



```css
adb shell setprop log.tag.GenericRequest DEBUG
```

如果你将DEBUG替换为VERBOSE，还可以看到详细的请求时间日志。

如果你想禁用log输出，执行：



```css
adb shell setprop log.tag.GenericRequest ERROR
```

