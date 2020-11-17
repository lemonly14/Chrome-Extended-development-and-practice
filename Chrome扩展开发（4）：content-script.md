# 4. Chrome扩展中的联络员：content-script

>作者：雷宇（leiyu@star-net.cn）

## content-script 简介

所谓 *content-scripts* ，其实就是 Chrome 扩展中向页面注入脚本的一种形式（虽然名为 script ，其实还可以包括 css 的），借助 *content-scripts* 我们可以实现通过配置的方式轻松向指定页面注入 JS 和 CSS 。

在 Manifest 配置中，也是有关于 *content-script* 的配置信息。

``` js
{
  // 需要直接注入页面的脚本
  "content_scripts":
  [
    {
      // "matches": ["http://*/*", "https://*/*"],
      // "<all_urls>" 表示匹配所有地址
      "matches": ["<all_urls>"],
      // 多个JS按顺序注入
      "js": ["js/jquery-1.8.3.js", "js/content-script.js"],
      // JS的注入可以随便一点，但是CSS的注意就要千万小心了，因为一不小心就可能影响局样式
      "css": ["css/custom.css"],
      // 代码注入的时间，可选值： "document_start"、"document_end"、"document_idle"，最后一个表示页面空闲时，默认document_idle
      "run_at": "document_start"
    }
  ]
}
```
在 Manifest 中指定 *content-scripts* 可以使 Chrome 扩展向页面注入 JS 和 CSS 。 *content-scripts* 主要包含4种属性，分别为*matches*、*css*、*js* 和 *run_at* 。
* **matches**：匹配需要注入 js 和 css的页面
* **js**：需要注入的js文件，甚至可以注入jquery
* **css**：需要注入的css文件，这里需要注意css的注入可能会影响局部的样式
* **run_at**：代码注入的时间，可选值： "document_start"、"document_end"、"document_idle"，最后一个表示页面空闲时，默认为 document_idle。

-----------------------------

## content-scripts 的能力


* 共享页面 DOM

Chrome 扩展中的各个部分都有同对方通信的方式，为什么偏偏 *content-scripts* 是联络员呢？因为 *content-scripts* 具备了其他扩展成员不具备的能力——和页面“进行通讯”。通过 *content-scripts* ， Chrome 扩展可以获取和操作页面上的 DOM 。云助理核赔扩展获取医疗信息和回填医疗信息就是使用了这个能力。

获取页面信息：

``` JS
// 获取出险人信息对象
function getInsurantInfo() {
  setTimeout(() => {
    let insurantInfo = {}
    insurantInfo.name = document.getElementById('opsnName').value
    insurantInfo.birthday = document.getElementById('opsnBirthDate').value
    insurantInfo.gender = sexMap[document.getElementById('opsnSex').value]
    insurantInfo.idType = idTypeMap[document.getElementById('opsnIdType').value]
    insurantInfo.idNo = document.getElementById('opsnIdNo').value
    insurantInfo.idValidDate = document.getElementById('opsnIdValidDate').value
    chrome.runtime.sendMessage({
      initInsurantInfo: insurantInfo
    })
  }, 1200)
}

```

回填识别信息（以回填出险信息为例）：

``` JS
let rowLists = document.getElementsByClassName('haloui-lg-row')
// 回填出险时间
function fillBeginTime(startTime) {
  // 出险时间输入框在第四个列表
  let rowList = rowLists[9]
  //出险时间输入框在列表的第二个框
  let formInput = rowList.getElementsByClassName('form-control')
  // 给出险时间输入框赋值
  formInput[2].value = startTime
}

```

* 使用部分 Chrome 提供的 api

*content-scripts* 只能使用部分的 Chrome 提供的 api ：

  1. **chrome.extension(getURL , inIncognitoContext , lastError , onRequest , sendRequest)**
  2. **chrome.i18n**
  3. **chrome.runtime(connect , getManifest , getURL , id , onConnect , onMessage , sendMessage)**
  4. **chrome.storage**

看到这里并不需要担心，因为 *content-scripts* 可以利用通信机制来联系大哥 *background* 帮助，并返回需要的信息。

----------------------------
## inject-script

*content-scripts* 虽然很出色地完成了联络员的使命：可以联系页面的 DOM 元素，但是有个较大的缺陷，就是 *content-scripts* 没有办法和页面的 JavaScript 部分打交道。这样就意味着，无法操作页面的 DOM 元素来访问 *content-scripts* 。但是，“通过操作页面上的元素比如点击页面按钮来调用扩展 api ”是一个很常见的需求。如果这样的话，Chrome 扩展的限制性就有点大，无法很好地支持前端工程师和产品经理天马行空的想法了！

这要怎么办呢？有条件要上，没有条件创造条件也要上！

我们在 *content-script* 中通过DOM方式向页面注入 *inject-script* :

```js

// 向页面注入JS
function injectCustomJs(jsPath) {
   jsPath = jsPath || 'js/inject.js';
   var temp = document.createElement('script');
   temp.setAttribute('type', 'text/javascript');
   // 获得的地址类似：chrome-extension://ihcokhadfjfchaeagdoclpnjdiokfakg/js/inject.js
   temp.src = chrome.extension.getURL(jsPath);
   temp.onload = function()
   {
   	 // 放在页面不好看，执行完后移除掉
   	 this.parentNode.removeChild(this);
   };
   document.head.appendChild(temp);
}
```
> 页面中直接访问插件中的资源的话必须显示声明才行，Manifest配置文件中增加如下：
> ``` JS
> {
>     // 普通页面能够直接访问的插件资源列表，如果不设置是无法直接访问的
>     "web_accessible_resources": ["js/inject.js"],
> }
> ```

