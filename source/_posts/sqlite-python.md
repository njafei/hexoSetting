---
title: SQLite-Python学习
date: 2017-04-28 11:05:29
tags: 
	 - sqlite
	 - python
categories: 数据库
---

> 注： 本文默认有基本的数据库和SQLite知识


最近在做SQLite数据库相关的自动化任务，所以学习了下`phthon`中如何使用`SQLite`，网上的教程如：[runoob](http://www.runoob.com/sqlite/sqlite-python.html)、 [docs.python](https://docs.python.org/2/library/sqlite3.html)等的说明不太详细，而且很多细节和demo都写的比较粗，遂有此文。

## 连接数据库

首先我们要连接数据库，要不然没法操作。`python`2.5之后，内置了`sqlite3`，所以我们可以直接用内置的命令。

介绍第一个命令

```
sqlite3.connect(database [,timeout ,other optional arguments]);
```

 - database： 数据库的path，如果不给，就会创建一个。如果给的“:memory:”，则会创建一个建在RAM上的数据库。
 - timeout: 默认是5s，当数据库锁定的时候，最长的等待时间
 - optional arguments： 暂时没查到有哪些参数可选

 比如我们想要新建一个数据库：
 
```
#!/usr/bin/python

import sqlite3

newData = sqlite3.connect('new.sqlite')
print "open new Database successfully";

```

这样，就建好了一个名为new的数据库，并连接上。

如果想建一个RAM上面的数据库，就使用`:memory`:

```
newData = sqlite3.connect(':memory:')

```

还可以设置10s的`timeout`：

```
newData = sqlite3.connect(':memory:'[,10]);

```

关闭数据库连接就比较简单了：

```
newData.close();
```

# 数据库执行`SQL`语句
#### 执行简单语句
这里主要有两种方式：

 - 数据库直接执行 `connection.execute()`
 - 通过cursor执行 `cursor.execute`

举个例子，我们希望建一个如下的数据表并插入数据：user

| id  | name | money |
| --- | ---  | ----- |
| 1 | jack | 100 |
| 1 | rose | 200 |


```
#create table
newData.execute("create table user (id, name, money)");

#insert data
newData.execute("insert into user values (1,'jack',100)");
newData.execute("insert into user values (2,'rose',200)");

#打印看下是否成功
users = newData.execute("select * from user");
print('row in new data');
for row in users:
	print row[0],
	print row[1]

newData.commit();
newData.close();
```

这里面有个注意点：一定要`commit` 和 `close` ，否则不会保存。

> commit: 该方法提交当前的事务。如果您未调用该方法，那么自您上一次调用 commit() 以来所做的任何动作对其他数据库连接来说是不可见的。

这里的`execute`也可以用cursor执行：

```
#获取cursor
c = localData.cursor()

#cursor执行语句
c.execute("select * from user");
```

#### 执行带参数的语句
如果我们有用到python里声明的变量，就只能通过`cursor`了。比如想要计算数组的和，放到amy的money中：

这里先介绍下有参数的命令：

```
cursor.execute(sql, seq_of_parameters)

cursor.execute("insert into people values (?, ?)", (who, age))
```
execute只接收两个参数，sql和param，sql中的变量名，都用?或者命名代替，然后依次写在参数中。记住，一定要放到一个括号中，否则会认为有三个参数，会报错：`TypeError: function takes at most 2 arguments (3 given)`

实现代码如下：

```
#获取cursor
c = localData.cursor()

#计算总和
name = 'amy';
total = money + money2 + money3;

#写入rose的钱中
c.execute("inset into user value (3, ?, ?)",(name,total));

#or 
#c.execute("inset into user value (3, name, money)",(name,total));

```

> 注意：这里的connect也可以执行带参数的语句，但是其实都是生成临时cursor执行，所以这两种方法本质是一样的，下文中遇到这两个，只会写一个作为示范。

#### 处理多组数据
如果我们希望针对一组数据，执行同样的sql呢？当然可以`for: in` 来挨个执行，也可以使用批量的数据处理方法：

 - `cursor.executemany(sql, seq_of_parameters)`
 - `connection.executemany(sql[, parameters])` 最终还是通过cursor执行

比如刚才的例子，我们不用一条一条数据插入，而是直接插入一个数组：

```
newUsers = [(1,'jack',100),(2,'rose',100)];
newData.executemany("insert into user values(?,?,?)",newUsers);
```
#### 处理多条语句
如果是想连续执行多条语句如何呢？

 - cursor.executescript(sql_script)
 - connection.executescript(sql_script)

比如：

```
sqlscript = """insert into user values(1,'jack',100);
			   insert into user values(2,'rose',200);"""

newData.executescript(sqlscript);
```

# 操作cursor
这里的cursor可以理解为我们变成时候的光标，光标在哪里，我们当前的操作点和关注点就在哪里。而当我们多次操作的时候，可能想要获取当前光标所在的位置。

还是举个例子：

| id  | name | money |
| --- | ---  | ----- |
| 1 | jack | 100 |
| 2 | rose | 200 |
| 3 | john | 200 |

我希望查到谁的money是200，而且我想获得当前的光标坐在的位置：

```
c = newData.cursor();
c.execute("select * from user where money = 200");
#查第一条
c.fetchone();//(2,rose,200)

# 查前两条
c.fetchmany(2);// (2,rose,200),(3,john,200)

# 查所有的
c.fetchall();//(2,rose,200),(3,john,200)

#如果没有结果返回none
c.execute("select * from user where money = 300");
c.fetchone();//none
```

说实话，刚开始看我是很疑惑的,看起来，这个光标的作用和我直接用一个数组赋值貌似没有什么区别，难道是这样比较方便来获取最后的结果么？

后来问了大神，发现原来光标是这样子的，fetch了一条，光标会下移，比如：

```
c.execute("select * from user where money = 200");
print c.fetchone();//(2,rose, 200)
print c.fetchone();//(3,john,200)
print c.fetchone();//none
```
再结合一些操作光标的操作，就可以实现一些比较复杂的用法了。


# 操作相关
数据库的操作和git还是有点相似的，都是有提交，回滚等，我们依次介绍下：

 - connection.commit() 提交修改，否则不会保存
 - connection.rollback() 回滚上次的commit
 - connection.close() 关闭数据库连接
 - connection.total_changes() 查看修改的总行数





