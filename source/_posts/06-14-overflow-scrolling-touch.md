title: overflow-scrolling-touch
date: 2016-06-14
tags: 
 - 移动端
---


其实在设置overflow:scroll后，安卓本身的滚动支持已经很流畅，IOS一直是卡卡的。我的初始解决方案是iscroll，但是iscroll我觉得它的使用场景更适合navbar导航和fixed布局,而且不设置回弹的话，IOS效果也并没有那么好。在安卓下，一些国产机如小米魅族，使用iscroll的效果还不如不用，总是会有一些兼容性的问题不那么如意。

然后发现了css属性overflow-scrolling:touch后解决。当然目前安卓overflow-scrolling并没有很好的支持。

```
-webkit-overflow-scrolling: touch;  
 overflow-scrolling: touch;  

```

原理：以下引用自`http://blog.csdn.net/hursing/article/details/9186199`

```
Safari真的用了原生控件来实现，对于有-webkit-overflow-scrolling的网页，会创建一个UIScrollView，提供子layer给渲染模块使用。创建时的堆栈如下：  

```

```
Thread 1, Queue : com.apple.main-thread  
#0	0x00086723 in -[UIScrollView initWithFrame:] ()  
#1	0x004ec3bd in -[UIWebOverflowScrollView initWithLayer:node:webDocumentView:] ()  
#2	0x001f1769 in -[UIWebDocumentView webView:didCreateOrUpdateScrollingLayer:withContentsLayer:scrollSize:forNode:allowHorizontalScrollbar:allowVerticalScrollbar:] ()  
#3	0x01d571bd in __invoking___ ()  
#4	0x01d570d6 in -[NSInvocation invoke] ()  
#5	0x01d5724a in -[NSInvocation invokeWithTarget:] ()  
#6	0x027fb6a1 in -[_WebSafeForwarder forwardInvocation:] ()  
#7	0x027fb8ab in __44-[_WebSafeAsyncForwarder forwardInvocation:]_block_invoke_0 ()  
#8	0x04ac753f in _dispatch_call_block_and_release ()  
#9	0x04ad9014 in _dispatch_client_callout ()  
#10	0x04ac97d5 in _dispatch_main_queue_callback_4CF ()  
#11	0x01d09af5 in __CFRunLoopRun ()  
#12	0x01d08f44 in CFRunLoopRunSpecific ()  
#13	0x01d08e1b in CFRunLoopRunInMode ()  
#14	0x01cbd7e3 in GSEventRunModal ()  
#15	0x01cbd668 in GSEventRun ()  
#16	0x00032ffc in UIApplicationMain ()  
#17	0x00002ae2 in main at /Users/liuhx/Desktop/UIWebView_Research/WebViewResearch/main.mm:16  

```

```
实际创建的是UIWebOverflowScrollView，它继承自UIScrollView，声明为：  

```

```
@class DOMNode, UIWebDocumentView, UIWebOverflowContentView, UIWebOverflowScrollListener;  
  
@interface UIWebOverflowScrollView : UIScrollView  
{  
 UIWebDocumentView *_webDocumentView;  
 UIWebOverflowScrollListener *_scrollListener;  
 UIWebOverflowContentView *_overflowContentView;  
 DOMNode *_node;  
 BOOL _beingRemoved;  
}  
  
@property(nonatomic, getter=isBeingRemoved) BOOL beingRemoved; // @synthesize beingRemoved=_beingRemoved;  
@property(retain, nonatomic) DOMNode *node; // @synthesize node=_node;  
@property(retain, nonatomic) UIWebOverflowContentView *overflowContentView; // @synthesize overflowContentView=_overflowContentView;  
@property(retain, nonatomic) UIWebOverflowScrollListener *scrollListener; // @synthesize scrollListener=_scrollListener;  
@property(nonatomic) UIWebDocumentView *webDocumentView; // @synthesize webDocumentView=_webDocumentView;  
- (void)setContentOffset:(struct CGPoint)arg1;  
- (void)_replaceLayer:(id)arg1;  
- (void)prepareForRemoval;  
- (void)fixUpViewAfterInsertion;  
- (id)superview;  
- (void)dealloc;  
- (id)initWithLayer:(id)arg1 node:(id)arg2 webDocumentView:(id)arg3;  
  
@end  

```

```
其还有一个子View作为ContentView，是给WebCore真正用作渲染overflow型内容的layer的容器。  
UIWebOverflowContentView的声明为：  

```

```
@interface UIWebOverflowContentView : UIView  
{  
}  
  
- (void)_setCachedSubviews:(id)arg1;  
- (void)_replaceLayer:(id)arg1;  
- (void)fixUpViewAfterInsertion;  
- (id)superview;  
- (id)initWithLayer:(id)arg1;  
  
@end  

```

```
再往底层跟，都是CALayer的操作。  
  
以上两个类都是UIKit层的实现，需要WebCore有硬件加速的支持才有实际意义，相关的逻辑被包含在  
  
ACCELERATED_COMPOSITING  
  
这个宏里。  
从SVN log看，在WebKit 108400版本左右才支持，所以iOS Safari应该是需要5.0。Android只在4.0以上支持。  
  
  
从前端开发的角度讲，只需要知道CSS的属性-webkit-overflow-scrolling是真的创建了带有硬件加速的系统级控件，所以效率很高。但是这相对是耗更多内存的，最好在产生了非常大面积的overflow时才应用。  

```

