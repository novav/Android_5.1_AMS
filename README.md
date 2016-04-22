# Android_5.1_AMS

AMS在系统启动期间所做的工作：
    Step-1 启动AMS
    Step-2 调用SetProcessService方法
    Step-3 调用installSystemProviders方法
    Step-4 调用systemReady方法


Step-1:启动AMS
    init AMS
    1.初始化BroadcastQueue
    2.初始化一些系统运行的相关变量，如：耗电量，CPU使用率和负载等信息
    3.获取系统配置信息，如OpenGL版本，字体，语言，屏幕等信息
    4.将自身加入Android WatchDog监控 

Step-2: 调用SetProcessService方法
    1.注册服务
    2.查询并处理ApplicationInfo
    3.创建并处理ProcessRecord

Step-3 调用installSystemProviders方法
    1.调用 generateApplicationProvidersLocked 查询Content Provider
    2.调用 ActivityThread.installSystemProviders 安装Content Provider
        <1>.调用installProvider方法创建ContentProvider，存入ContentProviderHolder中
        <2>.调用publishContentProviders方法向AMS发布该ContentProvider
            （1，根据调用者传入的IApplicationThread参数，获取进程的ProcessRecord
              2，将ContentProvider信息存入ActivityManagerService的成员变量mProviderMap中，存储方法有两种，即对应两种检索方法
              3，mLaunchingProvider中存储了Client请求使用的Content Provider信息，当前ContentProvider已启动，因此需要从mLaunchingProvider中移除该Content Provider的信息
              4，强Content Provider与其ProcessRecord关联
              5，通知Client，其请求使用Content Provider已发布）

Step-4 调用systemReady方法
    1.发送ACTION_PRE_BOOT_COMPLETED广播
        systemReady第一部分工作是发送ACTION_PRE_BOOT_COMPLETED广播，可以将该部分视为预启动完毕阶段，目前接受此广播的有三个模块：
            CalendarProvider：com.android.providers.calendar.CalederUpgradeReceiver
            ContactsProvider：com.android.providers.contacts.ContactsUpgradeReceiver
            MediaProvider：con.android.provider.media.MediaUpgradeReceiver
            这三个接受者的主要作用是在启动阶段对数据库做一些预处理工作，例如创建数据库，或者更新数据库版本。
            ACTION_PRE_BOOT_COMPELTED只被处理一次，处理过该广播的模块会在启动阶段存入data/system/called_pre_boots.dat文件中，因此第二次启动时，这些模块不需要再次处理该广播
            如果没有可以处理该广播的接收者，此时即将mSystemReady赋值为true直接进入第二部分处理工作
    2.清理预启动的非Persistent进程
        persistent进程是在AndroidManifest.xml中设置了Android:persistent="true"的应用程序，如framework-res.apk, SystemUI.apk, Phone.apk 这些进程需要常驻内存，不能被杀死
    3.读取Settings配置
    4.运行Runnable回调接口
    5.启动persistent应用程序和home