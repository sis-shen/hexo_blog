---
title: C++文件操作
date: 2024-05-14 10:19:32
tags: C++ 文件
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-05-21_19-59-20.png
---

*注:追求代码简洁,有一致的C++风格，可参阅本篇博客，若追求更高的读写效率，建议参阅C语言篇* [但文章还没写]()

本篇文章主要研究头文件`fstream`中的**函数**和**类**

目前C++文件操作主要有两种流派,一种是声明`fstream`对象,另一种是分开声明`ifstream`和`ofstream`

**注意，本文代码为了简洁，都是在展开std命名空间的前提下书写**

# fstream的使用
先写一段示例代码
```C++
#include <iostream>
#include <fstream>

using namespace std;

int main()
{
	fstream f("file.txt", ios::out);//调用构造函数以out模式打开文件file.txt 注:out模式下file.txt 会自动创建
	string str = "This is a sentence";//在内存中准备一段字符串
	f << str;//将字符串从内存写入文件
	f.close();//关闭文件流

	//f对象可以复用
	f.open("file.txt", ios::in);//以in模式打开file.txt
	string content;//声明变量
	f >> content;//从文件流读取数据写入变量(内存)
	cout << content;//打印出来看一眼
    f.close();

	return 0;
}

```
*输出结果*
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-05-14_18-49-56.png)

这一小段代码完成了文件的两种模式的打开和读写，已经体现了`fstream`的基本功能,接下来分别详细介绍`成员函数`和`操作符重载`

## fstreanm::open()
函数声明:`void open(const char* filename, ios_base::openmode mode)`

特别的,**C++11**增加了一个函数重载，第一个形参变为`const string& filename`

实际上，也可以通过`fstream`类的构造函数来打开文件，参数与`open()`函数相同

```C++
fstream f("filename",ios::out);
```

接下来分别介绍两个形参
### filename
**一般**第一个形参是文件名，可以传`字符串`/`char*`指针，C++11还支持传`string`对象

文件名没什么好说的，就是有后缀的文件要**注意后缀**,以及文件名要写对，勤检查

**但实际上**,第一个形参是`文件路径`,且支持`相对路径`,~~绝对路径我测不出来~~

*代码示例如下*
```C++
f.open("this,txt",ios::in);///打开当前文件夹的文件,this.txt是文件名
f.open("../father.txt",ios::in);//打开父级文件夹的文件,father.txt是文件名
f.open("./dir/child.txt",ios::in);//打开子级文件夹的文件(dir是文件夹名称)child.txt是文件名
```

### mode

**用前须知**:这些`mode`都存在于`ios_base`的类域中,但由于`ios`继承自`ios_base`,混用二者皆可，本文为了简洁，指定类域时，使用`ios`，如`f.open("file.txt",ios::out)`

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-05-16_19-22-46.png)

*上图是继承关系图，箭头指向父类*


这里依然统一使用`fstream`类声明一个`f`对象

```C++
fstream f;
```

打开模式比较多，下面先放一张表

| mode | stands for | 描述 |
| --- | --- | --- |
| out | output | 打开文件用于`写入`,即`内存->文件`,且会**完全覆盖**原文件,内置的流缓存(`internal stream buffer`)支持输出操作(类似`cout`)|
| in  | input | 打开文件用于读取,即`文件->内存`,内置的流缓存支持输入操作(`类似cin`)|
| app | append | 所有的输出操作都**追加在文件末尾**,向已有内容追加文本 |
| trunc | truncate | 在打文件**前**,**清除所有内容** |
| binary | binary | 所有操作都以`二进制`的形式，而不是文本 |
| ate | at end | 输出操作在文件末尾开始 |

*注*,这些`mode`能用`|`操作符连接,**同时**使用这些`mode`

**但是**

-使用`trunc | app`会打开失败
-使用`trunc`而未使用`|`连接`out`,也会失败

接下来逐一介绍这些`mode`

#### out 和 in
`out`是最常用的模式之一,用于`覆盖`写入文件,而且当文件不存在时，会按文件名**新建**一个文件并执行写入操作(哪怕是空文件)

`in`也是最常用的模式之一，用于`只读`地读取文件，且当文件不存在时，会**抛异常**(*这里挖个坑*)



- 使用`out`时，`f`对象支持`<<`流插入操作符

```C++
string str("123456");
f<<str<<endl;//和输出内容到终端(cout)是一样的
```

- 使用`in`时,`f`对象支持`>>`流提取和作为`getline`函数的参数

```C++
string s1,s2;
f>>s1>>s2;//和从终端提取内容(cin)是一样的

string line;
getline(f,line);//从文件流中提取一行,存入line对象

while(getline(f,line))//逐行提取至文件末尾
{
	//....
}
```

- 使用`in | out`时，`f`同时支持以上操作

但是`写入`操作又和单一个`out`不一样，`单out`是完全覆盖，不考虑原文件内容,而`in | out`时，是**不完全**覆盖：从头开始覆盖，新写入的内容没有原文件长时，剩下的原文件**依然保留**

>*示例代码*
```C++
int main()
{
	fstream f;
	f.open("ff.txt", ios::out);
	f << "LongWord";//准备一个有内容的原文件
	f.close();

	f.open("ff.txt", ios::in | ios::out);
	f << "1234";//写入1234
	f.close();
	//最后ff.txt的内容为"1234Word",不完全覆盖
	return 0;
}

```

但是，同时进行流插入和流提取时，文件的操作结果会比较诡异，所以并不建议对**同一个文件流**同时进行读写操作。

在一段`f.open()`和`f.close()`之间依然还是只进行读取**或**写入中的**一种**，而不要混合操作

**缺省参数**：其实`mode`形参是有缺省参数的，正是`ios_base::in | ios_base::out`,也就是说在明确只使用`out`或`in`的情况下，且执行覆盖写入操作时，单写一个`f.open(文件名)`即可

#### 其它mode

##### app
正如表格里描述的，使用`app`时，写入时不会覆盖远内容，而是`追加`到文件末尾。其中与`out`一样，当文件不存在时，会自动创建并写入内容。(即使没内容，也会创建空文件)
```C++
int main()
{
	fstream f;

	f.open("file.txt",ios::out);
	f << "111";//准备一个内容为111的文件
	f.close();
	f.open("file.txt", ios::app);
	f << "222";//111后面追加222
	f.close();
	//文件内容为111222
	return 0;
}
```

##### trunc
因为不加`out`会打开失败，所以`trunc`算是个`out`的修正,在以`out`模式打开前，清空原文件的内容

乍一看，因为`单out`是`完全覆盖写入`,似乎`trunc`没什么用

但是使用`ios::out | ios::in | ios::trunc`时是不完全覆盖写入，所以提前清空内容还是很有意义的。~~(那为啥不用单out呢)~~

##### ate
全称`at end`,单用`ate`也会打开失败,当然，`ios::ate | ios::out`没有意义，因为还是完全覆盖写入,`ate`在`ios::in | ios::out | ios::in`更加有用，可以从文件末尾追加内容

##### binary
虽然表格里是那么说了，有没有用`binary`我是真测不出差别(~~真要用的话，另寻高就把~~)

但是单用`binary`依然会打开失败,需要再连个`out`或`in`

---

# ifstream 和 ofstream
![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-05-17_11-18-29.png)

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/PixPin_2024-05-17_11-18-48.png)

上图分别为二者的继承关系,实际上二者用起来和`fstream`是基本一样的，只不过在打开文件时，一个始终自带`ios::in`,另一个始终自带`ios::out`

# 读取文件流的一些方法

## `>>`操作符
这是最容易理解的方式，和从终端读取`数据`到变量里是一样的,只要类型匹配，不一定要存到字符串里

## `getline()`函数
用`getline`可以读取一行(即读到`\n`或文件末尾`EOF`)

但`getline`主要有两种，存在于不同的头文件中，且参数不同

### &lt;string&gt;中的getline
`istream& getline (istream& is, string& str);`

函数声明如上，第一个参数是文件流(`fstream`类或`istream`类都可以),第二个参数就是个`string`对象

下面是一个逐行提取的例子
```C++
#include <iostream>
#include <fstream>
#include <string>

using namespace std;

int main()
{
	//提前准备一个待提取文件
	fstream f;
	f.open("ff.txt", ios::out);
	f << "line 1" << endl << "line 2" << endl;
	f.close();
	//================
	f.open("ff.txt");
	string str;
	while (getline(f, str))//当f为空时，循环停止
	{
		cout << str << endl;//打印每行,str内不含换行符
	}
	f.close();
	return 0;
}
```

### 成员函数中的getline
`std::istream::getline`

根据继承关系,`fstream`继承了来自`istream`的`getline`成员函数,也就是说它们的对象都能调用这一成员函数

`istream& getline (char* s, streamsize n );`

函数声明如上，可以看到，第一个参数是`char*`，要传给它一个`字符数组`,第二个则是读入字符数的最大值,当实际读入的字符数**小于**`n`时，会自动在结尾加一个`\0`

~~讲真这个函数更像是来自C语言的函数~~

示例代码
```C++
int main()
{
	fstream f;
	f.open("ff.txt", ios::out);
	f << "line 1" << endl << "line 2" << endl;
	f.close();

	f.open("ff.txt");
	char str[256] = { 0 };
	f.getline(str, 256);//str里存了line 1\0
	f.close();
	cout << str;
	return 0;
}
```

## get() 函数
```C++
int get();
istream& get (char& c);
```

如上是两种常用的函数重载,均为继承来的`成员函数`,逐字符提取的话就能提取到`\n``\r`之类的转义字符

代码示例
```C++
int main()
{
	//提前准备一个待提取文件
	fstream f;
	f.open("ff.txt", ios::out);
	f << "line 1" << endl << "line 2" << endl;
	f.close();
	//================
	//方法1
	f.open("ff.txt");
	char ch;
	while (f.get(ch))//获取字符
	{
		cout << ch;//打印字符
	}
	f.close();
	//方法2
	f.open("ff.txt");
	while ((ch = f.get()) != EOF)//因为优先级的问题，必须加括号
	{
		cout << ch;//打印字符
	}
	f.close();
	return 0;
}
```

# 小结
以上就是C++文件操作的大部分常用内容了。挖一挖确实也不少内容了,值得总结。

