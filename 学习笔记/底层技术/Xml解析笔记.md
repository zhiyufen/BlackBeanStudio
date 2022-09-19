## Xml解析

**XML**的解析方式有四种：

- DOM解析
- SAX解析
- DOM4J解析
- JDOM解析

### DOM解析

### SAX解析

#### 概述

SAX，全称Simple API for XML，是一种以事件驱动的XMl API，是XML解析的一种新的替代方法，解析XML常用的还有DOM解析，PULL解析（Android特有），SAX与DOM不同的是它边扫描边解析，自顶向下依次解析，由于边扫描边解析，所以它解析XML具有速度快，占用内存少的优点，对于Android等CPU资源宝贵的移动平台来说是一个巨大的优势。

- SAX的优点：
  1. 解析速度快
  2. 占用内存少

- SAX的缺点：
  - 无法知道当前解析标签（节点）的上层标签，及其嵌套结构，仅仅知道当前解析的标签的名字和属性，要知道其他信息需要程序猿自己编码
  - 只能读取XML，无法修改XML
  - 无法随机访问某个标签（节点）

- SAX解析适用场合
  - 对于CPU资源宝贵的设备，如Android等移动设备
  - 对于只需从xml读取信息而无需修改xml

#### SAX解析的步骤
解析步骤很简单，可分为以下四个步骤

- 得到xml文件对应的资源，可以是xml的输入流，文件和uri
- 得到SAX解析工厂（SAXParserFactory）
- 由解析工厂生产一个SAX解析器（SAXParser）
- 传入输入流和handler给解析器，调用parse()解析

#### `ContentHandler`接口

一般新建一个类XmlParseHandler.java，该类需要继承DefaultHandler或者实现ContentHandler接口，这里我们通过继承DefaultHandler（实现了ContentHandler接口）的方式，该类是SAX解析的核心所在，我们要重写以下几个我们关心的方法。

```java
//文档解析开始时调用，该方法只会调用一次
startDocument()
//标签（节点）解析开始时调用: uri：xml文档的命名空间; localName：标签的名字; qName：带命名空间的标签的名字; attributes: 标签的属性集
startElement(String uri, String localName, String qName,Attributes attributes)
//解析标签的内容的时候调用: ch：当前读取到的TextNode(文本节点)的字节数组;start：字节开始的位置，为0则读取全部; length：当前TextNode的长度
characters(char[] ch, int start, int length)
//标签（节点）解析结束后调用
endElement(String uri, String localName, String qName)
//文档解析结束后调用，该方法只会调用一次
endDocument()
```



### DOM4J解析

### JDOM解析