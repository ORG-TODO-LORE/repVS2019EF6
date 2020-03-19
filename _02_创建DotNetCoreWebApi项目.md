# 创建DotNetCoreWebApi项目

## 项目创建
1. 使用VS2019创建Asp.Net Core Web 应用程序
2. Core版本选择2.1 选择API项目，移除 Https 支持

## 添加Area
1. 项目上添加“新搭建基架的项目”，选择MVC区域，点击添加，输入名称，点击确定
2. 添加后确保以下内容在Startup.cs中
```
//app.UseMvc();
            app.UseMvc(routes =>
            {
                routes.MapRoute(
                  name: "areas",
                  template: "{area:exists}/{controller=Home}/{action=Index}/{id?}"
                );
            });
```

## 添加测试的Controller与Action
1. 在刚添加的Area中添加一个ApiController
2. 添加一个Action，代码如下
```
[HttpGet]
public HttpResponseMessage TestAct1([FromQuery]string para1)
{
            HttpResponseMessage res = new HttpResponseMessage();
            res.StatusCode = System.Net.HttpStatusCode.OK;
            res.Content = new StringContent("{A:1, B:2}");
            return res;
}
```
