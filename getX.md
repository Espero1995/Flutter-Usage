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

get有两个不同的状态管理器： <font color=red>响应式状态管理器（GetX）</font>和<font color=red>简单状态管理器（GetBuilder）</font>。

#### 1.响应式状态管理器 - 局部自动控制【自动挡 - GetX】

使用 Get 的响应式编程就像使用 setState 一样简单。让我们想象一下，你有一个名称变量，并且希望每次你改变它时，所有使用它的小组件都会自动刷新。

```dart
var name = 'Jonatas Borges';
```

要想让它变得可观察，你只需要在它的末尾加上".obs"。

```dart
var name = 'Jonatas Borges'.obs;
```

在视图中，你只需要把这个变量放在`Obx()`这个Widget里面就可以了。`Obx`是相当聪明的，只有当`controller.name`的值发生变化时才会改变。

```dart
Obx (() => Text (controller.name));
```

当你需要对更新的内容进行**精细的控制时，**GetX()** 可以帮助你。

如果你不需要 "unique IDs"，比如当你执行一个操作时，你的所有变量都会被修改，那么就使用`GetBuilder`。 因为它是一个简单的状态更新器(以块为单位，比如`setState()`)，只用几行代码就能完成。 它做得很简单，对CPU的影响最小，只是为了完成一个单一的目的（一个_State_ Rebuild），并尽可能地花费最少的资源。

✨✨✨声明一个响应式变量：你有3种方法可以把一个变量变成是 "可观察的"。

1 - 第一种是使用 **`Rx{Type}`**。

```dart
// 建议使用初始值，但不是强制性的
final name = RxString('');
final isLogged = RxBool(false);
final count = RxInt(0);
final balance = RxDouble(0.0);
final items = RxList<String>([]);
final myMap = RxMap<String, int>({});
```

2 - 第二种是使用 **`Rx`**，规定泛型 `Rx<Type>`。

```dart
final name = Rx<String>('');
final isLogged = Rx<Bool>(false);
final count = Rx<Int>(0);
final balance = Rx<Double>(0.0);
final number = Rx<Num>(0)
final items = Rx<List<String>>([]);
final myMap = Rx<Map<String, int>>({});
// 自定义类 - 可以是任何类
final user = Rx<User>();
```

3 - 第三种更实用、更简单、更可取的方法，只需添加 **`.obs`** 作为`value`的属性。

```dart
final name = ''.obs;
final isLogged = false.obs;
final count = 0.obs;
final balance = 0.0.obs;
final number = 0.obs;
final items = <String>[].obs;
final myMap = <String, int>{}.obs;
// 自定义类 - 可以是任何类
final user = User().obs;
```

案例：

controller类中：

```dart
class CountController extends GetxController {
  final _count1 = 0.obs;
  set count1(value) => _count1.value = value;
  get count1 => _count1.value;
  final _count2 = 0.obs;
  set count2(value) => _count2.value = value;
	get count2 => _count2.value;
	//求和
  int get sum => _count1.value + count2.value;
}

```

展示页中：

```dart
class StateGetxView extends StatelessWidget {
  StateGetxView({Key? key}) : super(key: key);
  final controller = CountController();
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          children: <Widget>[
            GetX<CountController>(
              init: controller,
              initState: (_) {
                print("初始化：$_");
              },
              builder: (_) {
                print("GetX - 1");
                return Text("value 1【count1】 -> ${_.count1}");
              },
            ),
            GetX<CountController>(
              init: CountController(),
              initState: (_) {},
              builder: (_) {
                print("GetX - 2");
                return Text('value 2【count2】 -> ${_.count2}');
              },
            ),
            GetX<CountController>(
              init: controller,
              initState: (_) {},
              builder: (_) {
                print("GetX - 3");
                return Column(
                  children: [
                    Text('value 3【sum】 -> ${_.sum}'),
                  ],
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}

```

如果我们把`count1.value++`递增，就会打印出来：

- `GetX - 1`
- `GetX - 3`

如果我们改变`count2.value++`，就会打印出来。

- `GetX - 2`
- `GetX - 3`

因为`count2.value`改变了，`sum`的结果现在是`2`。

GetxController中有自己的一套生命周期【workers】，如下

```dart
	class CountController extends GetxController {
  final _count = 0.obs;
  set count(value) => _count.value++;
  get count => _count.value;
  @override
  void onInit() {
    super.onInit();
		// 初始化
    // 每次 可以拿来当购物车使用
    ever(_count, (value) {
      print("every -> " + value.toString());
    });

    // 第一次 用户登录信息 null
    once(_count, (value) {
      print("once -> " + value.toString());
    });

    // 防抖 2s 内
    debounce(
      _count,
      (value) {
        print("debounce -> " + value.toString());
      },
      time: const Duration(seconds: 2),
    );

    // 定时器 1s
    interval(
      _count,
      (value) {
        print("interval -> " + value.toString());
      },
      time: const Duration(seconds: 1),
    );
  }

  @override
  void onReady() {
    super.onReady();
  }

  @override
  void onClose() {
    super.onClose();
  }
    
   @override
  void dispose() {
    super.dispose();
     // controller销毁的地方
  }
}
```



#### 2.简单状态管理器 - 局部手动控制【手动挡 - GetBuilder】

Get有一个极其轻巧简单的状态管理器，它不使用ChangeNotifier，可以满足特别是对Flutter新手的需求，而且不会给大型应用带来问题。

GetBuilder正是针对多状态控制的。想象一下，你在购物车中添加了30个产品，你点击删除一个，同时List更新了，价格更新了，购物车中的徽章也更新为更小的数字。这种类型的方法使GetBuilder成为杀手锏，因为它将状态分组并一次性改变，而无需为此进行任何 "计算逻辑"。GetBuilder就是考虑到这种情况而创建的，因为对于短暂的状态变化，你可以使用setState，而不需要状态管理器。

这样一来，如果你想要一个单独的控制器，你可以为其分配ID，或者使用GetX。这取决于你，记住你有越多的 "单独 "部件，GetX的性能就越突出，而当有多个状态变化时，GetBuilder的性能应该更优越。

用法：

```dart
// 创建控制器类并扩展GetxController。
class Controller extends GetxController {
  int counter = 0;
  void increment() {
    counter++;
    update(); // 当调用增量时，使用update()来更新用户界面上的计数器变量。
  }
}
// 在你的Stateless/Stateful类中，当调用increment时，使用GetBuilder来更新Text。
GetBuilder<Controller>(
  init: Controller(), // 首次启动
  builder: (_) => Text(
    '${_.counter}',
  ),
)
//只在第一次时初始化你的控制器。第二次使用ReBuilder时，不要再使用同一控制器。一旦将控制器标记为 "init "的部件部署完毕，你的控制器将自动从内存中移除。你不必担心这个问题，Get会自动做到这一点，只是要确保你不要两次启动同一个控制器。
```









## 附录 

#### 【1】getX github文档

https://github.com/jonataslaw/getx/blob/master/README.zh-cn.md

#### 【2】getX路由

https://github.com/jonataslaw/getx/blob/master/documentation/zh_CN/route_management.md

#### 【3】getX状态管理 

https://github.com/jonataslaw/getx/blob/master/documentation/zh_CN/state_management.md

#### 【4】依赖注入以及管理 

https://github.com/jonataslaw/getx/blob/master/documentation/zh_CN/dependency_management.md

#### 【5】利用get_cli进行创建带getX的项目

https://github.com/jonataslaw/get_cli/blob/master/README-zh_CN.md











