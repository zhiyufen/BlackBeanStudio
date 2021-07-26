## Android MeasureSpec笔记

在安卓View的measure过程中，measureSpec扮演着重要的作用。MeasureSpec是一个32位的int，前2位代表模式，其中00代表UNSPECIFIED，01代表EXACTLY，10代表ATMOST，后30位代表具体大小，例如1073741824模式为EXACTLY,大小为1080。这三种模式的具体意思如下图：

![](https://img-blog.csdnimg.cn/20190908173413534.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9xaW5tdXh1ZQ==,size_16,color_FFFFFF,t_70)





本文笔记来自：https://blog.csdn.net/zhaoqinmuxue/article/details/100631265