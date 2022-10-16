---
title: "Superset 实践及踩坑记录"
date: 2022-10-16T19:05:56+08:00
description: "从美观程度来说我更喜欢 ECharts，下次换 Echarts。"
tags: [Superset]
featured_image: "/superset_chart/featured_image.jpg"
draft: false
---

## Superset 怎么用
### 整体布局
- 首先整体的布局分为 Databases、Datasets、Charts、Dashboards 等几个部分。 

<center>

![](/superset_chart/home.png)  

<center>

### Databases
- `Select a database to connect`  
  填写需要连接的数据库，数据库连接配置信息可以参考[官方文档](https://superset.apache.org/docs/databases/installing-database-drivers/)填写。

<center>

![](/superset_chart/database.png)  

<center>

### Datasets
- `Add dataset`  
  添加连接的数据库中的具体的表，数据库连接成功后下拉即可点击选择需要的表。

<center>

![](/superset_chart/dataset.png)  

<center>

### Charts
- `Create a new chart`  
  从 Datasets 中选择一张需要可视化的表。

<center>

![](/superset_chart/chart.png)  

<center>

  - 根据实际的需求选择图表的类型，配置详细的展示信息。例如绘制地图类型的图表，展现不同省份的下单次数。

<center>

![](/superset_chart/map.png)  

<center>

- **踩坑问题**：
  这里就出现了一个非常坑的地方，绘制地图所用的表提供了两种编码用于标识省份，`iso_code`和`iso_code_3166_2`，用哪个都可以。  

<center>

![](/superset_chart/table.png)  

<center>

  但在 Superset 中选择第一种编码则只能显示港澳台地区的数据，其他地区无法显示；而选择第二种编码则港澳台地区无法显示，其他地区可以显示。一番搜索发现是 Superset 在港澳台和其他地区提供的编码方式不一致导致的问题。
- **解决方法**：
  - 按照[解决方法](https://blog.csdn.net/csncd/article/details/124559112)，进入 Superset 的安装目录下的目录。  
  `/lib/python3.8/site-packages/superset/static/assets/`
  - 找到对应的地图文件，`.geojson` 文件有很多，过滤找出文件，  
  `grep -rl 'Beijing'`  
  并修改，将编码方式修改一致即可。

### Dashboards
- 编辑布局，添加图表即可完成 Dashboard 绘制。
- 也可以对 Dashboards 进行刷新，根据需要设置`REFRESH FREQUENCY`。

<center>

![](/superset_chart/dashboard.png)  

<center>

## 参考链接
[article_124559112](https://blog.csdn.net/csncd/article/details/124559112)