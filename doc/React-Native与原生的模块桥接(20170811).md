## React-Native与原生的模块桥接(20170811)
- [iOS资料](http://reactnative.cn/docs/0.45/native-modules-ios.html#content)
- [Android资料](http://reactnative.cn/docs/0.45/native-modules-android.html#content)

### 3种交互通信方式
- 属性
   React-Native(属性)将信息(父到子)传送元素，推荐(callBack)将信息从（子到父）传递，（DeviceEventEmitter）将信息夸层级传递。
- 原生模块
- 原生UI组件封装

---------

 - ** `RCT_EXPORT_MODULE()`**

  ```objc
    #define RCT_EXPORT_MODULE(js_name)
    RCT_EXTERN void RCTRegisterModule(Class);
    + (NSString *)moduleName { return @#js_name; }
    + (void)load { RCTRegisterModule(self); }
  ```
    首先它将`RCTRegisterModule`这个函数定义为`extern`，这样该函数的实现对编译器不可见，
    但会在链接的时候可以获取到；同时声明一个`moduleName`函数，该函数返回该模块的js名称，
    如果你没有指定，默认使用类名；最后声明一个`load`函数（当应用载入后会加载所有的类，
    `load`函数在类初始化加载的时候就调用），然后调用`RCTRegisterModule`函数注册该模块，
    该模块会被注册添加到一个全局的数组`RCTModuleClasses`中。


 - **`RCT_EXPORT_METHOD()`**

   要暴露给js调用的API接口需要通过该宏定义声明，该宏定义会额外创建一个函数，形式如下：
   ```
  + (NSArray *)__rct_export__230
  {
    return @[ @"", @"addEvent:(NSString *)name location:(NSString *)location" ];
  }
  ```
  该函数名称以 rct_export 开头，同时加上该函数所在的代码行数，该函数返回一个包含可选的js名称以及一个函数签名的数组，他们的作用后面会说到。

- **`RCTBatchedBridge`**

  为了桥接js跟native，native层引入了RCTBridge这个类负责双方的通信，不过真正起作用的是RCTBatchedBridge这个类，这个类应该算是比较重要的一个类了，让我们来看看这个类主要做啥事情：

  ```objc
    //RCTBatchedBridge.m
    - (void)start
    {
    dispatch_queue_t bridgeQueue = dispatch_queue_create("com.facebook.react.RCTBridgeQueue", DISPATCH_QUEUE_CONCURRENT);

    // 异步的加载打包完成的js文件，也就是main.jsbundle，如果包文件在本地则直接加载，否则根据URL通过NSURLSession方式去下载
    [self loadSource:^(NSError *error, NSData *source) {}];

    // 同步初始化需要暴露给给js层的native模块
    [self initModules];

    //异步初始化JS Executor，也就是js引擎
    dispatch_group_async(setupJSExecutorAndModuleConfig, bridgeQueue, ^{
      [weakSelf setUpExecutor];
    });

    //异步获取各个模块的配置信息
    dispatch_group_async(setupJSExecutorAndModuleConfig, bridgeQueue, ^{
      config = [weakSelf moduleConfig];
    });

    //获取各模块的配置信息后，将这些信息注入到JS环境中
    [self injectJSONConfiguration:config onComplete:^(NSError *error) {}];

    //开始执行main.jsbundle
    [self executeSourceCode:sourceCode];

    }
  ```  
#### 具体函数说明

1. initModules( // 同步初始化需要暴露给给js层的native模块)

  ```objc
  - (void)initModules
  {
    NSMutableArray<RCTModuleData *> *moduleDataByID = [NSMutableArray new];
    NSMutableDictionary<NSString *, RCTModuleData *> *moduleDataByName = [NSMutableDictionary new];
    SEL setBridgeSelector = NSSelectorFromString(@"setBridge:");
    IMP objectInitMethod = [NSObject instanceMethodForSelector:@selector(init)];

    //RCTGetModuleClasses()返回之前提到的全局RCTModuleClasses数组，也就是模块类load时候会注册添加的数组
    for (Class moduleClass in RCTGetModuleClasses()) {
      NSString *moduleName = RCTBridgeModuleNameForClass(moduleClass);
      //如果该类或者父类没有重写了init方法或实现了setBridge方法，则，创建一个类的实例
      //React认为开发者期望这个模块在bridge第一次初始化时会实例化，确保该模块只有一个实例对象
      if ([moduleClass instanceMethodForSelector:@selector(init)] != objectInitMethod ||
          [moduleClass instancesRespondToSelector:setBridgeSelector]) {
        module = [moduleClass new];
      }

      //创建RCTModuleData模块信息，并保存到数组中
      RCTModuleData *moduleData;
      if (module) {
        if (module != (id)kCFNull) {
          moduleData = [[RCTModuleData alloc] initWithModuleInstance:module
                                                              bridge:self];
        }
      }
      moduleDataByName[moduleName] = moduleData;
      [moduleDataByID addObject:moduleData];
    }
  }
```
当创建完模块的实例对象之后，会将该实例保存到一个`RCTModuleData`对象中，`RCTModuleData`里包含模块的类名，名称，方法列表，实例对象、该模块代码执行的队列以及配置信息等，js层就是根据这个对象来查询该模块的相关信息。

2. setUpExecutor( //异步初始化JS Executor，也就是js引擎)

  `reactnative`的js引擎在初始化的时候会创建一个新的线程，该线程的优先级跟主线层的优先级一样，同时创建一个`runloop`，这样线程才能循环执行不会退出。所以执行js代码不会影响到主线程，而且`RCTJSCExecutor`使用的是`JavaScriptCore`框架，所以`react`只支持iOS7及以上的版本。

  ```objc
  //RCTJSCExecutor.m
  - (instancetype)init
  {
    NSThread *javaScriptThread = [[NSThread alloc] initWithTarget:[self class]
                                                         selector:@selector(runRunLoopThread)
                                                           object:nil];
    javaScriptThread.name = @"com.facebook.React.JavaScript";
    //设置该线程的优先级处于高优先级
    if ([javaScriptThread respondsToSelector:@selector(setQualityOfService:)]) {
      [javaScriptThread setQualityOfService:NSOperationQualityOfServiceUserInteractive];
    } else {
      javaScriptThread.threadPriority = [NSThread mainThread].threadPriority;
    }
    [javaScriptThread start];
    return [self initWithJavaScriptThread:javaScriptThread context:nil];
  }
  - (void)addSynchronousHookWithName:(NSString *)name usingBlock:(id)block
  {
    __weak RCTJSCExecutor *weakSelf = self;
    [self executeBlockOnJavaScriptQueue:^{
      //将该block函数添加到js的context中，javascriptcore会将block函数转成js function
      weakSelf.context.context[name] = block;
    }];
  }

  - (void)setUp
  {
    [self addSynchronousHookWithName:@"noop" usingBlock:^{}];
    [self addSynchronousHookWithName:@"nativeRequireModuleConfig" usingBlock:^NSString *(NSString *moduleName) {
      //获取该模块的具体配置信息，包含方法以及导出的常量等信息
      NSArray *config = [strongSelf->_bridge configForModuleName:moduleName];
      NSString *result = config ? RCTJSONStringify(config, NULL) : nil;
      return result;
    }];
    [self addSynchronousHookWithName:@"nativeFlushQueueImmediate" usingBlock:^(NSArray<NSArray *> *calls){
      [strongSelf->_bridge handleBuffer:calls batchEnded:NO];
    }];
  }
  ```
  可以看到`setup`的时候会注册几个方法到js的上下文中供后面js调用，比如``'nativeFlushQueueImmediate'`和 ``'nativeRequireModuleConfig'`方法等，当js调用相应方法时会执行对应的`block`，`javascriptcore`框架会负责`js function`和`block`的转换。

3. moduleConfig(//异步获取各个模块的配置信息)

  ```objc
  - (NSString *)moduleConfig
  {
    NSMutableArray<NSArray *> *config = [NSMutableArray new];
    for (RCTModuleData *moduleData in _moduleDataByID) {
      [config addObject:@[moduleData.name]];
    }
    return RCTJSONStringify(@{
      @"remoteModuleConfig": config,
    }, NULL);
  }
```
  从实现可以看出仅仅该过程是将模块的名称保存到一个数组中，然后生成一个json字符串的配置信息，包含所有的模块名称，类似如下：
```objc
  {"remoteModuleConfig":[["MyModule"],["RCTStatusBarManager"],["RCTSourceCode"],["RCTAlertManager"],["RCTExceptionsManager"],...]}

  ```


#### oc 与 js 间类型
![](http://upload-images.jianshu.io/upload_images/458529-c67085793e9f170e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
