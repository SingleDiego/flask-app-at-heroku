# Heroku 部署 Flask 应用（数据库设置）


在 [上一章](https://www.jianshu.com/p/b3478aaa82d3) 我们了解到了在 heroku 创建用户，创建应用，并利用 Git 和 heroku CLI 工具来把用于代码推送到云端并部署的基本操作。

本章我们更深入一步，讨论如何在我们的应用中使用 heroku 自带的 PostgreSQL 数据库，并且和 ``Flask-SQLAlchemy`` 无缝衔接。


<br>
<hr>
<br>


### 在 heroku 创建 PostgreSQL  数据库

假设我们已经在 heroku 创建好应用并且已经下载完成 heroku CLI 命令行工具和 Git 版本控制工具。

在 heroku CLI 登录我们的账号后，用命令创建一个 PostgreSQL  数据库：
```
heroku addons:create heroku-postgresql:hobby-dev --app <app name>
```

这条命令里的 ``hobby-dev``，表示创建一个“爱好者”级别的数据库，这在 heroku 的服务中是免费的。

``<app name>`` 部分请换成你创建的应用的名称。

这时候我们用 ``heroku config`` 命令来查看环境变量：
```
$ heroku config
=== flask-microblog-diego Config Vars
DATABASE_URL: postgres://uqyxkahrktzbdo...@.amazonaws.com:5432/d96...
```

创建数据库完毕后，环境变量中就多了 ``DATABASE_URL`` 这一项，这是我们应用的数据库的地址，我们要在应用中使用数据库，应该从环境变量中引用该变量。

除了在命令行查看外，在 heroko 的应用管理页面的 ``Resources`` 选项中我们也可以看到创建的数据库。

![](https://upload-images.jianshu.io/upload_images/2070024-30e155524b625841.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击数据库管理页面后选择 ``Setting / Database Credentials``，在这里我们也可以查看到包括数据库地址在内的相关信息。

![](https://upload-images.jianshu.io/upload_images/2070024-f2a1c486a4929cee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br>
<hr>
<br>


### 准备 Flask 文件

我们的 Flask 文件并无太多特别值得说明的地方，我只会讲解几个关键点，其余详情请翻看源码。数据库部分我会使用 ``Flask-SQLAlchemy`` 和 ``Flask-Migrate`` 作为 ORM 实例化和数据库管理的工具；这都是 Flask 的常规操作。

##### 输出依赖文件

为了用 Python 操作 PostgreSQL 我们需要先安装 ``psycopg2`` 库：
```
(venv) $ pip install psycopg2
```

相关的库安装完成后，我们把依赖文件导出：
```
pip freeze >requirements.txt
```

##### 编写 Procfile

编写 ``Procfile`` 文件如下：
```
web: flask db upgrade; gunicorn microblog:app
```

与上一章相比，增加了 ``flask db upgrade;`` 命令，这是为了让数据库初始化并创建各表。

需要注意的是 ``gunicorn`` 语句中的冒号后不可加空格。

##### 编写 config

编写配置部分是本章的重中之重，因为我们应用的数据库路径就在这部分定义。我们先来看代码：
```
# config.py

import os

basedir = os.path.abspath(os.path.dirname(__file__))

class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'

    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL', '').replace(
        'postgres://', 'postgresql://') or \
        'sqlite:///' + os.path.join(basedir, 'app.db')

    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

应用的数据库路径 ``SQLALCHEMY_DATABASE_URI `` 是在环境变量中获取 ``DATABASE_URL`` 而生成的，这里为什么要用一个 ``replace`` 函数呢？因为 ``Flask-SQLAlchemy`` 链接 PostgreSQL 数据库时候使用的路径开头部分是 ``'postgresql://'``，所以我们需要把 ``'postgres://'`` 替换为 ``'postgresql://'``。

这里还使用了一个 ``OR`` 关键字，当环境变量中不存在 ``DATABASE_URL`` 时，就使用会原来的 ``sqlite`` 数据库。


<br>
<hr>
<br>


### 设置 heroku 环境变量

还记得我们在启动 Flask 应用时候，要先配置环境变量指定 ``FLASK_APP`` 的参数吧，在 heroku 也同意可以这样做。
```
￥ heroku config:set FLASK_APP=microblog
```

完成后我们再来查看下环境变量：
```
$ heroku config
=== flask-microblog-diego Config Vars
DATABASE_URL: postgres://uqy...zbdo:ca855c...@ec2-34....amazonaws.com:5432/...8lu
FLASK_APP:    microblog
```


<br>
<hr>
<br>


### 使用 Git 推送到云端

如果是第一次操作，需要先进行 Git 的初始化，并设置 heroku 远程仓库
```
$ git init
$ heroku git:remote -a <app-name>
```

推送的云端的方法和上一篇一样，先 ``add``，在 ``commit`` 再 ``push``。

```
$ git add .
$ git commit -am "make it better"
$ git push heroku master
```

现在应用已经成功部署再 heroku 上，并已经使用了 PostgreSQL 数据库。


<br>
<hr>
<br>


本文源码：https://github.com/SingleDiego/flask-app-at-heroku
