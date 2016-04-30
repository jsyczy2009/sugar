# sugar.js
---

## 1. 组件模块化机制
根函数模块 `src/main/root.js` 实现 sugar 底层模块的继承方法 `extend`，每个模块都是从 `Root` 为根源继承而来， `extend` 是一个通过原型实现的类式继承（单继承），对传入的每个子类方法会挂载一个 `Super` 来实现对父类的调用。


系统基础模块 `Module` 的定义在 `src/main/module.js` 中，`Module` 继承于 `Root`，定义了 sugar 的模块系统和基本形态，实现系统模块的管理、创建和消息通信等基础功能。


视图组件基础模块 `Component` (`src/main/component.js`) 继承于 `Module` ，在 `Module` 的基础上扩展成一个视图容器类（构造函数），实现容器的生成、模板的拉取、视图的渲染。以 `Component` 为基础可以继承出自定义形态的视图组件。


sugar 的模块继承关系：

<img src="http://7xodrz.com1.z0.glb.clouddn.com/sugar-extend-cp">


## 2. 视图组件定义
sugar 的视图组件都是从 `Component` 继承而来，通过 `init` 方法来定义视图的初始数据和状态：

```javascript
var sugar = reuqire('dist/sugar.min');

var Page = sugar.Component.extend({
	// 配置字段可以缺省也可以自定义
	// `init` 传入的参数 config 是生成(创建) Page 实例时的配置
	// 生成实例时如有定义 config 将会覆盖掉原有配置，从而实现功能相似模块的复用
	init: function(config) {
		config = this.cover(config, {
			// 视图创建的目标容器
			'target'  : null,
			// 视图元素的标签
			'tag'     : 'div',
			// 元素的 class
			'class'   : '',
			// 元素的 css
			'css'     : null,
			// 元素的 attr
			'attr'    : null,
			// 视图布局内容
			'html'    : '',
			// 视图模板拉取地址
			'template': '',
			// 模板拉取请求参数
			'tplParam': null,
			// mvvm 数据模型对象
			'model'   : null,
			// 视图渲染完成后的回调函数
			'cbRender': 'viewReady'
		});
		// 调用父类 (sugar.Component) 的 init 方法来完成视图的渲染
		this.Super('init', arguments);
	}
});
```


## 3. 视图组件创建
将视图组件创建到 DOM 的方式有两种：

1. 通过核心实例 `sugar.core` 创建顶层模块：

	```javascript
	var sugar = reuqire('dist/sugar.min');

	var Page = sugar.Component.extend({
		init: function() {
			// ...
		}
	});

	sugar.core.create('page', Page, {
		'target': document.querySelector('body')
	});
	```
2. 在视图组件内部创建子视图组件：

	调用自身 `create` 方法创建：

	```javascript
	var sugar = reuqire('dist/sugar.min');

	var SubPage = sugar.Component.extend({
		init: function() {
			// ...
		}
	});

	var Page = sugar.Component.extend({
		init: function() {
			// ...
		},
		// 每个视图组件渲染完成之后默认调用 viewReady 方法
		viewReady: function() {
			// 创建 SubPage 到当前视图容器
			this.create('subpage', SubPage, {
				'target': this.el
			});
		}
	});

	// 创建 Page 到 body
	sugar.core.create('page', Page, {
		'target': document.querySelector('body')
	});
	```


# API
---

## 1、Module 基础模块实例方法
系统模块基础类，实现所有模块的通用方法

* #### `create(name, Class, config)` 同步创建子模块实例

	* 参数说明：
		```json
		{String}   name   <必选> [子模块名称]
		{Function} Class  <必选> [用于创建子模块实例的类（生成函数），通常是继承于 sugar.Component 的类]
		{Object}   config <可选> [子模块的配置参数]
		```

	* 返回值：创建成功后的子模块实例

	* 用法示例：
		```javascript
		// sugar.Component 为视图组件基础模块的类，继承于 sugar.Module ，用于定义视图组件

		var Page = sugar.Component.extend({
			// 定义视图组件
		});
		// 或者采用引入方式：var Page = require('/path/to/page');

		sugar.core.create('page', Page, {
			// 模块配置参数
		});
		```


* #### `getParent()` 获取当前模块实例的父模块实例（创建者）

	* 参数说明：不需要参数

	* 返回值：父模块实例或 null，null 表示该模块是由 sugar.core 实例创建的顶层模块

	* 用法示例：
		```javascript
		// 定义 header 视图组件
		var Header = sugar.Component.extend({
			// viewReady 为每个视图组件创建完毕后默认调用的方法
			viewReady: function() {
				var parent = this.getParent(); // => Page实例
			}
		});

		// 定义页面视图组件
		var Page = sugar.Component.extend({
			viewReady: function() {
				// 创建头部模块
				this.create('header', Header, {
					// 创建header模块的配置参数
				});

				var parent = this.getParent(); // => sugar.core实例
			}
		});

		var main = sugar.core.create('page', Page, {
			// 模块配置参数
		});
		var parent = main.getParent(); // => null
		```


* #### `getChild(name)` 获取当前模块实例的指定子模块实例

	* 参数说明：
		```json
		{String} name <必选> [子模块的名称]
		```

	* 返回值：子模块实例，不存在则返回null

	* 用法示例：
		```javascript
		// 定义 header 视图组件
		var Header = sugar.Component.extend({/* 定义视图组件…… */});

		// 定义页面视图组件
		var Page = sugar.Component.extend({
			viewReady: function() {
				// 创建头部模块
				this.create('header', Header, {
					// 创建header模块的配置参数
				});

				var head = this.getChild('header'); // => header模块实例
			}
		});
		```


* #### `getChilds(returnArray)` 获取当前模块实例的所有子模块实例

	* 参数说明：
		```json
		{Boolean} returnArray <可选> [返回的子模块集合是否是数组的形式，否则为映射对象形式]
		```

	* 返回值：子模块集合

	* 用法示例：
		```javascript
		// 定义 header 视图组件
		var Header = sugar.Component.extend({/* …… */});

		// 定义 footer 视图组件
		var Footer = sugar.Component.extend({/* …… */});

		// 定义 sidebar 视图组件
		var Sidebar = sugar.Component.extend({/* …… */});

		// 定义页面模块类
		var Page = sugar.Component.extend({
			viewReady: function() {
				// 创建三个子模块
				this.create('header', Header);
				this.create('footer', Footer);
				this.create('sidebar', Sidebar);

				var chids = this.getChilds();
				/*  {
						'header' : Object
						'footer' : Object
						'sidebar': Object
					}
				 */

				var chids = this.getChilds(true);
				// [header, footer, sidebar]
			}
		});
		```


* #### `setTimeout(callback, time, param)` 修正作用域的定时器

	* 参数说明：
		```json
		{Function|String} callback <必选> [定时器回调函数，字符串时为模块属性]
		{Number}          time     <可选> [定时器延迟时间毫秒数，不传为0]
		{Mix}             param    <可选> [回调函数参数]
		```

	* 返回值：定时器的id

	* 用法示例：
		```javascript
		var Page = sugar.Component.extend({
			viewReady: function() {
				this.$name = 'SUGAR';

				this.setTimeout(function() {
					console.log(this.$name); // 匿名函数作用域也是this，所以打印结果为'SUGAR'
				}, 100);

				this.setTimeout('delayLog', 1000, this.$name);
			},
			delayLog: function() {
				console.log(arguments); // [SUGAR']
			}
		});
		```


* #### `fire(name, param, callback)` 模块通信：冒泡方式发送消息

	* 参数说明：
		```json
		{String}   name     <必选> [消息名称]
		{Mix}      param    <可选> [消息参数（内容）]
		{Function} callback <可选> [消息被接受者接收完毕后的回调函数]
		```

	* 返回值：无

	* 用法示例：
		```javascript
		// 定义导航视图组件
		var Nav = sugar.Component.extend({
			viewReady: function() {
				// 向父模块发送消息，将会冒泡到每一层父模块
				this.fire('navCreated', 123);
			},
			// 默认接收消息方法用on + 消息名首字母大写，以区分其他类型的方法
			// 消息会从本模块开始触发接收方法
			onNavCreated: function(ev) {
				// ev.param => 123
			}
		});

		// 定义 Header 视图组件
		var Header = sugar.Component.extend({
			viewReady: function() {
				// 创建导航
				this.create('nav', Nav);
			},
			// Header 是 Nav 的父模块，能接收到 navCreated 消息
			onNavCreated: function(ev) {
				// ev.param => 123
				// return false // 假如在这里返回false，则不会将消息冒泡到Header的父模块Page中
			}
		});

		// 定义 Page 模块
		var Page = sugar.Component.extend({
			viewReady: function() {
				// 创建头部模块
				this.create('header', Header);
			},
			// Page 是 Header 的父模块，也能接收到 navCreated 消息
			onNavCreated: function(ev) {
				// ev.param => 123
				// 假如没有任何一层的消息return false，此消息还会将继续冒泡到Page的父模块，直到模块树的根
			}
		});

		// 以上，消息的传递顺序为子模块->父模块：Nav -> Header -> Page
		```


* #### `broadcast(name, param, callback)` 模块通信：向下广播方式发送消息

	* 参数说明：
		```json
		{String}   name     <必选> [消息名称]
		{Mix}      param    <可选> [消息参数（内容）]
		{Function} callback <可选> [消息被接受者接收完毕后的回调函数]
		```

	* 返回值：无

	* 用法示例：
		```javascript
		// 定义导航视图组件
		var Nav = sugar.Component.extend({
			// Nav 是 Header 的子模块，能接收到 pageReady 消息
			onPageReady: function(ev) {
				// ev.param => 456
			}
		});

		// 定义 LOGO 视图组件
		var LOGO = sugar.Component.extend({
			// LOGO 是 Header 的子模块，能接收到 pageReady 消息
			onPageReady: function(ev) {
				// ev.param => 456
			}
		});

		// 定义 Header 子模块
		var Header = sugar.Component.extend({
			viewReady: function() {
				// 创建导航
				this.create('nav', Nav);
				// 创建 LOGO
				this.create('logo', LOGO);
			},
			// Header 是 Page 的子模块，能接收到 pageReady 消息
			onPageReady: function(ev) {
				// ev.param => 456
				// return false // 假如在这里返回false，则不会将消息继续广播到Header的子模块Nav和LOGO中
			}
		});

		// 定义 Page 模块
		var Page = sugar.Component.extend({
			viewReady: function() {
				// 创建头部模块
				this.create('header', Header);

				// 向所有子模块发送广播消息
				this.broadcast('pageReady', 456);
			},
			// 默认接收消息方法用 on + 消息名首字母大写，以区分其他类型的方法
			// 消息会从本模块开始触发接收方法
			onPageReady: function(ev) {
				// ev.param => 456
			}
		});

		// 以上，消息的传递顺序为父模块->子模块：Page -> Header -> Nav、LOGO
		```


* #### `notify(receiver, name, param, callback)` 模块通信：向指定的模块发送消息

	* 参数说明：
		```json
		{String}   receiver <必选> [接受消息的模块实例名称，有层级时以.分隔并且要求完整层级]
		{String}   name     <必选> [消息名称]
		{Mix}      param    <可选> [消息参数（内容）]
		{Function} callback <可选> [消息被接受者接收完毕后的回调函数]
		```

	* 返回值：无

	* 特殊说明：fire 和 broadcast 方式要求通信的模块之间存在父子关系，而 notify 方式可以用于任何两个模块之间的通信，并且不会在发送者实例上开始触发消息

	* 用法示例：
		```javascript
		var PageA = sugar.Component.extend({
			viewReady: function() {
				this.notify('page_b', 'helloFromA', 'Are you ok?');
			}
		});

		var PageB = sugar.Component.extend({
			onHelloFromA: function(ev) {
				// ev.param => 'Are you ok?'
			}
		});

		sugar.core.create('page_b', PageB);
		```


* #### `destroy(notify)` 模块销毁

	* 参数说明：
		```json
		{Boolean} notify <可选> [为 true 时，销毁后会向父模块发送 subDestroyed 消息]
		```

	* 返回值：无

	* 特殊说明：模块在 destroy 在之前会调用当前模块的 beforeDestroy 方法，销毁后会调用 afterDestroy 方法

	* 用法示例：
		```javascript
		var Page = sugar.Component.extend({/* …… */});

		var page = sugar.core.create('page', Page);

		page.destroy(); // page => null
		```


## 2、core 实例方法
core 是由 sugar 中的核心实例（继承于 Module 类），所以拥有以上基础模块所有的实例方法，另外拓展了两个自身方法：

* #### `get(name)` 获取顶层模块实例

	* 参数说明：
		```json
		{String} name <必选> [子模块的名称]
		```

	* 返回值：由 sugar.core 创建的顶层模块实例，不存在或已销毁则返回 null

	* 特殊说明：该方法只是实现用 sugar.core.get(name) 来代替 sugar.core.getChild(name) 而已，两者相同


* #### `globalCast(name, param)` 模块通信：全局广播消息

	* 参数说明：
		```json
		{String} name  <必选> [消息名称]
		{Mix}    param <可选> [消息参数（内容）]
		```

	* 返回值：无

	* 用法示例：
		```javascript
		var user = 'xxx';
		// 全局广播 userLogin，将触发系统模块树中所有的模块实例的 onUserLogin 方法
		sugar.core.globalCast('userLogin', user);
		```


## 3、Component 视图组件实例方法
Component 类继承于 Module 类，所以 Component 的实例也有基础模块实例的所有方法。为了完善视图操作，自身拓展了一些方法和属性

* #### `init(config, parent)` 视图组件状态/参数初始化

	* 参数说明：
		```json
		{Object} config [视图组件生成实例的配置参数，可以覆盖和拓展 Component 类的参数]
		{Object} parent [父模块实例，没特殊需求基本上用不到]
		```

	* 返回值：无

	* 特殊说明：init 方法的作用是定义整个视图组件的初始状态、配置参数、MVVM 视图层初始化的，不建议在 init 方法里面含有任何业务逻辑的代码（虽然可以有）。视图组件在被创建成实例之后会默认调用 init 方法（详见 Module 的 create 方法实现）， init 方法会根据配置参数进行视图布局的渲染，在渲染完成后会调用 cbRender 参数指定的方法（默认为 viewReady ），业务逻辑比如事件绑定、数据加载等建议在 viewReady 中处理

	* 用法示例：
		```javascript
		// 这里的 Page 只是定义一个 Page 类（生成函数），这个 Page 类是继承于 Component 类的
		var Page = sugar.Component.extend({
			init: function(config) {
				// sugar.cover(子, 父)：子模块覆盖父模块参数
				config = sugar.cover(config, {
					'target'  : document.querySelector('body'), // 模块插入的目标容器
					'class'   : 'page-main',
					'tag'     : 'div',
					'html'    : 'I am page main',
					'cbRender': 'afterRender'
				});

				// Super 方法为调用父类方法，这里就是调用 sugar.Component.init(config)
				this.Super('init', [config]);
			},
			afterRender: function() {
				console.log('视图渲染完成，模块可以做后续处理了！');
			}
		});

		/*
		 *	需要注意的是：当调用模块创建方法时才会生成 Page 的一个实例
		 *	定义一个 Page 类，可以生成 N 个 Page 的实例
		 */
		var page1 = sugar.core.create('page1', Page);
		// page1 的 DOM 呈现为：<div class="page-main">I am page main</div>

		var page2 = sugar.core.create('page2', Page, {
			'class': 'page2-main',
			'tag'  : 'p'
		});
		// page2 的 DOM 呈现为：<p class="page2-main">I am page main</p>

		var page3 = sugar.core.create('page3', Page, {
			'html': 'I am page3'
		});
		// page3 的 DOM 呈现为：<div class="page-main">I am page3</div>

		/*
		 *	以上 page1、page2 和 page3 都是由 Page 生成的实例，可以通过配置不同参数使实例输出不同的形态
		 *	实例之间功能完全相同并且相互独立互不影响
		 */
		```


* #### `getConfig(name)` 获取模块配置

	* 参数说明：
		```json
		{String} name <可选> [获取指定的参数配置，不传则返回整个配置对象]
		```

	* 返回值：参数值或参数对象

	* 用法示例：
		```javascript
		var Page = sugar.Component.extend({
			init: function(config) {
				config = sugar.cover({
					'target': document.querySelector('body')
					'class' : 'page-main',
					'tag'   : 'b',
					'html'  : 'I am a page'
				});
				this.Super('init', arguments);
			},
			viewReady: function() {
				var html = this.getConfig('html');
				// 'I am a page'

				var cfg = this.getConfig();
				// {'class': 'page-main', 'tag': 'b', 'html': 'I am a page'}
			}
		});
		```


* #### `setConfig(name, value)` 设置模块配置

	* 参数说明：
		```json
		{String} name  <必选> [需要设置的配置参数名]
		{Mix}    value <必选> [设置的值，null则会删除该配置参数]
		```

	* 返回值：操作成功与否的Boolean

	* 用法示例：
		```javascript
		var Page = sugar.Component.extend({
			init: function(config) {
				config = sugar.cover({
					'target': document.querySelector('body')
					'class' : 'page-main',
					'tag'   : 'b',
					'html'  : 'I am a page'
				});
				this.Super('init', arguments);
			},
			viewReady: function() {
				var html = this.getConfig('html'); // 'I am a page'

				this.setConfig('html', 'I change my mind');

				html = this.getConfig('html'); // 'I change my mind'
			}
		});
		```



* #### `query(selector)` 获取/查找视图组件的DOM元素

	* 参数说明：
		```json
		{String} selector <必选> [元素选择器]
		```

	* 返回值：DOMElement

	* 用法示例：
		```javascript
		var Page = sugar.Component.extend({
			init: function(config) {
				config = sugar.cover(config, {
					'target': document.querySelector('body')
					'class' : 'page',
					'tag'   : 'p',
					'html'  : '<div class="header"><div class="nav"></div></div>'
				});
				this.Super('init', arguments);
			},
			viewReady: function() {
				var dom = this.el; // <p class="page">……</p>
				var nav = this.query('.nav'); // <div class="nav"></div>
			}
		});
		```


* #### `queryAll(selectors)` 获取/查找视图组件的DOM元素

	* 参数说明：
		```json
		{String} selectors <必选> [元素选择器]
		```

	* 返回值：NodeList


* #### `bind(elm, event, callback, capture)` 为元素添加绑定事件

	（参见 addEventListener 方法）


* #### `unbind(elm, event, callback, capture)` 从元素上移除事件绑定

	（参见 removeEventListener 方法）


## 4、ajax 实例方法
* #### `sugar.ajax.get(uri, param, callback, context)` Ajax GET 请求

	* 参数说明：
		```json
		{String}   uri      <必选> [请求地址]
		{Object}   param    <可选> [请求参数]
		{Function} callback <必选> [请求回调]
		{Object}   context  <可选> [回调函数执行上下文]
		```

	* 返回值：无

	* 用法示例：
		```javascript
		var Page = sugar.Component.extend({
			viewReady: function() {
				var param = {'page': 1, 'limit': 10};
				sugar.ajax.get('/article/list.php', param, this.dataBack);

				// 回调也可写成字符串形式：
				sugar.ajax.get('/article/list.php', param, 'dataBack', this);
			},
			dataBack: function(err, data) {
				// err 为请求错误或者服务器返回信息错误的对象，data 为请求成功的数据
			}
		});
		```


* #### `sugar.ajax.post(uri, param, callback, context)` Ajax POST 请求

	（参照 get 请求）


* #### `sugar.ajax.load(uri, param, callback, context)` 加载静态文本文件

	* 参数说明：（参照 get 请求）

	* 返回值：无

	* 用法示例：（参照 get 请求）
