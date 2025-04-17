---
title: Rust框架入门之Axum+Sea-ORM实现MySQL增删查改
date: 2025-03-29 16:59:15
tags:
---
# 框架介绍

## Axum
一款基于`tokio`的web框架
TODO

## Sea_ORM
一款提供对象关系映射服务的项目
# 环境准备
+ 编程环境: 安装好rust
+ 编码环境：vscode + rust analyzer插件
+ 数据库：安装好本机上的`MySQL Server`

# 开始上手

## 数据库准备
虽然`Sea-ORM`提供数据库的建库和建表操作，但是为了简化项目难度，这次我们就手动创建了

这里给出初始化的sql语句
```sql
DROP DATABASE if EXISTS RustMysql;
CREATE DATABASE RustMysql DEFAULT CHARACTER SET utf8mb4;

use RustMysql;

create table account(
    `id` BIGINT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
    `name` varchar(16) not null unique
)ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## 安装swa-orm-cli客户端
这是一个专门用于自动生成sea-orm相关文件的命令行客户端程序，我们接下来的操作依赖它，所以要提前安装
```shell
cargo install sea-orm-cli@1.1.0
```
## 创建项目
在命令行输入`cargo new rust_mysql_forblog`创建项目,这里的`rust_mysql_forblog`为项目名，后文用`<prj>`代替

## 配置项目环境变量
在项目的根目录`<prj>/`下创建配置文件`.env`,然后在里面写入数据库的URL和我们服务器的监听地址和端口

```.env
DATABASE_URL=mysql://blogroot:Password!1@127.0.0.1:3306/RustMysql
HOST=0.0.0.0
PORT=7878
```

## 生成sea-orm的Entity实例
在配置好数据库、配置好环境变量和安装好`sea-orm-cli`后，我们就可以在项目中生成`Entity`了

首先进入项目的根目录`<prj>/`,然后在命令行输入指令:
```shell
sea-orm-cli generate entity -o entity/src
```
**注：这里不用输入-u指定数据库url是因为它会自动读取.env文件中的DATABASE_URL变量**

完成后，我们可以看到项目下多了个`entity`包,现在我们先把它放着

## 初始化entity包
sea-orm仅仅只是生成了`.rs`源文件，但是如果要作为一个包导入，就得先初始化

首先输入下面的指令完成初始化
```
cargo init entity --lib
```

然后删除自动生成的`lib.rs`,把`mod.rs`文件重命名为`lib.rs`改为包根

以及编辑新增的`Cargo.toml`新增如下依赖

+ `sea-orm`依赖,保证包能够被成功编译,内容如下
+ `serde`依赖，使结构体能够有序列化特征和反序列化特征

```toml
sea-orm = {version = "1.1.0",features = ["sqlx-mysql","runtime-tokio-rustls","macros"]}
serde = { version = "1", features = ["derive"] }
```

修改`Model`的`derive宏`,为其添加`Serialize`和`Deserialize`
```rs
use serde::{Deserialize,Serialize};
#[derive(Clone, Debug, PartialEq, DeriveEntityModel, Eq,Serialize,Deserialize)]
#[sea_orm(table_name = "account")]
pub struct Model {
    #[sea_orm(primary_key)]
    pub id: u64,
    #[sea_orm(unique)]
    pub name: String,
}
```

## 创建service包
首先进入项目的根目录`<prj>/`,然后在命令行输入指令:
```shell
cargo new service --lib
```

## 配置service包的依赖
我们编辑`<prj>/service/Cargo.toml`文件内的`[dependencies]`项,我们添加如下依赖

+ `axum`: 提供web框架的接口
+ `tokio`: 提供异步io功能
+ `sea-orm`: 提供对象关系映射接口，与数据库完成交互
+ `dotenv`: 提供对`.env`环境变量的解析
+ `anyhow`: 用于让函数返回任意的`Result`类型
+ `tera`: 根据模板文件自动生成html页面内容
+ `serde`: 提供序列化和反序列化的特征
+ `tower-cookies`: 解析Cookie
+ `entity`:本地生成的包,手动提供路径

```toml
[dependencies]
axum = { version = "0.8.1", features = ["tokio"] }
tokio = { version = "1.25.0", features = ["full", "macros", "rt-multi-thread"] }
sea-orm = {version = "1.1.0",features = ["sqlx-mysql","runtime-tokio-rustls","macros"]}
dotenv = "0.15"
anyhow = "1.0.75"
tera = "1.19.1"
serde = "1.0.193"
tower-cookies = "0.11"
entity = { path = "../entity" }
```

## 编写service包的接口
对于要`use`哪些依赖，我们会在编程的过程中逐步给出

### 定义状态结构体
我们要使用一个状态结构体，用于把`seo-orm`的数据库的连接实例存储起来，并用于在各个异步函数中传递

**注：sea-orm依赖在代码中被重命名成了sea_orm**
```rs
use axum::extract::State;
use sea_orm::{Database,DatabaseConnection};

#[derive(Clone)]
struct AppState{
    db:DatabaseConnection
}
```

### 编写主函数体 完成初始化
使用`#[tokio::main]`标记异步主函数,我们主要有以下步骤
1. 解析`.env`文件并提取变量
2. 获取数据库连接实例并存入状态中
3. 创建路由应用并注入状态
4. 开启监听

```rs
use std::env;
use std::fmt::format;
use axum::{Router,routing::get,routing::post};
use tower_cookies::{Cookies,CookieManagerLayer};


//中间代码略

#[tokio::main]
pub async fn main()->anyhow::Result<()>
{
    //加载配置文件
    dotenv::dotenv().expect(".env文件加载失败");

    //提取变量
    let db_url = env::var("DATABASE_URL").unwrap();
    let host = env::var("HOST").expect(".env文件中未配置 HOST 变量");
    let port = env::var("PORT").expect(".env文件中未配置 HOST 变量");
    //拼接监听url
    let server_url = format!("{host}:{port}");
    //获取db实例
    let db = Database::connect(db_url).await?;
    //获取模板
    let templates = tera::Tera::new(concat!(env!("CARGO_MANIFEST_DIR"),"/templates/**/*")).expect("获取Tera模板失败");

    //创建状态变量s
    let shared_state = AppState{db,templates};
    //构建路由 ,注入Cookie层和状态对象 TODO
    let app = Router::new()
    .layer(CookieManagerLayer::new())
    .with_state(shared_state);
    //开启监听
    println!("server start on {}",server_url);
    let listener = tokio::net::TcpListener::bind(&server_url).await?;
    axum::serve(listener,app).await?;
    Ok(())
}
```

至此，web的主题框架已经完成，剩下地便是编写各种接口函数`handler`，并注册到路由中

## 各种handler函数

<table>
    <tr>
        <td><b>接口</b></td>
        <td><b>用途</b></td>
    </tr>
    <tr>
        <td>(GET)/users/all</td>
        <td>获取用户列表</td>
    </tr>
    <tr>
        <td>(POST)/users/add</td>
        <td>新增一个用户</td>
    </tr>
    <tr>
        <td>(GET)/users/get/&lt;username&gt;</td>
        <td>查看指定用户</td>
    </tr>
    <tr>
        <td>(POST)/user/del/</td>
        <td>删除指定用户</td>
    </tr>
</table>

### 创建资源文件
我们创建一个`<prj>/res/html`文件夹，专门用于存放`html`文件

然后编写一些内容用于显示部分网页

> index.html
```rs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Welcome to Auxm</h1>
    <h1>go to path <a href="/users">/users</a> to get more info about operations about users</h1>
</body>
</html>
```

> users_info.html
```rs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <table>
        <tr>
            <td><b>接口</b></td>
            <td><b>用途</b></td>
        </tr>
        <tr>
            <td>(GET)/users/all</td>
            <td>获取用户列表</td>
        </tr>
        <tr>
            <td>(POST)/users/add</td>
            <td>新增一个用户</td>
        </tr>
        <tr>
            <td>(GET)/users/get/&lt;username&gt;</td>
            <td>查看指定用户</td>
        </tr>
        <tr>
            <td>(POST)/user/del/</td>
            <td>删除指定用户</td>
        </tr>
    </table>

</body>
</html>
```

> 404.html
```rs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>资源不存在！</title>
</head>
<body>
    <h1>404 NOT FUND!</h1>
</body>
</html>
```

### get_404()
返回404页面的函数最先写出，防止资源不存在，却没有方法返回404页面,同时这里使用可能`panic!`的展开，防止运行时连404页面都显示不出来

```rust
use tokio::fs::read_to_string;
use axum::{
    Json,
    response::Html,
    http::StatusCode,
};
//中间代码省略

async fn get_404()->Result<Html<String>, (StatusCode, &'static str)>{
    let body = read_to_string("./res/html/404.html").await.expect("连404页面都找不到了？这服务还是别开了");

    Ok(Html(body))
}
```

### 创建模板文件
这里用到了`Tera`包用于自动把结构体解析成`html`页面文件，所以要提供相关的模板文件

我们把`templates`文件夹创建在`<prj>/service/`下，即我们的`service`包的根目录，然后写入如下文件

+ layout.html.tera
+ users_select.html.tera

> layout.html.tera
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Axum Example</title>
    <meta name="description" content="Axum - SeaOrm integration example" />
    <meta name="author" content="Yoshiera Huang" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <link
      href="//fonts.googleapis.com/css?family=Raleway:400,300,600"
      rel="stylesheet"
      type="text/css"
    />
    <link rel="stylesheet" href="/static/css/normalize.css" />
    <link rel="stylesheet" href="/static/css/skeleton.css" />
    <link rel="stylesheet" href="/static/css/style.css" />
    <link rel="icon" type="image/png" href="/static/images/favicon.png" />
  </head>
  <body>
    <div class="container">
      <p><!--Nothing to see here --></p>
      {% block content %}{% endblock content %}
    </div>
  </body>
</html>

```

> users_select.html.tera
```html
{% extends "layout.html.tera" %} {% block content %}
<div class="container">
  <p><!--Nothing to see here --></p>
  <h1>Account</h1>
  {#{% if flash %}
  <small class="field-{{ flash.kind }}-flash">
    {{ flash.message }}
  </small>
  {% endif %}#}
  <table>
    <tbody>
      <thead>
        <tr>
          <th>id</th>
          <th>name</th>
          <th>age</th>
        </tr>
      </thead>
      {% for user in users %}
      <tr class="user" onclick="window.location='/users/{{ user.id }}';">
        <td>{{ user.id }}</td>
        <td>{{ user.name }}</td>
        <td>{{ user.age }}</td>
      </tr>
      {% endfor %}
    </tbody>
    {#<tfoot>
      <tr>
        <td></td>
        <td>
          {% if page == 1 %} Previous {% else %}
          <a href="/?page={{ page - 1 }}&posts_per_page={{ posts_per_page }}"
            >Previous</a
          >
          {% endif %} | {% if page == num_pages %} Next {% else %}
          <a href="/?page={{ page + 1 }}&posts_per_page={{ posts_per_page }}"
            >Next</a
          >
          {% endif %}
        </td>
        <td></td>
      </tr>
    </tfoot>
  </table>#}

  {#<div class="twelve columns">
    <a href="/new">
      <input type="button" value="add post" />
    </a>
  </div>#}
</div>
{% endblock content %}


```

### get_index()
返回首页的html文件,为了应对文件不存在的情况，这里简单地用一下match匹配

```rust
async fn get_index()->Result<Html<String>, (StatusCode, &'static str)>{
    let body  = match read_to_string("./res/html/index.html").await{
        Ok(content)=>content,
        Err(_)=>return get_404().await,
    };
    Ok(Html(body))

}
```

### get_info 
```rust
async fn get_info()->Result<Html<String>, (StatusCode, &'static str)>{
    let body = match read_to_string("./res/html/users_info.html").await{
        Ok(content) =>content,
        Err(_) => return get_404().await,
    };
    Ok(Html(body))
}
```

### get_users_all
前面的都只是返回静态网页，还只是开胃小菜，接下来要编写的是与数据库交互的接口。

特别的，这里就要用到`tera`去渲染页面了。

```rust
use entity::account;
use entity::prelude::Account;

async fn get_users_all(
    State(state):State<AppState>,
    _:(),
)-> Result<Html<String>, (StatusCode, &'static str)>{
    let users = Account::find()
    .all(&state.db)
    .await
    .map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR,"数据库查询失败"))?;

    let mut ctx = tera::Context::new();

    ctx.insert("users", &users);

    let body = state
    .templates
    .render("users_select.html.tera",&ctx)
    .map_err(|_| (StatusCode::INTERNAL_SERVER_ERROR, "Template error"))?;

    Ok(Html(body))
}

```

### get_user_by_id
这里使用`find_by_id`方法即可
```rust
async fn get_user_by_id(
    State(state):State<AppState>,
    Path(id):Path<u64>,
)-> Result<Html<String>, (StatusCode, &'static str)>{
    let user:account::Model = Account::find_by_id(id).
        one(&state.db)
        .await
        .map_err(|_|(StatusCode::INTERNAL_SERVER_ERROR,"数据库查询失败"))?
        .unwrap();

    Ok(Html(format!("{:?}",user)))
}
```


### get_users_by_name
由于这里的筛选条件,要用到sea-orm的
```rust
use sea_orm::entity::prelude::*;

async fn get_users_by_name(
    State(state):State<AppState>,
    Path(name):Path<String>
)-> Result<Html<String>,(StatusCode,&'static str)>{
    let users = Account::find()
        .filter(account::Column::Name.contains(&name))
        .all(&state.db)
        .await
        .map_err(|_|(StatusCode::INTERNAL_SERVER_ERROR,"数据库查询失败") )?;

    let mut ctx = tera::Context::new();

    ctx.insert("users", &users);

    let body = state
    .templates
    .render("users_select.html.tera",&ctx)
    .map_err(|_| (StatusCode::INTERNAL_SERVER_ERROR, "Template error"))?;

    Ok(Html(body))
}
```

### post_add_user
为了添加记录，需要从参数获取表单数据，以及创建`ActiveModel`对象。

```rust
async fn post_add_user(
    State(state):State<AppState>,
    mut _cookie:Cookies,
    form:Form<account::Model>
)->Result<Html<String>, (StatusCode, &'static str)>{
    let form =form.0;

    account::ActiveModel{
        id:sea_orm::ActiveValue::NotSet,
        name:sea_orm::Set(form.name),
        age:sea_orm::Set(form.age)
    }
    .save(&state.db)
    .await
    .map_err(|_| (StatusCode::INTERNAL_SERVER_ERROR, "保存失败"))?;

    Ok(Html(r#"<h1>添加用户成功</h1>"#.to_string()))
}
```

## 依赖项汇总
最后我们把各种`use`到的依赖汇总一下

```rust
use axum::{
    extract::{Path, State},
    Form,
    Json,
    response::Html,
    http::StatusCode,
};
use axum::{Router,routing::get,routing::post};
use sea_orm::{Database,DatabaseConnection, EntityTrait, QueryFilter};
use sea_orm::entity::prelude::*;

use entity::{account,prelude::Account};

use std::env;
use tokio::fs::read_to_string;

use tower_cookies::{Cookies,CookieManagerLayer};
use serde::{Deserialize,Serialize};
```