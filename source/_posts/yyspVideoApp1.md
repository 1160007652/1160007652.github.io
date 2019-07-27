---
title: 鹰眼视频 (二)
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2018-12-23 14:44:15
tags:
    - flutter
copyright:
password:
---

## Flutter - 鹰眼视频

鹰眼视频 App 相关技术知识点,主要介绍如下几点技巧!

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=505820138&auto=0&height=66">
</iframe>

{% asset_img home.png 鹰眼视频 %}

- TabBar
- TabBarView
- DefaultTabController
- tab切换时阻止widget重绘

<!-- more -->
### TabBar

Tab页的Title控件，切换Tab页的入口，一般放到AppBar控件下使用，内部有*Title属性。其子元素按水平横向排列布局，如果需要纵向排列，请使用Column或ListView控件包装一下。子元素为Tab类型的数组。

### TabBarView

Tab页的内容容器，其内放置Tab页的主体内容。子元素可以是多个各种类型的控件。

### DefaultTabController

DefaultTabController，是一个默认的tabView控制器。不需要自己去维护controller,由组件去维护。



### tab切换时阻止widget重绘

因为Flutter为了节约内存不会保存widget的状态，widget都是临时变量。当我们使用TabBar，TabBarView是我们就会发现，切换tab后再重新切换回上一页面，这时候tab会重新加载重新创建，体验很不友好。不过Flutter还是为我们提供了解决办法。我们可以强制widget不显示情况下保留状态，下回再加载时就不用重新创建了。

AutomaticKeepAliveClientMixin 是一个抽象状态，使用也很简单，我们只需要用我们自己的状态继承这个抽象状态，并实现 wantKeepAlive 方法即可。

继承这个状态后，widget在不显示之后也不会被销毁仍然保存在内存中，所以慎重使用这个方法。

```dart
class _VideoList extends State<VideoList> with AutomaticKeepAliveClientMixin {
  
  // 当页面不显示的时候也常驻在内存中
  @protected
  bool get wantKeepAlive=>true;

}
```

### Demo 源码

```dart
import 'package:flutter/material.dart';
import 'package:yysp/view/VideoList.dart';

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {

// 定义需要展示的 选项卡
final List<Tab> myTabs = <Tab>[
    new Tab(text: '推荐'),
    new Tab(text: '搞笑'),
    new Tab(text: '健康'),
    new Tab(text: '美女'),
    new Tab(text: '历史'),
    new Tab(text: '汽车')
  ];

  @override
  Widget build(BuildContext context) {
    return new DefaultTabController(
        length: myTabs.length,
        child: new Scaffold(
            appBar: AppBar(
                bottom: new TabBar(
                  tabs: myTabs,
                  isScrollable: true,
                ),
            ),
            body: new TabBarView(
                children: myTabs.map((Tab tab) {
                    return new Center(
                        // 此处我把VideoList进行了封装，导入使用就可以。
                        child: new VideoList(tab.text)
                    );
                }).toList(),
            )
        )
    );
  }

}
```

### 总结

Dart语言还是有着独特的魅力，总是诱导我走向错误的边缘。Dart是一个强类型的语言可不能按照js的标准去使用，要按照TypeScript标准去适应。
到了Flutter中，就需要大量的使用泛型、Map、List Json与ModelClass的转换，差不多都是java的那一套方法论！