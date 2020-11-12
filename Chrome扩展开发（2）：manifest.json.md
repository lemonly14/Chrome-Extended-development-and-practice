# Chrome 扩展开发与实践（一）：manifest.json

>作者：雷宇（leiyu@star-net.cn）
## manifest.json 简介

每一个 Chrome 都包括一个 **Manifest** 文件———— manifest.json ，这个文件可以告诉 Chrome 关于这个扩展的相关信息，它是整个扩展的入口，也是 Chrome 扩展必不可少的部分。

Chrome 扩展的 Manifest 必须包含`name、version`和`manifest_version`属性，目前来看`manifest_version`属性值只能为数字 **2**。

其他常用的可选属性还有 *browser_action* 、*background* 、*content_page*、  *permissions* 等等，详细的可选属性可以参考[官方文档](https://developer.chrome.com/extensions/manifest)，这里的官方文档需要科学上网才能访问到。后续会有一个福利章节其中会介绍我本人正在使用非常好用的 Chrome 扩展，其中包括了利用 Chrome 扩展科学上网 ~~

---------------------------

## 云助理核赔扩展的 manifest.json 配置

这边贴上云助理核赔扩展的配置，我也将进行一一介绍。


* **manifest.json**
``` JSON
  {
  //清单文件的版本，必须项且值必须为2
  "manifest_version": 2,
  //扩展名称
  "name": "__MSG_extName__",
  //扩展版本
  "version": "1.0.0",
  //扩展主页
  "homepage_url": "http://localhost:8080/",
  //扩展描述
  "description": "回填插件",
  // 默认语言
  "default_locale": "zh_CN",
  //申请权限
  "permissions": [
    "contextMenus",
    "tabs",
    "notifications",
    "webRequest",
    "webRequestBlocking",
    "storage",
    "activeTab",
    "<all_urls>",
  ],
  //图标
  "icons": {
    "16": "icons/16.png",    //用于扩展页面的图标
    "48": "icons/48.png",    //用于扩展管理页面(chrome://Extensions)
    "128": "icons/128.png"   //在安装和 Chrome 网络商店中使用
  },
  //常驻的后台
  "background": {
    "scripts": [
      "js/background.js"
    ]
  },
  "content_security_policy": "script-src 'self' 'unsafe-eval'; object-src 'self'",
  // 需要直接注入页面的JS
  "content_scripts": [
    {
      "matches": [
        "<all_urls>"   //<all_urls>表示匹配所有的地址
      ],
      "js": [
        "js/content.js"   //注入的JS
      ],
      "run_at": "document_end"  //代码注入的时间，可选值： "document_start", "document_end", or "document_idle"，最后一个表示页面空闲时，默认document_idle
    }
  ],
  // 浏览器右上角图标设置，browser_action、page_action、app必须三选一
  "browser_action":
	{
		"default_icon": "img/icon.png",
		// 图标悬停时的标题，可选
		"default_title": "人寿ocr识别插件",
		"default_popup": "popup.html"
	},
  // 当某些特定页面打开才显示的图标
	/*"page_action":
	{
		"default_icon": "img/icon.png",
		"default_title": "我是pageAction",
		"default_popup": "popup.html"
	},*/

  // Chrome40以前的插件配置页写法
  "options_page": "options.html",
  // Chrome40以后的插件配置页写法，如果2个都写，新版Chrome只认后面这一个
  "options_ui": {
    "page": "options.html",
    "chrome_style": true
  }
}
```

- [ `matches` 的语法参考](https://developer.chrome.com/extensions/match_patterns)
