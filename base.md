### 1、创建flutter项目

```bash
sudo flutter create flutter_app01 #文件名不能出现大写字母
```

##### 创建指定iOS/Android项目语言类型：

###### （1）Android使用Kotlin，iOS使用Swift。

```base
flutter create -i swift -a kotlin flutter_app01
```

###### （2）Android使用Java，iOS使用Swift。

```bash
flutter create -i swift -a java flutter_app01
```

###### （3）Android使用Kotlin，iOS使用Objective-C。

```bash
flutter create -i objc -a kotlin flutter_app01
```

###### （4）Android使用Java，iOS使用Objective-C。

```bash
flutter create -i objc -a java flutter_app01
```

### 2、执行命令修改权限

```bash
sudo chmod -R 777 flutter_app01 # sudo:管理员权限 chmod:文件权限修改 -R:表示递归 777:表示可读可写可执行
# 有可能也需要修改SDK的权限，命令如下
cd /Users/wangxiao/flutter
sudo chmod -R 777 *
```

### 3、VSCode的终端中执行命令

```bash
flutter run
```



### 附录、Flutter Demo说明

##### 【一】flutter_app01 自定义widget、center组件、text组件、MaterialApp组件、Scaffold组件介绍。

##### 【二】flutter_app02	Container组件、Text组件详解。

```text
用法描述：
1、Container:
（1）child:设置子元素
（2）decoration:设置Container样式。【如果只设置颜色可以不需要这个属性，直接与child同层写color属性】
（3）直接设置宽高。
附：SizedBox此组件与Container类似，但是只能设置宽高。可作为空白物填充页面。

2、Text:
（1）style:TextStyle() 设置文本样式。
（2）overflow:设置超出文本缩略 TextOverflow.ellipsis。
```



##### 【三】flutter_app03 图片组件Image、本地图片、远程图片、图片裁剪。

```text
用法描述：
1、Image:
（1）网络图片：Image.network()。fit:BoxFit.xxx图片在父元素的填充情况
（2）本地图片：配置相当繁琐。需要在pubspec.yaml下配置，方可使用。
        uses-material-design: true
          assets:
            - images/small.png
     注：如果小图标需要指定的分辨率，则图片文件夹需要2.0x,3.0x,4.0x这样进行分类。yaml文件无需每个图片进行引入，直接引入文件		夹，会自动去匹配搜索图片。
（3）ClipOval组件 图片的裁剪。	
```



##### 【四】flutter_app04 ListView基础列表组件、水平列表组件、图标组件。

```text
用法描述：
1、ListView组件 children:[],  scrollDirection:Axis.xxx设置滚动方向【超出屏幕默认会滚动】。
2、ListTile组件:
（1）leading:类似cell前面的Icon。
（2）trailing:类似cell后面的Icon。
（3）title:主标题。
（4）subtitle:副标题。
```



##### 【五】flutter_app05 ListView列表组件、动态列表。

```dart
用法描述：
动态循环遍历数据展示列表。
1、ListView第二种创建方式：
ListView.builder(
	itemCount: listData.length,
          itemBuilder: (context, index) {
            return ListTile(
              leading: Image.network(listData[index]["imageUrl"]),
              title: Text(listData[index]["title"]),
              subtitle: Text(listData[index]["author"]),
            );
          },
			)
```



##### 【六】flutter_app06 GridView组件以及动态GridView。

```dart
用法描述：
1、GridView。
（1）第一种创建方式：
		GridView.count(
        crossAxisCount: 2, //垂直方向【默认方向】表示多少列，水平方向则表示有多少行
        crossAxisSpacing: 15.0, //纵向间距
        mainAxisSpacing: 15.0, //主轴间距
        childAspectRatio: 2.0, //子widget宽高比例
        children: [],
     )
（2）第二种创建方式：
    GridView.builder(
            padding: const EdgeInsets.all(15.0),
            itemCount: listData.length,
            itemBuilder: (context, index) {
            	return xxx;//用于循环展示样式（listData[index]["xxx"]）
            },
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              crossAxisSpacing: 15.0, //纵向间距
              mainAxisSpacing: 15.0, //主轴间距
              childAspectRatio: 0.9, //子widget宽高比例
            ),
          ),
```



##### 【七】flutter_app07 页面布局Padding Row Column Expanded组件详解。

```dart
用法描述：
1、Padding组件:设置内边距 【padding, child】
2、Row组件:设置行布局。
	Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,//设置主轴布局
          crossAxisAlignment: CrossAxisAlignment.end,//设置副轴布局
          children: [],
        )
3、Column组件:设置列布局。与Row用法同。
4、Expanded组件:设置布局分布。多与Row组件、Column组件混用。
		Expanded(
			child:xxx,
			flex:1,//设置占比
		)
5、  2、3、4混用例子如下：占比分布 1:2:1
		Row(
        children: <Widget>[
          Expanded(
            flex: 1,
            child: xxx
          ),
          Expanded(
            flex: 2,
            child: xxx
          ),
          Expanded(
            flex: 1,
            child: xxx
          )
        ],
      ),
  6、shadow属性用法:
  shadows: <Shadow>[
                    Shadow(
                      offset: Offset(5.0, 5.0),
                      blurRadius: 30.0,
                      color: Color.fromRGBO(0, 0, 0, 0.6),
                    ),
                  ],
```



##### 【八】flutter_app08 页面布局Stack层叠组件 Stack与Align Stack与Positioned实现定位布局。

```dart
用法描述：
1、Align组件:调整位置类似CSS的相对定位。
2、Positioned组件：类似CSS中的绝对定位。
3、Stack组件:层叠布局，常与Align，Positioned组件组合使用。
	Stack(
		alignment: Alignment.center, //const Alignment(-1, -1)
		children:[
			 Positioned(
                    left: 10,
                    child:null,
                  ),
       Align(
                    alignment: Alignment(-0.1, 0),
                    child: null,
             ),
		],
	)
```



##### 【九】flutter_app09 AspectRatio、Card卡片组件。

```text
用法描述：
1、AspectRatio组件：可以设置宽高比【只需要设置宽或者高即可】
2、Card组件：clipBehavior: Clip.antiAlias, //裁剪超过card的部分
3、ClipRRect()裁剪。
	ClipRRect(
          	borderRadius: const BorderRadius.only(
            			bottomLeft: Radius.circular(5.0),
                  bottomRight: Radius.circular(5.0),
                 ),
             child: null,
             fit: BoxFit.cover,
           ),
```



##### 【十】flutter_app10 页面布局Wrap组件【流布局】。

```text
用法描述：
1、Wrap组件：类似Row和Column，但是不同于他们的是，如果设置行方向/列方向，mainAxis上的空间不足时，超出的部分会换行/换列。
Wrap(
        direction: Axis.horizontal, //方向
        alignment: WrapAlignment.spaceEvenly,//主轴布局方式
        runAlignment:WrapAlignment.center,//副轴布局方式
        spacing: 15.0,//主轴间距
        runSpacing: 10.0,//副轴间距
        children: [],
      ),
```



##### 【十一】StatefulWidget有状态组件。

```text
用法描述：
StatelessWidget是无状态组件，状态不可变的widget。
StatefulWidget是有状态组件，持有的状态可能在widget生命周期中改变。通俗的讲，如果我们想改变页面中的数据的话这个时候就需要用到StatefulWidget。
1、安装插件Flutter Widget Snippets，输入statefulW即可构建出StatefulWidget代码模板，同时使用setState((){});来保存变量状态。
```



##### 【十二】🚀🚀🚀BottomNavigationBar 自定义底部导航条、以及实现页面切换。

```text
用法描述：
1、利用Scaffold组件中的BottomNavigationBar组件进行切换，
2、其中默认至少两个，超过3个需要设置bottomNavigationBar的属性type: BottomNavigationBarType.fixed,才可以进行渲染展示。
3、点击切换，以及对应页面的展示即需要通过StatefulWidget这个组件以及搭配setState((){});来进行实现。
4、在有BottomNavigationBar时候想设置有的页面无导航栏，可在各自的页面单独再设置Scaffold,此组件可一直嵌套下去。
```



##### 【十三】Flutter路由。🚀🚀🚀

```dart
用法描述：
Flutter中的路由通俗的讲就是页面跳转。在Flutter中通过Navigator组件管理路由导航。并提供了管理堆栈的方法。如Navigator.push和Navigator.pop。
Flutter给我们提供了两种配置路由跳转的方式：1、基本路由；2、命名路由。
1、基本路由
（1）路由跳转
			Navigator.of(context).push(
                  MaterialPageRoute(
                    builder: (context) => const SearchPage(),
                  ),
                );
      其中，SearchPage页面需要使用搭配Scaffold组件使用。
      return Scaffold(
      	appBar: AppBar(
        	title: const Text("搜索页"),
      	),
     	 	body: const Center(
        	child: Text("我是搜索页面！"),
      	),
    	);
 （2）返回
			Navigator.of(context).pop();
			
2、命名路由
（1）普通使用
在main.dart文件中添加routes。
		return MaterialApp(
      home: const Tabs(),
      routes: {
        '/search': (context) => const SearchPage(),
        '/form': (context) => FormPage(),
      },
    );
使用时Navigator.pushNamed(context, '/search');即可。
（2）传参使用
			方案一、使用 MaterialApp下的属性【routes:routes】。假设【A page -> B page】
          1.在 A page页面 传参：
              Navigator.pushNamed(
                      context,
                      '/search',
                      arguments: {"id": '5526378273', "title": "标题搜索"},
                    );
          2.在 B page页面 接收参数。
              在build下实现下列方法进行获取参数。
              final arguments = ModalRoute.of(context)!.settings.arguments as Map;
			
			方案二、使用 MaterialApp下的方法【onGenerateRoute】，有此方法时删除routes:routes属性。【A page -> B page】
					1.路由中需添加变量argument
                '/search': (context, {arguments}) => SearchPage(
                      arguments: arguments,
                    ),
					2.main.dart中
							onGenerateRoute: (RouteSettings settings) {
                  //统一处理
                  /**
                   * settings
                   * settings.name:路由名
                   * settings.arguments:路由参数Map对象
                   */
                  final String? name = settings.name;
                  final Function pageContentBuilder = routes[name] as Function;
                  print("settingsname:${settings.arguments}");

                  if (pageContentBuilder != null) {
                    if (settings.arguments != null) {
                      final Route route = MaterialPageRoute(
                        builder: (context) =>
                            pageContentBuilder(context, arguments: settings.arguments),
                      );
                      return route;
                    } else {
                      final Route route = MaterialPageRoute(
                        builder: (context) => pageContentBuilder(context),
                      );
                      return route;
                    }
                  } else {
                    return MaterialPageRoute(builder: (context) => const NotFoundPage());
                  }
                },
                
          3.在 A page页面 传参：
              Navigator.pushNamed(
                      context,
                      '/search',
                      arguments: {"id": '5526378273', "title": "标题搜索"},
                    );
          4.在 B page页面 接收参数。
              定义变量final arguments; 通过this.arguments即可获取传参。
            注：(1)如果是有状态组件，可以通过widget.arguments获取。
            	  (2)对于组件传参，区分下两种方式。
            	  		1、'/search': (context, {arguments}) => SearchPage(
                            arguments: arguments,
                          ),
                        这种方式的时候接受参数  SearchPage({required this.arguments, Key? key}) : super(key: key);
                    2、'/search': (context, {arguments}) => SearchPage(arguments),
                    		这种方式的时候接受参数  SearchPage(this.arguments, {Key? key}) : super(key: key);
```



##### 【十三】Flutter路由。🚀🚀🚀

```dart
用法描述：路由替换 和 返回根路由 的使用方法。
（1）路由替换方式：Navigator.of(context).pushReplacementNamed('/registerSecond');//这是处在倒数第二个界面按钮的跳转方法。
（2）返回根目录
			1.通过（1）的这种方式直接使用Navigator.pop(context)即可返回到根页面。
			
			2.如果push方式为正常的Navigator.pushNamed('context','/xxxx')方式。那么返回根目录就使用如下方式。
					Navigator.of(context).pushAndRemoveUntil(
                    MaterialPageRoute(builder: (context) => const Tabs()),
                    (route) => route == null);
          其中Tabs是自定义的底部工具栏组件，会默认到index为0的HomePage()界面。
          
      3.若要返回和1情况一样的根目录：设置实现Tabs组件的构造函数，可以从外部传入currentIndex即可。
```



##### 【十四】Flutter自定义AppBar，定义顶部Tab切换。

```dart
用法描述：
（1）方法一：
设置tab切换需要在最外层 添加DefaultTabController组件，然后设置length和child:Scaffold()。length的数量与tabs、TabBarView均相同。
TabBar组件可以在AppBar的title属性下也可以在其bottom属性下。
其结构大致如下：
return DefaultTabController(
      length: 2,
      child: Scaffold(
        appBar: AppBar(
          title: const Text("AppBarDemoPage"),
          //AppBar的Tab
          bottom:const tabs:  <Widget>[
              Tab(
                text: "热门",
              ),
              Tab(
                text: "推荐",
              ),
            ],
          ),
        ),
        body: TabBarView(
          children: <Widget>[
            ListView(
              children: const <Widget>[
                ListTile(
                  title: Text("第一个热门tab"),
                ),
              ],
            ),
            ListView(
              children: const <Widget>[
                ListTile(
                  title: Text("第一个推荐tab"),
                ),
              ],
            ),
          ],
        ),
      ),
    );
    
    （2）方法二：
    使用TabController方式。
    1、创建有状态组件StatefulWidget，并使用混入 with SingleTickerProviderStateMixin。
    2、创建变量late TabController _tabController; 并实现initState方法。
    		//初始化方法
        void initState() {
          super.initState();
          _tabController = TabController(length: 2, vsync: this); //注意
          _tabController.addListener(() {
            print('监听tab的index:' + _tabController.index.toString());
            // setState(() {
            //可以设置自定义操作
            // });
          });
        }
    3、使用方法与上同，只是不需要DefaultTabController了，而且在设置TabBar和TabBarView时，均需要添加controller:_tabController,该属性。
```



##### 【十五】Drawer侧边栏、DrawerHeader、UserAccountsDrawerHeader以及侧边栏内容布局。

```dart
用法描述：Drawer侧边栏组件应在Scaffold组件层级下定义方可生效。故可根据Scaffold组件的位置来控制Drawer侧边栏是全局的还是局部页面使用。
  
按钮集合：ElevatedButton、OutlinedButton、FloatingActionButton
```



##### 【十六】表单、TextField单行文本框、多行文本框、CheckBox、CheckBoxListTile。

```text
一、TextField：
	1.限制字数：maxLength
	2.限制行数：maxLines
	3.加边框以及设置圆角弧度：decoration -> border(OutlineInputBorder) -> borderRadius
	4.文本框内容的内边距：contentPadding
	5.密码框：(是否显示密码按钮)
	6.默认显示文本：controller -> TextEditingController
	
二、Checkbox:
	1.value：值
	2.onChanged：改变状态的方法
	3.CheckboxListTile：类似cell的Checkbox
	 
三、Radio: 【至少两个组合组合在一起】
	1.value：值
	2.groupValue：共用变量，可以实现按钮组
	3.onChanged：改变状态的方法
	4.RadioListTile：类似cell的Radio
	
四、Switch：
1.value：值
2.onChanged：改变状态的方法
注：上述的Switch是安卓的，如果想要类似iOS的请使用CupertinoSwitch，用法完全相同。

```



##### 【十七】日期和时间戳、格式化日期库、Flutter异步、官方自带日期组件showDatePicker、时间组件showTimePicker以及国际化。

```dart
当前时间：var now = DateTime.now();
print("当前日期：$now"); //2022-05-05 10:03:21.329704
print("时间戳：${now.millisecondsSinceEpoch}"); // 1651719360253  2022-05-05 10:56:00
print("时间戳转成时间:${DateTime.fromMillisecondsSinceEpoch(1651719360253)}"); //2022-05-05 10:56:00.253

//🚀🚀🚀 引入第三方库的方法
1.进入 https://pub.dev 网址，寻找第三方库。【以 date_format 为例】
2.Installing进行安装，将其复制到 pubspec.yaml 文件对应的目录下，进行保存即可，若未成功执行 flutter packages get 进行安装。
3.在Example查看如何引入以及使用即可。
print(formatDate(DateTime.now(), [yyyy, '年', mm, '月', dd, ' ', HH, ':', nn, ':', ss])); //2022年05月06 19:33:21

组件学习：
  1.showDatePicker:
			DateTime _nowDate = DateTime.now();
					 showDatePicker(
      				context: context,
      				initialDate: _nowDate,
     				  firstDate: DateTime(1980),
      				lastDate: DateTime(2100),
      				locale: const Locale('zh'), //非必须 国际化时候需要
    				).then((res){
             print("获取到日期:$res");
           })
             
  2.showTimePicker:
			var _nowTime = const TimeOfDay(hour: 12, minute: 20);			 
      showTimePicker(context: context, initialTime: _nowTime).then((res){
        print("获取到时间:$res");
      })
    
  3.InkWell 带水墨纹的提供点击事件的组件。
        InkWell(
                child: subWidget,
                onTap: function(){},
              ),

  4.async await 与JavaScript用法一致
    
  5.国际化:
		(1).在pubspec.yaml文件中进行配置
      	dependencies:
					flutter_localizations:
    				sdk: flutter
    (2).在入口文件main.dart中进行引入并进行全局配置如下:
				import 'package:flutter_localizations/flutter_localizations.dart';
				const MeterialApp(
        	 // 全局配置会根据操作系统来辨识语言
      			localizationsDelegates: [
        			//此处为国际化代理
        			GlobalMaterialLocalizations.delegate,
        			GlobalWidgetsLocalizations.delegate,
      			],
      			supportedLocales: [
        			//此处为支持的国家
       				Locale('zh', 'CH'),
        			Locale('en', 'US'),
      			],
        );
		(3).flutter会默认根据你的系统语言选择，
      	也可强制设置语言类型，提供的系统组件里配置locale: const Locale('zh') 如上showDatePicker组件所示。
```



##### 【十八】flutter_swiper、Dialog、AlertDialog、SimpleDialog、showModalBottomSheet、showToast以及自顶Dialog

```dart
注：如何解决在一个限制死高度的内容区域，有着超出组件高度的内容，并让其滚动。
方法一:用 LimitedBox -> SingleChildScrollView。 不过这种方法的坏处就是要自己限制高度。无法动态布局。、
			const Padding(
                padding: EdgeInsets.all(10),
                child: LimitedBox(
                  maxHeight: 120,
                  child: SingleChildScrollView(
                    child: Text("xxxxx",
                      textAlign: TextAlign.left,
                    ),
                  ),
                ),
              ),
方法二:用Flexible -> SingleChildScrollView。
  		const Flexible(
                flex: 1,
                child: SingleChildScrollView(
                  // physics: ClampingScrollPhysics(),
                  padding: EdgeInsets.all(15),
                  child: Text("xxxxx",
                    textAlign: TextAlign.left,
                  ),
                ),
              ),

定时器Timer:
		var timer;
    timer = Timer.periodic(const Duration(milliseconds: 3000), (t) {
      //coding... 3s后触发回调函数
      t.cancel(); //定时器取消
    });
```



##### 【附】模块一 Flutter性能测试

```text
【一】VSCode -> 运行和调试 -> 添加配置 -> Dart/Flutter -> 选择【flutter_performance (profile mode)】进行运行。
cmd+shift+P 选择 Dart:Open DevTools。
1、【Performance】模块：将会进行UI耗时测量。
使用方式：点击耗时的柱状图，然后会出现对应的火焰图，点击最底层。然后在底部模块"CPU Profile"处，选择"CPU Flame Chart"。进行筛选剔除dart以及flutter资源库所带来的消耗。即可定位消耗CPU的问题所在。
2、【CPU Profiler】模块：CPU耗时测量。
使用方法：点击Record按钮进行录制，然后对耗时操作的界面进行操作，操作完毕后点击Stop进行停止录制，便会生成分析日志。与上同，进行筛选剔除dart以及flutter资源库所带来的消耗即可定位耗时严重的方法从而进行分析优化。
3、【Memory】模块：GPU性能测试。
使用方法：进行耗时操作的界面进行操作，过段时间后点击Pause进行暂停。耗时的操作会以三角形的标记出现，然后点击进行分析（events）
4、【Performance Overlay】模块：
使用方法：点击后，界面会出现Raster和UI数据可以进行查看分析。
5、【包分析】
使用方法：再开一个命令行 flutter build ios --analyze-size分析iOS模块，将build目录下生成的最后一次的flutter_size_xx下的
snapshot.arm64.json文件拖入【Open DevTools】的【App Size】模块进行分析。

【二】VSCode -> 运行和调试 -> 添加配置 -> Dart/Flutter -> 选择【flutter_performance】进行运行。
cmd+shift+P 选择 Dart:Open DevTools。
1、【Highlight Repaints】会进行描边，组件重绘次数越多颜色越深。

【三】测试包体积 flutter build apk 或 flutter build ios 等生成的默认发行版本。
https://flutter.cn/docs/perf/app-size
大小拆分分析：
flutter build apk --analyze-size
flutter build appbundle --analyze-size
flutter build ios --analyze-size
flutter build linux --analyze-size
flutter build macos --analyze-size
flutter build windows --analyze-size
```



##### 【其他使用Tip】

```text
1.父传子的方式：在组件进行构造方法时，括号里是命名参数，括号外是匿名函数。如下
（1）命名参数：
		子组件：TestComp({Key?key, required this.arguments}):super(key:key);
		父组件：TestComp(arguments:'xxx')
		
（2）匿名参数：
		子组件：TestComp(this.arguments, {Key?key}):super(key:key);
		父组件：TestComp('xxx')
		
2.子传父的方式：
		子组件定义： 
							ValueChanged<data type> xxxCallBack;
							TestComp({Key?ket, required this.xxxCallBack}) : super(key: key);
		子函数传参：
							xxxCallBack(data);//data的类型与定义时保持一致。
							
		父组件获取到子组件的回调值：
							TestComp( xxxCallBack:(value) {
									//对value的处理，一般多用于StatefulWidget组件中，与setState()搭配使用。
							})
							
		【注:如果不传值的话使用VoidCallback即可】
							
	3.生成实体类工具：
	https://app.quicktype.io/【vscode 插件 Paste JSON as Code】 使用 Apifox + 此网站【JSON】
	
```



##### 【fvm命令】flutter版本切换

fvm 安装教程:
https://wiki.ducafecat.tech/flutter/%E5%B7%A5%E5%85%B7/fvm%E7%89%88%E6%9C%AC%E7%AE%A1%E7%90%86.html#%E5%AE%89%E8%A3%85

```bash
fvm releases //查看flutter所有版本
fvm install 版本号 //安装指定版本
fvm list //查看本地版本
fvm remove 版本号 //移除指定版本
fvm global 版本号 //设置全局默认的flutter sdk版本
fvm use 版本号 //在需要使用指定版本的项目根目录使用这条命令 
```

