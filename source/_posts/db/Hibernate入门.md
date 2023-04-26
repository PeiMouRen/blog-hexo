---
title: Hibernate入门
date: 2022-08-16 14:20:16
categories: ORM
tags: 
  - Hibernate
---

[TOC]



# 下载

## 不使用maven

```http
# 官网5.4版本zip下载地址
https://hibernate.org/orm/releases/5.4/
```

![资源包下载](资源包下载.jpg)

下载完成后引入`lib/required`目录下的jar包即可。

## 使用maven

```http
# mvn仓库搜索hibernate
https://mvnrepository.com/
```

```xml
<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-core -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.18.Final</version>
</dependency>
```

# 使用

## 创建配置文件

`resources`目录下新建`hibernate.cfg.xml`文件，配置文件示例在`${hibernate包根目录}/project/etc`下的`hibernate.cfg.xml`和`hibernate.properties`中查看。

```xml
<!--
  ~ Hibernate, Relational Persistence for Idiomatic Java
  ~
  ~ License: GNU Lesser General Public License (LGPL), version 2.1 or later.
  ~ See the lgpl.txt file in the root directory or <http://www.gnu.org/licenses/lgpl-2.1.html>.
  -->
<!DOCTYPE hibernate-configuration PUBLIC
	"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
	<session-factory>
		<!-- 数据库驱动 -->
		<property name="hibernate.connection.driver_class">org.postgresql.Driver</property>
		<!-- 数据库url -->
		<property name="hibernate.connection.url">jdbc:postgresql://192.168.5.112:5432/genericdb</property>
		<!-- 数据库连接用户名 -->
		<property name="hibernate.connection.username">postgres</property>
		<!-- 数据库连接密码 -->
		<property name="hibernate.connection.password">postgres</property>
		<!-- 数据库方言
			不同的数据库中,sql语法略有区别. 指定方言可以让hibernate框架在生成sql语句时.针对数据库的方言生成.
			sql99标准: DDL 定义语言  库表的增删改查
					  DCL 控制语言  事务 权限
					  DML 操纵语言  增删改查
			注意: MYSQL在选择方言时,请选择最短的方言.
		 -->
		<property name="hibernate.dialect">org.hibernate.dialect.PostgreSQL82Dialect</property>


		<!-- #hibernate.show_sql true
			 #hibernate.format_sql true
		-->
		<!-- 将hibernate生成的sql语句打印到控制台 -->
		<property name="hibernate.show_sql">true</property>
		<!-- 将hibernate生成的sql语句格式化(语法缩进) -->
		<property name="hibernate.format_sql">true</property>
		<!--
		## auto schema export  自动导出表结构. 自动建表
		#hibernate.hbm2ddl.auto create		自动建表.每次框架运行都会创建新的表.以前表将会被覆盖,表数据会丢失.(开发环境中测试使用)
		#hibernate.hbm2ddl.auto create-drop 自动建表.每次框架运行结束都会将所有表删除.(开发环境中测试使用)
		#hibernate.hbm2ddl.auto update(推荐使用) 自动生成表.如果已经存在不会再生成.如果表有变动.自动更新表(不会删除任何数据).
		#hibernate.hbm2ddl.auto validate	校验.不自动生成表.每次启动会校验数据库中表是否正确.校验失败.
		 -->
		<property name="hibernate.hbm2ddl.auto">update</property>
        
        <!-- 配置线程上下文 -->
		<property name = "hibernate.current_session_context_class">thread</property>
        
        <!-- 加载映射文件 -->
		<mapping resource="mapper/Department.hbm.xml" />
		
	</session-factory>
</hibernate-configuration>
```

## 创建实体类

```java
package com.lhx.orm.entity;

import lombok.Data;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Department {

    private Integer deptNo;
    private String deptName;
    private String location;

}
```

## 添加映射文件

在`resources`目录下创建`mapper`目录，存放实体类和数据库表的映射文件，格式为`entityName.hbm.xml`，如创建`Department`实体的映射文件`Department.hbm.xml`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<!-- 配置表与实体对象的关系 -->
<!-- package属性：填写一个包名，在元素内部凡是需要书写完整类名的属性，可以直接写类名 -->
<hibernate-mapping package="com.lhx.orm.entity">
    <!--
        class元素：配置实体与表的对应关系的
        name:完整类名
        table：数据库表名

     -->
    <class name="Department" table="t_department">
        <!-- id元素：配置主键映射的属性
            name：填写主键对应属性名
            colunm（可选）：填写表中的主键的列名，默认值：列名会默认使用属性名
            type(可选)：填写列（属性）的类型。如果不写hibernate会自动检测属性类型，
                    但是如果手写可以有三种语法java类型|hibernate类型|数据库类型
            not-null(可选)：配置该属性（列）是否不能为空，默认值为：false
            length（可选）:：配置数据库中列的长度，默认值：使用数据库类型的最大长度。如：varchar（255位），int（9位）
         -->
        <id name="deptNo" type="java.lang.Integer" column="deptNo">
            <!--
                主键生成策略
                assigned:程序生成主键，默认策略
                increment：主键按数值顺序递增。此方式的实现机制为在当前应用实例中维持一个变量，以保存着当前的最大值，之后每次需要生成主键的时候将此值加1作为主键
                sequence：采用数据库提供的sequence 机制生成主键。如Oralce 中的Sequence
                native：由Hibernate根据底层数据库自行判断采用identity、hilo、sequence其中一种作为主键生成方式
            -->
            <generator class="native"></generator>
        </id>
        <!-- property除di之外的普通属性映射
            name：填写主键对应属性名
            colunm：填写表中的列名
            type(可选)：填写列（属性）的类型。如果不写hibernate会自动检测属性类型，
                    但是如果手写可以有三种语法java类型|hibernate类型|数据库类型
            not-null(可选)：配置该属性（列）是否不能为空，默认值为：false
            length（可选）:：配置数据库中列的长度，默认值：使用数据库类型的最大长度。如：varchar（255位），int（9位）

         -->
        <property name="deptName" type="java.lang.String" column="deptName"></property>
        <property name="location" type="java.lang.String" column="location"></property>
    </class>
</hibernate-mapping>
```

## 加载映射文件

将映射文件`Department.hbm.xml`配置到`hibernate.cfg.xml`里。

```xml
<!DOCTYPE hibernate-configuration PUBLIC
	"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
	<session-factory>
		<!-- 其他配置 -->

		<!-- 加载映射文件 -->
		<mapping resource="mapper/Department.hbm.xml" />
	</session-factory>
</hibernate-configuration>
```

## 测试

### 新增部门

```java
package com.lhx.orm;

import com.lhx.orm.entity.Department;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;
import org.junit.Test;

public class DepartmentTest {

    @Test
    public void testAddDep() {
        // 读取hibernate配置文件
        Configuration configure = new Configuration().configure();
        // 创建SessionFactory对象
        SessionFactory sessionFactory = configure.buildSessionFactory();
        // 创建Session对象
        Session session = sessionFactory.openSession();
        // 开启事务
        Transaction transaction = session.beginTransaction();
        // 进行持久化操作
        Department department = new Department();
        department.setDeptName("技术部");
        department.setLocation("北京");
        try {
            session.save(department);
            // 提交事务
            transaction.commit();
        } catch (Exception e) {
            e.printStackTrace();
            // 回滚事务
            transaction.rollback();
        }

        // 释放资源
        session.close();
        sessionFactory.close();
    }

}
```

### 创建工具类

```java
package com.lhx.orm.util;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {

    private static SessionFactory sessionFactory;

    static {
        // 读取hibernate配置文件
        Configuration configure = new Configuration().configure();
        // 创建SessionFactory对象
        sessionFactory = configure.buildSessionFactory();
    }

    public static Session getSession() {
        // 此方式需要在hibernate.cfg.xml中配置
        // <property name = "hibernate.current_session_context_class">thread</property>
        // return sessionFactory.getCurrentSession();
        return sessionFactory.openSession(); // 此方式需要在最后手动关闭session
    }

    public static void closeSession(Session session) {
        if (session != null && session.isOpen()) {
            session.close();
        }
    }
}
```

### session其他api

```java
// 根据主键查询
// 不管后续有没有用到，都会查库，主键不存在返回null
Department department = session.get(Department.class, 2);
// 后续用到了才去查库，主键不存在报错
Department department = session.load(Department.class, 2);

// 删除
Department department = session.get(Department.class, 2);
session.delete(department);

// 更新
Department department = session.get(Department.class, 2);
department.setLocation("武汉");
session.update(department);
```

## HQL

```java
// 查询列表
String hql = "from Department where deptName like :name and location = :location"; // Deparment是实体名，deptName和location是属性名
Query query = session.createQuery(hql);
query.setParameter("name", "%技%");
query.setParameter("location", "上海");
List<Department> list = query.list();

// 返回唯一记录
String hql = "select count(1) from Department";
Query query = session.createQuery(hql);
Long count = (Long)query.uniqueResult();

// 分页查询
String hql = "from Department order by deptNo desc";
Query query = session.createQuery(hql);
List<Department> list = query.setFirstResult(0).setMaxResults(2).list();

// 投影查询
String hql = "select deptName from Department";
Query query = session.createQuery(hql);
List<Object> list = query.list();

String hql = "select deptName, location from Department";
Query query = session.createQuery(hql);
List<Object[]> list = query.list();

String hql = "select new Department(deptName, location) from Department"; // 需要有对应的构造方法
Query query = session.createQuery(hql);
List<Department> list = query.list();

// 原生sql查询
String sql = "select * from t_department where deptno > :deptno";
NativeQuery nativeQuery = session.createSQLQuery(sql).addEntity(Department.class);
nativeQuery.setParameter("deptno", 2);
List<Department> list = nativeQuery.list();
```

## 一对多关系映射

假设存在**一个部门对应多个员工**的映射关系，则实体类和映射文件如下：

**部门实体类**

```java
package com.lhx.orm.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.HashSet;
import java.util.Set;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Department {

    private Integer deptNo;
    private String deptName;
    private String location;
    // 属于该部门的员工信息
    private Set<Employee> employees = new HashSet<>();

}
```

**员工实体类**

```java
package com.lhx.orm.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Employee {
    private Integer empNo;
    private String empName;
    // 保存员工对应的部门
    private Department department;
}
```

**部门映射文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<!-- 配置表与实体对象的关系 -->
<!-- package属性：填写一个包名，在元素内部凡是需要书写完整类名的属性，可以直接写类名 -->
<hibernate-mapping package="com.lhx.orm.entity">
    <!--
        class元素：配置实体与表的对应关系的
        name:完整类名
        table：数据库表名

     -->
    <class name="Department" table="t_department">
        <!-- id元素：配置主键映射的属性
            name：填写主键对应属性名
            colunm（可选）：填写表中的主键的列名，默认值：列名会默认使用属性名
            type(可选)：填写列（属性）的类型。如果不写hibernate会自动检测属性类型，
                    但是如果手写可以有三种语法java类型|hibernate类型|数据库类型
            not-null(可选)：配置该属性（列）是否不能为空，默认值为：false
            length（可选）:：配置数据库中列的长度，默认值：使用数据库类型的最大长度。如：varchar（255位），int（9位）
         -->
        <id name="deptNo" type="java.lang.Integer" column="deptNo">
            <!--
                主键生成策略
                assigned:程序生成主键，默认策略
                increment：主键按数值顺序递增。此方式的实现机制为在当前应用实例中维持一个变量，以保存着当前的最大值，之后每次需要生成主键的时候将此值加1作为主键
                sequence：采用数据库提供的sequence 机制生成主键。如Oralce 中的Sequence
                native：由Hibernate根据底层数据库自行判断采用identity、hilo、sequence其中一种作为主键生成方式
            -->
            <generator class="native"></generator>
        </id>
        <property name="deptName" type="java.lang.String" column="deptName"></property>
        <property name="location" type="java.lang.String" column="location"></property>

        <!--
            配置一对多
            name:实体类的属性名
            cascade:级联操作
            inverse:维护关联关系的方向，默认为false，true表示不主动维护关系
        -->
        <set name="employees" cascade="all" inverse="true">
            <!-- 外键字段 -->
            <key column="deptNo" />
            <!-- 属性的类型 -->
            <one-to-many class="Employee"/>
        </set>
        
    </class>
</hibernate-mapping>
```

**员工映射文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.lhx.orm.entity">

    <class name="Employee" table="t_employee">
        <id name="empNo" type="java.lang.Integer" column="empNo">
            <generator class="native"></generator>
        </id>
        <property name="empName" type="java.lang.String" column="empName"></property>
        <!--
            多对一关系映射
            name：属性名
            column：表列名
            class：属性类型
        -->
        <many-to-one name="department" column="deptNo" class="Department"/>

    </class>
</hibernate-mapping>
```

### 测试

```java
@Test
// 测试保存一个员工
public void testSaveEmp() {
    Session session = HibernateUtil.getSession();
    Transaction transaction = session.beginTransaction();
    Department department = session.get(Department.class, 1);
    Employee employee = new Employee();
    employee.setEmpName("张三");
    employee.setDepartment(department);
    session.save(employee);
    transaction.commit();
    HibernateUtil.closeSession(session);
}

 @Test
// 添加部门的同时添加员工
// 这里遇到一个坑：注意使用lombok时，hashcode和tostring方法的递归调用，导致内存溢出
public void testSaveDeptAndEmp() {
    Session session = HibernateUtil.getSession();
    Transaction transaction = session.beginTransaction();
    Department department = new Department();
    department.setDeptName("设计部");
    department.setLocation("杭州");
    Employee employee = new Employee();
    employee.setEmpName("李四");
    employee.setDepartment(department);
    Employee employee1 = new Employee();
    employee1.setEmpName("王五");
    employee1.setDepartment(department);
    department.getEmployees().add(employee);
    department.getEmployees().add(employee1);
    session.save(department);
    transaction.commit();
    HibernateUtil.closeSession(session);
}
```

## 多对多关系映射

**存在用户和角色的多对多关系，实体类和映射文件如下**

**用户实体**

```java
package com.lhx.orm.entity;

import lombok.Getter;
import lombok.Setter;

import java.util.HashSet;
import java.util.Set;

@Setter
@Getter
public class User {

    private Integer id;
    private String name;
    private Set<Role> roles = new HashSet<>();

}
```

**角色实体**

```java
package com.lhx.orm.entity;

import lombok.Getter;
import lombok.Setter;

import java.util.HashSet;
import java.util.Set;

@Setter
@Getter
public class Role {

    private Integer id;
    private String name;
    private Set<User> users = new HashSet<>();

}
```

**用户映射文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.lhx.orm.entity">

    <class name="User" table="t_user">
        <id name="id" type="java.lang.Integer" column="id">
            <generator class="native"></generator>
        </id>
        <property name="name" type="java.lang.String" column="name"></property>

        <!--
            配置多对多映射
            name：属性名
            table：存放外键字段的表名
            cascade：级联关系
        -->
        <set name="roles" table="user_role" cascade="all">
            <!-- user_role表中与当前类相关的外键字段 -->
            <key column="user_id"/>
            <!--
                class:name属性的数据类型
                column:name属性相关的外键字段
             -->
            <many-to-many class="Role" column="role_id" />
        </set>

    </class>
</hibernate-mapping>

```

**角色映射文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.lhx.orm.entity">

    <class name="Role" table="t_role">
        <id name="id" type="java.lang.Integer" column="id">
            <generator class="native"></generator>
        </id>
        <property name="name" type="java.lang.String" column="name"></property>

        <!--
            配置多对多映射
            name：属性名
            table：存放外键字段的表名
            cascade：级联关系
        -->
        <set name="users" table="user_role" cascade="all">
            <!-- user_role表中与当前类相关的外键字段 -->
            <key column="role_id"/>
            <!--
                class:name属性的数据类型
                column:name属性相关的外键字段
             -->
            <many-to-many class="User" column="user_id" />
        </set>

    </class>
</hibernate-mapping>
```

### 测试

```java
@Test
// 添加角色和属性
public void saveUserAndRole() {

    Session session = HibernateUtil.getSession();
    Transaction transaction = session.beginTransaction();

    User user1 = new User();
    user1.setName("zhangsan");
    User user2 = new User();
    user2.setName("lisi");

    Role role1 = new Role();
    role1.setName("chaxun");
    Role role2 = new Role();
    role2.setName("shanchu");

    user1.getRoles().add(role1);
    user1.getRoles().add(role2);
    user2.getRoles().add(role1);
    user2.getRoles().add(role2);

    try {
        session.save(user1);
        session.save(user2);
        transaction.commit();
    } catch (Exception e) {
        e.printStackTrace();
        transaction.rollback();
    } finally {
        HibernateUtil.closeSession(session);
    }

}
```

## 使用注解

使用注解可代理实体类的Entity.hbm.xml映射文件，常用注解如下：

- @Entity 声明类为持久化类
- @Table 为持久化类指定映射的表名
- @Id 声明属性为表的主键
- @GenerateValue 声明主键的生成策略
- @SequenceGenerator 定义序列产生器
- @Column 声明属性为表中的一列
- @Transient 忽略属性
- @OneToMany 一对多关系映射
- @ManyToOne 多对一关系映射
- @JoinColumn 映射列
- @ManyToMany 多对多关系映射
- @JoinTable 映射表
- @OneToOne 一对一关系映射

示例：

```java
package com.lhx.orm.entity;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Setter
@Getter
@Entity
@Table(name = "t_user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;
    @Column(name = "name")
    private String name;
    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(name = "user_role",
            joinColumns = @JoinColumn(name = "user_id"),
            inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();

}
```

`hibernate.cfg.xml`中引入该实体类

```xml
<mapping class="com.lhx.orm.entity.User"/>
```

