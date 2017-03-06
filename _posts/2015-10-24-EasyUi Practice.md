---
layout: post
title: EasyUi数据表单小结
date: 2015-10-24
categories: blog
tags: [技术]
description: 这两天都在用EasyUi表单。
---

2015年10月24日19:36:51

最近在写管理系统，必定少不了增删改查，以前做练习功能彼此关联性不怎么强，用了EasyUi的数据表单才发现有这么强大的工具，将前台整合起来。

记录一下，以后忘记了还能回顾下。

这是系统部分界面
![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-24-1.jpg)
要实现的功能：
- 展示
- 异步添加数据，完成后顶端展示
- 实现多选删除
- 选择一条数据修改

我采用了EasyUi中的[Basic CRUD Application](http://www.jeasyui.com/demo/main/index.php?plugin=Application&theme=default&dir=ltr&pitem=)
另CRUD DataGrid也是非常好的模板。
区别主要在添加与修改：Basic CRUD为弹窗形式；CRUD DataGrid为表单内添加行。
![](http://7xnfbg.com1.z0.glb.clouddn.com/2015-10-24-2.jpg)

以下为部分代码：

页面：
```
#窗口标题
<div id="resultList" data-options="region:'west',collapsed:false,split:true" title="监测信息"  style="width:420px;overflow:none" >

 <div class="easyui-tabs" id="ftpTabs"  style="width:100%;height:100%">
#人员监测栏
	<div id="ftpGroupTab" title="人员监测" style="width:100%;height:100%">
#数据展示区
		<table id="gs_query" class="easyui-datagrid" title="" style="width:100%;height:93%"></table>
#添加弹出窗口
		<div id="dlg" class="easyui-dialog" style="width:400px;height:280px;padding:10px 20px"
		closed="true" buttons="#dlg-buttons">
			<div class="ftitle">监测信息</div>
				<form id="fm" method="post" novalidate>
					<div class="fitem">
						<label>当前监测值:</label>
						<input name="dense.monitoring" class="easyui-textbox" required="true" id="monitoring">
					</div>
					<div class="fitem">
						<label>最大监测值:</label>
						<input name="dense.maxMonitoring" class="easyui-textbox" required="true">
					</div>
					<div class="fitem">
						<label>地址:</label>
						<input name="dense.address" class="easyui-textbox" required="true">
					</div>
					<div class="fitem">
						<label>监测日期:</label>
						<input name="dense.collectionDate" class="easyui-textbox" validType="date">
					</div>
					<div class="fitem">
						<label>监测人:</label>
						<input name="dense.person" class="easyui-textbox" >
					</div>
				</form>
			</div>
	#弹窗保存与取消
		<div id="dlg-buttons">
			<a href="javascript:void(0)" class="easyui-linkbutton c6" iconCls="icon-ok" onclick="saveUser()" style="width:90px">保存</a>
			<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-cancel" onclick="javascript:$('#dlg').dialog('close')" style="width:90px">取消</a>
		</div>
```
JS：
```
/**
 * 初始进来，分页显示ftp组列表数据
 */
function loadKsQuery() {
#datagrid控件加载数据
	$("#gs_query").datagrid(
			{
				collapsible : false, 
				rownumbers : true,    		//True 就会显示行号的列
				fit : true,					//设置为true将自动使列适应表格宽度以防止出现水平滚动。
				pageSize : 20,				//当设置了 pagination 特性时，初始化页码尺寸
				pageList : [ 10, 20, 30 ],	//当设置了 pagination 特性时，初始化页面尺寸的选择列表
				/*url : 'str/ksQuery.do',*/
				url : 'str/getAllDense!getAllDense.action',		//请求后台数据
				loadMsg : '数据装载中......',
				columns : [ [ {				//显示的列
					title : '当前监测值',
					field : 'monitoring',
					align : 'center',
					width : 110
				},  {
					title : '最大监测值',
					field : 'maxMonitoring',
					align : 'center',
					width : 110
				}, {
					title : '地址',
					field : 'address',
					align : 'center',
					width : 180
				}, {
					title : '监测日期',
					field : 'collectionDate',
					align : 'center',
					width : 80
				}, {
					title : '监测人',
					field : 'person',
					align : 'center',
					width : 80
					
				},
				{
					title : 'XY',
					field : 'point',
					align : 'center',
					width : 80
					
				},
				{
					title : 'x',
					field : 'pointx',
					align : 'center',
					width : 80
					
				},
				{
					title : 'y',
					field : 'pointy',
					align : 'center',
					width : 80
				},] ],
				pagination : true,		// 是否显示底部分页工具栏
				singleSelect : false,	//设置为true将只允许选择一行
				selectOnCheck: false,	//如果设置为true，点击checkbox将永远选择这行。如果设置为false，选择一个行将不会选择checkbox
				checkOnSelect: true,	//如果设置为 true，当用户点击一行的时候 checkbox checked(选择)/unchecked(取消选择)。 如果为false，当用户点击刚好在checkbox的时候，checkbox checked(选择)/unchecked(取消选择)
				rownumbers : true,		// 当true时显示行号
				fitColumns : false,     //宽度自适应窗口改变
				onClickRow : function(rowIndex, rowData) {		//当用户点击行时触发
					// 获取选中行的数据
					var chks = $('#gs_query').datagrid('getChecked');
					if (null != chks && chks.length >= 1) {
						for(var i=0;i<chks.length;i++){
						/*
							$.ajax({
							url:'str/getAllDense!deleteOneDense.action',
							type:'post',
							data:'dense.did='+chks[i].did,
							dataType:'json',
							success:function(result){
								window.location.reload();
							}
						});
						*/
						}
					}
				},
			//工具栏
			toolbar: [{
			//添加按钮及事件
			text: '添加', iconCls: 'icon-add', handler: function () {
			  $('#dlg').dialog('open').dialog('center').dialog('setTitle','添加监测信息');
			  $('#fm').form('clear');
			  url = 'str/getAllDense!addOneDense.action';
			}
		}, '-', {
			text: '撤销', iconCls: 'icon-redo', handler: function () {
				editRow = undefined;
				$("#gs_query").datagrid('rejectChanges');
				$("#gs_query").datagrid('unselectAll');
			}
		}, '-', {
			text: '删除', iconCls: 'icon-remove', handler: function () {
				var row = $("#gs_query").datagrid('getSelections');
					if (null != row && row.length >= 1) {
						for(var i=0;i<row.length;i++){
							$.ajax({
							url:'str/getAllDense!deleteOneDense.action',
							type:'post',
							data:'dense.did='+row[i].did,
							dataType:'json',
							success:function(result){
							}
						});
						}
					}
					 $('#gs_query').datagrid('reload');
			}
		}, '-', {
			text: '修改', iconCls: 'icon-edit', handler: function () {
					var row = $('#gs_query').datagrid('getSelected');
				if (row){
					$('#dlg').dialog('open').dialog('center').dialog('setTitle','Edit User');
					$('#fm').form('load',row);
					url = 'update_user.php?did='+row.did;
				}
			}
		}]
	});
		//点击保存触发的事件
		function saveUser(){
			$('#fm').form('submit',{
				url: 'str/getAllDense!addOneDense.action',
				onSubmit: function(){
					
					return $(this).form('validate');
				},
				success: function(result){
				//	alert(11);
				//  var result = eval('('+result+')');
					
					if (result==null){
						$.messager.show({
							title: 'Error',
							msg: result.errorMsg
						});
					} else {
						$('#dlg').dialog('close');        // close the dialog
						$('#gs_query').datagrid('reload');    // reload the user data
					}
				}
			});
		}
```
暂时做了这么多，应该还有很多可以优化的地方。