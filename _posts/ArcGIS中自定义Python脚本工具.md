---
title: ArcGIS中自定义Python脚本工具
date: 2020-07-27 23:24:41
tags: [ArcGIS, Python]
---

ArcGIS中的ArcToolBox包含许多工具，涉及网络分析、路径分析、格式转换等诸多领域，再结合详尽的帮助文档，可以快速掌握上手一个工具的使用。当然，ArcGIS也提供自定义脚本工具，以满足用户的定制化需求。这篇文章旨在介绍ArcGIS中Python脚本工具的编写与使用，以及其中遇到的问题，以飨读者。在文章末尾给出一个自定义脚本工具的源代码，供读者参考。笔者的ArcGIS版本为10.6，自带的Python版本为2.7.14，与目前主流的python3略有差别，需留心注意。<!--more-->

## ArcPy简介

ArcPy 是以 arcgisscripting 模块为基础并继承了 arcgisscripting 功能进而构建而成的站点包。目的是为以实用高效的方式通过 Python  执行地理数据分析、数据转换、数据管理和地图自动化创建基础。在导入 ArcPy 之后，可以运行随 ArcGIS 安装的标准工具箱中的所有地理处理工具，包括有分析工具箱、制图工具箱、转换工具箱、数据管理工具箱、编辑工具箱、地理编码工具箱、线性参考工具箱、多维工具箱和空间统计工具箱。

ArcPy由一系列模块组成，包括：

|                    模块中文名                    |    模块名     |                             描述                             |
| :----------------------------------------------: | :-----------: | :----------------------------------------------------------: |
|           数据访问模块（Data Access）            |   arcpy.da    | 数据访问模块 (arcpy.da) 是一个用于处理数据的 Python  模块。通过它可控制编辑会话、编辑操作、改进的游标支持（包括更快的性能）、表和要素类与 NumPy  数组之间相互转换的函数以及对版本化、复本、属性域和子类型工作流的支持。 |
|               制图模块（Mapping）                | arcpy.mapping | 主要是用于操作现有地图文档 (.mxd) 和图层文件 (.lyr) 的内容。此外，还提供自动执行导出和打印的功能。Arcpy.mapping 可用于自动执行地图生产；它扩展了数据驱动页面的功能，同时，因其包含导出至  PDF 文档、创建和管理 PDF 文档的函数，而为构建完整地图册所必需。最后，可将  arcpy.mapping 脚本发布为地理处理服务，并将脚本功能提供给 Web 应用程序。 |
| ArcGIS空间分析扩展模块（ArcGIS Spatial Analyst） |   arcpy.sa    | Spatial Analyst 模块是用于分析栅格数据的 Python 模块，该模块在进行分析时将使用 ArcGIS Spatial Analyst  扩展模块 提供的功能。借助该模块可访问 Spatial Analyst 工具箱中提供的所有地理处理工具以及其他帮助程序函数和类 |
| ArcGIS网络分析扩展模块（ArcGIS Network Analyst） |   arcpy.na    | arcpy.na 是用于使用 ArcGIS Network Analyst  扩展模块 提供的网络分析功能的 Python 模块。通过它可访问 Network  Analyst 工具箱中提供的所有地理处理工具以及允许您通过 Python 使 Network Analys  工作流自动化的其他帮助程序函数和类。 |
|                     时间模块                     |  arcpy.time   | arcpy.time 模块包含在 Python 中处理时间增量和时区时会用到的类、方法以及属性。 |

## 自定义工具箱和脚本工具的步骤

1. ArcMap中点开右侧“目录”，展开“工具箱”，右键单击“我的工具箱”，选择“新建”工具箱，为工具箱命名；
2. 右键单击新工具箱，选择“添加”脚本，进入脚本工具编辑页面；
3. “名称”用于系统内部处理，全英文且不能包含空格、连接线等非法字符；“标签”是脚本工具的显示名称，便于阅读；勾选“存储相对路径名”，以便制作好的工具箱分享出去可以直接使用；“脚本文件”选择对应的.py文件；最后，设置脚本中所需要的参数，并指定好类型。

## 编写py文件

### 工具参数的传入

比如想要编写一个按掩膜提取的脚本工具，需要输入待提取的栅格图层和作为掩膜的矢量图层，输出提取完成的栅格图层。普通.py脚本文件中，参数的传递非常容易，但是如何获取输入参数提供给ArcGIS的脚本工具呢？这是需要使用arcpy.GetParameterAsText函数，按顺序获取输入参数，并赋给特定变量以供使用。

```python
import arcpy

yp_road_shp = arcpy.GetParameterAsText(0)
pond_tif = arcpy.GetParameterAsText(1) 
jsonPath = arcpy.GetParameterAsText(2)
```

### 自定义进度对话框中的消息

使用arcpy.AddMessage函数向脚本工具的消息中添加自定义消息，仅需要一个字符串类型的参数做文本输出。用于自定义写入消息的四个ArcPy函数如下所示，可根据需要使用相应的函数。其中，值得注意的一点是，message需为全英文，否则单独运行无误，但作为脚本工具在ArcGIS中使用会产生意想不到的错误。

| 函数名                                                       |                        描述                        |
| :----------------------------------------------------------- | :------------------------------------------------: |
| AddMessage(message)                                          |          用于一般信息性消息（严重性 = 0）          |
| AddWarning(message)                                          |             用于警告消息（严重性 = 1）             |
| AddError(message)                                            |             用于错误消息（严重性 = 2）             |
| AddIDMessage(message_type, messageID, add_argument1, add_arrgument2) | 用于错误和警告（由 message_type 参数确定严重性。） |

### 添加描述信息

右键单击自定义的脚本工具，选择“项目描述”，单击左上角的“编辑”，即可为脚本工具和参数添加说明信息，以便工具可读性的提高。

## 杨浦区洪水风险样例的脚本工具源码

上海市杨浦区，路网图层与洪水图层叠加，输出road_id（道路编码），name（道路名称），level（道路洪水等级），levelType（洪水等级范围）到json文件。

```python
'''# -*- coding: utf-8 -*-'''
# python version: 2.7.14
# Author: Wan Zixin
# Date: 2020.07.23

import arcpy
import json
import codecs
import os

def TifBand2ToShp(pond_tif):
    rasterTif_band2 = arcpy.MakeRasterLayer_management(pond_tif, "rasterTif_band2", "#", band_index="2") # 波段索引从1开始计
    rasterTif_band2_shp_name = byproductDir+'\\'+os.path.basename(pond_tif).split('.')[0]+'.shp'
    rasterTif_band2_shp = arcpy.RasterToPolygon_conversion(rasterTif_band2, rasterTif_band2_shp_name, "NO_SIMPLIFY")
    #arcpy.AddMessage("Step1 tif文件band2转为shp文件已完成。")
    arcpy.AddMessage("Step1 tif's ban2 to shapefile is done.")
    return rasterTif_band2_shp_name

def InteractTwoShps(shp1,shp2,jsonPath):
    infos = []
    interact_shp_name = byproductDir+'/'+"myInteract_"+ os.path.basename(shp1)
    interact_shp = arcpy.Intersect_analysis([shp1, shp2], interact_shp_name)
    # arcpy.AddMessage("Step2 shp文件叠加操作已完成。")
    arcpy.AddMessage ('Step2 shapefile interact is done.')

    
    fields = ['road_id', 'name', 'gridcode']
    with arcpy.da.SearchCursor(interact_shp, fields) as cursor:
        for row in cursor:
            info={}
            info["road_id"]=row[0];
            info["name"]=row[1];

            g_legend = getLegend(row[2])
            info["level"] = g_legend;
            g_code = getLevelType(g_legend)
            info["levelType"] = g_code;            
            infos.append(info)

    infos_new = []
    infos_new = reviseInfos(infos);
    jsonData = json.dumps(infos_new, ensure_ascii = False, encoding="gb2312")
    f = codecs.open(jsonPath, 'w',encoding='utf-8')
    f.write(jsonData)
    f.close()
    # arcpy.AddMessage("Step3 JSON文件生成已完成。JSON文件路径为： " + jsonPath)
    arcpy.AddMessage("Step3 JSON file is created. JSON file path is  " + jsonPath)

def reviseInfos(infos):
    # 按road_id排序
    infos.sort(key=lambda x: x['road_id'])
    # 设置一个哨兵字典,初值为列表第一个元素
    info_flag = {}
    info_flag = infos[0]
    # 定义一个新列表，作返回值
    global infos_new
    infos_new = []
    infos_new.append(info_flag)
    # 定义一个info_max存储road_id相等中level最大的字典元素
    # info_max = {}
    # 从列表第二个元素开始
    global i 
    i = 1 
    while i < len(infos):
        if(info_flag['road_id'] == infos[i]['road_id']):
            info_max = infos_new[-1] # info_max是待返回列表的最后一个元素,定义一个info_max存储road_id相等中level最大的字典元素
            if(info_max['level'] < infos[i]['level']):
                info_max = infos[i]
                infos_new.pop(-1)
                infos_new.append(info_max)
        else:
            infos_new.append(infos[i])

        info_flag = infos[i]
        i += 1

        # i_new = i
        # i_new += 1
        # i = i_new # python2中没有自增和自减

    return infos_new


def getLegend(g):
    if g == 255:
        return 0
    elif g == 203:
        return 0.1
    elif g == 167:
        return 0.2
    elif g == 135:
        return 0.3
    elif g == 96:
        return 0.4
    elif g == 64:
        return 0.5
    elif g == 36:
        return 1.0
    elif g == 0:
        return 1.1
    else:
        # print "g的数值有误"
        return None

def getLevelType(g_legend):
    if g_legend < 0.1:
        return "<0.1"
    elif g_legend >= 0.1 and g_legend < 0.3:
        return "0.1-0.3"
    elif g_legend >= 0.3 and g_legend <= 0.5:
        return "0.3-0.5"
    elif g_legend >0.5:
        return ">0.5"
    else:
        # print "g_legend数值有误"
        return None


byproductDir = arcpy.GetParameterAsText(3)
if not os.path.exists(byproductDir):
    os.mkdir(byproductDir)
# byproductDir = 'E:/PondRisk/ArcPyHand/byproduct'

def main():
    arcpy.env.overwriteOutput = True # 允许覆盖已有文件,注意提醒用户

    yp_road_shp = arcpy.GetParameterAsText(0)
    pond_tif = arcpy.GetParameterAsText(1) # tif数据确保包含有两个波段及以上
    jsonPath = arcpy.GetParameterAsText(2)

    #pond_tif = 'E:/PondRisk/ArcPyHand/lib/workspace/YP_20190810000000_max.tif'
    #yp_road_shp = 'E:/PondRisk/ArcPyHand/lib/workspace/YP_roads.shp'
    #jsonPath = 'C:/Users/user/Desktop/info.json'
    
    tif_band2_shp_path = TifBand2ToShp(pond_tif)
    InteractTwoShps(tif_band2_shp_path, yp_road_shp,jsonPath)

if __name__ == "__main__":
    main()
```

