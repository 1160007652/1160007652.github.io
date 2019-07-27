---
title: 鹰眼视频 (三)
abstract: '欢迎来到我的博客, 这是一篇加密文章。'
message: 请输入密码后按回车键
date: 2019-01-06 16:09:19
tags:
    - flutter
copyright:
password:
---

## Flutter - 鹰眼视频

鹰眼视频 App 相关技术知识点,主要介绍如下几点技巧!

- 状态管理 - bloc
- 动态设置视频比例

<!-- more -->
## 状态管理

Flutter 的状态管理 有多种方案，如：
- RxDart
- Redux
- Bloc

这里只介绍一种，基于Stream概念的状态管理方案。Bloc是对RxDart的进一步封装。对于Flutter有Flutter_bloc粘性组件，负责去呈现最新的状态。

### Bloc 官方概念
> Bloc可以轻松地将表示与业务逻辑分离，使代码快速，易于测试和重用。

Bloc的设计考虑了三个核心价值观：

- 简单
    易于理解，便于使用。

- 强大
    通过组合较小的组件来制作令人惊叹的复杂应用程序。

- 可测试
    轻松测试应用程序的各个方面，以便我们可以放心地进行迭代。

Bloc尝试通过调节何时可以发生状态更改来实现状态更改，并在整个应用程序中强制执行单一方式来更改状态。

由于Bloc是通过Event去传递的。所以我们需要去定义相应的事件源。

```dart
abstract class ThemeEvent {}

class ChangeTheme extends ThemeEvent {
  @override
  String toString() => 'ChangeTheme';
}
```
抽象类 ThemeEvent 是事件祖宗，所有的事件类我们都去继承它。这样在Bloc中就可以方便的判断这个事件是否是 ThemeEvent 类型下的以及 事件传递的 单一原则。避免混淆 想传递多个，泛型是个好东西！

接下来，我们需要再定义ThemeBloc类，这个类必须去继承 Bloc 基础类.
Bloc 支持所有类型， 这里只用到 布尔类型，就不在编写一个 类去做状态存放了。直接使用形参即可。

```dart
class ThemeBloc extends Bloc<ThemeEvent,bool>{

  void changeTheme() async{
    dispatch(ChangeTheme());
  }
  
  @override
  bool get initialState => true;

  @override
  void onTransition(Transition<ThemeEvent, bool> transition) async{
    print(transition.toString());
  }

  @override
  Stream<bool> mapEventToState(bool state, ThemeEvent event) async* {
    if(event is ChangeTheme){
      yield !state;
    }
  }
}
```

onTransition 提供了对stream的过滤操作。

### 反应组件

在ui中去呈现BLoc中状态时，需要使用Flutter_bloc组件库去做桥梁。Flutter_bloc是进一步的封装。

我们需要在一个APP程序的开始处引入Bloc。

> import 'package:flutter_bloc/flutter_bloc.dart';

```dart
class RootPage extends State<Root> {

  ThemeBloc themeBloc = ThemeBloc();
  // new 一个 themeBloc 状态管理对象

  @override
  Widget build(BuildContext context) {

    // 通过 BlocProvider 去注入要处理的 bloc. 这里我们传入themeBloc!
    return BlocProvider<ThemeBloc>(
        bloc: themeBloc,

        // 通过 BlocBuilder 去 监听要实时相应的bloc.这里我们传入themeBloc!
        // 该组件库 还提供了 builder 回掉事件，只要状态发生改变，这里就会自动重新build
        child: BlocBuilder(
          bloc: themeBloc,
          builder: (BuildContext context, bool state) {
            return MaterialApp(
              title: '鹰眼视频',
              // 在这里 去使用状态值， 判断是否为true 实现 主题的重新绑定。
              theme: state ? MyTheme.lightTheme : MyTheme.darkTheme,
              home: new HomePage(),
              routes: <String, WidgetBuilder>{
                '/aboutPage': (BuildContext context) => new AboutPage()
              }
            );
          },
        ));
  }
}
```

### 事件To状态

在这里，我们能感受到 Bloc 的美，它把event转换成了state.

```dart
@override
Widget build(BuildContext context) {

    //  取到上层注入的bloc,可使用 BlocProvider.of<ThemeBloc>(context);

    final ThemeBloc themeBloc = BlocProvider.of<ThemeBloc>(context);
   
    return IconButton(
        icon: new Icon(Icons.remove_red_eye),
        onPressed: () {
            // 在这里去触发我们之前定义的事件，实现状态的改变
            themeBloc.changeTheme();
        }
    );
}

```

### 小结

stream 可以看作一个 污水过滤器， 把污水从一端注入，从另一端流出清水。 污水转成清水 就是onTransition的功劳了，他负责了事件的执行，状态的修改！

其他页面的主题色可以通过**Theme.of(context)**去继承。

## 动态设置视频比例

在前2篇文章中，我介绍了**video_player**视频播放的组件库。

```dart
_videoPlayController = VideoPlayerController.network(_videoInfo.src )
    ..addListener(() {
        // .. 是Dart的语法糖。 链式调用的 写法
        //  根据视频设置 尺寸比例
        final double aRatio = _videoPlayController.value.aspectRatio;
        if (_aRatio != aRatio) {
        setState(() {
            _aRatio = aRatio;
        });
        }
    });
```

想动态改变视频的比列，就需要知道每一个视频的大小。那么我们只需要去添加监听回掉方法即可。
通过**addListener**去实现监听。在**_videoPlayController**控制器中，是存在视频信息的。可以获取到size及aspectRatio.
