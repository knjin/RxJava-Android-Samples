Learning RxJava for Android by example
==============

This is a repository with real-world useful examples of using RxJava with Android. [It usually will be in a constant state of "Work in Progress" (WIP)](http://blog.kaush.co/2014/09/15/learning-rxjava-with-android-by-example/).

I've also been giving talks about Learning Rx using many of the examples listed in this repo.

* [Learning RxJava For Android by Example : Part 1](https://www.youtube.com/watch?v=k3D0cWyNno4) \[[slides](https://speakerdeck.com/kaushikgopal/learning-rxjava-for-android-by-example)\] (SF Android Meetup 2015)
* [Learning Rx by Example : Part 2](https://vimeo.com/190922794) \[[slides](https://speakerdeck.com/kaushikgopal/learning-rx-by-example-2)\] (Øredev 2016)

## Examples:

1. [Background work & concurrency (using Schedulers)](#1-background-work--concurrency-using-schedulers)
2. [Accumulate calls (using buffer)](#2-accumulate-calls-using-buffer)
3. [Instant/Auto searching text listeners (using Subjects & debounce)](#3-instantauto-searching-text-listeners-using-subjects--debounce)
4. [Networking with Retrofit & RxJava (using zip, flatmap)](#4-networking-with-retrofit--rxjava-using-zip-flatmap)
5. [Two-way data binding for TextViews (using PublishSubject)](#5-two-way-data-binding-for-textviews-using-publishsubject)
6. [Simple and Advanced polling (using interval and repeatWhen)](#6-simple-and-advanced-polling-using-interval-and-repeatwhen)
7. [Simple and Advanced exponential backoff (using delay and retryWhen)](#7-simple-and-advanced-exponential-backoff-using-delay-and-retrywhen)
8. [Form validation (using combineLatest)](#8-form-validation-using-combinelatest)
9. [Pseudo caching : retrieve data first from a cache, then a network call (using concat, concatEager, merge or publish)](#9-pseudo-caching--retrieve-data-first-from-a-cache-then-a-network-call-using-concat-concateager-merge-or-publish)
10. [Simple timing demos (using timer, interval or delay)](#10-simple-timing-demos-using-timer-interval-and-delay)
11. [RxBus : event bus using RxJava (using RxRelay (never terminating Subjects) and debouncedBuffer)](#11-rxbus--event-bus-using-rxjava-using-rxrelay-never-terminating-subjects-and-debouncedbuffer)
12. [Persist data on Activity rotations (using Subjects and retained Fragments)](#12-persist-data-on-activity-rotations-using-subjects-and-retained-fragments)
13. [Networking with Volley](#13-networking-with-volley)
14. [Pagination with Rx (using Subjects)](#14-pagination-with-rx-using-subjects)
15. [Orchestrating Observable: make parallel network calls, then combine the result into a single data point (using flatmap & zip)](#15-orchestrating-observable-make-parallel-network-calls-then-combine-the-result-into-a-single-data-point-using-flatmap--zip)
16. [Simple Timeout example (using timeout)](#16-simple-timeout-example-using-timeout)

## Description

### 1. Background work & concurrency (using Schedulers)

A common requirement is to offload lengthy heavy I/O intensive operations to a background thread (non-UI thread) and feed the results back to the UI/main thread, on completion. This is a demo of how long-running operations can be offloaded to a background thread. After the operation is done, we resume back on the main thread. All using RxJava! Think of this as a replacement to AsyncTasks.

The long operation is simulated by a blocking Thread.sleep call (since this is done in a background thread, our UI is never interrupted).

To really see this example shine. Hit the button multiple times and see how the button click (which is a UI operation) is never blocked because the long operation only runs in the background.

### 2. Accumulate calls (using buffer)

This is a demo of how events can be accumulated using the "buffer" operation.

A button is provided and we accumulate the number of clicks on that button, over a span of time and then spit out the final results.

If you hit the button once, you'll get a message saying the button was hit once. If you hit it 5 times continuously within a span of 2 seconds, then you get a single log, saying you hit that button 5 times (vs 5 individual logs saying "Button hit once").

Note:

If you're looking for a more foolproof solution that accumulates "continuous" taps vs just the number of taps within a time span, look at the [EventBus Demo](https://github.com/kaushikgopal/Android-RxJava/blob/master/app/src/main/java/com/morihacky/android/rxjava/rxbus/RxBusDemo_Bottom3Fragment.java) where a combo of the `publish` and `buffer` operators is used. For a more detailed explanation, you can also have a look at this [blog post](http://blog.kaush.co/2015/01/05/debouncedbuffer-with-rxjava/).

### 3. Instant/Auto searching text listeners (using Subjects & debounce)

This is a demo of how events can be swallowed in a way that only the last one is respected. A typical example of this is instant search result boxes. As you type the word "Bruce Lee", you don't want to execute searches for B, Br, Bru, Bruce, Bruce, Bruce L ... etc. But rather intelligently wait for a couple of moments, make sure the user has finished typing the whole word, and then shoot out a single call for "Bruce Lee".

As you type in the input box, it will not shoot out log messages at every single input character change, but rather only pick the lastly emitted event (i.e. input) and log that.

This is the debounce/throttleWithTimeout method in RxJava.

### 4. Networking with Retrofit & RxJava (using zip, flatmap)

[Retrofit from Square](http://square.github.io/retrofit/) is an amazing library that helps with easy networking (even if you haven't made the jump to RxJava just yet, you really should check it out). It works even better with RxJava and these are examples hitting the GitHub API, taken straight up from the android demigod-developer Jake Wharton's talk at Netflix. You can [watch the talk](https://www.youtube.com/watch?v=aEuNBk1b5OE#t=2480) at this link. Incidentally, my motivation to use RxJava was from attending this talk at Netflix.

(Note: you're most likely to hit the GitHub API quota pretty fast so send in an OAuth-token as a parameter if you want to keep running these examples often).

### 5. Two-way data binding for TextViews (using PublishSubject)

Auto-updating views are a pretty cool thing. If you've dealt with Angular JS before, they have a pretty nifty concept called "two-way data binding", so when an HTML element is bound to a model/entity object, it constantly "listens" to changes on that entity and auto-updates its state based on the model. Using the technique in this example, you could potentially use a pattern like the [Presentation View Model pattern](http://martinfowler.com/eaaDev/PresentationModel.html) with great ease.

While the example here is pretty rudimentary, the technique used to achieve the double binding using a `Publish Subject` is much more interesting.

### 6. Simple and Advanced polling (using interval and repeatWhen)

This is an example of polling using RxJava Schedulers. This is useful in cases, where you want to constantly poll a server and possibly get new data. The network call is "simulated" so it forces a delay before return a resultant string.

There are two variants for this:

1. Simple Polling: say when you want to execute a certain task every 5 seconds
2. Increasing Delayed Polling: say when you want to execute a task first in 1 second, then in 2 seconds, then 3 and so on.

The second example is basically a variant of [Exponential Backoff](https://github.com/kaushikgopal/RxJava-Android-Samples#exponential-backoff).

Instead of using a RetryWithDelay, we use a RepeatWithDelay here. To understand the difference between Retry(When) and Repeat(When) I wouuld suggest Dan's [fantastic post on the subject](http://blog.danlew.net/2016/01/25/rxjavas-repeatwhen-and-retrywhen-explained/).

An alternative approach to delayed polling without the use of `repeatWhen` would be using chained nested delay observables. See [startExecutingWithExponentialBackoffDelay in the ExponentialBackOffFragment example](https://github.com/kaushikgopal/RxJava-Android-Samples/blob/master/app/src/main/java/com/morihacky/android/rxjava/fragments/ExponentialBackoffFragment.java#L111).

### 7. Simple and Advanced exponential backoff (using delay and retryWhen)

[Exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff) is a strategy where based on feedback from a certain output, we alter the rate of a process (usually reducing the number of retries or increasing the wait time before retrying or re-executing a certain process).

The concept makes more sense with examples. RxJava makes it (relatively) simple to implement such a strategy. My thanks to [Mike](https://twitter.com/m_evans10) for suggesting the idea.

#### Retry (if error) with exponential backoff

Say you have a network failure. A sensible strategy would be to NOT keep retrying your network call every 1 second. It would be smart instead (nay... elegant!) to retry with increasing delays. So you try at second 1 to execute the network call, no dice? try after 10 seconds... negatory? try after 20 seconds, no cookie? try after 1 minute. If this thing is still failing, you got to give up on the network yo!

We simulate this behaviour using RxJava with the [`retryWhen` operator](http://reactivex.io/documentation/operators/retry.html).

`RetryWithDelay` code snippet courtesy:

* http://stackoverflow.com/a/25292833/159825
* Another excellent implementation via @[sddamico](https://github.com/sddamico) : https://gist.github.com/sddamico/c45d7cdabc41e663bea1
* This one includes support for jittering, by @[leandrofavarin](https://github.com/leandrofavarin) : http://leandrofavarin.com/exponential-backoff-rxjava-operator-with-jitter

Also look at the [Polling example](https://github.com/kaushikgopal/RxJava-Android-Samples#polling-with-schedulers) where we use a very similar Exponential backoff mechanism.

#### "Repeat" with exponential backoff

Another variant of the exponential backoff strategy is to execute an operation for a given number of times but with delayed intervals. So you execute a certain operation 1 second from now, then you execute it again 10 seconds from now, then you execute the operation 20 seconds from now. After a grand total of 3 times you stop executing.

Simulating this behavior is actually way more simpler than the prevoius retry mechanism. You can use a variant of the `delay` operator to achieve this.


### 8. Form validation (using [`.combineLatest`](http://reactivex.io/documentation/operators/combinelatest.html))

Thanks to Dan Lew for giving me this idea in the [fragmented podcast - episode #4](http://fragmentedpodcast.com/episodes/4/) (around the 4:30 mark).

`.combineLatest` allows you to monitor the state of multiple observables at once compactly at a single location. The example demonstrated shows how you can use `.combineLatest` to validate a basic form. There are 3 primary inputs for this form to be considered "valid" (an email, a password and a number). The form will turn valid (the text below turns blue :P) once all the inputs are valid. If they are not, an error is shown against the invalid inputs.

We have 3 independent observables that track the text/input changes for each of the form fields (RxAndroid's `WidgetObservable` comes in handy to monitor the text changes). After an event change is noticed from **all** 3 inputs, the result is "combined" and the form is evaluated for validity.

Note that the `Func3` function that checks for validity, kicks in only after ALL 3 inputs have received a text change event.

The value of this technique becomes more apparent when you have more number of input fields in a form. Handling it otherwise with a bunch of booleans makes the code cluttered and kind of difficult to follow. But using `.combineLatest` all that logic is concentrated in a nice compact block of code (I still use booleans but that was to make the example more readable).


### 9. Pseudo caching : retrieve data first from a cache, then a network call (using concat, concatEager, merge or publish)

We have two source Observables: a disk (fast) cache and a network (fresh) call. Typically the disk Observable is much faster than the network Observable. But in order to demonstrate the working, we've also used a fake "slower" disk cache just to see how the operators behave.

This is demonstrated using 4 techniques:

1. [`.concat`](http://reactivex.io/documentation/operators/concat.html)
2. [`.concatEager`](http://reactivex.io/RxJava/javadoc/rx/Observable.html#concatEager(java.lang.Iterable))
3. [`.merge`](http://reactivex.io/documentation/operators/merge.html)
4. [`.publish`](http://reactivex.io/RxJava/javadoc/rx/Observable.html#publish(rx.functions.Func1)) selector + merge + takeUntil

The 4th technique is probably what you want to use eventually but it's interesting to go through the progression of techniques, to understand why.

`concat` is great. It retrieves information from the first Observable (disk cache in our case) and then the subsequent network Observable. Since the disk cache is presumably faster, all appears well and the disk cache is loaded up fast, and once the network call finishes we swap out the "fresh" results.

The problem with `concat` is that the subsequent observable doesn't even start until the first Observable completes. That can be a problem. We want all observables to start simultaneously but produce the results in a way we expect. Thankfully RxJava introduced `concatEager` which does exactly that. It starts both observables but buffers the result from the latter one until the former Observable finishes. This is a completely viable option.

Sometimes though, you just want to start showing the results immediately. Assuming the first observable (for some strange reason) takes really long to run through all its items, even if the first few items from the second observable have come down the wire it will forcibly be queued. You don't necessarily want to "wait" on any Observable. In these situations, we could use the `merge` operator. It interleaves items as they are emitted. This works great and starts to spit out the results as soon as they're shown.

Similar to the `concat` operator, if your first Observable is always faster than the second Observable you won't run into any problems. However the problem with `merge` is: if for some strange reason an item is emitted by the cache or slower observable *after* the newer/fresher observable, it will overwrite the newer content. Click the "MERGE (SLOWER DISK)" button in the example to see this problem in action. @JakeWharton and @swankjesse contributions go to 0! In the real world this could be bad, as it would mean the fresh data would get overridden by stale disk data.

To solve this problem you can use merge in combination with the super nifty `publish` operator which takes in a "selector". I wrote about this usage in a [blog post](http://blog.kaush.co/2015/01/21/rxjava-tip-for-the-day-share-publish-refcount-and-all-that-jazz/) but I have [Jedi JW](https://twitter.com/JakeWharton/status/786363146990649345) to thank for reminding of this technique. We `publish` the network observable and provide it a selector which starts emitting from the disk cache, up until the point that the network observable starts emitting. Once the network observable starts emitting, it ignores all results from the disk observable. This is perfect and handles any problems we might have.

Previously, I was using the `merge` operator but overcoming the problem of results being overwritten by monitoring the "resultAge". See the old `PseudoCacheMergeFragment` example if you're curious to see this old implementation.

### 10. Simple timing demos (using timer, interval and delay)

This is a super simple and straightforward example which shows you how to use RxJava's `timer`, `interval` and `delay` operators to handle a bunch of cases where you want to run a task at specific intervals. Basically say NO to Android `TimerTask`s.

Cases demonstrated here:

1. run a single task after a delay of 2s, then complete
2. run a task constantly every 1s (there's a delay of 1s before the first task fires off)
3. run a task constantly every 1s (same as above but there's no delay before the first task fires off)
4. run a task constantly every 3s, but after running it 5 times, terminate automatically
5. run a task A, pause for sometime, then execute Task B, then terminate

### 11. RxBus : event bus using RxJava (using RxRelay (never terminating Subjects) and debouncedBuffer)  

There are accompanying blog posts that do a much better job of explaining the details on this demo:

1. [Implementing an event bus with RxJava](http://blog.kaush.co/2014/12/24/implementing-an-event-bus-with-rxjava-rxbus/)
2. [DebouncedBuffer used for the fancier variant of the demo](http://blog.kaush.co/2015/01/05/debouncedbuffer-with-rxjava/)
3. [share/publish/refcount](http://blog.kaush.co/2015/01/21/rxjava-tip-for-the-day-share-publish-refcount-and-all-that-jazz/)

### 12. Persist data on Activity rotations (using Subjects and retained Fragments)

A common question that's asked when using RxJava in Android is, "how do i resume the work of an observable if a configuration change occurs (activity rotation, language locale change etc.)?".

This example shows you one strategy viz. using retained Fragments. I started using retained fragments as "worker fragments" after reading this [fantastic post by Alex Lockwood](http://www.androiddesignpatterns.com/2013/04/retaining-objects-across-config-changes.html) quite sometime back.

Hit the start button and rotate the screen to your heart's content; you'll see the observable continue from where it left off.

*There are certain quirks about the "hotness" of the source observable used in this example. Check [my blog post](http://blog.kaush.co/2015/07/11/a-note-about-the-warmth-share-operator/) out where I explain the specifics.*

I have since rewritten this example using an alternative approach. While the [`ConnectedObservable` approach worked](https://github.com/kaushikgopal/RxJava-Android-Samples/blob/master/app/src/main/java/com/morihacky/android/rxjava/fragments/RotationPersist1WorkerFragment.java#L20) it enters the lands of "multicasting" which can be tricky (thread-safety, .refcount etc.). Subjects on the other hand are far more simple.  You can see it rewritten [using a `Subject` here](https://github.com/kaushikgopal/RxJava-Android-Samples/blob/master/app/src/main/java/com/morihacky/android/rxjava/fragments/RotationPersist2WorkerFragment.java#L22).

I wrote [another blog post](https://tech.instacart.com/how-to-think-about-subjects-part-1/) on how to think about Subjects where I go into some specifics.


### 13. Networking with Volley

[Volley](http://developer.android.com/training/volley/index.html) is another networking library introduced by [Google at IO '13](https://www.youtube.com/watch?v=yhv8l9F44qo). A kind citizen of github contributed this example so we know how to integrate Volley with RxJava.

### 14. Pagination with Rx (using Subjects)

I leverage the simple use of a Subject here. Honestly, if you don't have your items coming down via an `Observable` already (like through Retrofit or a network request), there's no good reason to use Rx and complicate things.

This example basically sends the page number to a Subject, and the subject handles adding the items. Notice the use of `concatMap` and the return of an `Observable<List>` from `_itemsFromNetworkCall`.

For kicks, I've also included a `PaginationAutoFragment` example, this "auto-paginates" without us requiring to hit a button. It should be simple to follow if you got how the previous example works.

Here are some other fancy implementations (while i enjoyed reading them, i didn't land up using them for my real world app cause personally i don't think it's necessary):

* [Matthias example of an Rx based pager](https://gist.github.com/mttkay/24881a0ce986f6ec4b4d)
* [Eugene's very comprehensive Pagination sample](https://github.com/matzuk/PaginationSample)
* [Recursive Paging example](http://stackoverflow.com/questions/28047272/handle-paging-with-rxjava)

### 15. Orchestrating Observable: make parallel network calls, then combine the result into a single data point (using flatmap & zip)

The below ascii diagram expresses the intention of our next example with panache. f1,f2,f3,f4,f5 are essentially network calls that when made, give back a result that's needed for a future calculation.


             (flatmap)
    f1 ___________________ f3 _______
             (flatmap)               |    (zip)
    f2 ___________________ f4 _______| ___________  final output
            \                        |
             \____________ f5 _______|

The code for this example has already been written by one Mr.skehlet in the interwebs. Head over to [the gist](https://gist.github.com/skehlet/9418379) for the code. It's written in pure Java (6) so it's pretty comprehensible if you've understood the previous examples. I'll flush it out here again when time permits or I've run out of other compelling examples.

### 16. Simple Timeout example (using timeout)

This is a simple example demonstrating the use of the `.timeout` operator. Button 1 will complete the task before the timeout constraint, while Button 2 will force a timeout error.

Notice how we can provide a custom Observable that indicates how to react under a timeout Exception.

## Rx 2.x

All the examples here have been migrated to use RxJava 2.X.

* Have a look at [PR #83 to see the diff of changes between RxJava 1 and 2](https://github.com/kaushikgopal/RxJava-Android-Samples/pull/83/files)
* [What's different in Rx 2.x](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0)

We use [David Karnok's Interop library](https://github.com/akarnokd/RxJava2Interop) in some cases as certain libraries like RxBindings, RxRelays, RxJava-Math etc. have not been ported yet to 2.x.

## Contributing:

I try to ensure the examples are not overly contrived but reflect a real-world usecase. If you have similar useful examples demonstrating the use of RxJava, feel free to send in a pull request.

I'm wrapping my head around RxJava too so if you feel there's a better way of doing one of the examples mentioned above, open up an issue explaining how. Even better, send a pull request.

## License

Licensed under the Apache License, Version 2.0 (the "License").
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

You agree that all contributions to this repository, in the form of fixes, pull-requests, new examples etc. follow the above-mentioned license.

以Android为例学习RxJava

这是一个使用Android使用RxJava的真实世界有用示例的存储库。它通常处于“正在进行中”（WIP）的不断状态。

我也使用这个回购中列出的许多例子来谈论学习Rx。

学习RxJava对于Android，例如：第1部分 [ 幻灯片 ]（SF Android Meetup 2015）
通过示例学习Rx：第2部分 [ 幻灯片 ]（Øredev2016）
例子：

后台工作和并发（使用调度程序）
累积通话（使用缓冲区）
即时/自动搜索文本听众（使用主题和去抖动）
网络与Retrofit和RxJava（使用zip，flatmap）
TextViews的双向数据绑定（使用PublishSubject）
简单和高级轮询（使用间隔和重复）
简单高级指数退避（使用延迟和重试）
表单验证（使用combineLatest）
伪缓存：首先从缓存中检索数据，然后进行网络调用（使用concat，concatEager，merge或publish）
简单的时序演示（使用定时器，间隔或延迟）
RxBus：使用RxJava的事件总线（使用RxRelay（从不终止主题）和debouncedBuffer）
持续关于活动旋转的数据（使用主题和保留的片段）
与排球网络
使用Rx分页（使用主题）
协调可观察：进行并行网络调用，然后将结果合并到单个数据点（使用flatmap＆zip）
简单的超时示例（使用超时）
描述

后台工作和并发（使用调度程序）

一个常见的要求是将冗长的I / O密集型操作卸载到后台线程（非UI线程），并将结果反馈到UI /主线程。这是一个可以将多长时间运行的操作卸载到后台线程的演示。操作完成后，我们恢复主线程。所有使用RxJava！认为这是AsyncTasks的替代品。

长时间的操作是通过阻塞Thread.sleep调用来模拟的（因为这是在后台线程中完成的，我们的UI永远不会中断）。

要真正看到这个例子闪耀。多次按下按钮，看看按钮单击（这是一个UI操作）是不会被阻止的，因为长操作只能在后台运行。

2.收集电话（使用缓冲区）

这是一个使用“缓冲”操作可以累积事件的演示。

提供一个按钮，我们在一段时间内累积该按钮的点击次数，然后排除最终结果。

如果您按下按钮一次，您将收到一条消息，指示按钮被击中一次。如果你在2秒的时间内连续5次，那么你会得到一个单一的日志，说你点击该按钮5次（对5个单独的日志说“按钮一次”）。

注意：

如果你正在寻找的是积累了更多的万无一失的解决方案“连续”水龙头VS的时间跨度内，水龙头的只是数量，看看EventBus演示其中的一个组合publish，并buffer使用运营商。有关更详细的解释，您还可以看看这篇博文。

3.即时/自动搜索文本听众（使用主题和去抖动）

这是一个演示，说明事件如何被吞下，只有最后一个被尊重的方式。一个典型的例子是即时搜索结果框。当你输入“李小龙”这个词的时候，你不想执行B，Br，Bru，Bruce，Bruce，Bruce L等的搜索，但是要等到几分钟的时间才能确定用户打完整个字，然后单击“李小龙”。

当您输入输入框时，它不会在每个单个输入字符更改时触发日志消息，而只是选择最后发送的事件（即输入）并记录该日志消息。

这是RxJava中的debounce / throttleWithTimeout方法。

4.使用Retrofit和RxJava进行联网（使用zip，flatmap）

从Square改造是一个令人惊奇的图书馆，帮助轻松的网络（即使你还没有跳到RxJava，你真的应该检查出来）。它使用RxJava更好的工作，这些都是GitHub API的例子，它们直接来自Android demigod开发人员Jake Wharton在Netflix的演讲。您可以在此链接中观看演讲。顺便说一下，我使用RxJava的动机来自于Netflix的这次演讲。

（注意：您很可能非常快速地触及GitHub API配额，因此如果您希望经常继续运行这些示例，请发送OAuth-token作为参数）。

5. TextViews的双向数据绑定（使用PublishSubject）

自动更新视图是一件非常酷的事情。如果您之前已经处理过Ang JS，那么它们有一个非常漂亮的概念，称为“双向数据绑定”，所以当一个HTML元素绑定到一个模型/实体对象时，它会不断地“监听”该实体的变化，根据模型自动更新其状态。使用本例中的技术，您可以轻松地使用“ 演示视图模型”模式。

虽然这里的例子很简单，但用来实现双重绑定Publish Subject的技术更加有趣。

6.简单高级轮询（使用间隔和重复）

这是使用RxJava调度程序进行轮询的示例。这在您想不断轮询服务器并可能获取新数据的情况下很有用。网络呼叫是“模拟的”，所以它强制一个延迟，然后返回一个合成的字符串。

这有两个变种：

简单的轮询：每当你想要每5秒钟执行一个特定的任务
增加延迟轮询：说出要在1秒钟之后首先执行任务，然后在2秒钟，然后3秒等等。
第二个例子基本上是指数退避的变体。

而不是使用RetryWithDelay，我们在这里使用RepeatWithDelay。要了解重试（When）和重复（When）之间的区别，我会建议丹的这个主题的梦幻般的帖子。

在不使用延迟轮询的另一种方法repeatWhen将是使用链式嵌套延迟可观察值。请参阅ExponentialBackOffFragment示例中的startExecutingWithExponentialBackoffDelay。

7.简单高级指数退避（使用延迟和重试）

指数退避是一种策略，其基于来自某个输出的反馈，我们改变进程的速率（通常减少重试次数或者在重试或重新执行某个进程之前增加等待时间）。

这个概念比较有道理。RxJava使得（相对）简单地实现这样的策略。我感谢迈克提出这个想法。

以指数回退重试（如果错误）

说你有一个网络故障。一个明智的策略是不要每隔1秒重试你的网络呼叫。这将是聪明的（不，优雅！）以越来越多的延迟重试。所以你尝试在第二次执行网络通话，没有骰子？尝试10秒...否定？20秒后尝试，没有cookie？1分钟后尝试。如果这件事情仍然失败，你必须放弃网络哟！

我们使用RxJava与retryWhen运算符模拟此行为。

RetryWithDelay 代码片段礼貌：

http://stackoverflow.com/a/25292833/159825
通过@ sddamico的另一个很好的实现：https：//gist.github.com/sddamico/c45d7cdabc41e663bea1
这包括@ leandrofavarin的抖动支持：http : //leandrofavarin.com/exponential-backoff-rxjava-operator-with-jitter
也看轮询例子，我们用一个非常相似的指数回退机制。

“重复”指数退避

指数退避策略的另一个变体是执行给定次数的操作，但是具有延迟的间隔。所以你从现在开始1秒钟执行某个操作，然后再从现在开始再执行一次，然后从现在开始20秒钟就执行该操作。总共3次​​总共停止执行。

模拟这种行为实际上比prevoius重试机制更简单。您可以使用delay运算符的变体来实现此目的。

8.表单验证（使用.combineLatest）

感谢Dan Lew在分散的播客 - 第4集（4:30左右）给我这个想法。

.combineLatest允许您在单个位置一次紧凑地监视多个可观测量的状态。演示的示例显示了如何使用它.combineLatest来验证基本形式。该表格有3个主要输入被认为是“有效的”（电子邮件，密码和号码）。一旦所有输入都有效，表单将变为有效（下面的文字变为蓝色：P）。如果不是，则会针对无效输入显示错误。

我们有3个独立的可观察者跟踪每个表单字段的文本/输入更改（RxAndroid WidgetObservable可以方便地监视文本更改）。在所有 3个输入都注意到事件发生变化后，结果为“合并”，并对表单进行有效性评估。

请注意，Func3检查有效性的功能仅在ALL 3输入接收到文本更改事件后才启动。

当您在表单中输入更多的输入字段时，此技术的值将变得更加明显。否则用一堆布尔处理它使得代码变得混乱，很难跟随。但是使用.combineLatest所有这些逻辑集中在一个很好的紧凑的代码块（我仍然使用布尔，但这是使示例更易读）。

9.伪缓存：首先从缓存中检索数据，然后进行网络调用（使用concat，concatEager，merge或publish）

我们有两个源观察器：磁盘（快速）缓存和网络（新鲜）调用。通常，“可观察”磁盘比网络“可观察”更快。但是为了演示工作，我们还使用了一个假的“较慢”磁盘缓存来查看操作员的行为。

这是使用4种技术进行演示：

.concat
.concatEager
.merge
.publish selector + merge + takeUntil
第四种技术可能是您最终想要使用的技术，但是通过技术的进步，了解原因是很有趣的。

concat是很棒的。它从第一个Observable（在我们的情况下是磁盘缓存）中检索信息，然后从后面的网络“可观察”中检索信息。由于磁盘缓存可能更快，所有这些都显得很好，并且磁盘缓存加载速度很快，一旦网络呼叫完成，我们会更换“新鲜”的结果。

问题concat在于，在第一个Observable完成之后，后续的observable甚至不会启动。这可能是一个问题。我们希望所有可观测资料同时开始，但以我们预期的方式产生结果。感谢RxJava介绍concatEager哪个是正确的。它启动两个可观察值，但缓冲后者的结果，直到前Observable完成。这是一个完全可行的选择。

有时候，您只是想立即开始显示结果。假设第一个可观察（因为一些奇怪的原因）需要很长的时间来运行所有的项目，即使第二个可观察的前几个项目已经下线，它将被强制排队。你不一定要等待任何可观察的。在这些情况下，我们可以使用merge运算符。它们在发射时对其进行交织。这很有效，一旦显示，就开始吐出结果。

与concat操作员类似，如果您的第一个Observable总是比第二个Observable更快，那么您将不会遇到任何问题。但是问题merge在于：如果出于某种奇怪的原因，缓存中会发出一个项目，或者在较新/更新的可观察器之后发现较慢的可观察值，则会覆盖较新的内容。单击示例中的“MERGE（SLOWER DISK）”按钮，以查看此问题。@JakeWharton和@swankjesse贡献转到0！在现实世界中，这可能是坏的，因为这意味着新的数据将被旧的磁盘数据覆盖。

为了解决这个问题，你可以使用合并与超级漂亮的publish操作符，它带有一个“选择器”。我在博客文章中写了这个用法，但是我有Jedi JW感谢提醒这个技巧。我们publish网络可观察，并提供一个选择器，开始从磁盘缓存发射，直到网络可观察开始发射的点。一旦网络可观察开始发射，它将忽略磁盘可见的所有结果。这是完美的，并处理我们可能遇到的任何问题。

以前，我使用的是merge运算符，但是通过监视“resultAge”来克服结果被覆盖的问题。看到老PseudoCacheMergeFragment例如，如果你很好奇，看看这个老的实现。

10.简单的时序演示（使用定时器，间隔和延迟）

这是一个超级简单明了的例子展示了如何使用RxJava的timer，interval和delay运营商来处理一堆案件要在特定时间间隔运行任务。基本上说不是对Android TimerTask的。

这里展示的案例如下：

在延迟2秒后运行一个任务，然后完成
每1秒持续运行任务（在第一个任务启动之前有1秒的延迟）
每1秒不间断运行任务（与上述相同但在第一个任务启动之前没有任何延迟）
每3秒运行一次任务，但运行5次后，自动终止
运行任务A，暂停一段时间，然后执行任务B，然后终止
11. RxBus：使用RxJava的事件总线（使用RxRelay（从不终止主题）和debouncedBuffer）

随附的博客文章做了更好的工作来解释这个演示的细节：

使用RxJava实现事件总线
DebouncedBuffer用于演示者的爱好者变体
分享/发布/引用计数
12.持续关于活动旋转的数据（使用主题和保留的片段）

在Android中使用RxJava时遇到的常见问题是“如果发生配置更改（活动轮换，语言区域变更等），我如何恢复可观察的工作？”

这个例子显示了一个策略，即 使用保留的片段。在Alex Lockwood读了这个梦幻般的帖子后，我开始使用保留的片段作为“工作者片段” 。

点击开始按钮，并将屏幕旋转到您的心脏的内容; 你会看到可观察的继续从它离开的地方。

关于本示例中使用的源可观察的“热度”有一定的怪癖。检查我的博客文章我在哪里解释具体细节。

我已经用另一种方法重写了这个例子。虽然这种ConnectedObservable方法工作，它进入“组播”的土地，这可能很棘手（线程安全，.refcount等）。另一方面，学科更简单。你可以看到它使用Subject这里改写。

我写了另一篇关于如何思考主题的博客文章。

与Volley进行联网

Volley是Google在IO 13推出的另一个网络图书馆。一个善良的github公民贡献了这个例子，所以我们知道如何将Volley与RxJava整合。

使用Rx分页（使用主题）

我利用这里的主题的简单使用。老实说，如果您没有通过Observable已经（通过改造或网络请求）下载您的项目，则没有任何理由使用Rx并使事情复杂化。

该示例基本上将页码发送到主题，并且主题处理添加项目。注意使用concatMap和的回归Observable<List>从_itemsFromNetworkCall。

对于踢，我也PaginationAutoFragment举了一个例子，这个“自动分页”，而我们不需要按一个按钮。如果你以前的例子如何工作，应该很简单。

这里有一些其他花哨的实现（虽然我喜欢阅读他们，我没有着陆使用他们为我的真实世界的应用程序，因为个人我不认为这是必要的）

基于Rx的寻呼机的Matthias示例
尤金非常全面的分页样本
递归寻呼示例
15.协调可观察：进行并行网络调用，然后将结果合并成单个数据点（使用flatmap＆zip）

下面的ascii图表示了我们下一个例子的意图。f1，f2，f3，f4，f5本质上是网络呼叫，当做出，回馈未来计算所需的结果。

         (flatmap)
f1 ___________________ f3 _______
         (flatmap)               |    (zip)
f2 ___________________ f4 _______| ___________  final output
        \                        |
         \____________ f5 _______|
这个例子的代码已经由一个Mr.skehlet在Interwebs中写过。转到代码的要点。它是用纯Java（6）编写的，所以如果你已经理解了以前的例子，这是非常可以理解的。当时间允许时，我会再次将其清除，或者我已经用尽了其他令人信服的例子。

16.简单超时示例（使用超时）

这是一个简单的示例，演示了.timeout操作员的使用。按钮1将在超时限制之前完成任务，而按钮2将强制超时错误。

请注意，我们如何提供一个自定义的Observable，指示如何在超时异常情况下作出反应。

Rx 2.x

这里的所有示例都已迁移到使用RxJava 2.X.

看看PR＃83看到RxJava 1和2之间的变化的差异
Rx 2.x有什么不同？
在某些情况下，我们使用David Karnok的Interop库，因为像RxBindings，RxRelays，RxJava-Math等某些库尚未移植到2.x.

贡献：

我试图确保这些例子不是过分的设计，而是反映了一个现实世界的使用。如果您有类似的示例演示使用RxJava的有用示例，请随时在拉取请求中发送。

我也围绕RxJava包围我的头，所以如果你觉得有更好的方式做一个上面提到的例子，打开一个问题解释如何。更好的是发送拉请求。

执照

根据Apache许可证版本2.0（“许可证”）许可。您可以获得许可证的副本

http://www.apache.org/licenses/LICENSE-2.0

除非适用法律要求或书面同意，根据许可证分发的软件以“按现有”的基础分发，不附有明示或暗示的任何形式的担保或条件。有关许可证下的权限和限制的特定语言，请参阅许可证。

您同意，以上述许可证的方式，对该存储库的所有贡献，以修复，拉取请求，新示例等形式。
