---
title: Manage System
tags: java
date: 2019-05-09 10:13:34
---

# 模板
## mapper.xml
```xml
<!-- 根据条件获取用户信息 -->
<select id="getUserByCondition" resultMap="BaseResultMap" parameterType="com.zjx.model.Users">
select
<include refid="Base_Column_List" />
from users
<where>
  <if test="id != null">
	and id = #{id}
  </if>
  <if test="mobile != null">
	and mobile = #{mobile}
  </if>
  <if test="nickname != null and nickname != ''">
	and nickname like concat('%',#{nickname},'%')
  </if>
  <if test="password != null and password != ''">
	and password = #{password}
  </if>
</where>
</select>
```

## DaoMapper.java
增加一个方法返回列表数据，其他方法自动生成
```java
List<Tag> getTagList(Tag tag);
```

## service
```java
PageInfo<Tag> getTagList(int pageNum,int pageSize,Tag tag);

Tag tagDetail(Integer id);

int addOrModify(Tag tag);

int delTag(Integer id);
```
## serviceImpl
```java
@Override
public PageInfo<Tag> getTagList(int pageNum,int pageSize, Tag tag) {
	PageHelper.startPage(pageNum,pageSize);
	return new PageInfo<Tag>(tagMapper.getTagList(tag));
}

/**
 * @author: wjy
 * @description: 获取所有tag 不分页
 */
@Override
public List<Tag> getTagList(Tag tag) {
	return tagMapper.getTagList(tag);
}

/**
 * @author: wjy
 * @description: tag详情
 */
@Override
public Tag tagDetail(Integer id) {
	return tagMapper.selectByPrimaryKey(id);
}

/**
 * @author: wjy
 * @description: 新增/修改
 */
@Override
public int addTag(Tag tag) {
	int r = 0;
	if(null != tag.getId()){
		r = tagMapper.updateByPrimaryKeySelective(tag);
	}else{
		r = tagMapper.insertSelective(tag);
	}
	//其他业务逻辑
	
	
	return r;
}


/**
 * @author: wjy
 * @description: 删除tag
 */
@Override
public int delTag(Integer id) {
	return tagMapper.deleteByPrimaryKey(id);
}
```

## controller
```java
/**
 * @author: wjy
 * @description: 列表
 */
@RequestMapping("/index")
public ModelAndView flagManage(@RequestParam(value = "pageNum", required = false, defaultValue = "1") Integer pageNum,
							   @RequestParam(value = "pageSize", required = false, defaultValue = "10") Integer pageSize,
							   FlagDto flagDto) {
	ModelAndView mv = new ModelAndView("flag/flagManage");
	mv.addObject("model",flagDto);
	mv.addObject("pageInfo",flagService.getAllFlag(pageSize, pageNum, flagDto));
	return mv;
}

/**
 * @author: wjy
 * @description: 详情
 */
@RequestMapping("/detail")
public ModelAndView detail(Flag flag){
	ModelAndView mv = new ModelAndView("flag/flagDetail");
	mv.addObject("model",flagService.getFlagDetail(flag.getId()));
	return mv;
}

/**
 * @author: wjy
 * @description: 新增/修改(审核)
 */
@ResponseBody
@RequestMapping("/addOrModify")
public int addOrModify(Flag flag){
	return tagManageService.addOrModify(flag);
}

/**
 * @author: wjy
 * @description: 删除
 */
@ResponseBody
@RequestMapping("/del")
public int del(Integer id){
	return tagManageService.delTag(id);
}
```