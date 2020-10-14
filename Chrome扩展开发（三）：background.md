# Chrome 扩展中的支柱——background

>作者：雷宇（leiyu@star-net.cn）

## background 简介

在第二节中的人寿 OCR 识别 Chrome 扩展的 **Manifest** 配置中，我们已经见过了 *background* 这个角色了。

``` json
//常驻的后台
  "background": {
    "scripts": [
      "js/background.js"
    ]
  }
```
在 **Manifest** 中指定 **background** 可以使扩展常驻后台。*background* 可以包含3种属性，分别为*scripts* 、*page* 和 *persistent* 。
* **scripts**：如果指定 *scripts* 属性则 Chrome 会在扩展中自动生动一个包含指定脚本的页面；
* **page**：如果指定了*page*属性，则 Chrome 会在扩展启动时创建一个 HTML 文件作为后台页面运行。
* **persistent**：定义常驻后台的方式，当其值为 true 时，表示扩展将一直在后台运行；当其值为 false 时，表示扩展按需运行，这也是*Event Page*。使用 *Event Page* 可以有效减小扩展对内存的消耗。目前正常的电脑对于这种级别的吃内存，可以说轻松应对。


但是一般情况下 *background* 这个页面我们根本看不到以至于不需要的除非需要构建特殊的HTML，所以我们只需要用 *scripts* 属性即可。在人寿 OCR 识别扩展中也是只使用   *scripts* 属性。

------------------------------------------------------------
### Event Page

这里顺便提一下 **Event Page** ，毕竟第一节立了 flag ~

它是一个什么东西呢？鉴于 *background* 生命周期太长，长时间挂载后台可能会影响性能，所以 Google 又弄一个 *event-pages* ，在配置文件上，它与 *background* 的唯一区别就是多了一个 persistent 参数

它的生命周期是：在被需要时加载，在空闲时被关闭。

``` json
//常驻的后台

  "background":
  {
  	"scripts": ["event-page.js"],
  	"persistent": false
  }
```

在人寿 OCR 识别扩展中，我需要频繁的利用到 *background* 的能力，所以我并没有使用到 *Event Page* 。既然说到我频繁利用 *background* 的能力，我一定要好好吹 *background* 一下。 *background* 作为扩展程序中的唯一大爹实在太强了，脏活累活一把抓。

----------------------------
## background 的能力

* 储存数据

上面提到的 *background* 可以使扩展常驻后台，这意味的什么呢？这使得 *background* 的生命周期超长。可以这么说， Chrome 只要开着 *background* 就在，有点狗皮膏药的感觉。我在项目中利用这点存储数据。在其他地方取数据时也是十分方便。
其实也可以开启扩展的 `storage` 权限类似与 cookies 一样实现存储数据，后续章节会提及扩展的权限其中包括 `storage` 。

``` js

window.initInsurantInfo = {}  //出险人信息

window.compareInsurantInfo = {}  //出险人对比信息

window.initAccidentInfo = {} //出险信息

window.compareAccidentInfo = {}  //出险对比信息

window.initApplicantInfo = {}  //申请人信息

window.compareApplicantInfo = {} //申请人对比信息

window.initBankInfo = {}  //银行信息

window.compareBankInfo = {}  //对比银行信息

```

* 发起请求

这也是 *background* 非常强力的地方，可以无限制跨域，也就是可以跨域访问任何网站而无需要求对方设置 CORS 。前端也不需要像写多余的配置应对跨域问题。

``` js

//引入项目需要的接口

import { checkIdenityInfo, checkDischargeSummary, checkElectronicReceipt, checkBank, getAngle, checkLock, getStatistics } from "./api";

window.checkIdenityInfo = checkIdenityInfo

window.checkDischargeSummary = checkDischargeSummary

window.checkElectronicReceipt = checkElectronicReceipt

window.checkBank = checkBank

window.getAngle = getAngle

window.checkLock = checkLock

window.getStatistics = getStatistics
```

``` js
//其他页面不能发起请求，都需要经过background。
const bgPage = chrome.extension.getbackgroundPage();
const idenityInfo = await bgPage.checkIdenityInfo(
  imageList,
  angleList,
  id,
  retrieve
);
```

 `chrome.extension.getbackgroundPage()` 是 popup 页面与 background 通讯的一种方式,后续的关于通讯的章节会有提及。


* 使用扩展开启的额外权限

*background* 是作为扩展中的唯一大爹权限非常的高，几乎可以调用所有的 Chrome 扩展 API （除了 devtools ）。在 Manifest 配置申请的权限都是可以使用的。

``` js
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
```

这里是在 *background* 利用 webRequest 权限监听网络请求获取请求参数并存入缓存
``` js
// 监听网络请求
chrome.webRequest.onBeforeRequest.addListener(
  function (details) {
    // console.log('看看有哪些请求', details)
    if (details.url.endsWith('/claimReg/get')) {
      chrome.tabs.query({
        active: true,
        currentWindow: true
      }, (tabs) => {
        let message = {
          type: '3',
        }
        chrome.tabs.sendMessage(tabs[0].id, message, res => {
          console.log('监听网络请求之后发起的请求')
        })
      })
    }
    if (details.url.endsWith('/getClaimCntrAndClaimCntrExtList')) {
      // 缓存案子的id
      window.initCaseId = details.requestBody.formData.claimNo[0]
      chrome.storage.local.get('caseId', (info) => {
        if (window.initCaseId !== info.caseId) {
          window.initBgPage()
        }
      })
      chrome.storage.local.set({
        caseId: window.initCaseId
      })
    }
  },
  { urls: ["<all_urls>"] },
  ["blocking", "requestBody"],
)

```
