## 自定义View杂事

[TOC]

### Paint字体的使用

在Android SDK中使用Typeface类来定义字体，能够经过经常使用字体类型名称进行设置，如设置默认黑体： canvas;

  * Typeface.DEFAULT //常规字体类型 字体
  * Typeface.DEFAULT_BOLD //黑体字体类型 spa
  * Typeface.MONOSPACE //等宽字体类型 code
  * Typeface.SANS_SERIF //sans serif字体类型 对象
  * Typeface.SERIF //serif字体类型 图片

```java
Paint mp = new paint();
mp.setTypeface(Typeface.DEFAULT_BOLD)
```

除了字体类型设置以外，还能够为字体类型设置字体风格，如设置粗体：

```java
Paint mp = new Paint();
//String familyName = “宋体”;
//Typeface font = Typeface.create(familyName,Typeface.BOLD);
Typeface font = Typeface.create(Typeface.SANS_SERIF, Typeface.BOLD);
p.setTypeface( font );
```

经常使用的字体风格名称还有： 

  * Typeface.BOLD //粗体 字符串

  * Typeface.BOLD_ITALIC //粗斜体

  * Typeface.ITALIC //斜体

  * Typeface.NORMAL //常规j

可是有时上面那些设置在绘图过程当中是不起做用的，因此还有以下设置方式:

```
Paint mp = new Paint();
mp.setFakeBoldText(true); //true为粗体，false为非粗体
mp.setTextSkewX(-0.5f); //float类型参数，负数表示右斜，整数左斜
mp.setUnderlineText(true); //true为下划线，false为非下划线
mp.setStrikeThruText(true); //true为删除线，false为非删除线
```

更多方法可参考：https://www.jianshu.com/p/1fc5f6d771ef

