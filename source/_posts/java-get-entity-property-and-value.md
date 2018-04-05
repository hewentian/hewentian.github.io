---
title: java 获取对象的所有属性和它们的值
date: 2017-11-10 00:16:59
tags:
categories: java
---
下面我们展示如何获取对象的所有属性和它们的值，首先我们新建一个实体类：
``` java
/**
 * 
 * <p>
 * <b>User</b> 是 测试用的类
 * </p>
 *
 * @author <a href="mailto:wentian.he@qq.com">hewentian</a>
 * @date 2017年11月10日 上午12:19:18
 * @since JDK 1.8
 *
 */
public class User {
	/** id */
	private Integer id;

	/** 姓名 */
	private String name;

	/** 年龄 */
	private Integer age;

	public User(Integer id, String name, Integer age) {
		super();
		this.id = id;
		this.name = name;
		this.age = age;
	}

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Integer getAge() {
		return age;
	}

	public void setAge(Integer age) {
		this.age = age;
	}
}
```

这里我们用到了`apache`提供的工具包`commons-beanutils-1.9.3.jar`，详细代码如下：
``` java

import java.beans.PropertyDescriptor;
import java.lang.reflect.InvocationTargetException;
import java.util.ArrayList;
import java.util.List;

import org.apache.commons.beanutils.PropertyUtils;

/**
 * 
 * <p>
 * <b>GetPropertyAndValue</b> 是 获取对象的所有属性和它们的值
 * </p>
 *
 * @author <a href="mailto:wentian.he@qq.com">hewentian</a>
 * @date 2017年11月10日 上午12:20:09
 * @since JDK 1.8
 *
 */
public class GetPropertyAndValue {
	public static void main(String[] args) {
		User user = new User(1, "Tim Ho", 23);

		List<String> properties = new ArrayList<String>();

		System.out.println("获取所有属性");
		PropertyDescriptor[] pds = PropertyUtils.getPropertyDescriptors(user.getClass());
		for (PropertyDescriptor pd : pds) {
			System.out.println(pd.getName() + "\t" + pd.getPropertyType());
			properties.add(pd.getName());
		}

		System.out.println("\n根据属性获取这个属性的值");
		properties.stream().forEach(p -> {
			try {
				Object value = PropertyUtils.getProperty(user, p);
				System.out.println(p + "\t" + value);
			} catch (IllegalAccessException | InvocationTargetException | NoSuchMethodException e) {
				e.printStackTrace();
			}
		});
	}
}
```

输出结果如下：
``` java
获取所有属性
name	class java.lang.String
id	class java.lang.Integer
class	class java.lang.Class
age	class java.lang.Integer

根据属性获取这个属性的值
name	Tim Ho
id	1
class	class com.hewentian.User
age	23
```

end.
