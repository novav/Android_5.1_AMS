#  Android_5.1_AMS

## AMS在系统启动期间所做的工作：
-    Step-1 启动AMS
-    Step-2 调用SetProcessService方法
-    Step-3 调用installSystemProviders方法
-    Step-4 调用systemReady方法


Step-1:启动AMS
-    init AMS
-    1.初始化BroadcastQueue
-    2.初始化一些系统运行的相关变量，如：耗电量，CPU使用率和负载等信息
-    3.获取系统配置信息，如OpenGL版本，字体，语言，屏幕等信息
-    4.将自身加入Android WatchDog监控

Step-2: 调用SetProcessService方法
-    1.注册服务
-    2.查询并处理ApplicationInfo
-    3.创建并处理ProcessRecord

Step-3 调用installSystemProviders方法
-    1.调用 generateApplicationProvidersLocked 查询Content Provider
-    2.调用 ActivityThread.installSystemProviders 安装Content Provider
      -  <1>.调用installProvider方法创建ContentProvider，存入ContentProviderHolder中
      -  <2>.调用publishContentProviders方法向AMS发布该ContentProvider
            （1，根据调用者传入的IApplicationThread参数，获取进程的ProcessRecord
              2，将ContentProvider信息存入ActivityManagerService的成员变量mProviderMap中，存储方法有两种，即对应两种检索方法
              3，mLaunchingProvider中存储了Client请求使用的Content Provider信息，当前ContentProvider已启动，因此需要从mLaunchingProvider中移除该Content Provider的信息
              4，强Content Provider与其ProcessRecord关联
              5，通知Client，其请求使用Content Provider已发布）

Step-4 调用systemReady方法
-    1.发送ACTION_PRE_BOOT_COMPLETED广播
        - systemReady第一部分工作是发送ACTION_PRE_BOOT_COMPLETED广播，可以将该部分视为预启动完毕阶段，目前接受此广播的有三个模块：
            CalendarProvider：com.android.providers.calendar.CalederUpgradeReceiver
            ContactsProvider：com.android.providers.contacts.ContactsUpgradeReceiver
            MediaProvider：con.android.provider.media.MediaUpgradeReceiver
            这三个接受者的主要作用是在启动阶段对数据库做一些预处理工作，例如创建数据库，或者更新数据库版本。
            ACTION_PRE_BOOT_COMPELTED只被处理一次，处理过该广播的模块会在启动阶段存入data/system/called_pre_boots.dat文件中，因此第二次启动时，这些模块不需要再次处理该广播
            如果没有可以处理该广播的接收者，此时即将mSystemReady赋值为true直接进入第二部分处理工作
-    2.清理预启动的非Persistent进程
        - persistent进程是在AndroidManifest.xml中设置了Android:persistent="true"的应用程序，如framework-res.apk, SystemUI.apk, Phone.apk 这些进程需要常驻内存，不能被杀死
-    3.读取Settings配置
        - RetrieveSettings主要是从设置中读去了一下四种配置信息
            - a>: Debug app 需要调试的应用程序包名
            - b>: Wait for dubugger如果设置为1，当启动debug_App时，该应用会等待调试器，如果设置为零，则正常启动
            - c>: Always finish activities如果设置为1， activity manager 会直接接触那些不再需要的activity，如果设置为零，咋遵循正常的生命周期。
            - d>: Font scale 与字体大小相关的设置
        这些配置信息最终都存入AMS的相关成员变量中。
-    4.运行Runnable回调接口
      - 执行回调接口即执行传入systemReady的Runnable参数的run方法。
      - 第四部分主要做了以下工作
           - a>: 启动系统服务：该服务会根据当前设备是平板电脑还是手机，分别启动不同的状态栏。跟服务定义于frameworks/Bese/packages/SystemUi。编译后为 SystemUi.apk
           - b>: 执行其他系统服务的SystemReady 方法：包括batteryService network management service 等
           - c>: 启动软件看门狗watch dog
      - 可见一些服务依赖于AMS的运行状态，只有AMS本身万事俱备，才会通知其他一些系统服务进入SystemReady状态
-    5.启动persistent应用程序和home
      -  可见, M S在地物分钟在第五部分中还隐藏着一个重要操作:发送开机完成广播。这部分内容需要集合消息处理机制和应用程序启动过程才能真正理解。因此这里只列出其主要流程。
      -  至此，M S在系统启动init2 阶段的使命已经完成了。


## 章节小结：
文章详细分析了AM核心组成部分AmS在系统启动时所做的四个阶段的工作,着重分析了 AThread, Context, ActivityStack,SettingsProvider,ApplicationThread, Action_Boot_Completed等主要内容的实现。
