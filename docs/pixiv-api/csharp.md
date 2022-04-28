# 储存库

[![](https://github-readme-stats.vercel.app/api/pin/?username=git-xiaocao&repo=pixiv-csharp-api&theme=omni)](https://github.com/git-xiaocao/https://github.com/git-xiaocao/pixiv-csharp-api)


## 特点

1. .NET 6.0
2. C# 10.0
3. 初始化和刷新AuthToken支持
4. WebView2登录支持
5. 免代理IP直连

## 使用方法

1. `NuGet` 搜索 `PixivCSharpAPI` 并安装  
![](csharp-resources/setp1.png)
2. 引入命名空间
```csharp
using PixivAPI;
```
3. 创建一个获取用户账号的函数  
```csharp
public UserAccountDTO GetUserAccount()
{
    return ...;
}
```
4. 创建`AuthClient`  
```csharp
var authClient = new AuthClient(GetUserAccount, new("zh-CN"));
```
5. 创建`ApiClient`  
```csharp
var apiClient = new ApiClient(authClient, new("zh-CN"));
```

