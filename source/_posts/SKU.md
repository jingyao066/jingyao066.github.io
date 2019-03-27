---
title: SKU
tags: 设计思路
date: 2018-12-09 22:52:56
---

# SKU
![](SKU/1.jpg)
淘宝购买商品时，商品有很多不同的属性和属性值，如：
颜色：白色、黑色、蓝色、紫色...
尺寸：170、175、180、185...
买衣服时，通常会发现，白色-175库存20件，但是白色-180有30件，
而且价格可能还不同，这是因为后台sku设置的价格和库存。
后台设置SKU的界面
![](SKU/2.png)

# SKU相关表设计
商品表 goods
```
CREATE TABLE `goods` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `goods_name` varchar(100) DEFAULT NULL COMMENT '商品名称',
  `cover_img` varchar(255) DEFAULT NULL COMMENT '封面图',
  `sales_volume` int(11) DEFAULT NULL COMMENT '销量(前台显示用，售出后程序累加)',
  `stock_no` int(11) DEFAULT NULL COMMENT '商品库存',
  `price` decimal(10,2) DEFAULT NULL COMMENT '价格(只前台显示用)',
  `classify_id` int(11) DEFAULT NULL COMMENT '分类id(子级分类id)',
  `status` int(11) DEFAULT '1' COMMENT '商品状态(0：下线  1：上线)',
  `is_free_delivery` int(2) DEFAULT NULL COMMENT '是否包邮(0:否 1：是)',
  `type` int(11) DEFAULT '1' COMMENT '类型(预留字段)',
  `goods_detail` text CHARACTER SET utf8mb4 COMMENT '商品详情',
  `seller_id` int(11) DEFAULT NULL COMMENT '商家id',
  `audit_status` int(4) NOT NULL DEFAULT '0' COMMENT '审核状态。0待审核，1通过，2未通过，3封禁',
  `audit_opinion` varchar(50) DEFAULT NULL COMMENT '审核原因',
  `video_url` varchar(200) DEFAULT NULL COMMENT '商品视频',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `is_hot` varchar(2) DEFAULT '0' COMMENT '是否热销(0：否  1：是)',
  `is_new` int(11) DEFAULT '0' COMMENT '是否最新上架(0:否 1：是)',
  `is_explode` int(255) DEFAULT '0' COMMENT '是否爆品(0：否  1：是)',
  `is_home` varchar(2) DEFAULT '0' COMMENT '是否首页推荐(0:否 1：是)',
  `home_sort` varchar(10) DEFAULT NULL COMMENT '首页推荐排序(废弃字段，暂停使用)',
  `del_flag` int(2) DEFAULT '0' COMMENT '删除标识( 0未删除；1已删除)',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `index_status_type` (`status`,`type`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=7872 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='商品表';
```
sku表
```
CREATE TABLE `goods_sku` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `sku_name` varchar(100) CHARACTER SET utf8 DEFAULT NULL COMMENT 'SKU名字',
  `goods_id` int(11) DEFAULT NULL COMMENT '商品id',
  `discount` float DEFAULT NULL COMMENT '折扣',
  `price` decimal(10,2) DEFAULT NULL COMMENT '价钱',
  `sell_price` decimal(10,2) DEFAULT NULL COMMENT '折后价格',
  `weight` double(10,2) DEFAULT NULL COMMENT '商品毛重',
  `stock` int(11) DEFAULT NULL COMMENT '库存',
  `expressage` decimal(10,2) DEFAULT NULL COMMENT '快递费',
  `status` int(11) DEFAULT NULL COMMENT '状态',
  `sku_group_id` varchar(100) CHARACTER SET utf8 DEFAULT NULL COMMENT '组合后的skuid,用逗号分隔',
  `create_time` datetime DEFAULT NULL COMMENT '添加时间',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `index_goods_id` (`goods_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=19545 DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='sku表';
```
商品属性表 goods_property
```
CREATE TABLE `goods_property` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `seller_id` int(11) DEFAULT NULL COMMENT '商家id',
  `property_name` varchar(255) DEFAULT NULL COMMENT '属性名',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=139 DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='商品属性表';
```
商品属性值表 goods_property_val
```
CREATE TABLE `goods_property_val` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `seller_id` int(11) DEFAULT NULL COMMENT '商家id',
  `property_id` int(11) DEFAULT NULL COMMENT '属性id',
  `property_val` varchar(255) DEFAULT NULL COMMENT '属性值',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=20176 DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='商品属性值表';
```
商品属性和属性值中间表 goods_pro_val
```
CREATE TABLE `goods_pro_val` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `goods_id` int(11) DEFAULT NULL COMMENT '商品id',
  `property_id` int(11) DEFAULT NULL COMMENT '属性id',
  `property_val_id` int(11) DEFAULT NULL COMMENT '属性值id',
  `property_name` varchar(100) DEFAULT NULL COMMENT '属性名字',
  `property_val` varchar(100) DEFAULT NULL COMMENT '属性值',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `index_goods_id` (`goods_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=6775 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='商品-属性-属性值>中间表';
```

# 后台操作流程
添加sku之前，需要先添加属性和属性值，因为sku是根据多种属性值组合构成的。
添加商品属性作为单独的模块，在此不在赘述。

1.选择属性
![](SKU/3.png)
2.已选属性
![](SKU/4.png)
3.生成sku
![](SKU/5.png)

4.提交

# JS代码
## 定义全局下标
```
//所有已经存在的属性和属性值json
var propertyJson;
//用来保存选中的属性值
var attrValue_arr = [];
//属性名字参数下标
var attrNameIndex = 0;
//sku属性下标
var skuIndex = 0;
//拼接后的SKU最终结果,以数组形式保存
var skuArr = [];
```

## 选择属性
```
//选择属性
function choosePro(propertyId, propertyName, onOff,isFirst) {
    //检测该属性是否存在
    var isExists = false;
    $(".attrName").each(function () {
        if ($(this).val() == propertyId) {
            isExists = true;
            if(isFirst){
                return;
            }else{
                layer.msg('该属性已存在');
                return;
            }
        }
    })
    if (isExists) {
        $('#myModal').modal('hide');
        return;
    }
    $.each(propertyJson.gpList, function (key, value) {
        if (propertyName == value.propertyName) {
            if (value.proValList.length == 0) {
                layer.msg('该属性下没有属性值');
                $('#myModal').modal('hide');
                return;
            }
            //属性值参数下标
            var attrValueIndex = 0;
            //循环属性下面的属性值集合
            var vhtml = "<div class='formControls col-8'>";
            $.each(value.proValList, function (key, i) {
                vhtml += "<input name='proList[" + attrNameIndex + "].proValList[" + attrValueIndex + "].propertyVal' " +
                    "class='attrName" + value.id + "' onclick='createSku()'  type='checkbox' data-value='" + i.propertyVal + "' " +
                    "data-id='" + i.id + "' value='" + i.propertyVal + "'/>" + i.propertyVal+
                    "<input name='proList[" + attrNameIndex + "].id' type='hidden' value='" + value.id + "' />" +
                    "<input name='proList[" + attrNameIndex + "].proValList[" + attrValueIndex + "].id' type='hidden' value='" + i.id + "' />";
                attrValueIndex++;
            })
            vhtml += "</div>";
            //为每一个属性拼接一个删除按钮
            if (onOff) {
                vhtml += "<div class='formControls col-2'>" +
                    "<span class='l' style='padding-left: 20px;'><a href='javascript:;' class='btn btn-danger radius' onclick='delPro(this)'>删除属性</a></span>" +
                    "</div>";
            } else {
                vhtml += "<div class='formControls col-2'>" +
                    "<span class='l' style='padding-left: 20px;'><a href='javascript:;' class='btn btn-danger radius' onclick=javascript:layer.msg('不能删除!');>删除属性</a></span>" +
                    "</div>";
            }
            //data 代表了属性实体类
            //属性名字
            var shtml = "<label class='form-label col-2'>" + value.propertyName + "</label>"
                + "<input type='hidden' class='attrName' value='" + value.id + "' />"
                + "<input type='hidden' name='proList[" + attrNameIndex + "].propertyName' value='" + value.propertyName + "' />"
                + "<input type='hidden' name='proList[" + attrNameIndex + "].propertyId' value='" + value.id + "' />"
                //属性值
                + vhtml;
            shtml += "<div class='formControls col-12'></div>";
            //参数下标+1
            attrNameIndex++;
            //将拼接好的div放到属性div里
            $("#choosePro").append(shtml);
            //关闭弹出层
            if (onOff) {
                $('#myModal').modal('hide');
            }
        }
    })
}
//删除已选属性
function delPro(obj) {
    var btn = $(obj).parent().parent();
    for (var i = 0; i < 5; i++) {
        $(btn).prev().remove();
    }
    $(btn).remove();
    //重置属性下标
    attrNameIndex = 0;
    createSku();
}
//获取商品属性和属性值list
function searchProperty() {
    $.ajax({
        url: "/goods/getInfoBySellerId",
        type: "post",
        dataType: "json",
        async:false,
        success: function (res) {
            propertyJson = res;
        },
        error: function () {
            talert(1, "网络错误", function () {
            })
        }
    })
}
```

## 点击属性生成SKU
```
//根据选择的属性值生成SKU
function createSku() {
    // attrNameIndex = 0;
    attrValue_arr = [];
    skuArr = [];
    skuIndex = 0;
    //首先获取添加了几个属性
    var attrName_arr = $(".attrName");

    for (var i = 0; i < attrName_arr.length; i++) {
        //获取所有选中的属性值，返回一个数组，并将数组push进外层数组
        var attrckeck = $("input[class='attrName" + $(attrName_arr[i]).val() + "']:checked");
        if (attrckeck.length > 0) {
            attrValue_arr.push(attrckeck);
        }
    }

    if (attrValue_arr.length == 0) {
        $("#sku_body").empty();
        return;
    }

    //递归方式生成已选中的所有属性的组合（笛卡尔积）
    //思路为 已第一个组选中的属性值为根，与其他所有已选中的属性值进行拼接
    //因为属性值会出现多组，所以采用递归方式进行拼接
    //将每一次循环的结果作为参数传递至下一次循环里
    for (var i = 0; i < attrValue_arr[0].length; i++) {
        //只选择了一个属性
        if (attrValue_arr.length == 1) {
            for (var i = 0; i < attrValue_arr[0].length; i++) {
                skuArr.push({
                    'attrValue': $(attrValue_arr[0][i]).attr("data-value"),
                    'attrId': $(attrValue_arr[0][i]).attr("data-id")
                });
            }
        } else {
            var forArray = [attrValue_arr[0][i]];
            concatSku(forArray, 1);
        }
    }

    //将拼接好的SKU属性显示
    //获取页面中已有的SKU，如果已经的将不在进行拼接
    var skuHtml = "";
    for (var i = 0; i < skuArr.length; i++) {
        skuHtml += "<tr class='text-c'>"
            + "<td>" + skuArr[i].attrValue + "</td>"
            + "<input type='hidden' name='goodsSkuList[" + skuIndex + "].skuName' value='" + skuArr[i].attrValue + "' />"
            + "<input type='hidden' name='goodsSkuList[" + skuIndex + "].skuGroupId' value='" + skuArr[i].attrId + "' />"
            + "<td><input name='goodsSkuList[" + skuIndex + "].price' type='number' class='form-control'/></td>"
            + "<td><input name='goodsSkuList[" + skuIndex + "].sellPrice' type='number' class='form-control'/></td>"
            + "<td><input name='goodsSkuList[" + skuIndex + "].stock' type='number' class='form-control' /></td>"
            + "<td><input name='goodsSkuList[" + skuIndex + "].weight' type='number' class='form-control' /></td>"
            + "</tr>";
        skuIndex++;
    }
    $("#sku_body").html(skuHtml);

}
```

```
//拼接属性值
//forArray 上一次循环的结果数组（已拼接好的）
//计数变量，记录本次要循环外层数组中的哪一个数组
function concatSku(forArray, forIndex) {
    var forResult = [];
    //如果计数变量+1没有超出外层数组长度，则证明没有循环完毕
    if (forIndex + 1 <= attrValue_arr.length) {
        for (var i = 0; i < forArray.length; i++) {
            var attrValue = $(forArray[i]).attr("data-value");
            var attrId = $(forArray[i]).attr("data-id");
            if (forIndex == 1 || attrValue_arr.length == 1) {
                attrValue += "-";
                attrId += ",";
            }
            if (attrValue == undefined) {
                attrValue = forArray[i].attrValue;
                attrId = forArray[i].attrId;
            }
            for (var j = 0; j < attrValue_arr[forIndex].length; j++) {
                var nextAttrValue = $(attrValue_arr[forIndex][j]).attr("data-value");
                var nextAttrId = $(attrValue_arr[forIndex][j]).attr("data-id");
                var ary;
                //将拼接属性值的结果push到结果集数组中
                if (forIndex + 1 == attrValue_arr.length) {
                    ary = {'attrValue': attrValue + nextAttrValue, 'attrId': attrId + nextAttrId}
                    //forResult.push(attrValue + "-" + nextAttrValue);
                    forResult.push(ary);
                } else {
                    ary = {'attrValue': attrValue + nextAttrValue + "-", 'attrId': attrId + nextAttrId + ","}
                    //forResult.push(attrValue + "-" + nextAttrValue+"-");
                    forResult.push(ary);
                }

            }
        }
        //本次循环完毕后将计数变量+1，以便下次使用
        forIndex++;
        //递归调用
        concatSku(forResult, forIndex);
    } else {
        skuArr = skuArr.concat(forArray);
    }
}
```

属性是pojo，级联的name属性。
属性级联
`name='proList[" + attrNameIndex + "].proValList[" + attrValueIndex + "].propertyVal'`

sku级联
`name='goodsSkuList[" + skuIndex + "].skuName'`

所以ajax提交一定要
`var dataParam = $("#addForm").serializeArray();`

级联提交的难点在于，数组下标的控制。如果只是生成的话，还可以，但是如果涉及到回显后修改，以及删除，下标就不好玩了。注意吧，自己组装数据非常麻烦。

## 生成sku的代码，还有回显代码
```
function initGoods() {
    //获取商品id
    var goodsId = $("#goodsId").val();
    if (goodsId != null && goodsId != '') {
        $.ajax({
            url: "/goods/goods/getJsonGoodsInfo?id="+goodsId,
            type: 'post',
            async: false,
            success: function (data) {
                //循环回显该商品已有属性
                $.each(data.pvList, function (key,i) {
                    choosePro(i.propertyId, i.propertyName, false,true);
                    checkCheckBox(data.pvList);
                })
                //将拼接好的SKU属性显示
                var skuHtml = "";
                for (var i = 0; i < data.goodsSkuList.length; i++) {
                    skuHtml += "<tr class='text-c'>"
                        + "<td>" + data.goodsSkuList[i].skuName + "</td>"
                        + "<input type='hidden' name='goodsSkuList[" + skuIndex + "].skuName' class='hiddenSkuName'  value='" + data.goodsSkuList[i].skuName + "' />"
                        + "<input type='hidden' name='goodsSkuList[" + skuIndex + "].skuGroupId'   value='" + data.goodsSkuList[i].skuGroupId + "' />"
                        + "<td><input name='goodsSkuList[" + skuIndex + "].price' value='" + data.goodsSkuList[i].price + "' type='number' class='form-control'/></td>"
                        + "<td><input name='goodsSkuList[" + skuIndex + "].sellPrice' value='" + data.goodsSkuList[i].sellPrice + "' type='number' class='form-control'/></td>"
                        + "<td><input name='goodsSkuList[" + skuIndex + "].stock' value='" + data.goodsSkuList[i].stock + "' type='number' class='form-control' /></td>"
                        + "<td><input name='goodsSkuList[" + skuIndex + "].weight' value='" + data.goodsSkuList[i].weight + "' type='number' class='form-control' /></td>"
                        + "</tr>";
                    skuIndex++;
                }
                $("#sku_body").html(skuHtml);
            }
        })
    }
}

//将已选中的属性值设置为选中
function checkCheckBox(proValList){
    //获取当前所有属性
    $("input[type='checkbox']").each(function(){
        var proValCb = $(this);
        $.each(proValList,function(key,i){
            if($(proValCb).attr("data-id") == i.propertyValId){
                $(proValCb).prop("checked",true);
            }
        })
    })
}
$(document).ready(function () {
        searchProperty();
        //回显商品信息
        initGoods();
    });
```
发现类似实现方式，可做参考：
https://www.cnblogs.com/purediy/archive/2012/12/02/2798454.html
