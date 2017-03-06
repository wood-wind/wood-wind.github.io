---
layout: post
title: Arcgis自动建库(Python)
date: 2016-05-11
categories: blog
tags: [技术]
description: Python自动建库
---

2016年05月11日22:10:38


之前用Java读excel生成py文件的方式实现的建库，但还是觉得比较麻烦，后来经过改进实现了用python读取excel直接建库。

以下为实现过程

### python实现思路

* 利用python对excel进行操作，对模板化的excel表数据进行读取然后直接对arcgis进行操作
* 直接写进利用SDE连接oracle的arcgis
* 同时对写进oracle的表字段写进注释

### python源代码

**head.py主文件**

```python
# -*- coding: UTF-8 -*-
import arcpy
import xlrd,sys,os
import to_oracle
import parameter
reload(sys)
sys.setdefaultencoding('utf-8')

from arcpy import env

def main():
    home=os.getcwd() #得到本文件路径
    homedir=home.replace('\\','/') #将\替换为程序能读取的/
    data=to_oracle.open_excel(file=home+'\sourcefile.xls')#获取excel
    excel_len=len(data.sheets())#得到excel表数量
#循环表
    try:
        for by_index in range(0,excel_len):
    #        excelname=data.sheets()[by_index].name
            sheet=data.sheets()[by_index]#得到第by_index个表
            excel_table_byindex(by_index,sheet)
        to_oracle.to_oracle()
    except Exception,e:
        print str(e)

#创建要素类并新建字段   
def excel_table_byindex(by_index,sheet):
        arcpy.env.workspace = parameter.sdeworkspace#设定工作空间
        out_name=sheet.cell(0,1).value.encode('utf-8') #要素类输出名
        geometry_type =sheet.cell(0,2).value.encode('utf-8') #要素类类型
        template =sheet.cell(0,3).value.encode('utf-8')
        has_m=sheet.cell(0,4).value.encode('utf-8')
        has_z=sheet.cell(0,5).value.encode('utf-8')
        try:
            spatial_reference = arcpy.Describe(sheet.cell(0,6).value).spatialReference #空间参考
            #创建要素类
            arcpy.CreateFeatureclass_management(arcpy.env.workspace,out_name,geometry_type,template,has_m,has_z,spatial_reference)
            nrows = sheet.nrows #行数 
            ncols=sheet.ncols #列数
            print 'env.workspace='+arcpy.env.workspace
            print 'out_name='+out_name
            print 'geometry_type='+geometry_type
            print 'template='+template
            print 'spatial_reference='+sheet.cell(0,6).value.encode('utf-8')
            print 'field total='+str(nrows-2)
    #   colnames = sheet.row_values(colnameindex)  
    #得到excel表信息
            for row in range(2,nrows):
                    rowdata=sheet.row_values(row) #得到表中第row行
                    if (rowdata[1] == 'SHAPE' or rowdata[1] == 'OBJECTID')==0: #判断字段名是否为SHAPE和OBJECTID
                            #新建字段
                            arcpy.AddField_management(out_name,rowdata[1],rowdata[2],rowdata[3],rowdata[4],rowdata[5],rowdata[6],rowdata[7],rowdata[8])
                            #输出字段信息
                            print  ("No:%-2d name:%-10s type:%-8s accuracy:%-5s bits:%-5s length:%-5d alias:%-20s"%(rowdata[0],rowdata[1],rowdata[2],rowdata[3],rowdata[4],rowdata[5],rowdata[6]))
        except Exception,e:
            print str(e)
         
if __name__=="__main__":
    main()

```

**to_oracle.py 对oracle中表字段写进注释**

```python
# -*- coding: UTF-8 -*-
import xlrd,sys
import parameter
reload(sys)
sys.setdefaultencoding('utf-8')
import cx_Oracle

#打开需要读取的excel
def open_excel(file):
    try:
        data = xlrd.open_workbook(file,formatting_info=True)
        return data
    except Exception,e:
        print str(e)


def to_oracle():
    data=open_excel(file='sourcefile.xls')#获取excel
    excellen=len(data.sheets())#得到excel表数量
    conn = cx_Oracle.connect(parameter.dataconnect)  #连接数据库  
    cursor = conn.cursor ()  
#循环表
    for by_index in range(0,excellen):
#        excelname=data.sheets()[by_index].name
        sheet=data.sheets()[by_index]#得到第by_index个表
        excel_table_byindex(by_index,sheet,cursor)
    cursor.close ()  
    conn.close ()
	
#向oracle表字段添加注释
def excel_table_byindex(by_index,sheet,cursor):
    nrows = sheet.nrows #行数 
#    ncols=sheet.ncols
    element_name=sheet.cell(0,1).value.encode('utf-8')
    for row in range(2,nrows):
                    rowdata=sheet.row_values(row) #得到表中第row行
                    print 'comment on column '+ element_name+'.'+rowdata[1]+' is '+'\''+rowdata[6]+'\''
                    if (rowdata[1] == 'SHAPE' or rowdata[1] == 'OBJECTID')==0:
                        cursor.execute ('comment on column '+ element_name+'.'+rowdata[1]+' is '+'\''+rowdata[6]+'\'')

```

**parameter.py参数文件**

```python
dataconnect='LD/123456@192.168.199.108/db1'
sdeworkspace='C:\\Users\\Administrator\\AppData\\Roaming\\Esri\\Desktop10.2\\ArcCatalog\\MySDE.sde'
```
### python注意事项

* 需要配置python环境变量
* 需要安装xlrd或其它读取excel的插件
* 需要安装操作oracle数据库的cx_Oracle插件
* 读取excel同样需要固定格式
* 参考坐标系放进mdb库中

文件列出如下

![file](http://7xnfbg.com1.z0.glb.clouddn.com/2016-05-11-1.JPG)

### 结果展示

其中yc是参考坐标系要素

![1](http://7xnfbg.com1.z0.glb.clouddn.com/2016-05-11-2.JPG)
![2](http://7xnfbg.com1.z0.glb.clouddn.com/2016-05-11-3.JPG)

