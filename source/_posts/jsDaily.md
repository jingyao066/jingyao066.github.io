---
title: jsDaily
tags: js
date: 2019-05-20 10:04:27
---

# input设置为disabled，后台无法接受到数据
W3C的规范，`disabled=”disabled”`不能向后台提交，改为`readonly = “readonly”`即可

# 两种提交表单的方式
1. `$("#addForm").serialize`
将表单数据收集成json字符串，`name=小红&age=25`的形式

2. `$("#addForm").serializeArray();`
以数组形式收集表单数据，这个方法可以通过push方法向结果集中添加数据，如果想提交表单之外的数据，可以用这种方式：
`dataParam.push({"name": "specName", "value": specName});`

另外：需要在ajax的中设置参数
`traditional: true,`

# Js实现返回上一页并刷新
`self.location=document.referrer`

# 复选框全选，全不选
```
$('#checkAll').click(function(){ 
	    $(".checkSon").prop("checked",this.checked); 
	});
//用attr不行,因为jq版本问题,复选框最好用prop
```

# 限制input框输入只能为非负数且不能有小数点
```js
<input type="number" name="quantity" value="1" min='1' onkeyup="onlyNonNegative(this)"/>
//通过2步做到输入的为非负数
//1.去掉多余的小数点
//2.保证只能输入小数点或数字
function onlyNonNegative(obj) {
    var inputChar = event.keyCode;
    //alert(event.keyCode);
    //1.判断是否有多于一个小数点
    if (inputChar == 190) {//输入的是否为.
        var index1 = obj.value.indexOf(".") + 1;//取第一次出现.的后一个位置
        if (index1 == 1) {//如果第一个值就是点
            obj.value = obj.value.replace(/[^\d]/g, '')
        }
        var index2 = obj.value.indexOf(".", index1);
        while (index2 != -1) {
            //alert("有多个.");
            obj.value = obj.value.substring(0, index2);
            index2 = obj.value.indexOf(".", index1);
        }
    }
    //2.如果输入的不是.或者不是数字，替换 g:全局替换
    obj.value = obj.value.replace(/[^\d]/g, '');
}
```

# 选中颜色添加边框,相邻的去除边框
```
function getColor(obj){
	$(obj).siblings('div').removeClass('selected');
	$(obj).addClass('selected');
}
规定一个css属性
.selected{
  border:3px solid #339999 !important;
}	
Html
<div onclick="getColor(this)">
```

# JQ获取选中复选框的值
本以为：
`var arr = $('input[name="ck"]:checked').val();`
返回的直接是一个数组，错了。必须
```
var checkArr = [];
$('input[name="ck"]:checked').each(function(index,i){
      checkArr.push(i);
})
console.log("选中的"+checkArr.length);
```
选中之后遍历，向数组中push

# vue给input绑定value值
```
<tr v-for="business in businessList.list">
  <td><input type="checkbox" name="ck" :value="business.id"></td>
  <td>{{business.businessName}}</td>
  <td>{{business.businessMobile}}</td>
</tr>
```

# JQ ajax向后台传递数组
因为jQuery需要调用jQuery.param序列化参数，jQuery.param( obj, traditional )，
默认的话，traditional为false，即jquery会深度序列化参数对象，我们可以通过设置traditional 为true阻止深度序列化，然后序列化结果如下：
p: ["123", "456", "789"]    =>    p=123&p=456&p=456
随即，我们就可以在后台通过request.getParameterValues()来获取参数的值数组了，
所以，比如我们前台有多个checkbox，js代码可以写成：
```
var values = $("input[type=checkbox]").map(function(){
	return $(this).val();
}).get();
$.ajax{
  url:"xxxx",
  traditional: true,
  data:{
	p: values 
  }
}
```
# td中的内容，超出长度显示省略号
内容必须包在p标签中
```
.msg{
    width: 200px;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
}
<td>
    <p class="msg">
         ${(i.message)!''}
    </p>
</td>
```

# ajax 提交富文本 数据截断
富文本中有特殊符号，如：&，富文本的字符串会被截断。
解决方法：
```
var data = $("#addForm").serializeArray();
var content =  $.trim($('#summernote').summernote('code'));
data.push({"name": "content", "value": content});
```