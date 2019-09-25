## Day05

### 学习目标

- 分页器的相关方法
- flask-bootstrap
- flask-debugtoolbar
- flask-cache
- 勾子
- 四大内置对象
- 扩展-路径问题
- Json数据

### 学习课件

#### 1.分页器方法

```
分页器
	BaseQuery.paginate()
		page
		per_page
		False
	Pagination
		items
		pages         （获取总页数）
		prev_num      （上一页的页码）
		has_prev      （是否有上一页）
		next_num      （下一个页码）
		has_next	  （是否有下一页）
		iter_pages
```

#### 2.flask-bootstrap

```
插件安装
	pip install  flask-bootstrap
ext中初始化
	Bootstrap（app=app）
	
bootstrap案例--bootstrap模板   {% extends ‘bootstrap/base.html’%}
```

#### 3.flask-debugtoolbar

```
辅助调试插件
安装
	pip install flask-debugtoolbar
初始化  ext 
    app.debug = True (最新版本需要添加)
	debugtoolbar = DebugToolBarExtension()
	debugtoolbar.init_app(app=app)
```

#### 4.缓存flask-cache

```
1 缓存目的：
          缓存优化加载，减少数据库的IO操作
2 实现方案
          （1）数据库
          （2）文件
          （3）内存
          （4）内存中的数据库 Redis
3 实现流程
          （1）从路由函数进入程序
          （2）路由函数到视图函数
          （3）视图函数去缓存中查找
          （4）缓存中找到，直接进行数据返回
          （5）如果没找到，去数据库中查找
          （6）查找到之后，添加到缓存中
          （7）返回到页面上
4 使用
          （1）安装 flask-cache
				pip  install flask-cache
		 （2）初始化
                 指定使用的缓存方案
                      cache = Cache(config={'CACHE_TYPE':默认是simple})
                      cache.init_app(app=app)
          （3）使用
              在路由的下面添加@cache.cached(timeout=30)
              要配置config
                  TYPE
                  还可以配置各种缓存的配置信息
                      cache=Cache(config={'CACHE_KEY_PREFIX':'python'})
          （4）用法
                  ①：装饰器
                      @cache.cached(timeout=xxx)
                  ②：原生
                      get
                      set
                      例子：
                      chache.get('ip')
                      cache.set('ip',ip)
```

#### 5.钩子

```
@blue.before_request
eg:
	  @blue.before_request
      def aop():
         print('i am aop simida')

      @blue.route('/add/')
      def add():
          print('i am add')
          return 'add'

      @blue.route('/delete/')
      def delete():
          print('i am delete')
          return 'delete'
```

#### 6.四大内置对象

```
四大内置对象
	(1)request
			 请求的所有信息
	(2)session
			 服务端会话技术的接口
	(3)config
              当前项目的配置信息
              模板中可以直接使用
                  config
              在python代码中
                  current_app.config
                  当前运行的app
                  使用记得是在初始化完成之后
                  用在函数中
              eg:
                  for c in current_app.config:
                      print(c)
	(4)g
              global  全局
              可以帮助开发者实现跨函数传递数据
              eg:
                  @blue.route('/g/')
              def g():
                  g.ip = request.remote_addr
                  return 'ok'

              @blue.route('/testg1/')
              def testg1():
                  print(g.ip)
                  return 'ok1'
```

#### 7.路径

```
	BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
	init中app = Flask（__name__,static_folder=static_folder）
		 static_folder=os.path.join(settings.BASE_DIR,'static')
```

#### 8.前后端分离

```
整合网络和软件的一种架构模式
理解
	Representtational
		表现层
	State Transfer
		状态转换
	表现层状态转换
		资源（Resource）
	每一个URI代表一类资源
		对整个数据的操作
		增删改查
RESTful中更推荐使用HTTP的请求谓词（动词）来作为动作标识
	GET
	POST
	PUT
	DELETE
	PATCH
推荐使用json数据传输
```

#### 9.前后端分离-原生实现

概念：

```
概念：就是判断不同的请求方式，实现请求方法
高内聚，低耦合
	高内聚
	相同的数据操作封装在一起
	低耦合
MVC 没有模板--前后端分离
```

get：

```
get
		坑---类型
		eg：
		@blue.route("/users/<int:id>/", methods=["GET", "POST", "PUT", "DELETE", "PATCH"])
        def users(id):
            if request.method == "GET":
                page = int(request.args.get("page", default=1))
                per_page = int(request.args.get("per_page", default=3))

                users = User.query.paginate(page=page, per_page=per_page, error_out=False).items
                users_dict = []
                for user in users:
                    users_dict.append(user.to_dict())

                data = {
                    "message": "ok",
                    "status": "200",
                    "data": users_dict
                }

                return jsonify(data)
```

post：

```
post
		eg:
            elif request.method == "POST":
                  # 更新或创建
                  username = request.form.get("username")
                  password = request.form.get("password")

                  data = {
                      "message": "ok",
                      "status": "422"
                  }

                  if not username or not password:
                      data["message"] = "参数不正确"
                      return jsonify(data), 422

                  user = User()
                  user.u_name = username
                  user.u_password = generate_password(password=password)

                  try:

                      db.session.add(user)
                      db.session.commit()
                      data["status"] = "201"
                  except Exception as e:
                      data["status"] = "901"
                      data["message"] = str(e)
                      return jsonify(data), 422

                  return jsonify(data), 201
```

put:

```
put
		eg:
			elif request.method == "PUT":
                      username = request.form.get("username")
                      password = request.form.get("password")
                      user = User.query.get(id)

                      user.u_name = username
                      user.u_password = generate_password(password)

                      db.session.add(user)
                      db.session.commit()

                      data = {
                          "message": "update success",
                      }

                      return jsonify(data), 201
```

delete:

```
delete
		eg:
		    elif request.method == "DELETE":
                    user = User.query.get(id)

                    data = {
                        "message": "delete success"
                    }

                    if user:
                        db.session.delete(user)
                        db.session.commit()
                        return jsonify(data), 204
                    else:
                        data["message"] = "指定数据不存在"
                        return jsonify(data)
```

patch:

```

	patch
			eg:
			   elif request.method == "PATCH":
                        password = request.form.get("password")
                        user = User.query.get(id)
                        user.u_password = generate_password(password)

                        data = {
                            "messgage": "update success"
                        }

                        db.session.add(user)
                        db.session.commit()

                        return jsonify(data), 201
```

md5

```
md5
	    eg：
	    def generate_password(password):
              hash = hashlib.md5()
              hash.update(password.encode("utf-8"))
              return hash.hexdigest()
```

#### 10.flask-restful

```
框架简化开发
```

##### 使用

```
使用
	安装 pip install flask-restful
	初始化 
		urls---在init中调用init_urls
			api = Api()
			api.add_resource(Hello, "/hello/")
				Hello是一个类的名字  hello是路由
			def init_urls(app):
    		api.init_app(app=app)
    
	apis--基本用法
		继承自Resource
			class Hello(Resource):
		实现请求方法对应函数
			def get(self):
        		return {"msg": "ok"}

            def post(self):
                return {"msg": "create success"}
```

##### 定制输入输出

```
定制输入输出
	输出
		fields中的类型约束
			String
			Integer
			Nested
			List
				Nested
		@marshal_with的基本使用
			类型括号中还允许添加约束
				attribute
					指定连接对应名字
						attribute=名字
				default
					设置默认值
						default=404
		marshal_with特性
			        - 默认返回的数据如果在预定义结构中不存在，数据会被自动过滤
        	         - 如果返回的数据在预定义的结构中存在，数据会正常返回
                      - 如果返回的数据比预定义结构中的字段少，预定义的字段会呈现一个默认值
			        如果类型是Integer  那么默认值是  0
                     如果类型是String  那么默认值是null
                @marshal_with返回一个类对象
                @marshal_with返回一个列表
	输入
		使用了parser=reqparse.RequestParser()
		parser.add_argument("c_name", type=str)
        parser.add_argument("id", type=int, required=True, help="id 是必须的")
		parse = parser.parse_args()
		cat_name = parse.get("c_name")
        id = parse.get("id")
         print(id)
		在对象中添加字段
			对字段添加约束
			default
			required
				必须的参数
			help
				id是必须的
			action
				action=append
					c_name=tom&c_name=zs
		继承
			copy
			可以对已有字段进行删除和更新
			继承解析
    - 在整个项目中，通用字段可以创建一个基parser
    - 复用已有的部分参数转换数据结构
			parse_c parser.copy()   parse_c.remove_argument('')
```

