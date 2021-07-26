## Android辅助功能笔记

[TOC]

### 1. 基本原理

辅助功能（AccessibilityService）其实是一个Android系统提供给的一种服务，本身是继承Service类的。这个服务提供了增强的用户界面，旨在帮助身体不便或者可能暂时无法与设备充分交互的人们。

从开发者的角度看，其实就是提供两种功能：查找界面元素，实现模拟点击。

[官网AccessibilityService](https://developer.android.com/reference/android/accessibilityservice/AccessibilityService)

#### 2. 基本使用

#### 2.1 使用流程

参考下面或 直接看官网：https://developer.android.com/guide/topics/ui/accessibility/service

-  创建自定义的辅助功能服务类; 

  ```java
  package zhiyufen.learn.accessibility;
  
  import android.accessibilityservice.AccessibilityService;
  import android.view.accessibility.AccessibilityEvent;
  
  public class AccessibilityDemoService extends AccessibilityService {
  
      @Override
      protected void onServiceConnected() {
          super.onServiceConnected();
      }
  
      @Override
      public void onAccessibilityEvent(AccessibilityEvent event) {
  
      }
  
      @Override
      public void onInterrupt() {
  
      }
  }
  
  ```

- 配置自己的辅助功服务（注册、配置）

  AndroidManifest注册：

  ```xml
  <!--注册辅助功能服务-->
  <service android:name=".accessibility.AccessibilityDemoService"
      android:label="@string/accessibility_label"
      android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
      <intent-filter>
          <action android:name="android.accessibilityservice.AccessibilityService" />
      </intent-filter>
      <!--通过xml文件完成辅助功能相关配置，也可以在onServiceConnected中动态配置-->
      <meta-data
          android:name="android.accessibilityservice"                       	                   android:resource="@xml/accessibility_config"/>
  </service>
  ```

  accessibility_config.xml文件格式如下：

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
      android:accessibilityEventTypes=""
      android:accessibilityFeedbackType=""
      android:accessibilityFlags=""
      android:canRequestEnhancedWebAccessibility=""
      android:canRequestFilterKeyEvents=""
      android:canRequestTouchExplorationMode=""
      android:canRetrieveWindowContent=""
      android:description=""
      android:notificationTimeout=""
      android:packageNames=""
      android:settingsActivity=""
      android:summary="" />
  ```

  根据自己的需要配置相关属性，比如：

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
      android:canRequestEnhancedWebAccessibility="false"
      android:canRequestFilterKeyEvents="false"
      android:accessibilityFlags="flagDefault|flagRetrieveInteractiveWindows|flagIncludeNotImportantViews"
      android:canRetrieveWindowContent="true"
      android:description="@string/accessibility_description"
      android:notificationTimeout="10"/>
  ```

  如果需要动态设置的话，会需要在AccessibilityDemoService里的onServiceConnected进行设置，不过，使用此方法时，并非所有配置选项都可用。

  ```java
  @Override
  protected void onServiceConnected() {
      super.onServiceConnected();
      AccessibilityServiceInfo info = new AccessibilityServiceInfo();
      info.eventTypes = AccessibilityEvent.TYPES_ALL_MASK;
      info.feedbackType = AccessibilityServiceInfo.FEEDBACK_GENERIC;
      info.notificationTimeout = 100;
      info.packageNames = new String[]{"...", "..."};
      info.flags = AccessibilityServiceInfo.FLAG_REPORT_VIEW_IDS;
      setServiceInfo(info);
  
  ```

- 如果不是系统应用，是无法在代码中直接打开辅助功能的，我们需要跳转设置进行打开：

  ```java
  public class OpenAccessibilitySettingHelper {
      private static final String ACTION = "action";
      private static final String ACTION_START_ACCESSIBILITY_SETTING = "action_start_accessibility_setting";
  
      public static void jumpToSettingPage(Context context) {
          try {
              Intent intent = new Intent(context,  AccessibilityOpenHelperActivity.class);
              intent.putExtra(ACTION, ACTION_START_ACCESSIBILITY_SETTING);
              intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
              context.startActivity(intent);
          } catch (Exception ignore) {}
      }
  }
  ```

#### 2.2 [AccessibilityServiceInfo ](https://developer.android.com/reference/android/accessibilityservice/AccessibilityServiceInfo.html)配置参数说明

- accessibilityEventTypes: 此服务希望接收的事件类型
  | constant                                   | value    | 描述                                                         |
| ------------------------------------------ | -------- | ------------------------------------------------------------ |
| typeAllMask                                | ffffffff | 所有类型的事件                                               |
| typeAnnouncement                           | 4000     | 一个应用产生一个通知事件                                     |
| typeAssistReadingContext                   | 1000000  | 辅助用户读取当前屏幕事件                                     |
| typeContextClicked                         | 800000   | view中上下文点击事件                                         |
| typeGestureDetectionEnd                    | 80000    | 监测到的手势事件完成                                         |
| typeGestureDetectionStart                  | 40000    | 开始手势监测事件                                             |
| typeNotificationStateChanged               | 40       | 收到notification弹出消息事件                                 |
| typeTouchExplorationGestureEnd             | 400      | 触摸浏览事件完成                                             |
| typeTouchExplorationGestureStart           | 200      | 触摸浏览事件开始                                             |
| typeTouchInteractionEnd                    | 200000   | 用户触屏事件结束                                             |
| typeTouchInteractionStart                  | 100000   | 触摸屏幕事件开始                                             |
| typeViewAccessibilityFocusCleared          | 10000    | 无障碍焦点事件清除                                           |
| typeViewAccessibilityFocused               | 8000     | 获得无障碍的焦点事件                                         |
| typeViewClicked                            | 1        | 点击事件                                                     |
| typeViewFocused                            | 8        | view获取到焦点事件                                           |
| typeViewHoverEnter                         | 80       | 一个view的悬停事件                                           |
| typeViewHoverExit                          | 100      | 一个view的悬停事件结束，悬停离开该view                       |
| typeViewLongClicked                        | 2        | view的长按事件                                               |
| typeViewScrolled                           | 1000     | view的滚动事件，adapterview、scrollview                      |
| typeViewSelected                           | 4        | view选中，一般是具有选中属性的view，例如adapter              |
| typeViewTextChanged                        | 10       | edittext中文字发生改变的事件                                 |
| typeViewTextSelectionChanged               | 2000     | edittext文字选中发生改变事件                                 |
| typeViewTextTraversedAtMovementGranularity | 20000    | UIanimator中在一个视图文本中进行遍历会产生这个事件，多个粒度遍历文本。一般用于语音阅读context |
| typeWindowContentChanged                   | 800      | 窗口的内容发生变化，或者更具体的子树根布局变化事件           |
| typeWindowStateChanged                     | 20       | 新的弹出层导致的窗口变化（dialog、menu、popupwindow）        |
| typeWindowsChanged                         | 400000   | 屏幕上的窗口变化事件，需要API                                |

- [`accessibilityFeedbackType`](https://developer.android.com/reference/android/accessibilityservice/AccessibilityServiceInfo.html#attr_android:accessibilityFeedbackType): 此服务提供的反馈类型
  
| constant        | value    | 描述                 |
| --------------- | -------- | ---------------------- |
| feedbackAllMask | ffffffff | 取消所有的可用反馈方式 |
| feedbackAudible | 4        | 可听见的（非语音反馈） |
| feedbackGeneric | 10       | 通用反馈           |
| feedbackHaptic  | 2        | 触觉反馈（震动） |
| feedbackSpoken  | 1        | 语音反馈           |
| feedbackVisual  | 8        | 视觉反馈           |

- accessibilityFlags: 辅助功能附加的标志，多个使用'|'分隔
  | constant                            | value | 描述                                                                                                                                                                                                                                                                                |
| ----------------------------------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| flagDefault                         | 1     | 默认的配置                                                                                                                                                                                                                                                                       |
| flagEnableAccessibilityVolume       | 80    | 这个标志要求系统内所有的音频通道，使用由STREAM_ACCESSIBILTY音量控制USAGE_ASSISTANCE_ACCESSIBILITY                                                                                                                                                             |
| flagIncludeNotImportantViews        | 2     | 表示可获取到一些被表示为辅助功能无权获取到的view                                                                                                                                                                                                                |
| flagReportViewIds                   | 10    | 使用该flag表示可获取到view的ID                                                                                                                                                                                                                                              |
| flagRequestAccessibilityButton      | 100   | 如果辅助功能可用，提供一个辅助功能按钮在系统的导航栏API 26+                                                                                                                                                                                                 |
| flagRequestEnhancedWebAccessibility | 8     | 此类扩展的目的是为WebView中呈现的内容提供更好的辅助功能支持。这种扩展的一个例子是从一个安全的来源注入JavaScript。如果至少有一个具有此标志的辅助功能服务, 则系统将使能增强的web辅助功能。因此, 清除此标志并不保证该设备不会使能增强的web辅助功能, 因为可能有另一个使能的服务在使用它。 |
| flagRequestFilterKeyEvents          | 20    | 能够监听到系统的物理按键                                                                                                                                                                                                                                                  |
| flagRequestFingerprintGestures      | 200   | 监听系统的指纹手势 API 26+                                                                                                                                                                                                                                                   |
| flagRequestTouchExplorationMode     | 4     | 系统进入触控探索模式。出现一个鼠标在用户的界面                                                                                                                                                                                                                 |
| flagRetrieveInteractiveWindows      | 40    | 该标志知识的辅助服务要访问所有交互式窗口内容的系统，这个标志没有被设置时，服务不会收到TYPE_WINDOWS_CHANGE事件。                                                                                                                         |

- canRequestEnhancedWebAccessibility (boolean): 
  辅助功能服务是否能够请求WEB辅助增强的属性。例如: 安装脚本以使应用程序内容更易于访问。

- canRequestFilterKeyEvents (boolean): 
  辅助功能服务是否能够请求过滤KeyEvent的属性，是否可以请求KeyEvent事件流。
  与*flagRequestFilterKeyEvents*搭配使用；

- canRequestTouchExplorationMode (boolean)：
  此属性用于，能够让辅助功能服务通过手势，来请求触摸浏览模式，其被触摸的项，将被朗读出来。
  与*flagRequestTouchExplorationMode*搭配使用；

- canRetrieveWindowContent (boolean)
  辅助功能服务是否能够取回活动窗口内容的属性。 与上边的*flagRetrieveInteractiveWindows**搭配使用，无法在运行时更改此设置。*

- description：
  辅助功能服务目的或行为的简短描述。

- notificationTimeout：
  同一类型的两个辅助功能事件发送到服务的最短间隔（毫秒，两个辅助功能事件之间的最小周期）

- packageNames：
  此服务能接收到事件的软件包名称 (不适合所有软件包)（多个软件包用逗号分隔）。

- settingsActivity：
  允许用户修改辅助功能的activity组件名称

- summary：
  同description

#### 2.3 相关类使用说明

##### AccessibilityService类

常用方法：

- getRootInActiveWindow()
  获取窗体中的节点信息。 返回 `AccessibilityNodeInfo`
  如果需要使用该方法，我们需要在配置中申明 canRetrieveWindowContent，否则可能获取不到窗体的节点信息AccessibilityNodeInfo
- getServiceInfo()/setServiceInfo(AccessibilityServiceInfo info):
  获取/设置本服务的配置信息
- onAccessibilityEvent(AccessibilityEvent event)
  辅助功能事件(s)回调方法
- onInterrupt()
  辅助功能中断的回调
- performGlobalAction(int action)
  全局的点击方法

##### AccessibilityEvent类

当用户界面发生某些明显的事件时，AccessibilityEvent代表的无障碍事件会被系统发送，每一种事件类型是由该类暴露出的属性子集表示其特征的。在此类中为每一种事件类型定义了相应的常量。详细的事件类型请参照[AccessibilityEvent文档 Zh](http://www.siaa.org.cn/home/content/sharedetail/aid/1142)。

##### AccessibilityNodeInfo类

该类代表一个窗口内容节点和可以从源请求的操作。从 AccessibilityService的角度看，一个窗口内容被呈现为一个无障碍节点信息树，该树可能与视图层次一一映射，也可能不与视图层次一一映射。换句话说，一个自定义视图可灵活地将自己报告为一个无障碍节点信息树。一旦无障碍节点信息被发送给无障碍服务，该信息将会是不可改变的，且调用状态改变方法将会产生错误。[AccessibilityNodeInfo文档 Zh](http://www.siaa.org.cn/home/content/sharedetail/aid/1144)

嵌套类

- AccessibilityNodeInfo.AccessibilityAction: (API 21)
  定义了一个可以在 AccessibilityNodeInfo 上执行的操作的类

- AccessibilityNodeInfo.CollectionInfo
  具有一个节点是否是个集合的信息的类。

- AccessibilityNodeInfo.CollectionItemInfo
  具有一个节点是否是个集合项目的信息的类。

- AccessibilityNodeInfo.RangeInfo
  具有一个节点是否是个范围的信息的类。

常用方法

- performAction(Int Action)/performAction(Int Action, Bundle bundle)： 
  执行动作
- findAccessibilityNodeInfosByText(String text)
  使用文本找到 AccessibilityNodeInfo。
- findAccessibilityNodeInfosByViewId(String viewId)
  使用完全合格视图id的源名称找到AccessibilityNodeInfo，完全合格 id 的样式 如下“package:id/id_resource_name”。
- getActionList()
  获取可以在该节点上执行的操作。
- getBoundsInScreen(Rect outBounds)
  获取屏幕坐标中的节点边界。
- getChild(int index)
  获取给定索引下的子元素。
- getChildCount()
  获取子元素的数目。
- getClassName()
  获取该节点来自的类。
- getContentDescription()
  获取该节点的内容描述。
- getPackageName()
  获取该节点来自的包名。
- getParent()
  获取父级NodeInfo。
- getText()
  获取该节点的文本。
- getViewIdResourceName()
  获取源视图 id 的完全合格源名称。
- refresh()
  刷新视图呈现的最新状态信息。返回一个布尔值，表示是否刷新成功。



动作相关定义：
| Action                                                                                 | 释义                                                                                 |
| -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| ACTION_ACCESSIBILITY_FOCUS                                                             | ACTION_ACCESSIBILITY_FOCUS                                                             |
| 给节点添加无障碍焦点的操作。                                             | 给节点添加无障碍焦点的操作。                                             |
| ACTION_ARGUMENT_COLUMN_INT                                                             | ACTION_ARGUMENT_COLUMN_INT                                                             |
| 让指定集合列在屏幕上可见的参数。                                       | 让指定集合列在屏幕上可见的参数。                                       |
| ACTION_ARGUMENT_EXTEND_SELECTION_BOOLEAN                                               | ACTION_ARGUMENT_EXTEND_SELECTION_BOOLEAN                                               |
| 当以一定粒度移动时，是否扩大选择范围或反之移除的参数。      | 当以一定粒度移动时，是否扩大选择范围或反之移除的参数。      |
| ACTION_ARGUMENT_HTML_ELEMENT_STRING                                                    | ACTION_ARGUMENT_HTML_ELEMENT_STRING                                                    |
| 要移动到的下一个/上一个                                                     | 要移动到的下一个/上一个                                                     |
|  HTML 元素的参数。                                                               |  HTML 元素的参数。                                                               |
| ACTION_ARGUMENT_MOVEMENT_GRANULARITY_INT                                               | ACTION_ARGUMENT_MOVEMENT_GRANULARITY_INT                                               |
| 当遍历节点文本的时，使用哪种移动粒度的参数。。                  | 当遍历节点文本的时，使用哪种移动粒度的参数。。                  |
| ACTION_ARGUMENT_PROGRESS_VALUE                                                         | ACTION_ARGUMENT_PROGRESS_VALUE                                                         |
| 指定要设置的进度值的参数。                                                | 指定要设置的进度值的参数。                                                |
| ACTION_ARGUMENT_ROW_INT                                                                | ACTION_ARGUMENT_ROW_INT                                                                |
| 让指定集合行在屏幕上可见的参数。                                       | 让指定集合行在屏幕上可见的参数。                                       |
| ACTION_ARGUMENT_SELECTION_END_INT                                                      | ACTION_ARGUMENT_SELECTION_END_INT                                                      |
| 指定选择结束的参数。                                                         | 指定选择结束的参数。                                                         |
| ACTION_ARGUMENT_SELECTION_START_INT                                                    | ACTION_ARGUMENT_SELECTION_START_INT                                                    |
| 指定选择起始的参数。                                                         | 指定选择起始的参数。                                                         |
| ACTION_ARGUMENT_SET_TEXT_CHARSEQUENCE                                                  | ACTION_ARGUMENT_SET_TEXT_CHARSEQUENCE                                                  |
| 指定要设置的文本内容的参数。                                             | 指定要设置的文本内容的参数。                                             |
| ACTION_CLEAR_ACCESSIBILITY_FOCUS                                                       | ACTION_CLEAR_ACCESSIBILITY_FOCUS                                                       |
| 清除节点无障碍焦点的操作。                                                | 清除节点无障碍焦点的操作。                                                |
| ACTION_CLEAR_FOCUS                                                                     | ACTION_CLEAR_FOCUS                                                                     |
| 清除节点输入焦点的操作。                                                   | 清除节点输入焦点的操作。                                                   |
| ACTION_CLEAR_SELECTION                                                                 | ACTION_CLEAR_SELECTION                                                                 |
| 取消选择节点的操作。                                                         | 取消选择节点的操作。                                                         |
| ACTION_NEXT_AT_MOVEMENT_GRANULARITY                                                    | ACTION_NEXT_AT_MOVEMENT_GRANULARITY                                                    |
| 以给定移动粒度，请求去到该节点文本的下一个文本实体的操作。 | 以给定移动粒度，请求去到该节点文本的下一个文本实体的操作。 |
| ACTION_CLICK                                                                           | ACTION_CLICK                                                                           |
| 在节点信息上点击的操作.                                                     | 在节点信息上点击的操作.                                                     |
| ACTION_COLLAPSE                                                                        | ACTION_COLLAPSE                                                                        |
| 折叠一个可展开节点的操作。                                                | 折叠一个可展开节点的操作。                                                |
| ACTION_COPY                                                                            | ACTION_COPY                                                                            |
| 将当前选择拷贝到剪贴板的操作。                                          | 将当前选择拷贝到剪贴板的操作。                                          |
| ACTION_CUT                                                                             | ACTION_CUT                                                                             |
| 剪贴当前选项并放置到剪贴板的操作。                                    | 剪贴当前选项并放置到剪贴板的操作。                                    |
| ACTION_DISMISS                                                                         | ACTION_DISMISS                                                                         |
| 关闭一个可关闭节点的操作。                                                | 关闭一个可关闭节点的操作。                                                |
| ACTION_EXPAND                                                                          | ACTION_EXPAND                                                                          |
| 展开一个可展开节点的操作。                                                | 展开一个可展开节点的操作。                                                |
| ACTION_FOCUS                                                                           | ACTION_FOCUS                                                                           |
| 给节点添加输入焦点的操作。                                                | 给节点添加输入焦点的操作。                                                |
| ACTION_LONG_CLICK                                                                      | ACTION_LONG_CLICK                                                                      |
| 在节点上点击长按的操作。                                                   | 在节点上点击长按的操作。                                                   |
| ACTION_NEXT_HTML_ELEMENT                                                               | ACTION_NEXT_HTML_ELEMENT                                                               |
| 移动到给定类型的下一个                                                      | 移动到给定类型的下一个                                                      |
|  HTML 元素的操作。例如，移动到 BUTTON、INPUT、TABLE 等。               |  HTML 元素的操作。例如，移动到 BUTTON、INPUT、TABLE 等。               |
| ACTION_PASTE                                                                           | ACTION_PASTE                                                                           |
| 粘贴当前剪贴板内容的操作。                                                | 粘贴当前剪贴板内容的操作。                                                |
| ACTION_PREVIOUS_AT_MOVEMENT_GRANULARITY                                                | ACTION_PREVIOUS_AT_MOVEMENT_GRANULARITY                                                |
| 以给定移动粒度，请求去到该节点文本的上一个文本实体的操作。例如，移动到下一个字、词等。 | 以给定移动粒度，请求去到该节点文本的上一个文本实体的操作。例如，移动到下一个字、词等。 |
| ACTION_PREVIOUS_HTML_ELEMENT                                                           | ACTION_PREVIOUS_HTML_ELEMENT                                                           |
| 移动到给定类型的上一个                                                      | 移动到给定类型的上一个                                                      |
|  HTML 元素的操作。例如，移动到BUTTON、INPUT、TABLE 等。                |  HTML 元素的操作。例如，移动到BUTTON、INPUT、TABLE 等。                |
| ACTION_SCROLL_BACKWARD                                                                 | ACTION_SCROLL_BACKWARD                                                                 |
| 向后滚动节点内容的操作。                                                   | 向后滚动节点内容的操作。                                                   |
| ACTION_SCROLL_FORWARD                                                                  | ACTION_SCROLL_FORWARD                                                                  |
| 向前滚动节点内容的操作                                                      | 向前滚动节点内容的操作                                                      |
| ACTION_SELECT                                                                          | ACTION_SELECT                                                                          |
| 选择节点的操作。                                                               | 选择节点的操作。                                                               |
| ACTION_SET_SELECTION                                                                   | ACTION_SET_SELECTION                                                                   |
| 设置选择项的操作。执行该操作，并且无参数清除选项               | 设置选择项的操作。执行该操作，并且无参数清除选项               |
| ACTION_SET_TEXT                                                                        | ACTION_SET_TEXT                                                                        |
| 设置节点文本的操作。在没有参数的情况下执行该操作，使用      | 设置节点文本的操作。在没有参数的情况下执行该操作，使用      |
|  null 或者空CharSequence 将会清除文本。该操作也将会把光标放置到文本末尾 |  null 或者空CharSequence 将会清除文本。该操作也将会把光标放置到文本末尾 |
| FOCUS_ACCESSIBILITY                                                                    | FOCUS_ACCESSIBILITY                                                                    |
| 无障碍焦点。                                                                     | 无障碍焦点。                                                                     |
| FOCUS_INPUT                                                                            | FOCUS_INPUT                                                                            |
| 输入焦点。                                                                        | 输入焦点。                                                                        |
| MOVEMENT_GRANULARITY_CHARACTER                                                         | MOVEMENT_GRANULARITY_CHARACTER                                                         |
| 以字符为移动粒度位，遍历节点文本                                       | 以字符为移动粒度位，遍历节点文本                                       |
| MOVEMENT_GRANULARITY_LINE                                                              | MOVEMENT_GRANULARITY_LINE                                                              |
| 以行为移动粒度位，遍历节点文本。                                       | 以行为移动粒度位，遍历节点文本。                                       |
| MOVEMENT_GRANULARITY_PAGE                                                              | MOVEMENT_GRANULARITY_PAGE                                                              |
| 以页为移动粒度位，遍历节点文本。                                       | 以页为移动粒度位，遍历节点文本。                                       |
| MOVEMENT_GRANULARITY_PARAGRAPH                                                         | MOVEMENT_GRANULARITY_PARAGRAPH                                                         |
| 以段为移动粒度位，遍历节点文本。                                       | 以段为移动粒度位，遍历节点文本。                                       |
| MOVEMENT_GRANULARITY_WORD                                                              | MOVEMENT_GRANULARITY_WORD                                                              |
| 以字词为移动粒度位，遍历节点文本。                                    | 以字词为移动粒度位，遍历节点文本。                                    |

performGlobalAction：全局动作
| Action                       | 释义                       |
| ---------------------------- | ---------------------------- |
| GLOBAL_ACTION_BACK           | 相当于点击物理按键返回 |
| GLOBAL_ACTION_HOME           | 相当于点击物理按键Home键 |
| GLOBAL_ACTION_NOTIFICATIONS  | 相当于下滑打开通知  |
| GLOBAL_ACTION_RECENTS        | 相当于点击物理按键最近任务键 |
| GLOBAL_ACTION_QUICK_SETTINGS | 打开快速设置           |
| GLOBAL_ACTION_POWER_DIALOG   | 打开长按电源键的弹框 |

#### 2.4 自定义AccessibilityService类的常规使用

```java
public class MyAccessibilityService extends AccessibilityService {
 
    @Override
    protected void onServiceConnected() {
        super.onServiceConnected();
    }
 
    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
        // 1.AccessibilityEvent中一些常用的使用方法
        // 这里我们获取到该辅助功能的事件类型
        // 事件类型请参照 2.1.3中AccessibilityEventTypes表
        int eventType = event.getEventType();
        // 输出事件的字符串type
        String typeStr = event.eventTypeToString(eventType);
        // 根据事件类型来分发我们的需要的操作,这里以窗口变化为例
        if(AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED == eventType){
            // 判断我们的辅助功能是否在约定好的应用界面执行，以设置界面为例
            if("com.android.settings".equals(event.getPackageName()){
                // doSomeThing
            }
        } else if(AccessibilityEvent.TYPE_GESTURE_DETECTION_START == eventType) {
            // 在监测到手势的时候
        } else {
            // 在完成操作时，可以关闭自己的服务，下次使用再次开启。
            // API > = 24
            disableSelf();
        }
 
        // 2.通过event来遍历我们的nodeInfo
        if (VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2)
            // 这里使用getResource()方法其实是从AccessibilityEvent继承的
            // AccessibilityRecord中抽取AccessibilityNodeInfo
            // 实际调用的是AccessibilityRecord中的方法，返回的是AccessibilityNodeInfo mSourceNode
            AccessibilityNodeInfo info = event.getSource();
        else
            info = getRootInActiveWindow();
 
        // 3.遍历info中的子节点
        if (info.getChildCount() ！= 0){
            // 通过一个循环将info的子节点遍历
            for (int i = 0; i < info.getChildCount(); i++) {
                // 获取子节点中某个特定的node，这里通过以下方法通过ID查找
                List<AccessibilityNodeInfo> list = info.findAccessibilityNodeInfosByViewId("com.android" +".settings:id/xxxx");
                // 通过text查找
                List<AccessibilityNodeInfo> list = info.findAccessibilityNodeInfosByText("xxxx");
                // 打印nodeinfo的信息
                Log.e("InfoType: " + info.getClassName());
                Log.e("InfoText: " + info.getText());
                Log.e("InfoPkgName: " + info.getPackageName());
                Log.e("InfoViewId: " + info.getViewIdResourceName());
            }
        }
 
        // 4.为节点添加操作
        // 1) 首先获取到我们的节点
        AccessibilityNodeInfo info = event.getSource();
        // 2) 通过查找指定的ID、text来查找一个系列的节点，返回一个list，需要判断list.size()是否为空
        List<AccessibilityNodeInfo> list = info.findAccessibilityNodeInfosByViewId("pkgName." + "id");
        AccessibilityNodeInfo info = list.get(0);
        // 3) 为节点添加操作，点击事件（事件可参照AccessibilityNodeInfo表）
        info.performAction(AccessibilityNodeInfo.ACTION_CLICK);
        // 4) 添加node可用的action，给node添加一个可清除焦点的操作
        // 通过addAction(AccessibilityNodeInfo.AccessibilityAction action)
        info.addAction(AccessibilityNodeInfo.AccessibilityAction.ACTION_CLEAR_FOCUS);
        // 获取该node上可用的action属性list，可以用来查看该node的属性
        List<AccessibilityNodeInfo.AccessibilityAction> listAction = info.getActionList();
 
        // 5.获取该节点上子节点个数
        int childCount = info.getChildCount();
 
        // 6.操作完成后，回收实例，返回一个下次可用的实例
        info.recycle();
 
        // 需要注意的是，在node.performAction之后，调用本地广播的话，之后的globeAction不会起作用，例：
        sureStopNode.performAction(AccessibilityNodeInfo.ACTION_CLICK);
        sureStopNode.recycle();
        SystemMessage.getInstance().send(SystemMessage.ACTION_POWER_BOOSTER_NEXT);
        performGlobalAction();// 这句话是不起作用的
 
        // 7.辅助功能的一些适用场景：
        // 1)部分应用中获取短信验证码（通过开启辅助功能的方式获取）
        //  - 监听通知栏的消息(typeNotificationStateChanged)
        //  - 弹出通知栏的时候获取该通知的节点信息(event.getSource())
        //  - 遍历root节点，取得显示信息的文本Node(info.findAccessibilityNodeInfosByViewId("pkgName." + "id"))
        //  - 通过NodeInfo.getText()方法获取到相应的文本信息，并取出验证码
        //  - 取消掉通知栏弹出框
        //  - 获取到要输入的EditText获取直接在本应用给需要填写验证码的区域设置文本。
 
        // 2)部分应用中恢复APP的初始设置（清除APP的数据）
        //  - 跳转到应用详情，通过findAccessibilityNodeInfosByViewId查找节点
        //  - 通过节点info.performAction(AccessibilityNodeInfo.ACTION_CLICK)点击清除
        //  - 弹出确定对话框，同上方式找到确定节点，点击后返回
 
        // 3)监测应用是否在前台，APP的启动（通过windowstatechange事件获取到当前的event的pkgName）
        //  - 通过接收typeWindowStateChanged事件，获取event的pkgName确定哪个应用启动或者在最上层显示（悬浮窗不适用）
 
        // 4)自动安装与卸载软件
        //  - 同3，寻找相应的节点，点击事件
 
        // 5)自动化UI测试
 
        // 6)最常见的抢红包
 
        // 7)通过辅助功能开启一些权限(不需要用户手动点击开启了)
        //  - 在用户确定需要开启权限时，自动跳转，寻找相应的开关按钮
        //  - 需要注意的是，部分开关（switch button， checkbox）可能没有ID，需要通过info.getClassName()来判断，属于那种类型的view。
    }
 
    @Override
    public void onInterrupt() {
    }
 
}
```


https://blog.csdn.net/qq_24800377/article/details/78283662

https://blog.csdn.net/feiduclear_up/article/details/70482456

https://developer.android.com/guide/topics/ui/accessibility/service#gather-info