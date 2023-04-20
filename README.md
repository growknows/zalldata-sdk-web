# 1.开启全埋点

## 1.1. 代码示例

```js
<script charset='UTF-8' src="zallldata.min.js"></script>
<script>
      var zall = window['sensorsDataAnalytic201505'];
      zall.init({
	  server_url: 'http://127.0.0.1:8080/a?service=zall&project=test&gzip=0',
	  is_track_single_page:true, // 单页面配置，默认开启，若页面中有锚点设计，需要将该配置删除，否则触发锚点会多触发 $pageview 事件
	  heatmap: {
	      //是否开启点击图，default 表示开启，自动采集 $WebClick 事件，可以设置 'not_collect' 表示关闭。
	      clickmap:'default',
	     //是否开启触达注意力图，not_collect 表示关闭，不会自动采集 $WebStay 事件，可以设置 'default' 表示开启。
	     scroll_notice_map:'default'
	    } 
	});
      zall.quick('autoTrack');
</script>
```

Web JS SDK 全埋点包括三种事件：Web 页面浏览、Web 元素点击、Web 视区停留，单独对应的配置如下。

## 1.2. Web 页面浏览($pageview)

```js
  // 设置之后，SDK 就会自动收集页面浏览事件，以及设置初始来源。
  zall.quick('autoTrack')

  // 另外，如果想加额外的属性，可以如下方式（添加 platform 属性为 h5）
  zall.quick('autoTrack', {
	platform:'h5'
  })
```

## 1.3. Web 元素点击($WebClick)

```js
// SDK 初始化参数配置
  heatmap: {
      //是否开启点击图，default 表示开启，自动采集 $WebClick 事件，可以设置 'not_collect' 表示关闭。
      clickmap:'default'
   }
```



## 1.4. 视区停留事件($WebStay)

```js
// SDK 初始化参数配置
   heatmap: {
      //是否开启触达注意力图，default 表示开启，自动采集 $WebStay 事件，可以设置 'not_collect' 表示关闭。
      //需要 Web JS SDK 版本号大于 1.9.1
      scroll_notice_map:'default'
  }
```

- 
  如果发生页面滚动时候，之前的页面停留是有效停留，也就是超过默认的 4 秒或者自定义的时间，javascript SDK 就会发送一次页面停留事件。
- 关闭页面时，发送一次页面停留事件。 $WebStay 事件的采集是基于在 window 上绑定的 scroll 事件，如果页面滚动是绑定在 div 上，则无法采集。



# 2. 其他元素类型的元素点

## 2.1. 支持 div 类型元素的自动采集

在原来的全埋点（采集 a、button、input 、textarea 标签）基础上新增对 div 标签的采集，采集规则为：

1. div 为叶子结点（无子元素）时采集 div 的点击
2. div 中有且只有样式标签（['mark','strong','b','em','i','u','abbr','ins','del','s','sup']）时，点击 div 或者样式标签都采集 div 的点击

通过 collect_tags 配置是否开启 div 的全埋点采集，默认不采集。如需开启 ，配置 collect_tags 参数如下（注意：只支持配置 div）：

```js
   heatmap:{  
      collect_tags:{
           div : true
       }
    }
```

## 2.2. 支持其他类型元素的自动采集

### 2.2.1. 配置特殊属性：*data-sensors-click*

开启全埋点时，给需要自动采集点击事件的元素增加属性：***data-sensors-click***（备注：增加该属性无法让元素可以可视化全埋点）***。***

代码示例如下：

```xml
<div name='test' data-sensors-click>我是测试元素</div>
<li data-sensors-click class='test-li'>我是测试元素</li>
```

### 2.2.2. 配置自定义属性

开启全埋点时，支持配置带有指定属性的页面元素点击，自动采集点击事件（备注：增加该属性无法让元素可以可视化全埋点）。

代码示例如下：

```js
<script src="xxxx/xxx/zalldata.js"></script>
<script>
var zall = sensorsDataAnalytic201505;
zall.init({
  server_url: 'SERVER_URL', 
  heatmap: {
    track_attr: ['hotrep', 'anotherprop', "data-prop2"],
    clickmap:'default',
    scroll_notice_map: 'not_collect'
  }
});
</script>
...
<p hotrep id="test1">hotrep p tag</p>
<p><span anotherprop id="test2">another repo a.b.c</span></p>
<p><strong data-prop2 id="test3">strong with prop2 attribute</strong></p>
```

## 2.3. 代码埋点触发元素点击事件

如果你想要对非 a input button textarea 的其他元素采集点击事件的话，请在元素发生点击的时候，调用我们的这个方法。

```xml
// 下面演示一个 div 标签被点击时触发预置的元素点击事件
<div id="submit_order">提交订单</div>

<script type="text/javascript">
$('#submit_order').on('click', function() {
      zall.quick('trackHeatMap', this, {
	   customProp1: 'test1', //如果需要添加自定义属性需要将 SDK 升级到 1.13.7 及以上版本。
	   customProp2: 'test2'
      }, function() {
	  console.log('callback');
      });
});
</script>
```

// 注意: 在原生js中调用我们绑定的click事件时this指向的是元素节点，在其他框架采集点击事件时this指针也应该指向相应的元素节点，例如在vue中可以参考下面示例。

```xml
<div v-on:click="track">点击</div>

<script>
export default {
   methods: {
	track: function(event) {
	 zall.quick('trackHeatMap', event.target, {
		customProp1: 'test1', //如果需要添加自定义属性需要将 SDK 升级到 1.13.7 及以上版本。
		customProp2: 'test2'
	   }, function() {
	      console.log('callback');
		});
	   }
	}
}
</script>
```

SDK 增加新方法 trackAllHeatMap ，当调用这个方法时，此方法的第二个参数可以传入包括 a, input, button, textarea 标签和 div，img 等的所有标签。而 trackHeatMap 只采集除 a, input, button,textarea 之外的标签。同时，这两个方法均可以设置自定义属性，且支持回调函数（这两个方法的第四个参数都可以传入一个方法）。

```xml
// 下面演示一个 button 按钮被点击时触发预置的元素点击事件
<button id="testp">测试按钮</button>

<script type="text/javascript">
$('#testp').on('click', function() {
	zall.quick('trackAllHeatMap', this, {
		customProp1: 'test1', //如果需要添加自定义属性需要将 SDK 升级到 1.13.7 及以上版本。
		customProp2: 'test2'
	}, function() {
		console.log('callback');
	});
});
</script>
```

# 3. 全埋点相关参数配置

## 3.1. Web 元素点击

### 3.1.1. heatmap 相关参数，提供了对于 $WebClick 事件的自定义设置和处理。

heatmap 相关参数说明：

| 参数名            | 参数值说明                                                   | 备注                                                         |
| :---------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| clickmap          | 是否开启点击图，默认 default 表示开启，可以设置 'not_collect' 表示关闭。 |                                                              |
| scroll_notice_map | 是否开启触达注意力图，设置 default 表示开启，设置 'not_collect' 表示关闭。 |                                                              |
| loadTimeout       | 毫秒                                                         | 设置多少毫秒后开始渲染点击图,因为刚打开页面时候页面有些元素还没加载。 |
| collect_url       | 返回真会采集当前页面的元素点击事件，返回假表示不采集当前页面,设置这个函数后，内容为空的话，是返回假的。不设置函数默认是采集所有页面。 |                                                              |
| collect_element   | 用户点击页面元素时会触发这个函数，让你来判断是否要采集当前这个元素，返回真表示采集，返回假表示不采集。 |                                                              |
| custom_property   | 假如要在 $WebClick 事件增加自定义属性，可以通过标签的特征来判断是否要增加。 | 注意：如果同时使用了 trackAllHeatMap 或者 trackHeatMap 方法设置了自定义属性，那么这 两个方法中的自定义属性对象会覆盖 custom_property 返回值中的同名属性，它的优先级更高。 |
| collect_input     | 考虑到用户隐私，这里可以设置 input 里的内容是否采集。        | 如果返回真，表示采集 input 内容，返回假表示不采集 input 内容,默认不采集。 |
| element_selector  | SDK 默认优先以元素 ID 作为选择器采集点击事件，若不想以 ID 作为选择器，可以设置该参数为 'not_use_id'。 |                                                              |
| renderRefreshTime | 毫秒                                                         | 第二版点击图滚动滚动条，改变页面尺寸后延时多少毫秒重新渲染页面。 |

heatmap 相关参数代码示例：

```js
heatmap: {
	clickmap:'default',
	scroll_notice_map:'default',
	loadTimeout: 3000,
	collect_url: function(){
		//如果只采集首页
		if(location.href === 'xxx.com/index.html' || location.href === 'xxx.com/'){
			return true;
		}
	},
	//此参数针对预置 $WebClick 事件（包括 quick('trackHeatMap') quick('trackAllHeatMap') 触发的）生效。
	collect_element: function(element_target){
		// 如果这个元素有属性sensors-disable=true时候，不采集。
		if(element_target.getAttribute('sensors-disable') === 'true'){
			return false;
		}else{
			return true;
		}
	},
	//此参数针对预置 $WebClick 事件（包括 quick('trackHeatMap') quick('trackAllHeatMap') 触发的）生效。
	custom_property:function( element_target ){
		//比如您需要给有 data=test 属性的标签的点击事件增加自定义属性 name:'aa' ，则代码如下：
		if(element_target.getAttribute('data') === 'test'){
			return {
				name:'aa'
			}
		}
	},
	collect_input:function(element_target){
		//例如如果元素的 id 是a，就采集这个元素里的内容。
		if(element_target.id === 'a'){
			return true;
		}
	},
	element_selector:'not_use_id',
	renderRefreshTime:1000
}
```

## 3.2. Web 视区停留

### 3.2.1. 相关参数说明：

scrollmap 相关参数说明：

| 参数名      | 参数值说明                                                   | 备注 |
| :---------- | :----------------------------------------------------------- | :--- |
| collect_url | 返回真会采集当前页面的数据，返回假表示不采集当前页面,设置这个函数后，内容为空的话，是返回假的。不设置函数默认是采集所有页面。 |      |

heatmap 相关参数说明：

| 参数名                | 参数值说明                                                   | 备注                                                         |
| :-------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| scroll_notice_map     | 是否开启触达图，设置 default 表示开启，设置 'not_collect' 表示关闭。 |                                                              |
| scroll_delay_time     | 毫秒                                                         | 设置触达图的有效停留时间，默认超过 4 秒以上算有效停留。      |
| scroll_event_duration | 秒                                                           | 预置属性停留时长 event_duration 的最大值，默认 18000 秒，5 小时。 |

scrollmap 和 heatmap 相关代码示例：

```js
scrollmap: {
	collect_url: function(){
		//如果只采集首页
		if(location.href === 'xxx.com/index.html' || location.href === 'xxx.com/'){
			return true;
		}
	},
},
heatmap:{
	//是否开启触达注意力图，default 表示开启，自动采集 $WebStay 事件，可以设置 'not_collect' 表示关闭。
	//需要 Web JS SDK 版本号大于 1.9.1
	scroll_notice_map:'not_collect',
	scroll_delay_time: 4000, 
	scroll_event_duration: 18000 //单位秒，预置属性停留时长 event_duration 的最大值。默认5个小时，也就是300分钟，18000秒。
}
```

# 4. 单页面中事件的自动采集

## 4.1. Web 页面浏览事件采集

### 4.1.1. 自动模式

配置参数 is_track_single_page (推荐使用这种模式)，默认值为 false ，表示是否开启自动采集 web 浏览事件 $pageview 的功能 。
其原理是修改 window 对象的 pushState 和 replaceState 原生方法，在页面的 url 改变后自动采集 $pageview 事件，若用户浏览器不支持这两种方法或者是使用 hash 的路由模式，我们也会监听 popstate 和 hashchange 事件来自动触发 $pageview 事件。

使用方法示例：

```js
//默认值为false。
is_track_single_page:true
//注意：如果进首页不会自动 redirect 时，sa.quick('autoTrack') 是需要的，否则不需要。
```

SDK 的 is_track_single_page 参数增加 function(){} 的配置，必须 return 一个值。
配置一：return true 表示当前页面会发送数据，
配置二：return false 表示不会发数据，
配置三：return {} 对象，表示会把对象的属性新增到当前 $pageview 里。

使用方法示例：

```js
is_track_single_page : function (){
	return true 时候，使用默认发送的 $pageview
	return false 时候，不执行默认的 $pageview
	return {} 时候，把对象中的属性，覆盖默认 $pageview 里的属性。
}
//注意：如果进首页不会自动 redirect 时，sa.quick('autoTrack') 是需要的，否则不需要。
```

另外注意：
**必须保证切换页面前 Web JS SDK 的已经执行，否则的话可能第一次切换页面不会触发 $pageview。**

### 4.1.2. 手动模式

在页面切换的时候，手动调用 zall.quick('autoTrackSinglePage') 来采集 web 浏览事件 $pageview ，这个方法在页面 url 切换后调用。

```js
// 比如现在是在 react 中可以在全局的 onUpdate 里来调用。
onUpdate: function(){
	zall.quick('autoTrackSinglePage');
}

//vue 项目在路由切换的时候调用。
router.afterEach((to,from) => {
	Vue.nextTick(() => {
		zall.quick("autoTrackSinglePage");
	});
});
//注意： vue下因为首页打开时候就会默认触发页面更新，所以需要去掉默认加的 sa.quick('autoTrack')。
```

此方法也可添加自定义属性，

```js
zall.quick("autoTrackSinglePage",{platForm:"H5"});
```

## 4.2. 单页面的页面标题 $title 问题

对于单页面项目，SDK 全埋点的预置事件采集的页面标题属性，可能存在异常 。
具体问题：
1、title 如果没有更新赋值，获取的 title 一直是主页的 title 不会变化，切换页面发送的数据不会更新 $title 。
2、title 设置的时机晚于数据发送的时机，切换页面发送的 $pageview 事件携带的 $title 值为上一个页面的 title 。
解决方案：
在切换页面前完成 title 值的更新（以常见的 vue 为例）

```js
router.beforeEach((to, from, next) => { 
	document.title = '新页面的 title 值'; 
	next() 
}) 
```
