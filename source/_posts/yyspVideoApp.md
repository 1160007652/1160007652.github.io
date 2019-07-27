---
title: 鹰眼视频 (一)
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2018-11-04 20:00:30
tags:
    - flutter
copyright:
password:
---

## Flutter - 鹰眼视频

鹰眼视频 App 相关技术知识点

{% asset_img yyspflutterhome.png 鹰眼视频 %}

- 目录结构
- 页面拆封
- 封装 dio < Flutter http 插件 >
- 使用懒加载图片特效
- json自动序列化 model 映射
- ListView 组件
- GestureDetector 手势
- 路由导航
- 视频播放

### 目录结构

 ```
lib
    comments    // 组件封装
    model       // 序列号model
    page        // 页面
    serves      // http 相关的 目录 
    view        // 存放一些 组件组合的 控件
    main.dart   // Flutter入口dart文件 
 ```

 <!-- more -->

### 页面拆封

#### main.dart 入口文件内容

 ``` dart
import 'package:flutter/material.dart';
import 'package:yysp/page/homePage.dart';

// Flutter 入口main函数
void main()=>runApp(new RootPage());

class RootPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: '鹰眼视频',
      home: new HomePage()
    );
  }
}

 ```

RootPage组件类是根入口,用于汇集所有的 page页面。同时页面的路由配置就可以放到这个RootPage类中去管理。

导入其他文件时的2中方法，如下。

> import 'package:yysp/page/homePage.dart';
> import './page/homePage.dart';


#### homePage.dart 首页

该文件存放在 ** /page/ ** 目录下, ** page ** 目录用于管理所有的页面资源。

``` dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {

    @override
    Widget build(BuildContext context) {
        return new MaterialApp(
            home: new Scaffold(
                appBar: AppBar(
                    title: Text(
                        "鹰眼视频",
                        style: new TextStyle(fontSize: 20.0, letterSpacing: 3.0),
                    ),
                ),
                body: new Text('我是首页'),
            ),
        );
    }

    @override
    void initState() {
        super.initState();
    }
}
```

这是一个最基本的有状态组件,有状态组件可以做交互、让组件可控。如react中的 this.setState()后，组件就会重新绘制。不过Flutter中的修改状态值的方法是 ** setState((){ }) ** ,与React 一样。

### 封装自己的dio

Api.dart、httpUtil.dart 文件都存放在 ** /servers/**目录下。

Api.dart 文件内容

```dart
class Api {
    // 基础 Url
  static const String BaseUrl = 'https://436086407.dmi.net.cn/weapp/';
    // 视频列表
  static const String VideList = '$BaseUrl/videolist';
}
```

httpUtil.dart 文件内容

```dart
import 'package:dio/dio.dart';
import 'package:yysp/servers/Api.dart';

class HttpUtil {
  static HttpUtil instance;
  Dio dio;
  Options options;

  static HttpUtil getInstance() {
    print('getInstance');
    if (instance == null) {
      instance = new HttpUtil();
    }
    return instance;
  }

  HttpUtil() {
    print('dio赋值');
    options = Options(
      baseUrl: Api.BaseUrl,
      connectTimeout: 10000,
      receiveTimeout: 3000,
      headers: {},
    );
    dio = new Dio(options);
  }

  get(url, {data, options, cancelToken}) async {
    print('get请求启动! url：$url ,body: $data');
    Response response;
    try {
      response = await dio.get(
        url,
        data: data,
        cancelToken: cancelToken,
      );
      print('get请求成功!response.data：${response.data}');
    } on DioError catch (e) {
      if (CancelToken.isCancel(e)) {
        print('get请求取消! ' + e.message);
      }
      print('get请求发生错误：$e');
    }
    return response.data;
  }

  post(url, {data, options, cancelToken}) async {
    print('post请求启动! url：$url ,body: $data');
    Response response;
    try {
      response = await dio.post(
        url,
        data: data,
        cancelToken: cancelToken,
      );
      print('post请求成功!response.data：${response.data}');
    } on DioError catch (e) {
      if (CancelToken.isCancel(e)) {
        print('post请求取消! ' + e.message);
      }
      print('post请求发生错误：$e');
    }
    return response.data;
  }
}

```

这里httpUtil类，采用了单列模式。可以想一下 ** java ** 中连接数据库JDBC的情景。

### 使用懒加载图片特效

实现这个效果，需要使用第三方的插件。 在 ** pubspec.yaml ** 中添加。

Flutter与node 使用第三方库的区别。

- node 可以 npm i xxx ,也可以在package 中直接写库名，然后执行 npm i。
- flutter 需要手动写到 pubspec.yaml中，在执行 flutter packages get 。

加载图片特效插件：transparent_image

封装图片组件，代码示意图

```dart
// 图片懒惰加载
Widget _buildImage(String imgUrl) {
    return new FadeInImage.memoryNetwork(
        placeholder: kTransparentImage,
        image: imgUrl,
        height: 200.0,
        fit: BoxFit.fitWidth,
        fadeInDuration: const Duration(milliseconds: 300),
        fadeOutDuration: const Duration(milliseconds: 300),
    );
}
```

### json自动序列化 model 映射

我们在向服务器请求数据后，服务器往往会返回一段json字符串。而我们要想更加灵活的使用数据的话需要把json字符串转化成对象。由于flutter只提供了json to Map。而手写反序列化在大型项目中极不稳定，很容易导致解析失败。所以flutter团队推荐使用json_serializable 自动反序列化。

使用第三方库： json_annotation: ^2.0.0 \ build_runner \ json_serializable

```dart
dependencies:
  flutter:
    sdk: flutter
  json_annotation: ^2.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^1.1.0
  json_serializable: ^2.0.0

```

#### flutter中如何解析json对象

假如我们的mock数据是如下形式。

```json
{
    "id" : 8863,
    "list" : [ 8952, 9224, 8917 ],
    "score" : 111,
}

```
那么我们需要根据相应的json数据去创建对应的model实体类

```dart
import 'package:json_annotation/json_annotation.dart';

class Data{
  final int id;
  @JsonKey(name: 'list')
  final List<int> listData;
  final int score;

  Data({this.id, this.listData, this.score});
}
```
在这里使用了 ** @JsonKey(name: 'list) ** ,原因是，因为我们的json数据中使用了list关键字,所以我们给起个别名listData。并且与Json的list字断映射！

#### 生成Json解析文件

在这里，我们需要使用 ** build_runner ** 去生成dart代码，** build_runner **是dart团队提供的一个生成dart代码文件的外部包。

当前项目的目录下运行

> flutter packages pub run build_runner build

运行成功后，就可以在我们的model实体类下面看见一个叫 ** Data.g.dart ** 的文件，内容如下：

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'data.dart';

// **************************************************************************
// JsonSerializableGenerator
// **************************************************************************

Data _$DataFromJson(Map<String, dynamic> json) {
  return Data(
      id: json['id'] as int,
      kids: (json['kids'] as List)?.map((e) => e as int)?.toList(),
      score: json['score'] as int);
}

Map<String, dynamic> _$DataToJson(Data instance) => <String, dynamic>{
      'id': instance.id,
      'kids': instance.kids,
      'score': instance.score
    };

```
> 注释 ** GENERATED CODE - DO NOT MODIFY BY HAND" **, 意思是不要手写生成这个文件，使用根据去解决。

#### 关联实体类文件

```dart
import 'package:json_annotation/json_annotation.dart';

part 'data.g.dart';

@JsonSerializable()
class Data{
    final int id;
    @JsonKey(name: 'list')
    final List<int> listData;
    final int score;

    Data({this.id, this.listData, this.score});

    //反序列化
    factory Data.fromJson(Map<String, dynamic> json) => _$DataFromJson(json);
    //序列化
    Map<String, dynamic> toJson() => _$DataToJson(this);
}

```

### ListView 组件

在Flutter中,用ListView来显示列表项,支持垂直和水平方向展示,通过一个属性我们就可以控制其方向，在这里只做大数据的渲染介绍。

``` dart
import 'package:flutter/material.dart';
import 'package:yysp/servers/httpUtil.dart';
import 'package:yysp/servers/Api.dart';
import 'package:yysp/model/videos.dart';

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
    HttpUtil httpUtil = new HttpUtil();
    List<dynamic> _videosList = new List(); // 视频列表数据
    int _pageNo = 0; // 开始页
    @override
    Widget build(BuildContext context) {
        return new MaterialApp(
            home: new Scaffold(
                appBar: AppBar(
                    title: Text(
                        "鹰眼视频",
                        style: new TextStyle(fontSize: 20.0, letterSpacing: 3.0),
                    ),
                ),
                // 这是listView 实现 大数据渲染的 方法
                body: new ListView.builder(
                    itemCount: _videosList.length,
                    itemBuilder: this._buildRow,
                ),
            ),
        );
    }

    @override
    void initState() {
        super.initState();
    }

    // listView 子项 itemBuilder build方法 传入2个参数，( context 上下文, position 下标 - 计数器)
    Widget _buildRow(BuildContext context, int position) {
        print(_videosList[position].toString());
        return new Card(
            margin: EdgeInsets.only(bottom: 38.0),
            child: new Container(
                height: 256.0,
                child: this._buildImage(_videosList[position]['img'])
            )
        );
    }

    @override
    void initState() {
        super.initState();
        _getContent(_pageNo);
    }

    //获取网络数据
    void _getContent(pageNo) async {
        String url = Api.VideList;
        var data = {'page': pageNo};
        var response = await HttpUtil().get(url, data: data);
        Videos videos = Videos.fromJson(response);
        if (videos.code == 0) {
            setState(() {
                _videosList.addAll(videos.data.listData);
                _pageNo = pageNo + 1;
            });
        }
    }
}
```

上面介绍了 如何http 获取数据，并且把json数据映射到model实体类。其实listView最主要的就是，做大数据渲染。

```dart
//ListView 核心
new ListView.builder(
    itemCount: _videosList.length,
    itemBuilder: this._buildRow,
),

itemCount: 表示列表项的总数。
itemBuilder： 渲染子项的方法，使其list列表中显示不同的组件界面。
```
### GestureDetector 手势

GestureDetector 手势控件没有图像展示，只是检测用户输入的手势。当用户点击Container时，GestureDetector会调用onTap回调。也可以使用GestureDetector检测各种输入手势，包括点击、拖动和缩放。不过也有许多的控件使用GestureDetector为其他控件提供回调，比如IconButton、RaisedButton和FloatingActionButton控件有onPressed回调，当用户点击控件时触发回调。

#### 手势事件

|属性/回调 |描述|
|------|------|
|onTapDown|每次用户与屏幕联系时都会触发 OnTapDown|
|onTapUp|当用户停止触摸屏幕时，onTapUp 被调用|
|onTap|当短暂触摸屏幕时，onTap 被触发|
|onTapCancel|当用户触摸屏幕但未完成 Tap 时，将触发此事件|
|onDoubleTap|当屏幕被快速连续触摸两次时调用 onDoubleTap|
|onLongPress|用户触摸屏幕超过 500毫秒 时，onLongPress 被触发|
|onVerticalDragDown|当指针与屏幕接触并开始沿垂直方向移动时，onVerticalDown 被调用|
|onVerticalDragStart|当指针 开始 沿垂直方向移动时调用 onVerticalDragStart|
|onVerticalDragUpdate|每次指针在屏幕上的位置发生变化时都会调用此方法|
|onVerticalDragEnd|当用户停止移动时，拖动被认为是完成的，将调用此事件|
|onVerticalDragCancel|当用户突然停止拖动时调用|
|onHorizontalDragDown|当用户/指针与屏幕接触并开始水平移动时调用|
|onHorizontalDragStart|用户/指针已与屏幕接触并 开始 沿水平方向移动|
|onHorizontalDragUpdate|每次指针在水平方向/x轴上的位置发生变化时调用|
|onHorizontalDragEnd|在水平拖动结束时，将调用此事件|
|onHorizontalDragCancel|当指针未成功触发 onHorizontalDragDown 时调用|
|onPanDown|当指针与屏幕接触时调用|
|onPanStart|指针事件开始移动时，onPanStart 触发|
|onPanUpdate|每次指针改变位置时，调用 onPanUpdate|
|onPanEnd|平移完成后，将调用此事件|
|onScaleStart|当指针与屏幕接触并建立 1.0 的焦点时，将调用此事件|
|onScaleUpdate|与屏幕接触的指针指示了新的焦点|
|onScaleEnd|当指针不再与指示手势结束的屏幕接触时调用|

#### onTap 事咧
这里以onTap手势事件，为例。

```dart
// 传参数的手势。
new GestureDetector(
    child:Container(
        width: double.infinity,
        height: 200.0,
        child: this._buildImage(_videosList[position]['img']), // 返回的是自定义的图片组件
    ),
    onTap: this._videoPlayToggle(_videosList[position]['id']), // 这里是点击图片触发的事件，并且传入一个参数
),

Function _videoPlayToggle(id) {
    return () => parint('我是onTab事件，参数 $id');
}
```

```dart
// 不传参数的手势。
new GestureDetector(
    child:Container(
        width: double.infinity,
        height: 200.0,
        child: Text('无参数'), // 返回的是自定义的图片组件
    ),
    onTap: this._videoPlayToggle(), // 这里是点击图片触发的事件，并且传入一个参数
),

void _videoPlayToggle() {
    parint('我是onTab事件，无参数');
}
```

### 路由导航

在上节手势介绍文章中介绍了onTab事件，那么现在我们就看看app应用的多页面切换吧。看看Flutter的页面切换与 ** React-router ** 有什么不一样。

在Flutter中有着两种路由跳转的方式，一种是静态路由，在创建时就已经明确知道了要跳转的页面和值。另一种是动态路由，跳转传入的目标地址和要传入的值都可以是动态的。

#### 静态路由

```dart

import 'package:flutter/material.dart';
import 'package:yysp/page/homePage.dart';
import 'package:yysp/page/aboutPage.dart';

void main()=>runApp(new RootPage());

class RootPage extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
        return new MaterialApp(
            title: '鹰眼视频',
            home: new HomePage(),
            routes: <String, WidgetBuilder>{
                '/aboutPage': (BuildContext context) => new AboutPage() // 静态路由配置
            }
        );
    }
}
```
在MaterialApp中，是存在一个叫routers的参数的，用于配置静态路由。

#### 动态路由

在手势的小节中讲解了，如何去使用手势事件。那么这节继续扩充。点击事件后，让其携带参数切换页面。

```dart
Function _videoPlayToggle(id) { // 这里接收的是，事件传递的参数。
    return () => Navigator.of(context).push(
        new PageRouteBuilder(
            pageBuilder: (BuildContext context,
                Animation<double> animation, // 切换的动画。
                Animation<double> secondaryAnimation) {
                return new DetailsPage(id.toString()); // 这里负责把事件参数，传递到下一个页面。
            }
        )
    );
  }
```

原则就是：“简单的学习，快速的掌握”。至于动画效果，后续优化。先跑下框架结构！

> 下一个页面的，参数接收。

其实就是，通过构造方法实例话对象时传进去。就是这么简单。

```dart
class DetailsPage extends StatefulWidget {
    final String videoId; // 路由携带的 播放id 参数

    DetailsPage(this.videoId); // 构造方法

    @override
    _DetailsPageState createState() => _DetailsPageState(this.videoId); // 把 参数再次向下传入。
}

class _DetailsPageState extends State<DetailsPage> {
  final String _videoId; // 这里就是 传入的参数，这样就使用视频ID，在相应的生命周期中去获取数据了。
  _DetailsPageState(this._videoId); // 构造方法

```

记住，类对象必须构造方法传参数。
### 视频播放

视频播放使用了这2个插件，video_player、chewie

```dart
import 'package:flutter/material.dart';
import 'package:video_player/video_player.dart';
import 'package:chewie/chewie.dart';
import 'package:yysp/servers/httpUtil.dart';
import 'package:yysp/servers/Api.dart';
import 'package:yysp/model/video_detail.dart';

class DetailsPage extends StatefulWidget {
    final String videoId; // 路由携带的 播放id 参数

    DetailsPage(this.videoId);

    @override
    _DetailsPageState createState() => _DetailsPageState(this.videoId);
}

class _DetailsPageState extends State<DetailsPage> {
    final String _videoId;
    Video _videoInfo = new Video(0, '', '', '', '', '', '', '', '', '', 0, 0, '');
    // 实例化播放控制器，通过这个控制器去实现播放、暂停、视频切换。
    VideoPlayerController _videoPlayController = new VideoPlayerController.network('');
    HttpUtil httpUtil = new HttpUtil();
    _DetailsPageState(this._videoId);

    @override
    Widget build(BuildContext context) {
        return new MaterialApp(
            home: new Scaffold(
                appBar: AppBar(
                    title: Text(
                        "鹰眼视频",
                        style: new TextStyle(fontSize: 20.0, letterSpacing: 3.0),
                    ),
                    ),
                    body: Column(
                    children: <Widget>[
                        Container(
                        child: this._buildVideo()
                        ),
                        Text(_videoInfo.title)
                    ],
                )
            )
        );
    }
  

    @override
    void initState() {
        super.initState();
        this._getVideoInfo();
    }

    @override
    void dispose() {
        super.dispose();
        _videoPlayController.pause();
    }

    //获取网络数据 视频播放详情
    void _getVideoInfo() async {
        String url = Api.VideoInfo;
        var data = {'id': this._videoId};
        var response = await HttpUtil().get(url, data: data);
        VideoDetail videos = VideoDetail.fromJson(response);
        if (videos.code == 0) {
        // 视频获取回来，就初始化 播放控制器
        _videoPlayController = VideoPlayerController.network( videos.data.video.src );
        setState(() {
            _videoInfo = new Video(videos.data.video.id, videos.data.video.title, videos.data.video.url, videos.data.video.auther, videos.data.video.pubtime, videos.data.video.img, videos.data.video.src, videos.data.video.desc1, videos.data.video.cate, videos.data.video.pt, videos.data.video.collect, videos.data.video.count, videos.data.video.insertTime);
        });
        }
    }

    // 视频 组件，传递了VideoPlayController控制器
    Widget _buildVideo() {
        _videoPlayController.play();
        return new Chewie(
            _videoPlayController,
            aspectRatio: 3 / 2,
            autoPlay: true,
            looping: false,
        );
    }
}

```
####
总结：说了这么多，其实Flutter学的是框架，最主要的是学会dart语言。