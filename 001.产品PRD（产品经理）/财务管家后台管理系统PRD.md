# 财务管家后台管理系统 PRD

## 1. 项目概述

**项目名称**：财务管家后台管理系统

**项目目的**：为管理员提供查看和管理财务管家应用数据的后台系统，监控用户行为、统计数据等。

**前提约束**：
- 允许新增1个管理员表 `admins`，用于存储超级管理员账户
- 禁止修改现有数据库及数据库表结构
- 所有数据通过现有前端API的数据库获取

---

## 2. 现有系统分析

### 2.1 数据库表结构

| 表名 | 说明 |
|------|------|
| users | 用户表 |
| ledgers | 账本表 |
| accounts | 账户表 |
| account_types | 账户类型表 |
| categories | 分类表 |
| category_types | 分类类型表（支出/收入） |
| transactions | 交易记录表 |
| admins | 管理员表（新建，仅用于后台登录） |

### 2.2 现有API端点

| 端点 | 方法 | 功能 |
|------|------|------|
| /api/register | POST | 用户注册 |
| /api/login | POST | 用户登录 |
| /api/verify | GET | 验证Token |
| /api/user | GET/PUT | 获取/更新用户信息 |
| /api/accounts | GET/POST | 获取/添加账户 |
| /api/accounts/:id | PUT/DELETE | 更新/删除账户 |
| /api/categories | GET/POST | 获取/添加分类 |
| /api/categories/:id | PUT/DELETE | 更新/删除分类 |
| /api/transactions | GET/POST | 获取/添加交易记录 |
| /api/transactions/:id | DELETE | 删除交易记录 |
| /api/home | GET | 获取首页数据 |
| /api/statistics | GET | 获取统计数据 |
| /api/export | GET | 导出数据 |
| /api/account-types | GET | 获取账户类型 |
| /api/category-types | GET | 获取分类类型 |

---

## 3. 功能需求列表

### 3.1 超级管理员登录

- 管理员账号登录（用户名+密码）
- 使用独立的 `admins` 表存储管理员账户（可新建，仅此表）
- 登录后进入管理后台首页

### 3.2 数据概览（首页仪表盘）

| 功能 | 说明 |
|------|------|
| 总用户数 | 当前系统注册的用户总数 |
| 今日新增用户 | 今日注册的用户数量 |
| 总交易笔数 | 所有用户的交易记录总数 |
| 本月总收入 | 所有用户本月收入合计 |
| 本月总支出 | 所有用户本月支出合计 |
| 最近7天趋势 | 每日新增用户数折线图 |

### 3.3 用户管理

| 功能 | 说明 |
|------|------|
| 用户列表 | 分页展示所有用户（ID、用户名、昵称、注册时间、最后登录） |
| 用户搜索 | 按用户名/昵称搜索 |
| 用户详情 | 查看单个用户的详细信息 |
| 用户账户 | 查看该用户下的所有账户 |
| 用户交易记录 | 查看该用户的所有交易明细 |

### 3.4 交易记录管理

| 功能 | 说明 |
|------|------|
| 交易列表 | 分页展示所有交易记录（时间、用户、类型、分类、账户、金额） |
| 交易筛选 | 按日期范围、交易类型（收入/支出）、用户筛选 |
| 交易详情 | 查看单条交易的详细信息 |

### 3.5 数据统计

| 功能 | 说明 |
|------|------|
| 全局收入支出 | 按年度/月度统计所有用户的总收入和总支出 |
| 分类占比 | 所有用户支出按分类的占比统计（饼图） |
| 月度趋势 | 12个月的收入支出趋势图 |
| 用户排行 | 支出最多/收入最多的用户排行 |

### 3.6 分类管理（只读）

| 功能 | 说明 |
|------|------|
| 支出分类列表 | 查看系统现有的支出分类 |
| 收入分类列表 | 查看系统现有的收入分类 |

### 3.7 账户类型管理（只读）

| 功能 | 说明 |
|------|------|
| 账户类型列表 | 查看系统现有的账户类型 |

---

## 4. 页面结构

```
/admin-login          # 登录页
/admin-dashboard     # 数据概览（首页）
/admin-users        # 用户管理
/ admin-users/:id   # 用户详情
/admin-transactions # 交易记录
/admin-statistics   # 数据统计
/admin-categories  # 分类管理（只读）
/ admin-categories/expense  # 支出分类
/ admin-categories/income    # 收入分类
/admin-account-types  # 账户类型（只读）
```

---

## 5. 技术方案

### 5.1 实现方式

由于禁止修改数据库，采用以下方式获取数据：

1. **新建管理员表 `admins`**：仅新建此表用于存储超级管理员账户
2. **新建管理后台数据库连接**：直接连接现有数据库查询
3. **不依赖现有API**：管理后台需要查看所有用户数据，独立查询更灵活
4. **只读操作**：不对原有数据进行增删改，只做查询和统计

### 5.2 admins表结构

```sql
CREATE TABLE admins (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  username VARCHAR(50) NOT NULL UNIQUE,
  password VARCHAR(255) NOT NULL,
  nickname VARCHAR(50),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 5.3 需要新增的API

后台管理系统需要新增独立的API端点（可与前端API共存）：

| 端点 | 方法 | 功能 |
|------|------|------|
| /api/admin/login | POST | 管理员登录 |
| /api/admin/users | GET | 用户列表 |
| /api/admin/users/:id | GET | 用户详情 |
| /api/admin/users/:id/transactions | GET | 用户交易记录 |
| /api/admin/transactions | GET | 交易列表 |
| /api/admin/statistics/overview | GET | 数据概览 |
| /api/admin/statistics/trend | GET | 趋势数据 |
| /api/admin/statistics/ranking | GET | 用户排行 |
| /api/admin/categories | GET | 分类列表 |
| /api/admin/account-types | GET | 账户类型列表 |

---

## 6. 非功能需求

- 响应速度：首屏加载 < 2秒
- 分页：每页显示20条，支持分页查询
- 安全：管理员密码使用BCrypt加密存储

---

## 7. 待确认事项

1. 是否需要导出功能？（Excel导出统计数据）
    否
2. 是否需要定时任务？（自动生成统计报表）
    否


## 8. 数据库admins的创建
目前已经按照 “5.2 admins表结构” 的创建完毕。