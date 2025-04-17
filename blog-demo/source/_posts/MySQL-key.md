---
title: MySQL关键字专题
date: 2025-02-26 10:33:54
tags:
hiddent: true
---

# ER图关系模型->数据表数据模型

## 方法一
1. 实体集 == 表
2. 1->n , 主键->外键
3. n->m, 新表，主键=(外键1 + 外键2)
4. 1-1 , 选2或3

| 表名称 | 属性 |
|--- | ---| 
| person | driver_id(PK), address,name |
| car | license(PK) , model, year | 
| accident | report-number(PK), location, date | 
| owns | owner_driver_id(PK,FK),car_license(PK,FK) | 
| participated | driver_id(PK,FK),license(PK,FK),report_number(PK,FK),damage_amount | 

| 表名称 | 属性 |
|--- | ---| 
| 职员表 | 职员编号(PK),年龄，性别,从属部门(FK) |
| 部门表 | 部门编号(PK),部门属性码 |
| 项目表 | 项目编号(PK),项目时间，属性码,主管编号(FK) |
| 项目参与表 | 职员编号(PK,FK),项目编号(PK,FK) |

| 表名称 | 属性 |
|--- | ---| 
| 书表 | 版号(PK),书名,单价 |
| 作者表 |身份证(PK) 姓名，电话,出版社名称(FK) |
| 出版社表 | 地址 ，名称(PK) |
| 书店表 | 地址(PK) ，名称 |
| 书店售卖表 | 书店地址(PK),书本出版号(PK),出售时间(PK),本数|
| 作者写书表 | 作者身份证(PK),书本版号(PK),第几作者，稿费 |