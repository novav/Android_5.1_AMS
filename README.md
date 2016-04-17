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
