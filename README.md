# Ocelot.ConfigAuthLimitCache
此项目已经重写了Ocelot配置文件的获取方式，改为从数据库中进行获取，目前已经实现了mssql。具体其他的可以自行参考代码进行扩展。接下来将继续扩展认证以及限流方法。实现网关对每个客户单对每个API的访问进行限流



很多人都说配置文件的配置很繁琐，如果存储在数据库就方便很多，可以通过自定义UI界面在后台进行路由的配置，然后通过调用Administration API让修改后的路由规则立即生效。当然这都是后话了。今天就教你手把手的来把配置文件放到数据库中，然后在数据库中进行路由的配置。当然，我会在Github上开放源代码供大家参考。至于Nuget包的话，今天还没来得及弄，等明天晚上弄好，再发布Nuget包吧，今天先引用下源代码来使用吧。大家委屈一下吧。本文还是沿用之前的系列文章里面的Demo。所以可以先下载之前系列文章里面的Demo源码。https://github.com/yilezhu/OcelotDemo
## 实例教程集成步骤
1. Github上下载重写的配置文件的源代码，地址：https://github.com/yilezhu/Ocelot.ConfigAuthLimitCache  然后把项目文件拷贝到。系列文章的源代码下面，并添加项目引用。如下所示：

   ![1540302626763](https://img2018.cnblogs.com/blog/1377250/201810/1377250-20181023222341045-611883175.png)

项目添加进来后的结构如下所示：

![1540302716406](https://img2018.cnblogs.com/blog/1377250/201810/1377250-20181023222340531-2035653599.png)



2. OcelotDemo网关项目作如下修改，Programs.cs文件移除对Ocelot.json文件的引用，因为配置文件的获取方式已经改成了从数据库中获取，所以，你需要新建一个数据库，然后执行数据库脚本创建数据库表，这里只给出Mssql的数据库脚本，在项目源代码下面，大家自行下载。

3. ConfigureServices服务中Ocelot的注入的同时需要注入我们的扩展方法，如下所示：

   ```c#
   services.AddOcelot()//注入Ocelot服务
                       .AddAuthLimitCache(option=> {
                           option.DbConnectionStrings = "Server=.;Database=Ocelot;User ID=sa;Password=1;";
                       })
                       .AddConsul();
   ```

   > 注意：这里需要传入SqlServer的数据库连接字符串，由于博主扩展使用的Dapper+MSSQL所以这里需要传入步骤2中创建的数据库的链接字符串。
   > 

4. 我们在数据库中配置一个路由吧，如下所示：字段名称基本都是跟Ocelot原生配置名称一样，只是扩展了一些字段方便后期做限流的

   ![1540303270834](https://img2018.cnblogs.com/blog/1377250/201810/1377250-20181023222340182-1837600611.png)

   大家看到没有，这条路由的意思是接受/ss1/{通配符} 的路由，然后转到到下面就是/api/{通配符} 。

5. 路由配置好了，那就让我们启动一下项目看下效果吧。

   ![1540303513128](https://img2018.cnblogs.com/blog/1377250/201810/1377250-20181023222339859-844164595.png)

6. 上面是正常的访问结果，当我们访问一个错误的路由的时候，再看看吧。

   ![1540304187258](https://img2018.cnblogs.com/blog/1377250/201810/1377250-20181023222339374-1584876328.png)

   看到没有，返回了404的状态码，感觉不够友好，所以，我们也进行了改造。直接看结果吧

7. 为了看到效果，你需要在Configure中少做下修改

   ```c#
    app.UseAhphOcelot().Wait();
   ```

8. 然后我们重新启动下Ocelot网关项目，重新访问下6中的Url吧。

   ![1540304235799](https://img2018.cnblogs.com/blog/1377250/201810/1377250-20181023222338410-161695171.png)

   看到没有，返回的数据更友好，而且是200的状态。当然大家也可以忽略这个功能哈。

   
## 源码地址：
1. Demo地址:https://github.com/yilezhu/OcelotDemo
2. 扩展插件地址：https://github.com/yilezhu/Ocelot.ConfigAuthLimitCache

## 总结
本文主要通过实例讲述如何集成，将配置文件存储到数据库的插件。源码已经开源，今天暂时没有发布Nuget包，明天再发布吧。当然你可以自行扩展代码。实现你自己的业务。我把配置文件存储到数据库的目的就是方便后面做UI管理方便，还有就是可以基于这些路由在数据库中对每个客户端进行单独的限流。最后感谢大家的阅读。
## Ocelot简易教程目录
1. [Ocelot简易教程（一）之Ocelot是什么](https://www.cnblogs.com/yilezhu/p/9557375.html)
2. [Ocelot简易教程（二）之快速开始1](https://www.cnblogs.com/yilezhu/p/9563188.html)
3. [Ocelot简易教程（二）之快速开始2](https://www.cnblogs.com/yilezhu/p/9638417.html)
4. [Ocelot简易教程（三）之主要特性及路由详解](https://www.cnblogs.com/yilezhu/p/9664977.html)
5. [Ocelot简易教程（四）之请求聚合以及服务发现](https://www.cnblogs.com/yilezhu/p/9695639.html)
6. [Ocelot简易教程（五）之集成IdentityServer认证以及授权](https://www.cnblogs.com/yilezhu/p/9807125.html )
7. [Ocelot简易教程（六）之重写配置文件存储方式并优化响应数据](https://www.cnblogs.com/yilezhu/p/9839863.html)
