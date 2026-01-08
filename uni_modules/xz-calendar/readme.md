# xz-calendar 自定义日历
> 实现展示一周或者一个月的日历

### Props
| 属性名	| 类型		| 是否必填	|默认		|可选								|说明																																													|
|-------	|-------|-------	|-------|-------						|-------																																											|
| selected| Array	|false		|[]			|无									|打点数据；格式：[{date:'2025-10-25',info:'描述',mark:true,markColor:'red',infoColor:'red',...}]	|
| type		| String|false		|month	|month(月)、week(周)	|日历类型																																											|

### Emits
| 事件名		| 接收值																							|说明				|
|-------		|-------																						|-------		|
|changeDate	|{year:年,month:月,day:天,date:年-月-日,data:其他信息}	|当前选择日期	|
|changeMonth	|{year:年,month:月}	|当前选择月份	|

### 代码示例

#### 数据
```js
const selected = [
	{
		date:'2025-10-25',
    info:'已打卡',
    mark:true
	},
	{
		date:'2025-10-26',
		info:'已打卡',
		mark:true
	}
];
const type = 'month'
const changeDate = (dayInfo) =>{
  console.log('当前选择日期', dayInfo);
}
```

#### 组件
```vue
<template>
	<xz-calendar :selected="selected" :type="type" @changeDate="changeDate"></xz-calendar>
</template>
```
