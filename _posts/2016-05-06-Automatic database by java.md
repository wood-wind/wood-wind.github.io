---
layout: post
title: Arcgis自动建库(Java)
date: 2016-05-06
categories: blog
tags: [技术]
description: java自动建库
---

2016年05月06日22:26:11

###问题

最近项目中需要建库，但是一个字段一个字段的选和填既费时还容易出错，弄了几个要素类之后实在受不了了，于是想办法怎么能够实现自动化，花了几天时间终于被我鼓捣出来了，在此记录一下。

未实现自动化是这样的

![](http://7xnfbg.com1.z0.glb.clouddn.com/2016-05-04-1.JPG) 

需要左边输字段名右边选字段类型，下面填别名和字段长度等，通常是对照excel表

###实现思路

* 利用Java对excel进行操作，对模板化的excel表数据进行读取生成一个可执行的python文件
* Arcgis自带python，并且工具箱中许多脚本就是用python写的
* 执行生成的python文件

###Java源代码

```java
package topy;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

import jxl.Cell;
import jxl.Sheet;
import jxl.Workbook;
import jxl.read.biff.BiffException;

/**
 * 将excel中的内容复制到py中
 */
public class xls {
    static String filepath = path();
	// private static String py = filepath + "\\ld.py";
	private static String path = filepath + "\\sourcefile.xls";

	public static void main(String[] args) throws BiffException, IOException {
		// 创建流
		InputStream is = new FileInputStream(path);
		Workbook book = Workbook.getWorkbook(is);
		String[] sheetname = book.getSheetNames();
		// 读取excel中多个sheet
		for (int i = 0; i < book.getSheetNames().length; i++) {
			// 生成的python文件名与路径
			String py = filepath + "\\" + sheetname[i] + ".py";
			// 读取sheet
			Sheet sheet = book.getSheet(i);
			// 将sheet数据写入py中
			new xls().readExcel(path, py, true, sheet);
		}
		book.close();
	}

	// 读取当前文件路径
	public static String path() {
		File directory = new File("");// 参数为空
		String courseFile = null;
		try {
			courseFile = directory.getCanonicalPath();
		//	System.out.println(courseFile);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return courseFile;
	}

	// 读取Excel内的数据并写入目标文件中
	public boolean readExcel(String path, String py, boolean append, Sheet sheet) {
		try {
			// 得到所有的行数
			Integer rows = sheet.getRows();
			// 得到所有的列数
			Integer colus = sheet.getColumns();
			Cell cell;
			String filepath2=filepath;
			String spatial = sheet.getCell(6, 0).getContents().trim();
			// 写入py头部必要信息
			String CreateFeatureclass = "# -*- coding: UTF-8 -*-";
			new xls().insert(py, CreateFeatureclass + "\r\n", append);
			CreateFeatureclass = "import arcpy";// 引入arcpy
			new xls().insert(py, CreateFeatureclass + "\r\n", append);
			CreateFeatureclass = "from arcpy import env";
			new xls().insert(py, CreateFeatureclass + "\r\n", append);
			CreateFeatureclass = "env.workspace = \"" + filepath2.replace("\\", "/")
					+ "/database.mdb\"";// 定义工作空间
			new xls().insert(py, CreateFeatureclass + "\r\n", append);
			
			CreateFeatureclass = "spatial_reference = arcpy.Describe(\""+filepath2.replace("\\", "/")+"/spatial_reference/"+spatial+"\").spatialReference";
			
			new xls().insert(py, CreateFeatureclass + "\r\n", append);
			/**
			 * 新建要素类
			 */
			CreateFeatureclass = "arcpy.CreateFeatureclass_management(\""+filepath2.replace("\\", "/")+"/database.mdb\",";
			for (int i = 1; i < 6; i++) {
				String colname = sheet.getCell(i, 0).getContents().trim();
				CreateFeatureclass += "\""+colname + "\",";
			}
			new xls().insert(py, CreateFeatureclass +"spatial_reference"+ ")" + "\r\n"
					+ "\r\n", append);
			/**
			 * 添加字段
			 */
			for (int i = 3; i < rows; i++) {
				// 得到地2列字段名用做判断
				String getSheet = sheet.getCell(1, i).getContents().trim();
				// 得到新建要素类名
				String getField = sheet.getCell(1, 0).getContents().trim();
				// 跳过字段名为SHAPE和OBJECTID的字段
				if (getSheet.equals("SHAPE") == false
						|| getSheet.equals("OBJECTID") == false) {
					String topython = "arcpy.AddField_management(\"" + getField
							+ "\",\"";
					for (int a = 1; a < colus; a++) {
//						System.out.println(getSheet);
						String colname = sheet.getCell(a, i).getContents()
								.trim();
						topython += colname + "\",\"";
					}
					new xls().insert(py, topython + "\")" + "\r\n",
							append);
				}
			}
			return true;
		} catch (Exception e) {
			e.printStackTrace();
			return false;
		}
	}

	// 写入py文件流
	public void insert(String py, String content,boolean append) throws IOException {
		File file = new File(py);
		FileOutputStream writerStream = new FileOutputStream(file,append);    
		BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(writerStream, "UTF-8")); 
		writer.write(content);
		writer.close();    
    } 
}
```

###Java注意事项

* 需要下载能够读取excel的包
* 将代码导出成可执行的jar包，这样只需要有Java环境的电脑都可以执行了
* 因为读取时是模式化读取，需要固定excel格式
![](http://7xnfbg.com1.z0.glb.clouddn.com/2016-05-04-3.JPG)
    * 其中第一行为要素类的属性(要素名，类型，参考坐标系名等)
    * 没有值的就空着
* 同时mdb文件，参考坐标系的shp等也都需要放在固定的位置
![](http://7xnfbg.com1.z0.glb.clouddn.com/2016-05-04-2.JPG)
    * spatial_reference中放的是参考坐标系shp文件
    * database.mdb是新建的要素类存放的库
* 当执行zdjk.jar会生成三个py文件

![](http://7xnfbg.com1.z0.glb.clouddn.com/2016-05-04-4.JPG)

###生成的py文件

![](http://7xnfbg.com1.z0.glb.clouddn.com/2016-05-04-5.JPG)

###python注意事项

* 配置python环境变量(自己Google)输入python回车出现下图表示配置成功
![](http://7xnfbg.com1.z0.glb.clouddn.com/2016-05-04-6.JPG)
* 注意字符编码问题


以上

###后续

python不熟，用Java实现的，考虑用python一步实现
