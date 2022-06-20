## 【一】 国际化

1、在pubspec.yaml文件中我们需要dart 本身的国际化，在dependencies: 下加入

```yaml
dependencies:
  flutter_localizations:
      sdk: flutter
```

2、新增文件夹lang，locales（对应的各个语言字典：eg. locale_en.dart/locale_zh.dart等）以及新增index.dart【多语言文件入口】、translation.dart【多语言的配置项】、locale_keys.dart【语言字典对应的key】

locale_keys.dart：

```dart
/// 多语言 keys
class LocaleKeys {
  /// 测试
  static const testLogin = 'login';
}
```

locale_en.dart/locale_zh.dart:

````dart
/// 多语言 英文
Map<String, String> localeEn = {
  LocaleKeys.testLogin: 'logged in as @name with email @email',
};
// 多语言 简体中文
Map<String, String> localeZh = {
  LocaleKeys.testLogin: '登录用户 @name，邮箱账号 @email',
};	
````

translation.dart:

```dart
class Translation extends Translations {
  // 获取设备语言
  static Locale? get locale => Get.deviceLocale;
  //以下均需要在main.dart文件里进行配置
  static const fallbackLocale = Locale('en', 'US');//添加一个回调语言选项，以备上面指定的语言翻译不存在
  static const supportedLocales = [
    Locale('en', 'US'),
    Locale('zh', 'CN'),
  ];
  static const localizationsDelegates = [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ];
  @override
  Map<String, Map<String, String>> get keys => {
        'en': localeEn,
        'zh': localeZh,
      };
}
```

在main.dart文件中进行注入 - GetMaterialApp 国际化。

```dart
return GetMaterialApp(
      // 国际化
    	translations: Translation(), // 词典，你的翻译
      localizationsDelegates: Translation.localizationsDelegates, // 代理
      supportedLocales: Translation.supportedLocales, // 支持的语言种类
      locale: ui.window.locale == 'en_US'
          ? const Locale('en', 'US')
          : const Locale('zh', 'CN'), //ui.window.locale【系统语言】, // 将会按照此处指定的语言翻译
      fallbackLocale: Translation.fallbackLocale, // 添加一个回调语言选项，以备上面指定的语言翻译不存在
    );
```



## 【二】路由

首先，我们新建一个routers的路由文件夹，其中再新建app_routes.dart【路由名称】和app_pages.dart【路由名称所指向的对应页】两个文件。

app_roures.dart:

```dart
part of 'app_pages.dart';

//路由命名
abstract class AppRoutes {
  ///页面不存在
  static const notFoundRoute = '/notfound';
  //首页
  static const homeRoute = '/home';
  static const listRoute = '/list';
  static const detailRoute = '/detail';
  static const detailRoute_ID = '/detail/:id'; //参数传值
  static const loginRoute = '/login';
  static const myRoute = '/my';
}
```

app_pages.dart:

```dart
part 'app_routes.dart';
class AppPages {
  static const initRoute = AppRoutes.homeRoute; //初始化路由
  //所有路由路径
  static final routes = [
    /// 登录页面【白名单】
    GetPage(
      name: AppRoutes.loginRoute,
      page: () => const LoginView(),
    ),

    /// 需要登录 中间件控制是否需要授权才可以继续，详细业务在router_auth拦截
    GetPage(
      name: AppRoutes.myRoute,
      page: () => const MyView(),
      middlewares: [
        //数组 - 可以依靠priority调整优先级
        RouteAuthMiddleware(priority: 1),
      ],
    ),
    ///子页面======================
    // Home > List > Detail
    GetPage(
      name: AppRoutes.homeRoute,
      page: () => const HomeView(), 
      children: [
        GetPage(
          name: AppRoutes.listRoute,
          page: () => const ListIndexView(),
          children: [
            GetPage(
              name: AppRoutes.detailRoute, // /home/list/detail
              page: () => const DetailView(),
            ),
            GetPage(
              name: AppRoutes.detailRoute_ID, // /home/list/detail/123
              page: () => const DetailView(),
            ),
          ],
        ),
      ],
    ),
    //Other Page...
  ];

  /// 404 NotFound
  static final notFoundRoute = GetPage(
    name: AppRoutes.notFoundRoute, // /notfound
    page: () => const NotFoundView(),
    transition: Transition.fadeIn,
  );
}

```

对应的使用：

```dart
// 1.导航到新的页面
Get.toNamed("/home/list");

// 2.跟app_pages.dart里的一样具有层级关系。
Get.toNamed("/home/list/detail");

// 3.关闭SnackBars、Dialogs、BottomSheets或任何你通常会用Navigator.pop(context)关闭的东西。
Get.back();

// 4.进入下一个页面，但没有返回上一个页面的选项（用于SplashScreens闪屏页，登录页面等）
Get.off(const DetailView());

// 5.进入下一个界面并取消之前的所有路由（在购物车、投票和测试中很有用）
Get.offAll(const DetailView());

// 6.带值传入【两种传值方式 arguments / 链接中带参(适合web使用) 】 以及 pop时返回值
// (1)传值方式一：arguments
// A Page: 记得加async
var result = await Get.toNamed("/home/list/detail", arguments: {"id": 999});
print("result:${result["key"]}");
// B Page:
Map? arguments = Get.arguments;// 接收参数
Get.back(result: {"key": "value"});// 返回值方式

// (2)链接中带参(适合web使用)
// A Page: 记得加async
var result = await Get.toNamed("/home/list/detail?id=666");// /home/list/detail/666
print("result:${result["key"]}");     
// B Page:
Map? parameters = Get.parameters;// 接收参数
Get.back(result: {"key": "value"});// 返回值方式


// 7.要处理到未定义路线的导航（404错误），可以在GetMaterialApp中定义unknownRoute页面
Get.toNamed("/notfound"); // 前端使用居多
// 返回方式 - 动态网页链接的形式
// Get提供高级动态URL，就像在Web上一样。Web开发者可能已经在Flutter上想要这个功能了，Get也解决了这个问题。
Get.offAllNamed(AppRoutes.homeRoute);
```

在main.dart文件中进行注入 - GetMaterialApp路由。

```dart
return GetMaterialApp(
      initialRoute: AppPages.initRoute, //默认路由
      getPages: AppPages.routes, //路由表
      unknownRoute: AppPages.notFoundRoute, //未找到路由
    );
```

另外：有些界面需要路由拦截才可以进行展示，以“我的”为例，如果没有鉴权点击我的 => 登录页，否则就展示登录页。如下：

router_auth.dart

```dart
class RouteAuthMiddleware extends GetMiddleware {
  @override
  int? priority = 0; // 优先级
  RouteAuthMiddleware({ this.priority});
  @override
  RouteSettings? redirect(String? route) {
    print("route:$route");
    // 加自己的业务，若没有登录，去登录！！！
    bool isAuth = false;
    // ignore: dead_code
    if (!isAuth) {
      Future.delayed(
        const Duration(seconds: 1),
        () => Get.snackbar("提示", "请先登录APP"),
      );
      return const RouteSettings(name: AppRoutes.loginRoute);
    }
  }
}

// 使用 - 会自动进入路由拦截
Get.toNamed('/my'); 
```



## 【三】状态管理 🚀✨





## 附录 

#### 【1】getX github文档

https://github.com/jonataslaw/getx/blob/master/README.zh-cn.md

#### 【2】getX路由

https://github.com/jonataslaw/getx/blob/master/documentation/zh_CN/route_management.md

#### 【3】getX状态管理 

https://github.com/jonataslaw/getx/blob/master/documentation/zh_CN/state_management.md

#### 【4】依赖注入以及管理 

https://github.com/jonataslaw/getx/blob/master/documentation/zh_CN/dependency_management.md

####【5】利用get_cli进行创建带getX的项目

https://github.com/jonataslaw/get_cli/blob/master/README-zh_CN.md







