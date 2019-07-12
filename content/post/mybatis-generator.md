---
title: "MyBatis Generator"
date: 2019-05-22T08:34:36+08:00
lastmod: 2019-05-22T08:34:36+08:00
draft: false
keywords: ["mybatis"]
description: "mybatis generator"
tags: ["mybatis"]
categories: ["mybatis"]
# author: ""
# comment: false
toc: true
contentCopyright: false
reward: false
mathjax: false
---


## mybatis逆向工程

eclipse作IDE工具，mybatis官方提供了逆向工程，这样针对单表就可以自动生成相应的pojo、mapper、mapper.xml等，且eclipse也都有相应的插件，但是往往不尽人意，这里只是用代码生成，可以相对自定义配置生成想要的格式，对此做个记录以备后用。

<!--more-->

### 1)  generatorConfiguration.xml配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration>

	<!-- 引入配置文件 -->
	<properties resource="jdbc.properties" />
	<!-- 指定数据连接驱动jar地址 -->
	<classPathEntry location="${classpath}" />
	<!-- 一个数据库一个context -->
	<context id="dyb" targetRuntime="MyBatis3">
		<property name="javaFileEncoding" value="UTF-8" />
		<!-- 注释 -->
		<!-- <commentGenerator>
			关闭自动生成的注释 false时打开注释，true时关闭注释
			<property name="suppressAllComments" value="false" />
			生成时间戳 false时打开时间标志，true时关闭
			<property name="suppressDate" value="true" />
		</commentGenerator> -->
		<commentGenerator type="com.upload.controller.MyCommentGenerator" />
		<!-- jdbc连接 -->
		<jdbcConnection driverClass="${master.jdbc.driverClassName}"
			connectionURL="${master.jdbc.url}" userId="${master.jdbc.username}"
			password="${master.jdbc.password}">
			<property name="useInformationSchema" value="true" />
		</jdbcConnection>
		<!-- 类型转换 -->
		<javaTypeResolver>
			<!-- 是否使用bigDecimal， false可自动转化以下类型（Long, Integer, Short, etc.） -->
			<property name="forceBigDecimals" value="false" />
		</javaTypeResolver>

		<!-- 生成实体类地址 -->
		<javaModelGenerator targetPackage="${entitypackage}"
			targetProject="${projectdir}">
			<!-- 是否在当前路径下新加一层schema,eg：false路径com.example.user.model， true:com.example.user.model.[schemaName] -->
			<property name="enableSubPackages" value="false" />
			<!-- 是否针对string类型的字段在set的时候进行trim调用 -->
			<property name="trimStrings" value="true" />
			
		</javaModelGenerator>

		<!-- 生成mapper文件 -->
		<sqlMapGenerator targetPackage="${mapperpackage}"
			targetProject="${projectdir}">
			<!-- 是否在当前路径下新加一层schema,eg：false路径com.example.user.model， true:com.example.user.model.[schemaName] -->
			<property name="enableSubPackages" value="false" />
		</sqlMapGenerator>

		<!-- 生成mapxml -->
		<javaClientGenerator targetPackage="${mapperxmlpackage}"
			targetProject="${projectdir}" type="XMLMAPPER">
			<!-- 是否在当前路径下新加一层schema,eg：fase路径com.example.user.model， true:com.example.user.model.[schemaName] -->
			<property name="enableSubPackages" value="false" />
		</javaClientGenerator>

		<!-- 配置表信息 -->
		<table tableName="tablename" domainObjectName="javaentityname"
			enableCountByExample="false" enableDeleteByExample="false"
			enableSelectByExample="false" enableUpdateByExample="false">
			<property name="useActualColumnNames" value="false" />
		</table>
	</context>
</generatorConfiguration>
```

### 2)  jdbc.properties

```properties
master.jdbc.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8
master.jdbc.username=root
master.jdbc.password=root
#jar的绝对路径
classpath=mysql-connector-java-5.1.34.jar
entitypackage=com.example.entity
mapperpackage=com.example.mapper
mapperxmlpackage=com.example.mapper.xml
projectdir=D:XXXXXX/src/main/java
```

### 3) 自定义转换格式

```java
package com.example.controller;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Properties;
import static org.mybatis.generator.internal.util.StringUtility.isTrue;
import org.mybatis.generator.api.CommentGenerator;
import org.mybatis.generator.api.IntrospectedColumn;
import org.mybatis.generator.api.IntrospectedTable;
import org.mybatis.generator.api.dom.java.CompilationUnit;
import org.mybatis.generator.api.dom.java.Field;
import org.mybatis.generator.api.dom.java.InnerClass;
import org.mybatis.generator.api.dom.java.InnerEnum;
import org.mybatis.generator.api.dom.java.JavaElement;
import org.mybatis.generator.api.dom.java.Method;
import org.mybatis.generator.api.dom.java.Parameter;
import org.mybatis.generator.api.dom.java.TopLevelClass;
import org.mybatis.generator.api.dom.xml.XmlElement;
import org.mybatis.generator.config.MergeConstants;
import org.mybatis.generator.config.PropertyRegistry;

public class MyCommentGenerator implements CommentGenerator {
	private Properties properties;
	private Properties systemPro;
	private boolean suppressDate;
	private boolean suppressAllComments;
	private String currentDateStr;

	public MyCommentGenerator() {
		super();
		properties = new Properties();
		systemPro = System.getProperties();
		suppressDate = false;
		suppressAllComments = false;
		currentDateStr = (new SimpleDateFormat("yyyy-MM-dd")).format(new Date());
	}

	public void addJavaFileComment(CompilationUnit compilationUnit) {
		// add no file level comments by default
		return;
	}

	/**
	 * Adds a suitable comment to warn users that the element was generated, and
	 * when it was generated.
	 */
	public void addComment(XmlElement xmlElement) {
		return;
	}

	public void addRootComment(XmlElement rootElement) {
		return;
	}

	public void addConfigurationProperties(Properties properties) {
		this.properties.putAll(properties);

		suppressDate = isTrue(properties.getProperty(PropertyRegistry.COMMENT_GENERATOR_SUPPRESS_DATE));

		suppressAllComments = isTrue(properties.getProperty(PropertyRegistry.COMMENT_GENERATOR_SUPPRESS_ALL_COMMENTS));
	}

	/**
	 * This method adds the custom javadoc tag for. You may do nothing if you do
	 * not wish to include the Javadoc tag - however, if you do not include the
	 * Javadoc tag then the Java merge capability of the eclipse plugin will
	 * break.
	 * 
	 * @param javaElement
	 *            the java element
	 */
	protected void addJavadocTag(JavaElement javaElement, boolean markAsDoNotDelete) {
		javaElement.addJavaDocLine(" *");
		StringBuilder sb = new StringBuilder();
		sb.append(" * ");
		sb.append(MergeConstants.NEW_ELEMENT_TAG);
		if (markAsDoNotDelete) {
			sb.append(" do_not_delete_during_merge");
		}
		String s = getDateString();
		if (s != null) {
			sb.append(' ');
			sb.append(s);
		}
		javaElement.addJavaDocLine(sb.toString());
	}

	/**
	 * This method returns a formated date string to include in the Javadoc tag
	 * and XML comments. You may return null if you do not want the date in
	 * these documentation elements.
	 * 
	 * @return a string representing the current timestamp, or null
	 */
	protected String getDateString() {
		String result = null;
		if (!suppressDate) {
			result = currentDateStr;
		}
		return result;
	}

	public void addClassComment(InnerClass innerClass, IntrospectedTable introspectedTable) {
		if (suppressAllComments) {
			return;
		}
		StringBuilder sb = new StringBuilder();
		innerClass.addJavaDocLine("/**");
		sb.append(" * ");
		sb.append(introspectedTable.getFullyQualifiedTable());
		sb.append(" ");
		sb.append(getDateString());
		innerClass.addJavaDocLine(sb.toString().replace("\n", " "));
		innerClass.addJavaDocLine(" */");
	}

	public void addEnumComment(InnerEnum innerEnum, IntrospectedTable introspectedTable) {
		if (suppressAllComments) {
			return;
		}
		StringBuilder sb = new StringBuilder();
		innerEnum.addJavaDocLine("/**");
		sb.append(" * ");
		sb.append(introspectedTable.getFullyQualifiedTable());
		innerEnum.addJavaDocLine(sb.toString().replace("\n", " "));
		innerEnum.addJavaDocLine(" */");
	}

	public void addFieldComment(Field field, IntrospectedTable introspectedTable,
			IntrospectedColumn introspectedColumn) {
		if (suppressAllComments) {
			return;
		}
		StringBuilder sb = new StringBuilder();
		field.addJavaDocLine("/**");
		sb.append(" * ");
		sb.append(introspectedColumn.getRemarks());
		field.addJavaDocLine(sb.toString().replace("\n", " "));
		field.addJavaDocLine(" */");
	}

	public void addFieldComment(Field field, IntrospectedTable introspectedTable) {
		if (suppressAllComments) {
			return;
		}
		StringBuilder sb = new StringBuilder();
		field.addJavaDocLine("/**");
		sb.append(" * ");
		sb.append(introspectedTable.getFullyQualifiedTable());
		field.addJavaDocLine(sb.toString().replace("\n", " "));
		field.addJavaDocLine(" */");
	}

	public void addGeneralMethodComment(Method method, IntrospectedTable introspectedTable) {
		if (suppressAllComments) {
			return;
		}
		method.addJavaDocLine("/**");
		addJavadocTag(method, false);
		method.addJavaDocLine(" */");
	}

	public void addGetterComment(Method method, IntrospectedTable introspectedTable,
			IntrospectedColumn introspectedColumn) {
		if (suppressAllComments) {
			return;
		}
		method.addJavaDocLine("/**");
		StringBuilder sb = new StringBuilder();
		sb.append(" * ");
		sb.append(introspectedColumn.getRemarks());
		method.addJavaDocLine(sb.toString().replace("\n", " "));
		sb.setLength(0);
		sb.append(" * @return ");
		sb.append(introspectedColumn.getActualColumnName());
		sb.append(" ");
		sb.append(introspectedColumn.getRemarks());
		method.addJavaDocLine(sb.toString().replace("\n", " "));
		method.addJavaDocLine(" */");
	}

	public void addSetterComment(Method method, IntrospectedTable introspectedTable,
			IntrospectedColumn introspectedColumn) {
		if (suppressAllComments) {
			return;
		}
		method.addJavaDocLine("/**");
		StringBuilder sb = new StringBuilder();
		sb.append(" * ");
		sb.append(introspectedColumn.getRemarks());
		method.addJavaDocLine(sb.toString().replace("\n", " "));
		Parameter parm = method.getParameters().get(0);
		sb.setLength(0);
		sb.append(" * @param ");
		sb.append(parm.getName());
		sb.append(" ");
		sb.append(introspectedColumn.getRemarks());
		method.addJavaDocLine(sb.toString().replace("\n", " "));
		method.addJavaDocLine(" */");
	}

	public void addClassComment(InnerClass innerClass, IntrospectedTable introspectedTable, boolean markAsDoNotDelete) {
		if (suppressAllComments) {
			return;
		}
		StringBuilder sb = new StringBuilder();
		innerClass.addJavaDocLine("/**");
		sb.append(" * ");
		sb.append(introspectedTable.getFullyQualifiedTable());
		innerClass.addJavaDocLine(sb.toString().replace("\n", " "));
		sb.setLength(0);
		sb.append(" * @author ");
		sb.append(systemPro.getProperty("user.name"));
		sb.append(" ");
		sb.append(currentDateStr);
		innerClass.addJavaDocLine(" */");
	}

	public void addModelClassComment(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {

	}
}
```

### 4) 逆向生成文件入口

```java
package com.example.controller;

import java.io.IOException;
import java.io.InputStream;
import java.net.URISyntaxException;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.exception.InvalidConfigurationException;
import org.mybatis.generator.exception.XMLParserException;
import org.mybatis.generator.internal.DefaultShellCallback;

public class StartUp {
	 public static void main(String[] args) throws URISyntaxException {
	        try {
	            List<String> warnings = new ArrayList<String>();
	            boolean overwrite = true;
	            ClassLoader classloader = Thread.currentThread().getContextClassLoader();
	            InputStream is = classloader.getResourceAsStream("generatorConfiguration.xml");
	            ConfigurationParser cp = new ConfigurationParser(warnings);
	            Configuration config = cp.parseConfiguration(is);
	            DefaultShellCallback callback = new DefaultShellCallback(overwrite);
	            MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
	            myBatisGenerator.generate(null);
	            System.out.println("创建实体类结束");
	        } catch (SQLException e) {
	            e.printStackTrace();
	        } catch (IOException e) {
	            e.printStackTrace();
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        } catch (InvalidConfigurationException e) {
	            e.printStackTrace();
	        } catch (XMLParserException e) {
	            e.printStackTrace();
	        }
	    }
}
```

### 5) resources所需文件

```properties
# src/main/resources
jdbc.properties
generatorConfiguration.xml
# 所用jar包
mybatis-generator-core-1.3.5.jar
mysql-connector-java-5.1.34.jar
```
