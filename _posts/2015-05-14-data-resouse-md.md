---
layout:     post
title:      "Spring Boot 学习起步 十、数据访问"
subtitle:   " \"Spring Boot\""
date:       2015-05-14 16:33:13
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> 数据访问

### 数据源配置

修改pom.xml,添加spring-boot-starter-data-jpa及数据库驱动,下面以mysql为例：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>org.test</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.1.8.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<start-class>demo.Application</start-class>
		<java.version>1.7</java.version>
	</properties>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```
<!--more-->
修改application.properties，添加数据库连接信息：
```properties
spring.datasource.url=jdbc:mysql://localhost/demo
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

创建一个实体类：
```java
package demo.model;

import java.io.Serializable;
import java.util.Date;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Demo implements Serializable {
	private static final long serialVersionUID = 2829011618483752235L;

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	private Long id;
	private String name;
	private Date createTime;

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Date getCreateTime() {
		return createTime;
	}

	public void setCreateTime(Date createTime) {
		this.createTime = createTime;
	}

}

```
可以利用hibernate自动建表，只需修改application.properties：
```properties
spring.datasource.url=jdbc:mysql://localhost/demo
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

# JPA (JpaBaseConfiguration, HibernateJpaAutoConfiguration)
spring.jpa.openInView=true
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=create-drop
```
启动项目，控制台输出：
```
2014-11-01 10:50:54.093  INFO 5551 --- [           main] org.hibernate.tool.hbm2ddl.SchemaExport  : HHH000227: Running hbm2ddl schema export
Hibernate: drop table if exists demo
Hibernate: create table demo (id bigint not null auto_increment, create_time datetime, name varchar(255), primary key (id))
```
访问数据库。查看是否建表成功：
```shell
mysql> use demo;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_demo |
+----------------+
| demo           |
+----------------+
1 row in set (0.00 sec)

mysql> desc demo;
+-------------+--------------+------+-----+---------+----------------+
| Field       | Type         | Null | Key | Default | Extra          |
+-------------+--------------+------+-----+---------+----------------+
| id          | bigint(20)   | NO   | PRI | NULL    | auto_increment |
| create_time | datetime     | YES  |     | NULL    |                |
| name        | varchar(255) | YES  |     | NULL    |                |
+-------------+--------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

mysql>

```

### JdbcTemplate
数据源配置好后，可以在代码中通过注入直接使用JdbcTemplate以及NamedParameterJdbcTemplate
```java
package demo.service;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

import javax.transaction.Transactional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Service;

import demo.model.Demo;

@Service
@Transactional
public class DemoService {
	private final JdbcTemplate jdbcTemplate;

	@Autowired
	public DemoService(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}

	public List<Demo> findAll(){
		return jdbcTemplate.query("select * from demo", new RowMapper<Demo>() {

			@Override
			public Demo mapRow(ResultSet rs, int arg1) throws SQLException {
				Demo demo = new Demo();
				demo.setId(rs.getLong("id"));
				demo.setName(rs.getString("name"));
				demo.setCreateTime(rs.getDate("create_time"));

				return demo;
			}

		});
	}
}


```

### Spring Data JPA
也可以通过注入HibernateTemplate，EntityManager的方式操作数据库，但Spring Data JPA对一些诸如分页，排序之类的常用操作提供了更多的便利，看下面的例子：
```java
package demo.repository;

import org.springframework.data.repository.PagingAndSortingRepository;

import demo.model.Demo;

public interface DemoRepository extends PagingAndSortingRepository<Demo, Long> {

}
```
详细的代码参见[example/simple-demo](example/simple-demo).
