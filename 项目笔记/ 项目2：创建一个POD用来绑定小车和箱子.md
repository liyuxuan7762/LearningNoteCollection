# 项目2：创建一个POD用来绑定小车和箱子

## 1. 项目需求

用户可以输入小车号和箱子号，用户输入完箱子号以后，系统会根据箱子号查询到对应的SFC，只有当箱子已经绑定了SFC之后，才可以被添加到下面的表格中。否则就给出用户相应的提示。

此外，如果当前表格中已经存在了箱子数据，那么再次添加新的箱子的时候需要先获取SFC, 然后判断当前输入的箱子对应的SFC和表格中已经存在的SFC是否一致，如果不一致，则返回“不允许混批”。

当用户点击绑定按钮后，调用后台的接口进行绑定。

## 2. 业务场景

一个订单可能包含多个零件，这一个订单实际上就是一个SFC，一个SFC中可以包含多个零件，每一个零件都保存到一个箱子中。一个小车可以搭载多个箱子。

生产结束后，需要将生产完成的零件放到箱子中，然后将箱子放到小车上，一个小车可能放不了一整个SFC的零件，所以这时候，如果零件没有放完，那么SFC的状态就是incomplete，如果放完了，那么就设置SFC的状态的complete。当这一个SFC已经完成后，就需要调用相关的接口到ERP系统中进行报工。下面的这段代码就是判断SFC的状态，然后决定是否需要报工的代码。

```java
//如果SFC完成 if SFC complete
if("complete".equals(sfcState.toLowerCase())){
	//报工 report work
	ipInterfaceService.setSfcCompleteAndFeedback(workCenter, sfc);
	//删除排序状态 delete sorting status
	commonDao.deleteSfcData(new SFCBOHandle(site, sfc).getValue(), "Z_SFC_STATE", null);
}
```

## 3. 学习到的知识点

### 3.1 下拉列表的实现

之前的下拉列表的实现方式是使用了ComboBox组件，这一个组件有一个问题，也就是说虽然他可以下拉，但是显示的内容用户是可以进行编辑的，因此后来还写了相应的Validator来防止用户随意输入内容。如下：

```xml
<ComboBox id="status" required="true" width="100%"
	selectedKey="1"
	items="{
	path: '/status',
	sorter: { path: 'text' }
	}"
	change="handleChange">
	<core:Item text="{text}" key="{key}" />
</ComboBox>
```

这种方式非常麻烦，而且还需要在后台创建JSON对象绑定数据，所以并不是一个好方法，在这个项目中，实际上有Select组件，可以实现

- 用户无法编辑下拉列表的内容，只能选择
  
- 下拉列表里面的数据可以直接在xml中写入，无需创建JSON对象
  

```xml
<Select id="sfcStatus">
	<core:Item key="1" text="{i18n>/PackComplete}" />
	<core:Item key="2" text="{i18n>/PackIncomplete}" />
	<layoutData>
		<l:GridData span="XL8 L8 M8 S10" />
	</layoutData>
</Select>
```

### 3.2 表格的使用

#### 3.2.1 前端界面

```xml
<Table id="table" inset="false" items="{/dataList}" noDataText="{i18n>/PackNoData}" >
	<columns>
		<Column hAlign="Center">
			<Text text="{i18n>/PackBoxLabel}"/>
		</Column>
		<Column hAlign="Center">
			<Text text="{i18n>/PackSFCNoLabel}"/>
		</Column>
		<Column hAlign="Center">
			<Text text="{i18n>/PackDeleteLabel}"/>
		</Column>
	</columns>
	<items>
		<ColumnListItem vAlign="Middle">
			<cells>
				<Text text="{box}"/>
				<Text text="{SFC}"/>
				<Button icon="sap-icon://delete" press="onDelete"/>
			</cells>
		</ColumnListItem>
	</items>
</Table>
```

前端界面中没有很多需要注意的点，需要注意的是前端的数据绑定问题，items="{/dataList}这一个地方，以及在每一个item中，如何获取不同列的值text="{SFC}。

对应的JavaScript中的数据对象的定义如下：

```javascript
var oModel = new JSONModel({
	dataList : [
		{box : "1500000001", SFC : "0480-233223213"},
		{box : "1500000001", SFC : "0480-233223213"},
		{box : "1500000001", SFC : "0480-233223213"},
		{box : "1500000001", SFC : "0480-233223213"}
	]
});
```

#### 3.2.2 前端JavaScript代码

JavaScript中的代码主要集中于实现下面的几个个功能：

- 如何读取到表格中的数据？
  
- 如何遍历所有的表格数据生成一个JSON对象数组？
  
- 如何删除指定的某一行数据
  

**如何读取到表格中的数据？（以获取第一行第一列的数据为例）**

首先获取到表格中的所有记录

```javascript
this.getView().byId("table").getItems()
```

然后获取到第一行的记录对象

```javascript
var item = oItems[0];
```

然后获取到指定列的数据

```javascript
var sfc = item.getCells()[1].getText();
```

**如何遍历所有的表格数据生成一个JSON对象数组？**

和之前获取第一行的操作基本一样，首先获取到所有的记录

```javascript
var oItems = oTable.getItems();
```

然后利用循环依次读取每一行的数据即可

```javascript
for (var i = 0; i < oItems.length; i++) {
	var item = oItems[i];
	var boxNumber = item.getCells()[0].getText();
	boxList.push({
		boxId : boxNumber
	});
}
```

**如何删除指定行的数据**

```javascript
onDelete : function(oEvent) {
	var oItem = oEvent.getSource().getParent();

	var oContext = oItem.getBindingContext();
	var sPath = oContext.getPath();
	var oModel = oContext.getModel();
	var oData = oModel.getProperty(sPath);

	var aDataList = oModel.getProperty("/dataList");
	var iIndex = aDataList.indexOf(oData);
	if (iIndex !== -1) {
		// Remove one element starting from the iIndex index. 
		// For example, splice(iIndex, 1) means to remove the element corresponding to the iIndex index.
		aDataList.splice(iIndex, 1); // Delete the iIndex
		oModel.setProperty("/dataList", aDataList);
	}
},
```

### 3.3 如何自动获取POD的标题

通过下面的代码即可

```javascript
onInit : function() {
	// set the title of the POD
	this.getView().addEventDelegate({
		onBeforeShow : function(evt) {
			this.onBeforeShow(evt);
		}
	}, this);
},

onBeforeShow: function(evt) {
	// set the title
	this.byId("page").setTitle(evt.data.appName);
}
```
