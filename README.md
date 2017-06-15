# vuejs

手动配置自己:
	webpack+vue-loader

	webpack加载模块
-------------------------------------
如何运行此项目？
	1. npm install	或者    cnpm install
	2. npm run dev
		->  package.json
			"scripts":{
				"dev":"webpack-dev-server --inline --hot --port 8082"
			}

以后下载模块：
	npm install <package-name>  --save-dev

EADDRINUSE	端口被占用

少了:
	webpack-dev-server
	webpack
----------------------------------------
.vue 结尾，之后称呼组件
----------------------------------------
路由:
	vue-router

		->  如何查看版本:
			bower info vue-router

	路由使用版本: 0.7.13
配合vue-loader使用:
1. 下载vue-router模块
	cnpm install vue-router@0.7.13
2. import VueRouter from 'vue-router'

3. Vue.use(VueRouter);

4. 配置路由
	var router=new VueRouter();
	router.map({
		路由规则
	})
5. 开启
	router.start(App,'#app');

注意:
	之前: index.html	->   <app></app>
	现在: index.html	->   <div id="app"></div>

	App.vue	->   需要一个 <div id="app"></div>  根元素

home	news
---------------------------------------------
路由嵌套:
	和之前一模一样
--------------------------------------------
上线:
	npm run build
		->	webpack -p
--------------------------------------------
脚手架:
	vue-cli——vue脚手架
		帮你提供好基本项目结构

本身集成很多项目模板:
	simple		个人觉得一点用都没有
	webpack	可以使用(大型项目)
			Eslint 检查代码规范，
			单元测试
	webpack-simple	个人推荐使用, 没有代码检查	√

	browserify	->  自己看
	browserify-simple
	
--------------------------------------------
基本使用流程:
1. npm install vue-cli -g	安装 vue命令环境
	验证安装ok?
		vue --version
2. 生成项目模板
	vue init <模板名> 本地文件夹名称
3. 进入到生成目录里面
	cd xxx
	npm install
4. npm run dev
--------------------------------------------	





vuex2中使用mapGetters/mapActions报错解决方法
时间 2016-09-22 11:04:24  DaraW
原文  http://blog.daraw.cn/2016/09/22/vuex2-problems-with-mapgetter/
主题 Vuex
在尝鲜vuex2时，发现vuex2增加了 mapGetters 和 mapActions 的方法，借助stage2的 Object Rest Operator 特性,可以写出下面代码：

methods: {
  marked,
  ...mapActions([
    'getArticles'
  ])
}
但是在借助babel编译转换时发生了报错： BabelLoaderError: SyntaxError: Unexpected token 。

解决方案

在vuex的repo issues中有人提过这样的问题，后来是修改了eslint配置中对 Object Rest Operator 的支持解决了问题，然而我根本没有使用eslint。

接着在babel的issues中搜索这样的报错，原来是我babel预置的转换器是 babel-preset-es2015 ，并不能转换 Object Rest Operator 特性。

解决方法很简单了，可以安装整个stage2的预置器或者安装 Object Rest Operator 的babel插件 babel-plugin-transform-object-rest-spread 。

我选择了安装插件，接着在babel的配置文件 .babelrc 中应用插件：

{
  "presets": [
    ["es2015", { "modules": false }]
  ],
  "plugins": ["transform-object-rest-spread"]
}
重启webpack，就不会再有报错了。


vue配置mock模拟接口数据
https://github.com/carrotz/vue-cli-mock







	
