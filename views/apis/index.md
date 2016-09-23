# mobile-router.js — A lightweight single page bone for mobile web App.轻量级web端单页面骨架

[![NPM Version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]

[Online docs](http://mrdocs.aijc.net/)

[GitHub](https://github.com/dolymood/mobile-router.js)

[DEMO](http://demo.aijc.net/js/M/examples/)

[require.js DEMO](http://demo.aijc.net/js/M/examples/requirejs/)

[mobile-router.js-demo](https://github.com/dolymood/mobile-router.js-demo)是一个关于mobile-router.js如何使用的DEMO，暂时包含：前后端模板共享，后端输出首屏，与其他库自由搭配，动画转场。

[mobile-router.js-sample](https://github.com/dolymood/mobile-router.js-sample) - A mobile-router.js demo like [ui-router sample](http://angular-ui.github.io/ui-router/sample/)

gzip后不到9k

### 优势：

* 使用简单、方便、轻量。

* 利用`CSS animation`控制动画转场（页面切换）效果，也可设置关闭动画效果。

* 支持路由视图嵌套 (1.5.0+)。

* 完整生命周期管理。

* 无依赖，可与其他框架（库）搭配自由使用，例如：`jquery`, `zepto`, `iscroll`等。

* 任意选择字符串式模板引擎，当然最简单的就是自己拼接字符串了；同时支持异步（远程获取模板，或者去请求数据在前端构建模板）；可配置是否缓存结果模板。

* 考虑后端渲染首屏的情况，只需要按结构输出响应的片段即可，利于`SEO`，且可以实现前后端模板公用。

* 自动缓存部分画面，可配置缓存数量，默认3个。

* 根据`hash`，可自由跳转到对应`id`元素位置。

* `history`、`hashbang`（默认）、`abstract`三种`history`模式任选。

### 一些注意点：

* 不管画面是否已缓存在页面中，只要切换回显示了，那么就会调用`callback`，而`callback`中大多数情况需要处理监听事件、操作`DOM`，这时候可根据`this.cached`来区分；当没有缓存在页面上时为`false`，或者缓存在页面上了，但是模板更新了，这时候也为`false`。

* `getTemplate`配置方法，如果带有参数，那么该参数就是得到模板字符串后的回调函数，一定要回调的；如果没有参数，直接返回模板字符串即可。这样做，主要是为了考虑异步获取（render）模板的场景。

* `M.history`的默认的 base path 是页面中`base`元素的`href`的值，如果没有，则默认是`/`；也可以在`M.history.start()`时传入。

* 对于[history](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Manipulating_the_browser_history)、[window.onpopstate](https://developer.mozilla.org/en-US/docs/WindowEventHandlers.onpopstate)不支持或者支持不够好的浏览器来说，自动回退到`hashbang`模式，如果`hashbang`也不支持的话，就退回到`abstract`模式了；`abstract`模式能够正常匹配对应`route`，但是不会产生历史记录。

### 使用方法：

```js
M.router.init([
	{
		path: '/',
		cacheTemplate: false, // 针对于当前的route，是否缓存模板
		animation: true, // 针对于当前的route，是否有动画
		aniClass: 'slideup', // 针对于当前的route，动画类型（效果）
		getTemplate: function() {
			return '/index';
		},
		onActive: function() { // 1.5.5+ 当路由被激活时调用，一般此时还没创建 `page-view` 元素
			
		},
		callback: function() { // 页面展示出来之后
			if (this.cached) return;
			// 处理操作...
		},
		onDestroy: function() {
			// 例如，处理一些解绑操作，销毁和DOM关联
		},
		onEnter: function(paramName) { // 1.5.3+ // 页面将要显示的时候

		},
		onLeave: function() { // 1.5.3+ // // 页面将要隐藏的时候

		}
	},
	{ // 支持 redirectTo 语法，可以是直接的 url 还可以是函数 (1.5.5+)
		path: '/redirectTo/:rtPath',
		redirectPushState: false, // 默认 true, 当激活redirectTo的时候是否启用`pushState`
		redirectTo: function(rtPath) {
			console.log('redirectTo', arguments, this);
			return '/' + rtPath;
		}
	},
	{ // support redirectTo , string url or function (1.5.5+)
		// 如果当前的route配置有getTemplate的话，此时会表现的像正常的
		// route一样依旧会创建`page-view`、会调用回调函数们、会做动画等。
		// 当正常行为结束之后（route的callback被调用之后）会触发redirectTo的逻辑
		path: '/contacts',
		getTemplate: contacts.getTemplate,
		onEnter: contacts.onEnter,
		onLeave: contacts.onLeave,
		callback: contacts.controller,
		onDestroy: contacts.onDestroy,

		redirectTo: '/contacts/list',
		redirectPushState: false,

		children: { // Nested routes & views! (1.5.0+)
			viewsSelector: '.content',
			cacheViewsNum: 1,
			routes: [
				{
					// all contacts
					path: '/list',
					getTemplate: list.getTemplate,
					onEnter: list.onEnter,
					onLeave: list.onLeave,
					callback: list.controller,
					onDestroy: list.onDestroy
				}
			]
		}
	},
	{
		path: '/m/:paramName',
		cacheTemplate: false, // 针对于当前的route，是否缓存模板
		getTemplate: function(cb) {
			// 这里模拟异步得到模板内容
			var that = this;
			// that.params 参数信息
			// that.query query信息
			setTimeout(function() {
				cb('/m/' + that.params.paramName);
			}, 200);
		},
		callback: function(paramName) {
			if (this.cached) return;
			// 处理操作...
		},
		onDestroy: function() {
			// 例如，处理一些解绑操作，销毁和DOM关联
		}
	},
	{ // 嵌套！！(1.5.0+)
		path: '/b/:bid',
		getTemplate: function(cb) {
			var path = this.path.substr(1);
			setTimeout(function() {
				var lis = '';
				var t;
				for (var i = 1; i <= 4; i++) {
					t = path + '/s' + i;
					lis += '<li><a href="' + t + '">/' + t + '</a></li>';
					// 或者：（这样的话 不会对地址栏有影响）
					// lis += '<li><a href="#" data-href="' + t + '">/' + t + '</a></li>';
				}
				cb(
					'<ul class="nav">' + lis + '</ul>'
				);
			}, 200);
		},
		callback: function() {
			console.log('callback:/b', this, arguments);
		},
		onDestroy: function() {
			// 当前被销毁时调用
			console.log('destroy:/b', this, arguments);
		},

		children: {
			/* 这些配置项 默认继承自 parent */
			viewsSelector: '',
			viewClass: 'sub-view-b',
			maskClass: 'mask',
			showLoading: true,
			cacheViewsNum: 3,
			cacheTemplate: true,
			animation: true,
			aniClass: 'slide',

			routes: [
				{
					path: '/:subB', // '/b/:bid' + '/:subB'
					/* 这里依旧可以设置 */
					cacheTemplate: false, // 针对于当前的route，是否缓存模板
					animation: true, // 针对于当前的route，是否有动画
					aniClass: 'slideup', // 针对于当前的route，动画类型（效果）
					getTemplate: function(cb) {
						var that = this;
						setTimeout(function() {
							cb('<div>' + that.path + '<p>sub content</p></div>');
						}, 200);
					},
					callback: function() {
						console.log('sub callback b', this, arguments);
					},
					onDestroy: function() {
						console.log('sub destroy b', this, arguments);
					}
				}
			]
			
		}
	}
], {
	/*是否缓存模板*/
	cacheTemplate: true,

	/*views容器选择器 如果为空，或者没有符合元素，那么views的容器元素就为body了*/
	viewsSelector: '',

	/*view的class 默认会有 page-view 的 class 这里配置的是其他增加的 class */
	viewClass: 'page-view2 page-view3',

	/*是否有动画*/
	animation: true,
	/*有动画的话，动画的类型*/
	aniClass: 'slide',

	/*蒙层class 主要是显示loading时的蒙层*/
	maskClass: 'mask',
	/*显示loading*/
	showLoading: true,

	/*缓存view数*/
	cacheViewsNum: 3
});

// 也可以通过这种形式添加
M.router.add('/ddd/{dddID:int}', function(dddID) {
	// 这是 callback 回调
}, {
	cacheTemplate: true,
	getTemplate: function() {
		return '/ddd/' + this.params.dddID;
	},
	onDestroy: function() {
		// destroy
	}
});

/* 监听route change */
/* routeChangeStart 是刚开始的时候被触发，此时还没有调用getTemplate得到模板内容 */
M.router.on('routeChangeStart', function(currentRouteState) {
	
});
/*已经完成动画切换（如有动画效果的话）显示出来之后触发*/
M.router.on('routeChangeEnd', function(currentRouteState) {
	
});

// 开始 监听history
M.history.start({
	// base: '/', // base path
	// enablePushState: true, // 启用pushstate （2.x之前版本）
	// (2.0.0+)
	// `history`, `hashbang` or `abstract`
	// default `hashbang` 
	history: true,
	// or
	hashbang: true,
	// or
	abstract: true
});

```

### 关于配置

`animation`、`aniClass`和`cacheTemplate`配置，依次取的是：

链接元素上的`data-xxx`->单个route规则中对一个的配置项->整体route配置规则中的配置。

### examples中示例

* `index.html`: 基本使用，都是默认配置，主要是关于`getTemplate`的2中方式以及在链接元素加入`data-rel=back`（反方向动画）配置。

* `index1.html`: 在"/c"中利用`data-href`达到不更新浏览器地址切换示例，且演示了如何才能局部禁用动画切换效果。

* `index2.html`: 关闭动画示例。

* `index3.html`: 不缓存模板示例。

* `index4.html`: 全局更改动画class示例。

* `index5.html`: 局部更改动画class的两种方式示例。

* `index6.html`: 局部更改缓存模板的两种方式示例。

* `index7.html`: `M.history`禁用掉pushstate示例。

* `index8.html`: 嵌套路由视图示例。

* `requirejs/`: 使用 [require.js](http://requirejs.org/) 示例

### 后端渲染

只需要在响应时加入对应的页面结构即可：

```html
<div class="page-view">后端渲染内容</div>
```

这是因为默认第一次初始化时，会查找页面上带有`viewClass`的元素，如果找到了，且`innerHTML`不为空，那么就不会再去调用`getTemplate`来得到模板内容了。

### 代码风格

没有用空格，而是用的`tab`。

### 协议

[MIT](https://github.com/dolymood/mobile-router.js/blob/master/LICENSE)

[npm-image]: https://img.shields.io/npm/v/mobile-router.js.svg?style=flat
[npm-url]: https://npmjs.org/package/mobile-router.js
[downloads-image]: https://img.shields.io/npm/dm/mobile-router.js.svg?style=flat
[downloads-url]: https://npmjs.org/package/mobile-router.js
