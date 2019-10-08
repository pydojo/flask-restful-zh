.. _quickstart:

快速开始
==========

.. currentmodule:: flask_restful

是时候写你的第一个 REST API 网络应用了。
本指导文档都是建立在你已经理解并与
 `Flask <http://flask.pocoo.org>`_ 工作一段时间的基础上，
并且你已经安装完 Flask 和 Flask-RESTful。
如果还没有的话，阅读 :ref:`installation` 安装文档内容。

一个迷你 API
-------------

迷你型 Flask-RESTful API 看起来就像下面代码所实现的一样： ::

    from flask import Flask
    from flask_restful import Resource, Api

    app = Flask(__name__)
    api = Api(app)

    class HelloWorld(Resource):
        def get(self):
            return {'hello': 'world'}

    api.add_resource(HelloWorld, '/')

    if __name__ == '__main__':
        app.run(debug=True)

把这些代码保存到 `api.py` 文件里，然后在终端里用 Python 解释器来运行这个文件。 
注意：我们已经打开了调试模式
 `Flask debugging <http://flask.pocoo.org/docs/quickstart/#debug-mode>`_
这样提供了代码动态更新功能和更好的错误消息显示功能。 ::

    $ python api.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader

.. warning::

    调试模式永远不要用在生产服务器上！

现在打开一个新的终端来测试 API 效果，是用 `curl` 命令： ::

    $ curl http://127.0.0.1:5000/
    {"hello": "world"}

资源路由
-------------------
Flask-RESTful 提供的主体建筑块都是一些资源。
这些资源都建立在 `Flask pluggable views <http://flask.pocoo.org/docs/views/>`_
的顶部，这样你就可以使用多种 HTTP 方法来容易访问这些资源，
HTTP 方法是要定义在每一个资源上的，也就是通过写代码来实现。
一项基础的 CRUD 资源是 Create、Read、Update、Delete，
四种操作，对于一个应用来说我们必须要实现这四种操作，即：
建立、读取、更新、删除。那么代码如同下面一样： ::

    from flask import Flask, request
    from flask_restful import Resource, Api

    app = Flask(__name__)
    api = Api(app)

    todos = {}

    class TodoCURD(Resource):
        def post(self, todo_id):
            todos[todo_id] = request.form['data']
            return {todo_id: todos[todo_id]}

        def get(self, todo_id):
            return {todo_id: todos[todo_id]}

        def put(self, todo_id):
            todos[todo_id] = request.form['data']
            return {todo_id: todos[todo_id]}

        def delete(self, todo_id):
            try:
                del todos[todo_id]
                return 1
            except KeyError as e:
                pass

    class GetAll(Resource):
        def get(self):
            return todos

    api.add_resource(TodoCURD, '/<string:todo_id>')
    api.add_resource(GetAll, '/')

    if __name__ == "__main__":
        app.run(debug=True)

在另一个终端窗口常识如下测试命令： ::

    $ http://127.0.0.1:5000/create -d "data=C操作的HTTP方法是post，现在实用API来增加数据" -X POST
    {
    "create": "C\u64cd\u4f5c\u7684HTTP\u65b9\u6cd5\u662fpost\uff0c\u73b0\u5728\u5b9e\u7528API\u6765\u589e\u52a0\u6570\u636e"
    }
    $ curl http://127.0.0.1:5000/create
    {
        "create": "C\u64cd\u4f5c\u7684HTTP\u65b9\u6cd5\u662fpost\uff0c\u73b0\u5728\u5b9e\u7528API\u6765\u589e\u52a0\u6570\u636e"
    }
    $ curl http://127.0.0.1:5000/update -d "data=U操作的HTTP方法是PUT，现在实用API来更新数据，如果没有数据的话会起到增加数据的效果" -X PUT
    {
        "update": "U\u64cd\u4f5c\u7684HTTP\u65b9\u6cd5\u662fPUT\uff0c\u73b0\u5728\u5b9e\u7528API\u6765\u66f4\u65b0\u6570\u636e\uff0c\u5982\u679c\u6ca1\u6709\u6570\u636e\u7684\u8bdd\u4f1a\u8d77\u5230\u589e\u52a0\u6570\u636e\u7684\u6548\u679c"
    }
    $ curl http://127.0.0.1:5000/update
    {
        "update": "U\u64cd\u4f5c\u7684HTTP\u65b9\u6cd5\u662fPUT\uff0c\u73b0\u5728\u5b9e\u7528API\u6765\u66f4\u65b0\u6570\u636e\uff0c\u5982\u679c\u6ca1\u6709\u6570\u636e\u7684\u8bdd\u4f1a\u8d77\u5230\u589e\u52a0\u6570\u636e\u7684\u6548\u679c"
    }
    $ curl http://127.0.0.1:5000
    {
    "update": "U\u64cd\u4f5c\u7684HTTP\u65b9\u6cd5\u662fPUT\uff0c\u73b0\u5728\u5b9e\u7528API\u6765\u66f4\u65b0\u6570\u636e\uff0c\u5982\u679c\u6ca1\u6709\u6570\u636e\u7684\u8bdd\u4f1a\u8d77\u5230\u589e\u52a0\u6570\u636e\u7684\u6548\u679c",
    "create": "C\u64cd\u4f5c\u7684HTTP\u65b9\u6cd5\u662fpost\uff0c\u73b0\u5728\u5b9e\u7528API\u6765\u589e\u52a0\u6570\u636e"
    }
    $ curl http://127.0.0.1:5000/create -X DELETE
    1
    $ curl http://127.0.0.1:5000
    {
        "update": "U\u64cd\u4f5c\u7684HTTP\u65b9\u6cd5\u662fPUT\uff0c\u73b0\u5728\u5b9e\u7528API\u6765\u66f4\u65b0\u6570\u636e\uff0c\u5982\u679c\u6ca1\u6709\u6570\u636e\u7684\u8bdd\u4f1a\u8d77\u5230\u589e\u52a0\u6570\u636e\u7684\u6548\u679c"
    }


或者在 Python 里做测试，前提是你已经安装了 ``requests`` 库::

    >>> from requests import post, get, put, delete
    >>> post('http://127.0.0.1:5000/create', data={'data': 'C操作的HTTP方法是post，现在使用API来增加数据'}).json()
    {'create': 'C操作的HTTP方法是post，现在使用API来增加数据'}
    >>> get('http://127.0.0.1:5000').json()
    {'create': 'C操作的HTTP方法是post，现在使用API来增加数据'}
    >>> put('http://127.0.0.1:5000/create', data={'data': 'U操作的HTTP方法是PUT，现在使用API来更新数据，如果没有数据的话会起到增加数据的效果'}).json()
    {'create': 'U操作的HTTP方法是PUT，现在使用API来更新数据，如果没有数据的话会起到增加数据的效果'}
    >>> get('http://127.0.0.1:5000').json()
    {'create': 'U操作的HTTP方法是PUT，现在使用API来更新数据，如果没有数据的话会起到增加数据的效果'}
    >>> get('http://127.0.0.1:5000/create').json()
    {'create': 'U操作的HTTP方法是PUT，现在使用API来更新数据，如果没有数据的话会起到增加数据的效果'}
    >>> delete('http://127.0.0.1:5000/create').json()
    1
    >>> get('http://127.0.0.1:5000').json()
    {}

Flask-RESTful 能够理解多种从视图方法返回的值。
类似 Flask 一样，你可以返回任何一个可迭代对象，
迭代对象会转换成一个响应对象，包括生食 Flask 响应对象。
Flask-RESTful 也支持使用多个返回值配置响应代号和响应头部，
示例如下： ::

    class Todo1(Resource):
        def get(self):
            # Default to 200 OK
            return {'task': 'Hello world'}

    class Todo2(Resource):
        def get(self):
            # Set the response code to 201
            return {'task': 'Hello world'}, 201

    class Todo3(Resource):
        def get(self):
            # Set the response code to 201 and return custom headers
            return {'task': 'Hello world'}, 201, {'Etag': 'some-opaque-string'}


端点
---------

一个 API 多点访问，你的资源可以有多个 URLs 地址。
这样你可以在 `Api` 实例对象上用
:meth:`~Api.add_resource` 方法来增加多个地址规则。
每一条路线都会通往你的 :class:`Resource` 类 ::

    api.add_resource(HelloWorld,
        '/',
        '/hello')

你也可以用变量形式增加到路线中，匹配符合的用法。 ::

    api.add_resource(Todo,
        '/todo/<int:todo_id>', endpoint='todo_ep')


参数语法分析
----------------

在 Flask 提供容易访问请求数据的同时（例如，查询字符串或 POST 表单编码过的数据），
验证表单数据依然是一种痛苦。但 Flask-RESTful 具有内置支持，对请求数据验证来说
使用了一个类似 `argparse <http://docs.python.org/dev/library/argparse.html>`_
一样的库，名叫 `reqparse`。完整代码在 `examples/todo.py` 目录中。 ::

    from flask_restful import reqparse

    parser = reqparse.RequestParser()
    parser.add_argument('rate', type=int, help='Rate to charge for this resource')
    args = parser.parse_args()

.. note ::

    与 `argparse` 不同，其中 :meth:`reqparse.RequestParser.parse_args`
    方法返回的是一个 Python 字典数据类型，而不是自定义数据结构。

使用 :class:`reqparse` 模块中的类可以提供正常的错误消息给你。
如果一个参数验证失败， Flask-RESTful 会用 400 败坏请求代号
响应给你，并含有一个响应提醒错误。 ::

    $ curl -d 'rate=foo' http://127.0.0.1:5000/todos
    {
        "message": {
            "rate": "Rate to charge for this resource"
        }
    }


其中 :class:`inputs` 模块提供的类具有共性转换功能，
例如 :meth:`inputs.date` 和 :meth:`inputs.url` 方法。

调用 ``parse_args`` 方法时含有 ``strict=True`` 参数可以
让请求中的参数不符合你的语法分析器定义时确保抛出一个错误。 ::

    args = parser.parse_args(strict=True)

数据格式化
---------------

默认情况，你返回的可迭代对象中的所有区域会进行直译。
同时，当你处理 Python 数据结构时就有很棒的效果，
但与对象工作却变得非常麻烦。要解决对象问题，
Flask-RESTful 提供了 :class:`fields` 模块，
其中 :meth:`marshal_with` 装饰器方法解决了你的对象烦恼。
类似 Django ORM 和 WTForm 一样，你使用 ``fields`` 模块
来描述你的响应对象结构。 ::

    from flask_restful import fields, marshal_with

    resource_fields = {
        'task':   fields.String,
        'uri':    fields.Url('todo_ep')
    }

    class TodoDao(object):
        def __init__(self, todo_id, task):
            self.todo_id = todo_id
            self.task = task

            # This field will not be sent in the response
            self.status = 'active'

    class Todo(Resource):
        @marshal_with(resource_fields)
        def get(self, **kwargs):
            return TodoDao(todo_id='my_todo', task='Remember the milk')

上面的例子是得到一个 Python 对象后准备把这个对象做序列化处理。
其中 :meth:`marshal_with` 装饰器方法会把 ``resource_fields`` 字典进行变形处理。
只提取了 ``task`` 区域，而 :class:`fields.Url` 区域是一个特殊区域，
该特殊区域负责获得一个端点名后为响应对象中的端点生成一个 URL 地址。
你需要的许多区域类型都已经有了。阅读 :class:`fields` 模块指导内容了解完整清单。
在 `examples/todo.py` 中也可以找到这种示例代码。

完整的例子
------------

把下面代码保持在 `api.py` 文件中 ::

    from flask import Flask
    from flask_restful import reqparse, abort, Api, Resource

    app = Flask(__name__)
    api = Api(app)

    TODOS = {
        'todo1': {'task': 'build an API'},
        'todo2': {'task': '?????'},
        'todo3': {'task': 'profit!'},
    }


    def abort_if_todo_doesnt_exist(todo_id):
        if todo_id not in TODOS:
            abort(404, message="Todo {} doesn't exist".format(todo_id))

    parser = reqparse.RequestParser()
    parser.add_argument('task')


    # Todo
    # shows a single todo item and lets you delete a todo item
    class Todo(Resource):
        def get(self, todo_id):
            abort_if_todo_doesnt_exist(todo_id)
            return TODOS[todo_id]

        def delete(self, todo_id):
            abort_if_todo_doesnt_exist(todo_id)
            del TODOS[todo_id]
            return '', 204

        def put(self, todo_id):
            args = parser.parse_args()
            task = {'task': args['task']}
            TODOS[todo_id] = task
            return task, 201


    # TodoList
    # shows a list of all todos, and lets you POST to add new tasks
    class TodoList(Resource):
        def get(self):
            return TODOS

        def post(self):
            args = parser.parse_args()
            todo_id = int(max(TODOS.keys()).lstrip('todo')) + 1
            todo_id = 'todo%i' % todo_id
            TODOS[todo_id] = {'task': args['task']}
            return TODOS[todo_id], 201

    ##
    ## Actually setup the Api resource routing here
    ##
    api.add_resource(TodoList, '/todos')
    api.add_resource(Todo, '/todos/<todo_id>')


    if __name__ == '__main__':
        app.run(debug=True)


在终端里运行该示例： ::

    $ python api.py
      * Serving Flask app "api" (lazy loading)
     * Environment: production
       WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: on
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
     * Restarting with stat
     * Debugger is active!
     * Debugger PIN: 261-284-565

另一个终端里输入测试 API 命令 ::

    $ curl http://localhost:5000/todos
    {
        "todo1": {
            "task": "build an API"
        },
        "todo2": {
            "task": "?????"
        },
        "todo3": {
            "task": "profit!"
        }
    }

GET 单项任务命令 ::

    $ curl http://localhost:5000/todos/todo3
    {
        "task": "profit!"
    }

DELETE 一项任务命令 ::

    $ curl http://localhost:5000/todos/todo2 -X DELETE -v

    *   Trying ::1...
    * TCP_NODELAY set
    * Connection failed
    * connect to ::1 port 5000 failed: Connection refused
    *   Trying 127.0.0.1...
    * TCP_NODELAY set
    * Connected to localhost (127.0.0.1) port 5000 (#0)
    > DELETE /todos/todo2 HTTP/1.1
    > Host: localhost:5000
    > User-Agent: curl/7.54.0
    > Accept: */*
    > 
    * HTTP 1.0, assume close after body
    < HTTP/1.0 204 NO CONTENT
    < Content-Type: application/json
    < Server: Werkzeug/0.16.0 Python/3.7.4
    < Date: Tue, 08 Oct 2019 03:46:58 GMT
    < 
    * Closing connection 0

再增加一项任务的命令 ::

    $ curl http://localhost:5000/todos -d "task=something new" -X POST -v

    Note: Unnecessary use of -X or --request, POST is already inferred.
    *   Trying ::1...
    * TCP_NODELAY set
    * Connection failed
    * connect to ::1 port 5000 failed: Connection refused
    *   Trying 127.0.0.1...
    * TCP_NODELAY set
    * Connected to localhost (127.0.0.1) port 5000 (#0)
    > POST /todos HTTP/1.1
    > Host: localhost:5000
    > User-Agent: curl/7.54.0
    > Accept: */*
    > Content-Length: 18
    > Content-Type: application/x-www-form-urlencoded
    > 
    * upload completely sent off: 18 out of 18 bytes
    * HTTP 1.0, assume close after body
    < HTTP/1.0 201 CREATED
    < Content-Type: application/json
    < Content-Length: 32
    < Server: Werkzeug/0.16.0 Python/3.7.4
    < Date: Tue, 08 Oct 2019 03:49:02 GMT
    < 
    {
        "task": "something new"
    }
    * Closing connection 0

更新一项任务内容的命令 ::

    $ curl http://localhost:5000/todos/todo3 -d "task=something different" -X PUT -v

    *   Trying ::1...
    * TCP_NODELAY set
    * Connection failed
    * connect to ::1 port 5000 failed: Connection refused
    *   Trying 127.0.0.1...
    * TCP_NODELAY set
    * Connected to localhost (127.0.0.1) port 5000 (#0)
    > PUT /todos/todo3 HTTP/1.1
    > Host: localhost:5000
    > User-Agent: curl/7.54.0
    > Accept: */*
    > Content-Length: 24
    > Content-Type: application/x-www-form-urlencoded
    > 
    * upload completely sent off: 24 out of 24 bytes
    * HTTP 1.0, assume close after body
    < HTTP/1.0 201 CREATED
    < Content-Type: application/json
    < Content-Length: 38
    < Server: Werkzeug/0.16.0 Python/3.7.4
    < Date: Tue, 08 Oct 2019 03:50:28 GMT
    < 
    {
        "task": "something different"
    }
    * Closing connection 0

