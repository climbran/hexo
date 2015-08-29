title: spring、mybatis、postgresql整合
date: 2015-08-29 14:18:52
tags: [Spring MVC,mybatis,postgresql]
---

pom.xml增加以下依赖：
{% codeblock pom.xml %}

	<properties>
		......
        
        <jdbc.driver.groupId>postgresql</jdbc.driver.groupId>
        <jdbc.driver.artifactId>postgresql</jdbc.driver.artifactId>
        <jdbc.driver.version>9.1-901.jdbc4</jdbc.driver.version>
	</properties>
    
	<dependencies>
    	......
		
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
        </dependency>

		<!-- connection pool -->
        <dependency>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>tomcat-jdbc</artifactId>
            <version>${tomcat-jdbc.version}</version>
            <scope>runtime</scope>
        </dependency>

        <!-- jdbc driver -->
        <dependency>
            <groupId>${jdbc.driver.groupId}</groupId>
            <artifactId>${jdbc.driver.artifactId}</artifactId>
            <version>${jdbc.driver.version}</version>
            <scope>runtime</scope>
        </dependency>

        <!-- mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.3.0</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.2.2</version>
        </dependency>
	</dependencies>
{% endcodeblock %}

在applicationContext.xml中增加以下配置，其中${jdbc...}配置在application.properties中
{% codeblock applicationContext.xml %}	
	......
	
    <context:property-placeholder ignore-unresolvable="true" location="classpath*:/application.properties" />
 
    <!-- 事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
 	
    <!-- 创建SqlSessionFactory，同时指定数据源-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <!-- 自动扫描entity目录, 省掉Configuration.xml里的手工配置 -->
        <property name="typeAliasesPackage" value="com.climbran.spring.entity" />
        <!-- 显式指定Mapper文件位置 -->
        <property name="mapperLocations" value="classpath:/mybatis/*Mapper.xml" />
    </bean>
    
    <!-- 扫描basePackage下所有以@MyBatisRepository标识的 接口-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.climbran.spring.repository" />
        <property name="annotationClass" value="com.climbran.spring.repository.MyBatisRepository"/>
    </bean>

    <!-- 数据源配置, 使用Tomcat JDBC连接池 -->
    <bean id="dataSource" class="org.apache.tomcat.jdbc.pool.DataSource" destroy-method="close">
        <!-- Connection Info -->
        <property name="driverClassName" value="${jdbc.driver}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />

        <!-- Connection Pooling Info -->
        <property name="maxActive" value="${jdbc.pool.maxActive}" />
        <property name="maxIdle" value="${jdbc.pool.maxIdle}" />
        <property name="minIdle" value="0" />
        <property name="defaultAutoCommit" value="false" />
    </bean>
{% endcodeblock %}

在src/main/resources下新建application.properties文件：
{% codeblock application.properties %}
jdbc.driver=org.postgresql.Driver
jdbc.url=jdbc:postgresql://127.0.0.1:5432/test
jdbc.username=数据库用户名
jdbc.password=数据库密码
jdbc.pool.maxIdle=10
jdbc.pool.maxActive=50
{% endcodeblock %}

applicationContext.xml文件中，SqlSessionFactory的typeAliasesPackage配置entity的扫描目录，mapperLocations配置mybatis映射文件，示例如下：
{% codeblock User.java%}
package com.climbran.spring.entity;

/**
 * @author climbran
 */
public class User {
    private Integer id;
    private String username;
    private String schoolName;

    public Integer getId(){
        return id;
    }
    public void setId(Integer id){
        this.id = id;
    }

    public String getUsername(){
        return username;
    }

    public void  setUsername(String username){
        this.username = username;
    }

    public String getSchoolName(){
        return schoolName;
    }

    public void  setSchoolNamer(String schoolName){
        this.schoolName = schoolName;
    }
}
{% endcodeblock %}

在src/main/resources下新建文件夹mybatis,再在mybatis下新建文件UserMapper.xml
{% codeblock UserMapper.xml%}
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace必须指向Dao接口 -->
<mapper namespace="com.climbran.spring.repository.UserDao">
    <!--
        获取用户: 输出直接映射到对象, school_name列要"as schoolName"以方便映射
    -->
    <select id="get" parameterType="integer" resultType="User">
        select id, username,
        school_name as schoolName
        from public.user
        where id=#{id}
    </select>

</mapper>
{% endcodeblock %}
以前使用的JPA貌似public schema下载@Table注解中public可以省略，不过mybatis测试时from语句不写public会报错BadSqlGrammarException

applicationContext.xml中org.mybatis.spring.mapper.MapperScannerConfigurer配置了dao层文件所在位置，所以根据UserMapper.xml中的配置，对应的java文件为：
{% codeblock UserDao.java %}
package com.climbran.spring.repository;

import com.climbran.spring.entity.User;

/**
 * @author climbran
 */
@MyBatisRepository
public interface UserDao {
    User get(Integer id);
}
{% endcodeblock %}

org.mybatis.spring.mapper.MapperScannerConfigurer的annotationClass配置是表示已注解方式扫描dao，自动注册有value值对应注解的bean，例如上面UserDao的@MyBatisRepository注解，其是实现如下：
{% codeblock UserDao.xml%}
package com.climbran.spring.repository;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.stereotype.Component;

/**
 * 标识MyBatis的DAO,方便{@link org.mybatis.spring.mapper.MapperScannerConfigurer}的扫描。
 *
 * @author climbran
 *
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Component
public @interface MyBatisRepository {
    String value() default "";
}
{% endcodeblock %}

使用时在Service中通过@Autowired注入UserDao即可访问数据库。
最后是UserMapper.xml映射的表：
{% codeblock %}
-- Table: "user"

-- DROP TABLE "user";

CREATE TABLE "user"
(
  id serial NOT NULL,
  username character varying,
  school_name character varying,
  CONSTRAINT user_pkey PRIMARY KEY (id)
)
WITH (
  OIDS=FALSE
);
ALTER TABLE "user"
  OWNER TO postgres;
{% endcodeblock %}