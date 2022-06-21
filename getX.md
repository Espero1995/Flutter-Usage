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

当你需要对更新的内容进行**精细的控制时，**GetX() 可以帮助你。

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

✨✨✨注意：你可能想要一个更大的规模，而不是上面方式使用init属性。为此，你可以创建一个类并扩展Bindings类，并在其中添加将在该路由中创建的控制器。控制器不会在那个时候被创建，相反，这只是一个声明，这样你第一次使用Controller时，Get就会知道去哪里找。Get会保持懒加载，当不再需要Controller时，会自动移除它们。请看pub.dev的例子来了解它是如何工作的。

如果你导航了很多路由，并且需要之前使用的Controller中的数据，你只需要再用一次GetBuilder（没有init）。

```dart
class OtherClass extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: GetBuilder<Controller>(
          builder: (s) => Text('${s.counter}'),
        ),
      ),
    );
  }
```

如果你需要在许多其他地方使用你的控制器，并且在GetBuilder之外，只需在你的控制器中创建一个get，就可以轻松地拥有它。(或者使用`Get.find<Controller>()`)

```dart
class Controller extends GetxController {
  /// 你不需要这个，我推荐使用它只是为了方便语法。
  /// 用静态方法：Controller.to.increment()。
  /// 没有静态方法的情况下：Get.find<Controller>().increment();
  /// 使用这两种语法在性能上没有区别，也没有任何副作用。一个不需要类型，另一个IDE会自动完成。
  static Controller get to => Get.find(); // 添加这一行 / 这一行很关键🚀🚀🚀
  int counter = 0;
  void increment() {
    counter++;
    update();
  }
}
```

然后你可以直接访问你的控制器，这样：

```dart
FloatingActionButton(
  onPressed: () {
    Controller.to.increment(),// 使用方式🚀🚀🚀
  }
  child: Text("${Controller.to.counter}"),// 使用方式🚀🚀🚀
),
```

当你按下FloatingActionButton时，所有监听'counter'变量的widget都会自动更新。

<font color=red size=5>GetBuilder - 唯一的ID:</font>

如果你想用GetBuilder完善一个widget的更新控件，你可以给它们分配唯一的ID。

```dart
GetBuilder<Controller>(
  id: 'text', //这里，设置一个唯一的ID
  init: Controller(), // 每个控制器只用一次
  builder: (_) => Text(
    '${Get.find<Controller>().counter}', //here
  ),
),
```

并更新它：

```dart
update(['text']);
```

您还可以为更新设置条件。例如counter大于10时候触发更新。

```dart
update(['text'], counter < 10);
```

GetX会自动进行重建，并且只重建使用被更改的变量的小组件，如果您将一个变量更改为与之前相同的变量，并且不意味着状态的更改，GetX不会重建小组件以节省内存和CPU周期（界面上正在显示3，而您再次将变量更改为3。在大多数状态管理器中，这将导致一个新的重建，但在GetX中，如果事实上他的状态已经改变，那么widget将只被再次重建）



#### 3.其他 - 控制器的生命周期

GetxController中有自己的一套生命周期【workers】，如下：

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

	/// 关闭流用onClose方法，而不是dispose
  @override
  void onClose() {
    super.onClose();
    // controller销毁的地方
  }
    
}
```

控制器的生命周期。

- onInit()是创建控制器的地方。
- onClose()，关闭控制器，为删除方法做准备。
- deleted: 你不能访问这个API，因为它实际上是将控制器从内存中删除。它真的被删除了，不留任何痕迹。



## 【四】依赖管理🚀✨

- 注意：如果你使用的是Get的状态管理器，请多注意绑定api，这将使你的界面更容易连接到你的控制器。

```dart
Controller controller = Get.put(Controller()); // 而不是 Controller controller = Controller();
```

你是在Get实例中实例化它，而不是在你使用的类中实例化你的类，这将使它在整个App中可用。 所以你可以正常使用你的控制器（或类Bloc）。

**提示：** Get依赖管理与包的其他部分是解耦的，所以如果你的应用已经使用了一个状态管理器（任何一个，都没关系），你不需要全部重写，你可以使用这个依赖注入。

```dart
controller.fetchApi();
```

想象一下，你已经浏览了无数条路由，现在你需要拿到一个被遗留在控制器中的数据，Get会自动为你的控制器找到你想要的数据，而你甚至不需要任何额外的依赖关系。

```dart
Controller controller = Get.find();
//	是的，它看起来像魔术，Get会找到你的控制器，并将其提供给你。你可以实例化100万个控制器，Get总会给你正确的控制器。
//  static Controller get to => Get.find(); // 同样的，在 “简单状态管理器” 章节中也介绍了此方法，可以在类里写明这句话，整个App中你就可沟通类来调用方法变量了。
//  eg. Controller.to.xx属性/xx方法()
```

然后你就可以恢复你在后面获得的控制器数据。

```dart
Text(controller.textFromApi);
```

### <font color=red><u>1.实例方法：</u></font> 

这些方法和它的可配置参数是：

#### （1）Get.put

最常见的插入依赖关系的方式。例如，对于你的视图的控制器来说：

```dart
Get.put<SomeClass>(SomeClass());
Get.put<LoginController>(LoginController(), permanent: true);
Get.put<ListItemController>(ListItemController, tag: "some unique string");
```

这是你使用put时可以设置的所有选项。

```dart
Get.put<S>(
  // 必备：你想得到保存的类，比如控制器或其他东西。
  // 注："S "意味着它可以是任何类型的类。
  S dependency

  // 可选：当你想要多个相同类型的类时，可以用这个方法。
  // 因为你通常使用Get.find<Controller>()来获取一个类。
  // 你需要使用标签来告诉你需要哪个实例。
  // 必须是唯一的字符串
  String tag,

  // 可选：默认情况下，get会在实例不再使用后进行销毁
  // （例如：一个已经销毁的视图的Controller)
  // 但你可能需要这个实例在整个应用生命周期中保留在那里，就像一个sharedPreferences的实例或其他东西。
  //所以你设置这个选项
  // 默认值为false
  bool permanent = false,

  // 可选：允许你在测试中使用一个抽象类后，用另一个抽象类代替它，然后再进行测试。
  // 默认为false
  bool overrideAbstract = false,

  // 可选：允许你使用函数而不是依赖（dependency）本身来创建依赖。
  // 这个不常用
  InstanceBuilderCallback<S> builder,
)
```



#### （2）Get.lazyPut

可以懒加载一个依赖，这样它只有在使用时才会被实例化。这对于计算代价高的类来说非常有用，或者如果你想在一个地方实例化几个类（比如在Bindings类中），而且你知道你不会在那个时候使用这个类。

```dart
/// 1.只有当第一次使用Get.find<ApiMock>时，ApiMock才会被调用。
Get.lazyPut<ApiMock>(() => ApiMock());
/// 2.
Get.lazyPut<FirebaseAuth>(
  () {
    // ... some logic if needed
    return FirebaseAuth();
  },
  tag: Math.random().toString(),
  fenix: true
)
/// 3.
Get.lazyPut<Controller>( () => Controller() )
```

这是你在使用lazyPut时可以设置的所有选项。

```dart
Get.lazyPut<S>(
  // 强制性：当你的类第一次被调用时，将被执行的方法。
  InstanceBuilderCallback builder,
  
  // 可选：和Get.put()一样，当你想让同一个类有多个不同的实例时，就会用到它。
  // 必须是唯一的
  String tag,

  // 可选：类似于 "永久"，
  // 不同的是，当不使用时，实例会被丢弃，但当再次需要使用时，Get会重新创建实例，
  // 就像 bindings api 中的 "SmartManagement.keepFactory "一样。
  // 默认值为false
  bool fenix = false
)
```



#### （3）Get.putAsync

如果你想注册一个异步实例，你可以使用`Get.putAsync`。

```dart
Get.putAsync<SharedPreferences>(() async {
  final prefs = await SharedPreferences.getInstance();
  await prefs.setInt('counter', 12345);
  return prefs;
});
Get.putAsync<YourAsyncClass>( () async => await YourAsyncClass() )
```

这都是你在使用putAsync时可以设置的选项。

```dart
Get.putAsync<S>(
  // 必备：一个将被执行的异步方法，用于实例化你的类。
  AsyncInstanceBuilderCallback<S> builder,
  // 可选：和Get.put()一样，当你想让同一个类有多个不同的实例时，就会用到它。
  // 必须是唯一的
  String tag,
  // 可选：与Get.put()相同，当你需要在整个应用程序中保持该实例的生命时使用。
  // 默认值为false
  bool permanent = false
)
```

### <font color=red><u>2.使用实例化方法/类：</u></font> 

想象一下，你已经浏览了无数条路由，现在你需要拿到一个被遗留在控制器中的数据，你只需要让Get为你的控制器自动 "寻找"，你不需要任何额外的依赖关系。

```dart
// 1.在某个页面使用改控制器时
final controller = Get.find<Controller>();
// 或者
Controller controller = Get.find();
// 是的，它看起来像魔术，Get会找到你的控制器，并将其提供给你。
// 你可以实例化100万个控制器，Get总会给你正确的控制器。

// 2.到处需要这个控制器，比如 网络请求/ 本地存储。 你可以在你的控制器Controller.dart中如下定义，但是前提你得写好各自的set or   get方法
static Controller get to => Get.find(); // 同样的，在 “简单状态管理器” 章节中也介绍了此方法，可以在类里写明这句话，整个App中你就可沟通类来调用方法变量了。
//  eg. Controller.to.xx属性/xx方法()
```

然后你就可以在需要使用控制器的页面中恢复你在后面获得的控制器数据。

```dart
// 1.与上1对应
Text(controller.textFromApi);

// 2.与上2对应
Text(Controller.to.textFromApi); 
```

由于返回的值是一个正常的类，你可以做任何你想做的事情。

```dart
// 1.与上1对应
int count = Get.find<SharedPreferences>().getInt('counter');
print(count); // out: 12345

// 2.与上2对应
int count = Controller.to.getInt('counter');
print(count); // out: 12345
```

移除一个Get实例:

```dart
Get.delete<Controller>(); //通常你不需要这样做，因为GetX已经删除了未使用的控制器。
```



### <font color=red><u>3.Bindings：</u></font> 

这个包最大的区别之一，也许就是可以将路由、状态管理器和依赖管理器完全集成。 当一个路由从Stack中移除时，所有与它相关的控制器、变量和对象的实例都会从内存中移除。如果你使用的是流或定时器，它们会自动关闭，你不必担心这些。 在2.10版本中，Get完全实现了Bindings API。 现在你不再需要使用init方法了。如果你不想的话，你甚至不需要键入你的控制器。你可以在适当的地方启动你的控制器和服务来实现。 Binding类是一个将解耦依赖注入的类，同时 "Bindings "路由到状态管理器和依赖管理器。 这使得Get可以知道当使用某个控制器时，哪个页面正在显示，并知道在哪里以及如何销毁它。 此外，Binding类将允许你拥有SmartManager配置控制。你可以配置依赖关系，当从堆栈中删除一个路由时，或者当使用它的widget被布置时，或者两者都不布置。你将有智能依赖管理为你工作，但即使如此，你也可以按照你的意愿进行配置。

#### （1）Bindings类

```dart
class HomeBinding implements Bindings {}
```

你的IDE会自动要求你重写 "dependencies"方法，然后插入你要在该路由上使用的所有类。

```dart
class HomeBinding implements Bindings {
  @override
  void dependencies() {
    Get.lazyPut<HomeController>(() => HomeController());
    Get.put<Service>(()=> Api());
  }
}

class DetailsBinding implements Bindings {
  @override
  void dependencies() {
    Get.lazyPut<DetailsController>(() => DetailsController());
  }
}
```

现在你只需要通知你的路由，你将使用该 Binding 来建立路由管理器、依赖关系和状态之间的连接。

- 使用别名路由：

```dart
getPages: [
  GetPage(
    name: '/',
    page: () => HomeView(),
    binding: HomeBinding(),
  ),
  GetPage(
    name: '/details',
    page: () => DetailsView(),
    binding: DetailsBinding(),
  ),
];
```



#### （2）BindingsBuilder

创建Bindings的默认方式是创建一个实现Bindings的类，但是，你也可以使用`BindingsBuilder`回调，这样你就可以简单地使用一个函数来实例化任何你想要的东西。

例子:

```dart
getPages: [
  GetPage(
    name: '/',
    page: () => HomeView(),
    binding: BindingsBuilder(() {
      Get.lazyPut<ControllerX>(() => ControllerX());
      Get.put<Service>(()=> Api());
    }),
  ),
  GetPage(
    name: '/details',
    page: () => DetailsView(),
    binding: BindingsBuilder(() {
      Get.lazyPut<DetailsController>(() => DetailsController());
    }),
  ),
];
```

这样一来，你就可以避免为每条路径创建一个 Binding 类，使之更加简单。

两种方式都可以完美地工作，我们希望您使用最适合您的风格。



## 附录 

#### 【1】getX github文档

https://github.com/jonataslaw/getx/blob/master/README.zh-cn.md

#### 【2】getX路由

https://github.com/jonataslaw/getx/blob/master/documentation/zh_CN/route_management.md

#### 【3】getX状态管理 

https://github.com/jonataslaw/getx/blob/master/documentation/zh_CN/state_management.md

#### 【4】getX依赖管理 

https://github.com/jonataslaw/getx/blob/master/documentation/zh_CN/dependency_management.md

#### 【5】利用get_cli进行创建带getX的项目

https://github.com/jonataslaw/get_cli/blob/master/README-zh_CN.md











