---
title: 使用高德Web服务API生成shp文件
date: 2020-05-25 18:01:51
tags: [Python, Spider]
---

[高德Web服务API](https://lbs.amap.com/api/webservice/summary)向开发者提供HTTP接口，开发者可通过这些接口使用各类型的地理数据服务，返回结果支持JSON和XML格式。利用行政区域查询以获取行政区域坐标串，生成shp文件。利用搜索POI中的关键字搜索获取兴趣点的id，再将id传入[https://www.amap.com/detail/get/detail?id=](https://www.amap.com/detail/get/detail?id=)，获得POI的详细信息，其中包括面状POI的边界坐标串，再生成shp文件。但由于反爬机制的存在，这样的方法不可以短时间提交大量请求。还有另外一个方法[AOI边界查询](https://lbs.amap.com/api/webservice/guide/api/search#t8)，在高德开放平台属于高阶服务，使用前需要申请权限。本项目开源在GitHub，地址[指路](https://github.com/WanZixin/getShp)。<!--more-->![高德POI详细信息查询--武汉大学](https://cdn.jsdelivr.net/gh/WanZixin/picture@main/20210422/高德POI详细信息查询--武汉大学.png)

## 文件说明

lib文件夹，包含两个xls文件，分别是高德地图的城市编码表和POI分类编码表。

result/district_shp文件夹，用于存储生成的行政区shp文件。

result/aoi_shp文件夹，用于存储生成的aoi的shp文件。

config.ini文件，配置文件，填写高德地图web服务的key；填写要爬取的poi的类别编码；填写爬取城市的adcode。

getPoiShp.py文件，生成指定专题、指定城市的aoi的shp文件。

getDistrictShp.py文件，生成行政区划shp文件。

gcj02togps84.py文件，高德地图使用的是GCJ-02坐标系，用此py文件转换为WGS-84坐标系。

> GCJ-02是由中国国家测绘局（G表示Guojia国家，C表示Cehui测绘，J表示Ju局）制订的地理信息系统的坐标系统。它是一种对经纬度数据的加密算法，即加入随机的偏差。国内出版的各种地图系统（包括电子形式），必须至少采用GCJ-02对地理位置进行首次加密。

## 第三方依赖

- requests
- configparser
- [pyshp](https://github.com/GeospatialPython/pyshp)

## 注意事项

- result/district_shp文件夹中，分别包含有中国各省份、湖北各城市、武汉行政区的个人地理数据库。result/aoi_shp文件夹中，分别包含有武汉市高等教育院校、武汉市公园、武汉市景点的个人地理数据库。这些数据是在ArcMap中构建的数据库，一并上传，供需要的读者下载使用。
- 每一个shp文件写入成功后，在控制台会输出提示，注意查看。
- 若想研究pyshp的用法，推荐查阅pyshp的github页面，其作者的文档很详细。笔者额外加了写入.prj文件的代码。