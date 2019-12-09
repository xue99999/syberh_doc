## 前提
syberh_framework模块需要在OS2.1和OS4.1上一起运行起来，对我们框架而言，差别就在于不同qt版本的浏览器内核不一样
- OS2.1 内核是webkit
- OS4.1 内核是chrome

## 实现思路
### C++ 
通过宏来判断QT版本

### QML
qmldir 里面加版本号区分
页面适配

### js
C++ helper拓展一个方法，传给core.js，然后可以判断创建相应的qml文件


## 具体实现
### C++
1、 QT webview库

```C++
//App_Workspace.cpp  QT版本大于5.6
#if (QT_VERSION >= QT_VERSION_CHECK(5, 6, 0))
#include <qtwebengineglobal.h>
#endif
```

2、分辨率适配
OS4.1 中`cenvironment.h`文件实现了`env.dp(num)`方法用来做适配
OS2.1 中也有这个文件，但是并没有这个方法

我们的QML插件需要跑在二者上，在OS2.1上（QT版本小于5.6）我们自己实现了一个类，拓展了此方法`env.dp(1)`，页面就可以正常使用了

```C++
#include <cenvironment.h>
//App_Workspace.cpp QT版本大于5.6
#if (QT_VERSION >= QT_VERSION_CHECK(5, 6, 0))
#else
#include "../../vendor/syberh-framework/src/senvironment.h"
#endif

// QT版本大于5.6，选择进入特定的qml页面, qml适配， 266是9860手机的dpi值
#if (QT_VERSION >= QT_VERSION_CHECK(5, 6, 0))
CEnvironment::initAppDpi(266);
m_view->setSource(QUrl("qrc:/qml/main59.qml"));
#else
SEnvironment *env = new SEnvironment;
m_view->rootContext()->setContextProperty("env", env);
m_view->setSource(QUrl("qrc:/qml/main.qml"));
#endif
```

### QML
1、如何根据系统版本不同，实现2个入口
在`App_Workspace.cpp`中，根据QT版本把首页设置成`main.qml` 或者 `main59.qml`

```javascript
// syberh-framework/qmldir 中定义59.0版本
module com.syberos.api
Syberh 1.0 js/syber.js
Syberh 59.0 js/syber.js

SWebview 1.0 qml/SWebview.qml
SWebview 59.0 qml/SWebview59.qml

SPage 1.0 qml/SPage.qml
SPage 59.0 qml/SPage59.qml
```

```javascript
import com.syberos.api 59.0

//main59.qml
SPage59 {

}
```

```javascript
//SPage59.qml
SWebview59 {

}
```

`SWebview59` 和 `SWebview` 区别就在于，一个是`Webview`，一个是`WebengineView`实现的

2、QML元素适配

用到数字的地方用`env.dp(num)`来做适配

### JS
C++ `helper.h`拓展一个方法，传给`core.js`，然后可以判断创建相应的qml文件
`core.js` 中定义了一个方法
```javascript
  this.moduleVersion = helper.isGtQt56() ? '59.0' : '1.0';
```

webview创建新的qml页面，根据此参数来确定是`SWebview59`还是`SWebview`