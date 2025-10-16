# 

## Windows本地搭建
1. 从Github上下载并解压 SQLi-Labs 到 Web 根目录（例如 `D:\learning\phpstudy\phpstudy_pro\WWW`）。

2. Fatal error: Uncaught Error: Call to undefined function mysql_connect() in……

   原因：mysql_connect() 函数在 PHP 7 以后就被彻底移除了，而小皮使用的是PHP 7

   解决：将PHP版本更换为支持mysql_connect()的PHP 5

3. Failed to connect to MySQL……

   原因：未配置数据库密码

   解决：目录打开sql-connection下的db-creds.inc 补充数据库密码

   ![](/assets/images/sql-connect.png)
