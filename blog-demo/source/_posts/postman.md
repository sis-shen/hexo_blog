---
title: Postman服务端HTTP接口测试工具
date: 2025-03-13 08:22:38
tags:
---
`Postman`是一款强大的API开发协作工具，它的功能十分丰富，但是在这里我们仅选取要运用的接口测试相关的功能。

# 安装Postman
[点我去下载页面🔗https://www.postman.com/downloads/](https://www.postman.com/downloads/)

这里作者是`Windows`环境，所以下载的是`Windows`版本的安装包。*其实它还提供了`vscode 插件`、`命令行工具`等不同的安装形式*

# 认识Postman界面
首先是认识`左侧选项卡`和`底边选项卡`
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503130901946.webp)

然后我们着重认识一下集合管理和创建接口测试

# 开始准备测试

## 创建集合
集合类似于文件夹，可以把一组相关的请求接口放在同一个集合里。

为了创建集合，我们点击图中的加号,创建一个空集合,然后右键集合再重命名为`testForBlog`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503130909342.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503130911860.webp)

## 创建请求接口
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503130913119.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503130919778.webp)

如图，我们创建了一个请求接口，把它重命名为`test1`，调用的API默认为`GET`，然后输入了`baidu.com`的URL,发送之后我们就获得了响应报文。

这样就完成了一次接口的单次测试

## 使用其它请求接口
如图,`Postman`提供了多种接口

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503130927415.webp)\

## 编写请求报文和测试脚本
如图，这里使用了我一个项目的一个查询接口，它使用`POST`接口，报文是`JSON`格式，响应也是`JSON`格式，我们将会依据此编写响应的测试脚本

[戳我去官方的测试脚本文档🔗](https://learning.postman.com/docs/tests-and-scripts/write-scripts/test-scripts/#home)

### 配置URL和报文

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503130930568.webp)

### 配置测试脚本

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503130933091.webp)

关于测试脚本，有如下解释：

+ `创建测试`：在`Scripts`选项卡中，每次调用`pm.test()`接口都会创建一次测试
  + `参数1`:字符串，本次测试的名称，一般也用于顺便描述测试
  + `参数2`:函数，可用lambda表达式，用返回值的`true`或者`false`表示本次测试的通过与否
+ `在测试中获取响应`: `Postman`提供了`pm.response`用于获取本次请求获得的响应对象，可通过对它的成员的访问，完成对响应的访问
+ `格式化测试`：`PM`提供了简化的多条件测试方式，只需要在同一次测试中将不同的测试条件分行编写即可，示例如下:

*官网的不同测试代码样例,其中汉化为个人汉化*

1. 返回状态码测试
```js
pm.test("状态码测试", function () {
    pm.response.to.have.status(200);
});
```
2. 测试响应的报文为`JSON`格式，且有键`"dish"`，没有键`"error"`
```js
pm.test("响应的报文应该能够被正确处理", function () {
    pm.response.to.not.be.error;
    pm.response.to.have.jsonBody("");
    pm.response.to.not.have.jsonBody("error");
});
```

3. 测试响应正确，且有报文，且报文格式为`JSON`
```js
pm.test("响应合法且有报文", function () {
     pm.response.to.be.ok;
     pm.response.to.be.withBody;
     pm.response.to.be.json;
});
```
## 查看测试结果
以上的准备工作做好后，可以点击`Send`按钮获取响应并开始测试

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503131011498.webp)

# 单接口串行压力测试
在单次接口测试编写完成后，我们就可以开始使用`Runner`功能进行压力测试了

1. 我们点击底边栏的`Runner`按钮，然后将待测试的集合拖动到工作区内

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503131101309.webp)

2. 配置测试参数
   1. 选择待测试的接口（它们之间是轮询式串行运行）,因为这里是单接口测试，所以只选一个
   2. 设置测试模式为`Function`
   3. 设置执行模式，这里使用手动模式(`Run manually`)
   4. 设置每个接口测试多少次
   5. 设置发送延迟
3. 点击`Run '集合名称'`按钮

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503131108296.webp)

运行结果如下:

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503131346462.webp)


# 单接口并行压力测试
在单次接口测试编写完成后，我们就可以开始使用`Runner`功能进行压力测试了

1. 我们点击底边栏的`Runner`按钮，然后将待测试的集合拖动到工作区内

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503131101309.webp)

2. 配置测试参数
   1. 选择待测试的接口,因为这里是单接口测试，所以只选一个
   2. 设置测试模式为`Performance`
   3. 设置执行模式，这里使用手动模式(`Run manually`)
   4. 设置并发量(virtual user)的数量
   5. 设置发送延迟
   6. 设置并发测试的总时长
3. 点击`Run '集合名称'`按钮

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503131348456.webp)

运行过程及结果如下:

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503131348456.webp)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/202503131351527.webp)


