# Day02

### 学习目标

- request
- response
- 异常
- 会话
  - cookie
  - session
- 模板
- 模型

### 学习课程

#### 1.request

```
request是一个内置对象
内置对象：不需要创建就可以直接使用的对象
属性
	method  **
		请求方法
	base_url
		去掉get参数的url
	host_url
		只有主机和端口号的url
	url
		完整的请求地址
	remote_addr **
		 请求的客户端地址
		 
     一般情况下  get请求方式都是在浏览器的地址栏上显示
     eg：http：//www.baiduc.com/s?name=zs&age=18
     获取get请求方式的参数的方式：
	request.args.get('name')
	args  **
		1. args
         - get请求参数的包装，args是一个ImmutableMultiDict对象，类字典结构对象
         - 数据存储也是key-value
         - 外层是大列表，列表中的元素是元组，元组中左边是key，右边是value
     一般情况下 post请求方式都是通过表单形式使用的
     eg：
     	<form action='xxx' method='post'>
     		<input type='text' name='name'>
     获取post请求方式的参数的方式
     request.form.get('name')
	form   **
		2. form
         - 存储结构个args一致
         - 默认是接收post参数
         - 还可以接收 PUT，PATCH参数
	files   **   form标签中有一个参数 enctype=maltipart/form-data  请求方式必须是post  使用files接收
		文件上传
	headers	
		请求头
	path
		路由中的路径
	cookies
		请求中的cookie
	session	
		与request类似  也是一个内置对象  可以直接打印 print（session）
```

#### 2 Response

```
创建方式
	返回字符串
		如果只有字符串，就是返回内容，数据
		还有第二个返回，放的是状态码
		@blue.route('/response/')
        def get_response():
               return '德玛西亚',404
	render_template
		渲染模板
		将模板变成字符串
		@blue.route('/rendertemplate/')
        def render_temp():
              resp = render_template('Response.html')
              print(resp)
              print(type(resp))
              return resp,500
	make_response
		Response对象
		返回内容
		状态码
		@blue.route('/makeresponse/')
		def  make_resp():
              resp = make_response('<h2>xxxxxxxx</h2>',502)
              print(resp)
              print(type(resp))
              return resp
	redirect
		重定向
		@blue.route('/redirect/')
        def  make_redir():
             return redirect('/makeresponse/')
		反向解析 url_for
		@blue.route('/redirect/')
		def  make_redir():
       		return redirect(url_for('first.make_resp'))
	response()
```

##### 异常

```
abort
	直接抛出 显示错误状态码  终止程序运行
	abort(404)
	eg:
		@blue.route('/makeabort/')
         def make_abort():
              abort(404)
              return '天还行'
```

```
捕获
	@blue.errorhandler()
		- 异常捕获
		- 可以根据状态或 Exception进行捕获
		- 函数中要包含一个参数，参数用来接收异常信息
	eg:
	@blue.errorhandler(502)
    def handler502(exception):
        return '不能让你看到状态码'
```

#### 3.会话技术

```
1.请求过程Request开始，到Response结束
2.连接都是短连接
3.延长交互的生命周期
4.将关键数据记录下来
5.Cookie是保存在浏览器端/客户端的状态管理技术
6.Session是服务器端的状态管理技术
```

##### cookie

```
Cookie
	1.客户端会话技术
	2.所有数据存储在客户端
	3.以key-value进行数据存储层
	4.服务器不做任何存储
	5.特性
		支持过期时间
			max_age
			expries
		根据域名进行cookie存储
		不能跨网站（域名）
		不能跨浏览器
		自动携带本网站的所有cookie
	6.cookie是服务器操作客户端的数据
	7.通过Response进行操作
```

```
cookie登陆使用
	设置cookie    response.set_cookie('username',username)
		response
	获取cookie    username = request.cookies.get('username','游客')
		request
	删除cookie     response.delete_cookie('username')
		response
```

##### session

```
Session
	1.服务端会话技术
	2.所有数据存储在服务器中
	3.默认存在服务器的内存中
		- django默认做了数据持久化（存在了数据库中）
	4.存储结构也是key-value形势，键值对
	【注】单纯的使用session是会报错的，需要使用在__init__方法中配置app.config['SECRET_KEY']=‘110’
```

```
session登陆使用
	设置    session['username'] = username
	获取    session.get('username')
	删除
		   resp.delete_cookie('session')
		   session.pop('username')
```

##### session持久化问题

```
Session
	- django中对session做了持久化，存储在数据库中
    - 可以修改到redis中
		flask中没有对默认session进行任何处理
    - flask-session 可以实现session的数据持久化
    - 各种位置，更推荐使用redis
        - 缓存在磁盘上的时候，管理磁盘文件使用lru, 最近最少使用原则
	服务端会话技术
	Flask中没有对默认Session进行处理，默认存在内存中
	Session需要持久化  Redis中
	实现方案
		插件 flask-session
		pip install flask-session
		在国内源安装
			pip install flask-sessin -i https://pipy.douban.com/simple
		初始化Session对象 
			配置init中app.config['SESSION_TYPE'] = 'redis'
				持久化的位置
			初始化
				创建session的对象有2中方式 分别是以下两种
				1 Session(app=app)
				2 se = Session()   se.init_app(app = app)
			安装redis
				pip install redis
			需要配置SECRET_KEY='110'
			其他配置--视情况而定
				app.config['SESSION_KEY_PREFIX']='flask'
		查看redis内容
			redis-cli
			keys *
			get key
	session生存时间31天	
		ttl session
			flask的session的生存时间是31天，django的session生存时间是15天
```

#### 4.Template

```
简介：
	MVC中的View，MTV中的Template
	主要用来做数据展示的
	模板处理过程分为2个阶段
            1 加载
            2 渲染
     jinja2模板引擎
            1.本质上是html
            2.支持特定的模板语法
            3.flask作者开发的   一个现代化设计和友好的python模板语言  模仿的django的模板引擎
            4.优点
                    速度快，被广泛使用
                    HTML设计和后端Python分离
                    减少Python复杂度
                    非常灵活，快速和安全
                    提供了控制，继承等高级功能

```

```
模板
	1.静态html   前后端分离
	2.模板语言动态生成的html
		{{ var }}  变量的接收
			从views传递过来的数据
			前面定义出来的数据
		案例
			www.jq22.com/cdn/
		{% tag %}
			结构标签
				block
					首次出现挖坑操作
					第二次出现填坑操作
					第N次出现，填坑操作，会覆盖前面填的坑
					不想被覆盖，需要添加 {{ super() }}
				extends
					继承
				include
					包含，将一个指定的模板包含进来
			宏定义
				macro
					可以在模板中定义，调用函数
					无参
						{% macro say()%}
    							你饿了吗？？？
						{% endmacro %}
					有参
						{% macro createUser(name,age)%}
   								 欢迎{{ name }} 心理没点数吗 你都{{ age }}大了
						{% endmacro %}
						
					函数还是用来生成html的	
					外文件中的宏定义调用需要导入也可以include
					{% macro getUser(name)%}
    						欢迎光临红浪漫{{ name }},拖鞋手牌拿好,楼上2楼左转,男宾一位
					{% endmacro %}
						{% from ‘html文件’ import yyy %}
					{{ getUser('action') }}
			循环控制
				for
					for .. in 
					loop  循环信息
						索引  index
						第一个   first
						最后一个last
				if
					if 
					else
					elif
			过滤器 
				{{ var|xxx|yyy|zzz }}
				没有数量限制
				lower
				upper
				title
				trim
				reverse
				striptags
					渲染之前将值中的标签去掉
				safe
					标签生效
				eg:
				{% for c in config %}
                       <li>{{ loop.index0 }}:{{ loop.index}}:{{ c|lower|reverse }}</li>
    			{% endfor %}
```

#### 5.models

```
1.数据交互的封装
2.Flask默认并没有提供任何数据库操作的API
	Flask中可以自己的选择数据，用原生语句实现功能
		原生SQL缺点:
              代码利用率低，条件复杂代码语句越过长，有很多相似语句
              一些SQL是在业务逻辑中拼出来的，修改需要了解业务逻辑
              直接写SQL容易忽视SQL问题
	也可以选择ORM
		SQLAlchemy
		MongoEngine
		将对象的操作转换为原生SQL
		优点
			 易用性，可以有效减少重复SQL
              性能损耗少
              设计灵活，可以轻松实现复杂查询
              移植性好
3.Flask中并没有提供默认ORM
	ORM 对象关系映射
	通过操作对象，实现对数据的操作
```

```
flask-sqlalchemy
     使用步骤：1.pip install flask-sqlalchemy
              2.创建SQLALCHEMY对象
                                   ①：db=SQLAlchemy(app=app)
                                   ②：db=SQLAlchemy()  上面这句话一般会放到models中  因为需要db来调                                       用属性 db.init_app(app=app)
              3.config中配置  SQLALCHEMY_DATABASE_URI
                dialect+driver://username:password@host:port/database
                数据库 + 驱动 :// 用户:密码@ 主机:端口/数据库
                mysql是需要全部配置
                sqlite
                    轻量级数据库，配置简单
                    sqlite:///xxxx（sqlite3.db）
                执行
                    views中db.create_all()
                        有坑
                            primary-key
                                添加主键
                            SQLALCHEMY_TRAKE_MODIFICATIONS
                                app.config[‘SQLALCHEMY_TRAKE_MODIFICATIONS’]=False
                  
```

```
使用：
	定义模型
			继承Sqlalchemy对象中的model
	定义字段
              主键
                  一定要添加
              所需要字段
              语法
                  db.Column( db.类型（）,约束 )
     创建
	db.create_all()
	删除
	db.drop_all()
	修改表名
	__tablename__ = "Worker"
    数据操作
        创建对象
        添加
            db.session.add(对象)
            db.session.commit()
        查询
            模型.query.all()
```

