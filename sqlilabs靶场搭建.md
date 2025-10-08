# SQLi-Labs 渗透测试与防御报告

> 作者：zhenyu wu  
> 项目地址：[https://yourname.github.io/sqlilabs-report](https://yourname.github.io/sqlilabs-report)  
> 日期：2025-10  

本报告记录了本人在 SQLi-Labs 靶场进行 **手工 SQL 注入攻击与防御** 的全过程，包括环境搭建、漏洞分析、攻击步骤、PoC 演示、修复建议及总结。  
旨在展示对 **SQL 注入机制** 的理解与复现能力，供学习与面试展示使用。

---

## 📑 目录
- [1. 项目简介]()
- [2. 环境搭建](#2-环境搭建)
- [3. 靶场关卡分析]()
  - [3.1 Less-1：基于报错注入](# 3.1 Less-1：基于报错注入)
  - [3.2 Less-2：联合查询注入](#32-less2联合查询注入)
  - [3.3 ...后续关卡](#33-后续关卡)
- [4. 常用 Payload 与原理说明](#4-常用-payload-与原理说明)
- [5. 防御与加固](#5-防御与加固)
- [6. 总结与收获](#6-总结与收获)
- [7. 附录与参考资料](#7-附录与参考资料)

---

## 1. 项目简介
- **目标**：复现 SQL 注入漏洞原理，掌握攻击流程与防御策略  
- **工具环境**：
  - PHP 5.x / MySQL 5.x
  - Apache2
  - Burp Suite、Firefox、Kali Linux
- **实验范围**：SQLi-Labs 靶场 1–20 关  
- **产出**：完整渗透文档（本报告）、可运行 PoC、截图证据

---

## 2. 环境搭建
1. 下载 SQLi-Labs 并部署到本地 `phpstudy`/`XAMPP`
2. 数据库初始化：
    ```bash
    http://127.0.0.1/sqli-labs/setup-db.php
    ```
3. 常见报错与解决
    - `Access denied for user 'root'@'localhost'` → 检查 `db-creds.inc`
4. 环境架构图：

![环境架构图](images/env-architecture.png)

---

## 3. 靶场关卡分析

### 3.1 Less-1：基于报错注入
- **源码关键片段**
```php
$sql = "SELECT * FROM users WHERE id='$id' LIMIT 0,1";
```

- **漏洞原因**：参数 `$id` 未做过滤，直接拼接 SQL，导致报错回显。
- **攻击流程**：
  1. 输入 `?id=1'` → 触发语法错误
  2. 通过 `ORDER BY` 判断列数
  3. 利用 `UNION SELECT` 报错注入
- **演示截图：**





- **修复建议**：
  - 使用 **预编译语句（Prepared Statement）**
  - 严格验证输入为整数类型

