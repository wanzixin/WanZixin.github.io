---
title: Python下GDAL和OGR的使用
date: 2020-08-09 23:07:11
tags: [Python, GDAL, OGR]
---

![](https://img.shields.io/badge/python-3.8-blue.svg)

项目上需要实现两个功能，一个是shp文件（路网）与tif文件（积水）的相交，以获取各条道路的积水信息；一个是根据经纬度位置获取对应tif文件的像素值。某个项目上需要实现两个功能，一个是shp文件（路网）与tif文件（积水）的相交，以获取各条道路的积水信息；一个是根据经纬度位置获取对应tif文件的像素值。为了解决上述问题，首先考虑到ArcGIS Desktop自带的ArcPy，<!--more-->借助ArcPy可以调用标准工具箱的任意一项工具，但最大的缺点在于速度，仅仅在import语句中导入ArcPy模块，就需要花费十秒左右的时间，而且Python2.7的版本太过远古。如果仅仅是做单独脚本，或者是编写ArcGIS的脚本工具，ArcPy仍然是最好的选择。但要考虑到作为Web服务，时延太长就很不合理了。因而，选择了Python 3.8环境下的gdal和ogr，分别用于处理栅格数据和矢量数据；使用flask和flasgger，分别用于微型网络框架和Swagger-UI的使用。

## GDAL和OGR的安装

使用pip安装速度上很慢，使用国内镜像源速度快很多，共有两种方式在pip安装模块时使用镜像源。

- 临时修改。使用pip时加上`-i https://pypi.tuna.tsinghua.edu.cn/simple`，例如：`pip install gdal -i https://pypi.tuna.tsinghua.edu.cn/simple`即使用了清华的镜像源。

- 永久修改。Windows平台下的用户目录新建pip文件夹，再新建pip.ini配置文件，写入如下内容保存，以后的每次pip安装模块都会使用阿里云的镜像源了。

  ```
  [global]
  index-url = http://mirrors.aliyun.com/pypi/simple/
  [install]
  trusted-host = mirrors.aliyun.com
  ```

  使用`pip install gdal`命令即可，因为gdal和ogr都被集成在了osgeo中，ogr也会一并下载。使用`from osgeo import gdal, ogr`即可导入。


## GDAL和OGR的使用

Python下GDAL的官方文档，地址为[https://gdal.org/python](https://gdal.org/python)，以下将本次任务中使用到的部分做个简单的记录。

### 栅格数据的读取

```python
from osgeo import gdal
def readRaster(rasterPath):
    dataset = gdal.Open(rasterPath)
    #波段从1开始计数，这行语句获取栅格数据集的第二个波段
    band2 = dataset.GetRasterBand(2) 
    #获取栅格文件的仿射信息，是一个包含六个元素的数组,第三个元素和第五个元素带代表旋转信息，一般都为0
    geotransform = dataset.GetGeoTransform() 
    originX = geotransform[0] #左上角经度
    originY = geotransform[3] #左上角纬度
    pixelWidth = geotransform[1] #像元宽度
    pixelHeight = geotransform[5] #像元高度
```

### 矢量数据的读取

```python
from osgeo import ogr
def readShp(shpPath):
    driver = ogr.GetDriverByName('ESRI Shapefile')
    ds = driver.Open(shppath, 0) # read only
    layer = ds.GetLayer(0) #获取数据集的第一个图层
    ds_sr = layer.GetSpatialRef() #获取数据集的空间参考
    
    #获取图层的每一个要素与属性
    for feature in layer:
        geomtry = feature.GetGeometryRef()
        FID = feature.GetField('FID')
```

### 栅格矢量化

```python
def raster_to_shapefile(self,rasterpath):
    driver = ogr.GetDriverByName('ESRI Shapefile')
    src_dataset = gdal.Open(rasterpath)

    # get band2 in tif
    src_band2 = src_dataset.GetRasterBand(2)
    maskband = src_band2.GetMaskBand() #掩膜波段图层

    target_sr = osr.SpatialReference()
    target_sr.ImportFromEPSG(4326)

    dst_layerName = "byproduct/Polyginize_Stuff"
    dst_dateset = driver.CreateDataSource(dst_layerName + ".shp")
    dst_layer = dst_dateset.CreateLayer(dst_layerName, srs = target_sr)

    # DN代表地物反射率，这段代码在属性表创建了DN列，值即为波段2的值
    dst_fieldName = 'DN'
    fd = ogr.FieldDefn(dst_fieldName, ogr.OFTInteger)
    dst_layer.CreateField(fd)

    # 第四个参数是需要将DN值写入矢量字段的索引，应该不包括FID和Shape
    gdal.Polygonize(src_band2, maskband, dst_layer, 0, [], callback = None) 
    return dst_layerName
```

### 栅格数据中根据仿射信息将经纬度转换为行列号

```python
def getRGBByLnglat(self,lnglat,time):
    maxTifpath = self.getMaxTifpathByTime(time)
    dataset = gdal.Open(maxTifpath)

    geotransform = dataset.GetGeoTransform()
    originX = geotransform[0]
    originY = geotransform[3]
    pixelWidth = geotransform[1]
    pixelHeight = geotransform[5]

    xOffset = int((lnglat[0]-originX)/pixelWidth)
    yOffset = int((lnglat[1]-originY)/pixelHeight)

    rgb = []
    for i in range(1,4):
        band = dataset.GetRasterBand(i)
        # gridcode is a two-dimensional array with only an element
        gridcode = band.ReadAsArray(xOffset,yOffset,1,1) 
        rgb.append(gridcode[0][0])
    return str(rgb[0]) + ',' + str(rgb[1]) + ',' + str(rgb[2])
```

### 矢量数据的相交Intersect

在ogr中，读入一个矢量文件得到的是数据集，数据集的下一维度是图层（一般只有一个图层），图层的下一维度是要素，要素包含几何图形和属性。相交有两个维度的判断办法，一种是Layer（图层）的相交，一种是Geometry（几何图形）的相交。基于Layer的相交仅有一个函数Intersection，无论是否存在相交，返回的都是处理结果；而基于Geometry的相交则有三个函数，分别是Intersection，Intersect，Intersects，不同点在于后两个函数返回的是bool值，即只要几何图形的envelope重叠就返回True，否则返回False。

```python
def getDataSetFromShpPath(self, shppath):
    driver = ogr.GetDriverByName('ESRI Shapefile')
    ds = driver.Open(shppath, 0) # read only
    if ds is None:
        print(shppath + 'is wrong.')
        return
    return ds

def intersectTwoShp(roadShp,pondshp):
    pond_ds = getDataSetFromShpPath(pondshp)
    road_ds = getDataSetFromShpPath(roadShp)

    pond_layer = pond_ds.GetLayer(0)
    road_layer = road_ds.GetLayer(0)

    # 用于空间参考的一致性变换
    src_sr = road_layer.GetSpatialRef()
    target_sr = pond_layer.GetSpatialRef()
    ct = osr.CoordinateTransformation(src_sr,target_sr)

    infos = [] # 构建的相交信息
    i = 0 # 总循环次数
    j = 0 # 相交次数
    for pond_feature in pond_layer:
        pond_geom = pond_feature.GetGeometryRef()
        for road_feature in road_layer:
            i += 1
            road_geom = road_feature.GetGeometryRef()
            road_geom.Transform(ct)
            intersect = pond_geom.Intersect(road_geom)
            if intersect:
                # 以下为提取信息，可以根据需要做替换或是修改
                j += 1
                info = {}
                road_id = road_feature.GetField('road_id')
                name = road_feature.GetField('name')
                gridCode = pond_feature.GetField('DN')
                g_legend = self._getLegend(gridCode)
                g_code = self._getLevelType(g_legend)

                if g_legend < 0.1:
                    continue

                info["level"] = g_legend
                info["levelType"] = g_code
                info["name"] = name
                info["road_id"] = road_id
                infos.append(info)
	# print(i,j)
    infos_new = _reviseInfos(infos)
    return infos_new

def _reviseInfos(infos):
    # 按road_id排序
    infos.sort(key=lambda x: x['road_id'])
    # 设置一个哨兵字典,初值为列表第一个元素
    info_flag = {}
    info_flag = infos[0]
    # 定义一个新列表，作返回值
    infos_new = []
    infos_new.append(info_flag)
    # 从列表第二个元素开始
    i = 1 
    while i < len(infos):
        if(info_flag['road_id'] == infos[i]['road_id']):
             # info_max是待返回列表的最后一个元素,定义一个info_max存储road_id相等中level最大的字典元素
            info_max = infos_new[-1]
            if(info_max['level'] < infos[i]['level']):
                info_max = infos[i]
                infos_new.pop(-1)
                infos_new.append(info_max)
                else:
                    infos_new.append(infos[i])

             info_flag = infos[i]
             i += 1

    return infos_new
```

### 图层空间参考的修改

这个部分在2.5节中有应用，在这里主要是在Geometry级别的操作，在DataSet类中有Set SpatialRef函数可以实现数据集的空间参考修改。

```python
from osgeo import ogr,osr
def modifySR(srcShpPath, targetShpPath)
    road_ds = self.getDataSetFromShpPath(srcShpPath)
    pond_ds = self.getDataSetFromShpPath(targetShpPath)

    road_layer = road_ds.GetLayer(0)
    pond_layer = pond_ds.GetLayer(0)

    src_sr = road_layer.GetSpatialRef()
    target_sr = pond_layer.GetSpatialRef()
    ct = osr.CoordinateTransformation(src_sr,target_sr)
    
    # Geometry级别的修改空间参考，但图层本身并没有被修改
    for feature in road_layer:
        geom = feature.GetGeometryReF()
        geom.Transform(ct)
```

## 结语

最终的版本还有一些可以修改的地方，不是尽善尽美的，主要因为对gdal和ogr本身的类结构不熟悉。

在使用过程中参考了很多CSDN的博客，起了个入门的作用。如果真想要深入的使用gdal和ogr这么强大的工具，应该研究官方给出的文档，关键函数都有详细的说明，查看传入参数、返回值和函数名基本上能猜个八九不离十。另外，类图的阅读非常有必要，这可以帮助快速的掌握模块的组织架构，总的来说有所收获。

看得开心！