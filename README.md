# Single-page-application

## 前言
在单页面应用程序中，前后端采用了完全分离的方法，因此在前端实现路由的切换非常的重要。同时前端实现路由可以减少请求数，缓解后端的压力。在单页面中的路由主要有两种实现方法，一种是通过h5的history api来实现，还有一种是hash来实现。
## history
history的方法主要是应用了几个H5的histroy API：
#### API

* window.pushState(stateData, title, url)

在history中创建一个新的访问记录，不能跨域，且不造成页面刷新，stateData为当前页面的一些信息在触发popstate事件的时候可以调用；title为网页的标题；url是浏览器中显示的网址。

* window.replaceState(stateData, title, url)

修改当前的访问记录，不能跨域，且不造成页面刷新

* windows事件popstate

当回到页面前一页或者后一页的时候触发

#### API示意图

![image](https://images2015.cnblogs.com/blog/1094893/201706/1094893-20170621193746366-2004051541.png)
这张图表示了这些api的具体操作。

    pushState主要就是向当前页面的后面添加一个新的页面，新的页面地址是第三个参数。
    replaceState是把当前页面的地址替换成他的第三个参数，history并不会记录之前的地址
    这两个方法都不能进行页面的刷新，所以需要在调用方法的时候执行一下相关路由变化的操作，从而达到页面不切换的效果更改页面的单页面效果。

#### 举个栗子
    因此只要在页面跳转的时候改成pushState然后执行当前页面的加载函数，就可以实现不刷新的页面跳转了如：
    原来地址是：https://www.baidu.com
    然后点击了按钮进行页面跳转就执行函数：
    history.pushState(data, title, 'https://www.baidu.com/test')
    然后浏览器上面的地址已经变成了https://www.baidu.com/test然后同时还有返回的按钮
    然后调用一下test页面的渲染方法

#### 存在问题
这样子会存在一个问题，就是当页面直接打开https://www.baidu.com/test服务器会找不到这个页面而报404的错
#### 解决问题
对此错误我想到了两种解决方法

* 其一是你在服务器端设置一下当访问https://www.baidu.com/以下的所有地址全部转发到https://www.baidu.com/这个地址上来。然后前端通过对url的解析来确定用户想访问的页面，然后渲染这个页面
* 其二是你跳转页面用不同参数的形式来跳转页面，如
    
    原来地址是：https://www.baidu.com

    然后点击跳转的时候执行
    
    history.pushState(data, title, '?page=test')
    
    然后浏览器地址栏的地址是https://www.baidu.com?page=test
    
    然后当你访问https://www.baidu.com?page=test页面的时候会打开https://www.baidu.com然后你可以根据page参数的值来判断用户想访问的页面，进行不同的渲染
    
#### 使用了解决方法二写的一个history单页面的小栗子

##### html代码

```
<div id="msg"></div>
<br><br>
<span class="msgBtn" data-route='A'>toFirst</span>
<span class="msgBtn" data-route='B'>toSecond</span>
<span class="msgBtn" data-route='C'>toThird</span>
```
样式很简单就是一个显示信息的msg然后三个按钮对应不同的页面
![image](https://s15.postimg.cc/hooa99nsb/image.png)
##### js代码

```
	//路由注册
	const message = {
		undefined: 'massage',
		A: 'hahahh',
		B: 'hehehehe',
		C: 'hohohoho'
	};

	//页面分发加载
	function showMsg(el) {
		const _loc = location.href;
		let url_msg_id = GetRequest(_loc);
		el.html(message[url_msg_id.msg]);
	}

	//监听前进后退按钮
	window.addEventListener('popstate', function (e) {
		showMsg($('#msg'));
	});

	//点击切换路由
	$('.msgBtn').click(function (e) {
		console.log(e.currentTarget );
		history.pushState(null, null, '?msg=' + e.target.dataset.route);
		showMsg($('#msg'));
	});

	//处理url
	function GetRequest() {
		let url = location.search; //获取url中"?"符后的字串
		let theRequest = {};
		if (url.indexOf("?") !== -1) {
			let str = url.substr(1);
			let strs = str.split("&");
			for (let i = 0; i < strs.length; i++) {
				theRequest[strs[i].split("=")[0]] = unescape(strs[i].split("=")[1]);
			}
		}
		return theRequest;
	}

	//页面加载渲染
	showMsg($('#msg'));
```
* 在注册路由里面的这个对象是每个路由名对应不同的信息，然后如果是页面的话可以每个路由名对应不同页面的渲染方法，其中undefined是没有参数的时候的页面。
* showMsg方法在页面加载和路由切换的时候执行，作用是根据当前的地址来渲染不同的页面，这里只是用了$().html()来更改页面，如果是不同的页面的话可以来执行不同的渲染方法
* 监听popstate的作用是点击浏览器的前进后退的时候执行一下渲染方法
* 点击事件是点击不同的按钮来更改地址然后触发渲染方法
* GetRequest方法是一个提取url中的参数的方法

##### 实现过程
当你点击toFirst会把url的msg参数变成toFirst对应的data-route，然后触发showMsg方法，获取到当前的url的参数在message中找到对应要渲染的字符，渲染在msg上。
效果如下
![image](https://s15.postimg.cc/uhce9fcbf/image.png)

然后点击toSecond

![image](https://s15.postimg.cc/tf79nnkaj/image.png)

点击浏览器的返回按钮，通过监听来重新调用showMsg方法，所以会回到这个页面

![image](https://s15.postimg.cc/uhce9fcbf/image.png)

如果你直接打开http://localhost:63342/history/index.html?msg=C
也会在index.html里面进行showMsg处理来打开对应的页面

![image](https://s15.postimg.cc/8v2fpg76z/image.png)

#### 完整代码

```
<html>
<head>
    <title></title>
    <style type="text/css">
        div {
            margin: 10px;
        }
        .msgBtn {
            margin: 10px;
            padding: 10px;
            border: 1px solid black;
        }
    </style>
</head>
<body>
    <div id="msg"></div>
    <br><br>
    <span class="msgBtn" data-route='A'>toFirst</span>
    <span class="msgBtn" data-route='B'>toSecond</span>
    <span class="msgBtn" data-route='C'>toThird</span>
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
    <script type="text/javascript">
        //路由注册
        const message = {
            undefined: 'massage',
            A: 'hahahh',
            B: 'hehehehe',
            C: 'hohohoho'
        };
        //页面分发加载
        function showMsg(el) {
            const _loc = location.href;
            let url_msg_id = GetRequest(_loc);
            el.html(message[url_msg_id.msg]);
        }
        //监听前进后退按钮
        window.addEventListener('popstate', function (e) {
            showMsg($('#msg'));
        });
        //点击切换路由
        $('.msgBtn').click(function (e) {
            console.log(e.currentTarget );
            history.pushState(null, null, '?msg=' + e.target.dataset.route);
            showMsg($('#msg'));
        });
        //处理url
        function GetRequest() {
            let url = location.search; //获取url中"?"符后的字串
            let theRequest = {};
            if (url.indexOf("?") !== -1) {
                let str = url.substr(1);
                let strs = str.split("&");
                for (let i = 0; i < strs.length; i++) {
                    theRequest[strs[i].split("=")[0]] = unescape(strs[i].split("=")[1]);
                }
            }
            return theRequest;
        }
        //页面加载渲染
        showMsg($('#msg'));
    </script>
</body>
```

## hash
实现思路和之前的大概相同就是变化后面的哈希值，然后通过hashchange监听到哈希值的变化
demo地址 https://github.com/WindStormrage/Single-page-application/blob/master/history/index.html


## 总结
这样子是大概实现出了一个单页面的效果但是，还有几个单页面的要素没有体现出来，比如异步静态文件的加载。
hash的缺点是，只好设置一层路由如果还有子页面需要路由会比较麻烦，然后还有的是就是#这个字符让url显得非常不美观