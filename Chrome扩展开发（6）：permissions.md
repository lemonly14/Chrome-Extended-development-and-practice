# Chrome 扩展开发（3）： Chrome 扩展中的双刃剑 —— permissions

>作者：雷宇（leiyu@star-net.cn）

## permissions 简介

Chrome 扩展的功能十分丰富且强大，但是它不是一开始就这么强大，需要开发者根据需求开放相应的权限。比如开发者想要储存数据，可以开放 `storage` 权限将数据缓存在本地，我想查看或者修改请求和响应，开放 `webRequest` 即可。 Chrome 扩展能使用的权限可不止于此。

Chrome 扩展能够提供给开发者如此开放的权限的同时，也就意味用户其实是处在一个很危险的境地。比如开发者背地获取用户信息是很简单的事情，比如开发者使用 `cookies` 权限窃取用户的 cookie 信息。所以不建议使用不知名的 Chrome 扩展，还是使用业内有口皆碑的 Chrome 扩展比较保险。


## permissions 开放方式

Chrome 扩展开放权限的方式也是很简单:

* 方式一( Manifest 配置文件声明)，可以在 *permissions* 配置项中声明各项权限。

```JS
"permissions": [
   "contextMenus",    // 右键菜单
   "tabs",            // 标签
   "notifications",   // 通知
   "webRequest",      // web请求
   "storage",         // 本地存储
   "http://*/*",      // 可以通过executeScript或insertCSS访问网站
   "https://*/*",
   "<all_urls>",
],

```

* 方式二( [chrome.permissions][1] )，使用 Chrome 提供的 api ，用于实现可选权限。在 Chrome 扩展的运行过程中，而不是在安装的时候请求权限。这能帮助用户清楚为什么需要此权限，且仅在必要的时候才运行扩展使用此权限。

```JS
document.querySelector('#my-button').addEventListener('click', function(event) {
    chrome.permissions.request({
      permissions: ['tabs'],
      origins: ['http://www.google.com/']
    }, function(granted) {
      // The callback argument will be true if the user granted thepermissions.
      if (granted) {
        doSomething();
      } else {
        doSomethingElse();
      }
    });
});
```
>云助理核赔扩展毕竟只是一个功能简单的 Chrome 扩展，只使用了部分权限。更多权限可以去[官方网站][2]深入了解，发挥想象力写出超棒的 Chrome 扩展。

[1]:https://developer.chrome.com/extensions/permissions
[2]:https://developer.chrome.com/extensions/api_index
