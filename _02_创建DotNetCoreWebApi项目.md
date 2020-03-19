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
3. 尝试使用 http://localhost:10670/api/BIMComp/BIMCompFamily/TestAct1?para1=2 访问，发现无法访问
4. 修改刚添加的Controller的Route标记，修改后代码如下：
```
[Route("api/BIMComp/[controller]/[action]")]
[ApiController]
public class BIMCompFamilyController : ControllerBase
{
    [HttpGet]
    public HttpResponseMessage TestAct1([FromQuery]string para1)
    {
        HttpResponseMessage res = new HttpResponseMessage();
        res.StatusCode = System.Net.HttpStatusCode.OK;
        res.Content = new StringContent("{A:1, B:2}");
        return res;
    }
}
```
5. 访问后，发现数据不正确

## 添加 WebApiCompatShim
1. 使用程序包管理控制台，安装 WebApiCompatShim：Install-Package Microsoft.AspNetCore.Mvc.WebApiCompatShim -Version 2.1.3
2. 修改 services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1)，在其后追加：.AddWebApiConventions();
3. 调试：直接运行，再附加到进程，dotnet.exe，别忘了打断点

## 配置 Swagger
1. 安装 Swagger 要用的包：Install-Package Swashbuckle.AspNetCore
2. 引入 using Swashbuckle.AspNetCore.Swagger;
3. 将 Swagger 生成器添加到 Startup.ConfigureServices 方法中的服务集合中：
```
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    // 将 Swagger 生成器添加到 Startup.ConfigureServices 方法中的服务集合中：
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });
    });

    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1).AddWebApiConventions(); 
}
```
4. 在 Startup.Configure 方法中，启用中间件为生成的 JSON 文档和 Swagger UI 提供服务：
```
// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    //启用中间件服务生成Swagger作为JSON终结点
    app.UseSwagger();

    //启用中间件服务对swagger-ui，指定Swagger JSON终结点
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
    });

    //app.UseMvc();
    app.UseMvc(routes =>
    {
        routes.MapRoute(
          name: "areas",
          template: "{area:exists}/{controller=Home}/{action=Index}/{id?}"
        );
    });
}
```
5. 编译启动测试
6. 坑1：Internal Server Error /swagger/v1/swagger.json
    1. 请确保所有Action都加上了HttpPost或HttpGet等特性
