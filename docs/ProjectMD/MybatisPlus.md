## Mybatis Plus简介
官网地址：https://mp.baomidou.com/guide/

简介：
> Mybatis-Plus(简称MP)是一个Mybatis的增强工具，在MyBatis的基础上只做增强不做改变，为简化开发、提高效率而生。

特性：
- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

## 实战 第一个Mybatis-Plus程序
### 1.新建数据库mybatis_plus
```
DROP TABLE IF EXISTS user;
CREATE TABLE user
(
 id BIGINT(20) NOT NULL COMMENT '主键ID',
 name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
 age INT(11) NULL DEFAULT NULL COMMENT '年龄',
 email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
 PRIMARY KEY (id) );
```
###  2.新建User表
```
DELETE FROM user; 
 INSERT INTO user (id, name, age, email) VALUES
 (1, 'sxq', 18, 'test1@baomidou.com'), 
 (2, 'Jack', 20, 'test2@baomidou.com'), 
 (3, 'Tom', 28, 'test3@baomidou.com'), 
 (4, 'Sandy', 21, 'test4@baomidou.com'), 
 (5, 'Billie', 24, 'test5@baomidou.com');
```
### 3.创建项目
> 一个普通的springboot项目,添加依赖
```
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.3.1</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

```
###  4.编写application.properties文件(也可用yaml文件)
```
#mysql8以下用这个
#mysql数据库连接
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus
spring.datasource.username=root
spring.datasource.password=1234


#mysql8以上
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus?
serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=1234
```
###  5.写个实体类User
```
package com.sxq.mpdemo.entity;

import lombok.Data;

/**
 * @author sxq
 */
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}

```

###  6.写User的Mapper

```
package com.sxq.mpdemo.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.sxq.mpdemo.entity.User;
import org.springframework.stereotype.Repository;


/**
 * @author sxq
 *用Repository：是为了将UserMapper交给spring管理，后面偷懒要用。这里使用@Compont,@Service也行
 */
@Repository
public interface UserMapper extends BaseMapper<User> {
}
```

> 继承Mybatis-Plus的BaseMapper,这里面已经写好了很多方法，很方便。

###  7.测试一下，通过自带的test文件，非常方便

```
package com.sxq.mpdemo;

import com.sxq.mpdemo.entity.User;
import com.sxq.mpdemo.mapper.UserMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class MpdemoApplicationTests {

    @Autowired
    private UserMapper userMapper;

    //查询User表中的所有数据
    @Test
    void findAll() {

        List<User> users = userMapper.selectList(null);
        System.out.println(users);

    }

}


部分结果显示：
2021-10-23 16:33:50.222  INFO 27632 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
[User(id=1, name=sxq, age=18, email=test1@baomidou.com), User(id=2, name=Jack, age=20, email=test2@baomidou.com), User(id=3, name=Tom, age=28, email=test3@baomidou.com), User(id=4, name=Sandy, age=21, email=test4@baomidou.com), User(id=5, name=Billie, age=24, email=test5@baomidou.com)]
2021-10-23 16:33:50.311  INFO 27632 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated..
```

成功！！！
结束了？加个日志在application.properties


```
#mysql数据库连接
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus
spring.datasource.username=root
spring.datasource.password=1234
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
#再次运行结果
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@4bafe935] was not registered for synchronization because synchronization is not active
2021-10-23 16:58:41.620  INFO 27752 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-10-23 16:58:41.627  WARN 27752 --- [           main] com.zaxxer.hikari.util.DriverDataSource  : Registered driver with driverClassName=com.mysql.jdbc.Driver was not found, trying direct instantiation.
2021-10-23 16:58:42.832  INFO 27752 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
JDBC Connection [HikariProxyConnection@65894433 wrapping com.mysql.cj.jdbc.ConnectionImpl@3bbf841e] will not be managed by Spring
==>  Preparing: SELECT id,name,age,email FROM user
==> Parameters: 
<==    Columns: id, name, age, email
<==        Row: 1, sxq, 18, test1@baomidou.com
<==        Row: 2, Jack, 20, test2@baomidou.com
<==        Row: 3, Tom, 28, test3@baomidou.com
<==        Row: 4, Sandy, 21, test4@baomidou.com
<==        Row: 5, Billie, 24, test5@baomidou.com
<==      Total: 5
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@4bafe935]
[User(id=1, name=sxq, age=18, email=test1@baomidou.com), User(id=2, name=Jack, age=20, email=test2@baomidou.com), User(id=3, name=Tom, age=28, email=test3@baomidou.com), User(id=4, name=Sandy, age=21, email=test4@baomidou.com), User(id=5, name=Billie, age=24, email=test5@baomidou.com)]

```

连查询的sql语句都给你找出来

## 实现添加操作

### test中新建addUser方法

```
 //添加操作
    @Test
    void addUser() {
        User user = new User();
        user.setName("liuton");
        user.setAge(20);
        user.setEmail("liutong@qq.com");
        int insert = userMapper.insert(user);
        System.out.println("insert:"+insert);

    }
返回结果
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@507b79f7] was not registered for synchronization because synchronization is not active
2021-10-23 17:07:37.762  INFO 16740 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-10-23 17:07:37.767  WARN 16740 --- [           main] com.zaxxer.hikari.util.DriverDataSource  : Registered driver with driverClassName=com.mysql.jdbc.Driver was not found, trying direct instantiation.
2021-10-23 17:07:39.190  INFO 16740 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
JDBC Connection [HikariProxyConnection@1453650546 wrapping com.mysql.cj.jdbc.ConnectionImpl@3ee0b4f7] will not be managed by Spring
==>  Preparing: INSERT INTO user ( id, name, age, email ) VALUES ( ?, ?, ?, ? )
==> Parameters: 1451837706610008066(Long), liuton(String), 20(Integer), liutong@qq.com(String)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@507b79f7]
insert1
2021-10-23 17:07:39.266  INFO 16740 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-10-23 17:07:39.269  INFO 16740 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.

Process finished with exit code 0

```
> 在代码中没有设置id值，数据库中也没有设置id自增，但是在结果中确有id。**这个id值mp自动生成的，一共有19位**。

> 返回insert的结果是1，代表影响行数，就是新增的一条数据。

## 主键策略
Mybatis-Plus默认的主键策略是:ID_WORKER全局唯一ID

分布式系统唯一ID生成方案
https://www.cnblogs.com/haoxinyue/p/5208136.html

主键策略
- 1.自动增长 AOTO_INCREMENT
> 缺点，在分表时，新表的id值需要上一张表的最后一个id+1
- 2.UUID 每次生成一个随机唯一的值
> 缺点，没法进行排序
- 3.Redis生成ID
> 不依赖与数据库，灵活方便，且性能优于数据库。数字ID天然排序，对分页或者需要排序的结果很有帮助。缺点，需要引入新的组件，增加系统复杂度
- 4.mp自带策略snowflake算法

### @TableId(type = IdType.?)
AUTO:自动增长 

INPUT : 设置ID值

NONE : 没有策略

UUID : 随机唯一值

ASSING_UUID:mp自带策略

自3.3.0开始,默认使用雪花算法+UUID(不含中划线)

## 自动填充
> 原本实现的方式通过set方式，将属性值注入。mp简化了这个操作通过自动填充。

### 自动填充实现方式
1.在实体类中进行自动填充属性添加注解
```
 /**
     * create_time
     * 在添加的过程中，这个数据会赋值
     */
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;
    /**
     * update_time
     * 在添加和修改的时候都有值
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
```
 2.创建类，实现MetaObjectHandler实现接口里面的方法
```
package com.sxq.mpdemo.Handler;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * @author sxq
 * Component:交给spring管理
 */
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    /**
     * 使用mp实现添加操作，这个方法就会执行
     * @param metaObject
     */
    @Override
    public void insertFill(MetaObject metaObject) {

        this.setFieldValByName("createTime",new Date(),metaObject);
        this.setFieldValByName("updateTime",new Date(),metaObject);
    }
    /**
     * 使用mp实现修改操作，这个方法就会执行
     * @param metaObject
     */
    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("updateTime",new Date(),metaObject);
    }
}
```

## 乐观锁
> 主要解决丢失更新问题(多个人同时修改同一条数据，最后一个提交的人会把之前提交数据的人的数据覆盖)

悲观锁，就是一种串行的方式，每次只能有一个人执行。
### 乐观锁实现方式
- 去除记录时，获取当前version
- 更新时，带上这个version
- 执行更新时，set version = new Version where version = old version
- 如果version不对，就更新失败


### 具体实现

- 1.在表中添加字段version，作为乐观锁版本号

- 2.对应实体类添加版本号属性
    @Version
    private Integer version;
    
- 3.配置乐观锁插件
#新建一个config包，在里面新建MpConfig配置类

```
package com.sxq.mpdemo.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;

import com.baomidou.mybatisplus.extension.plugins.inner.OptimisticLockerInnerInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author sxq
 * Mybatis-Plus配置类
 */
@Configuration
@MapperScan("com.sxq.mpdemo.mapper")
public class MpConfig {


    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        /**
         * 乐观锁插件
         */
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        /**
         * 分页插件
         */
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.H2));

        return mybatisPlusInterceptor;
    }


}

```
- 4.测试乐观锁
注意：乐观锁触发要先查再改

```
	/**
     * 测试乐观锁
     * 先查再改
     */
    @Test
    void testOptimisticLocker(){

        //根据id查数据
        User user = userMapper.selectById(1452215762650460162l);
        //修改
        user.setAge(200);
        int row = userMapper.updateById(user);
        System.out.println(row);

    }
```

## mp查询
### - 1.根据id查询
> User user = userMapper.selectById(1452215762650460162l);

### - 2.根据多个id查询
>  List<User> users = userMapper.selectBatchIds(Arrays.asList(1l, 2l, 3l));

### - 3.根据id查询.通过封装map做一个简单的条件查询

```
	/**
     * 通过封装map做一个简单的条件查询
     */
    @Test
    public void testSelectByMap() {
        HashMap<String, Object> map = new HashMap<>();
        //条件为name="sxq",age=18
        map.put("name", "sxq");
        map.put("age", 18);
        List<User> users = userMapper.selectByMap(map);
        users.forEach(System.out::println);
    }
```
#结果

```java
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@71cb3139] was not registered for synchronization because synchronization is not active
2021-10-24 18:32:31.381  INFO 3256 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-10-24 18:32:31.386  WARN 3256 --- [           main] com.zaxxer.hikari.util.DriverDataSource  : Registered driver with driverClassName=com.mysql.jdbc.Driver was not found, trying direct instantiation.
2021-10-24 18:32:32.517  INFO 3256 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
JDBC Connection [HikariProxyConnection@1977493952 wrapping com.mysql.cj.jdbc.ConnectionImpl@fc807c1] will not be managed by Spring
==>  Preparing: SELECT id,name,age,email,create_time,update_time,version FROM user WHERE name = ? AND age = ?
==> Parameters: sxq(String), 18(Integer)
<==    Columns: id, name, age, email, create_time, update_time, version
<==        Row: 1, sxq, 18, test1@baomidou.com, null, null, null
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@71cb3139]
User(id=1, name=sxq, age=18, email=test1@baomidou.com, createTime=null, updateTime=null, version=null)
2021-10-24 18:32:32.600  INFO 3256 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-10-24 18:32:32.603  INFO 3256 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```

### - 4.分页查询

```java
 	/**
     * 分页查询测试
     */
        @Test
        void testPageHelper(){
        //1 new一个Page对象
        //2 传入两个参数：当前页和显示记录数
        Page<User> userPage = new Page<>(1,3);
        //调用map的分页查询方法
        //调用map分页查询的过程中，底层封装，将所有数据都封装到userPage对象中
        userMapper.selectPage(userPage,null);
        //通过userPage对象获取分页数据
        System.out.println(userPage.getCurrent());//获取当前页
        System.out.println(userPage.getRecords());//每页数据list集合
        System.out.println(userPage.getSize());   //每页数据记录数
        System.out.println(userPage.getTotal());  //获取表中所有记录数
        System.out.println(userPage.getPages());  //得到当前的总页数

        System.out.println(userPage.hasNext());   //是否有下一页
        System.out.println(userPage.hasPrevious());//是否有上一页

}
```

#结果

```java
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@39008c9f] was not registered for synchronization because synchronization is not active
2021-10-24 19:04:44.075  INFO 22392 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-10-24 19:04:44.080  WARN 22392 --- [           main] com.zaxxer.hikari.util.DriverDataSource  : Registered driver with driverClassName=com.mysql.jdbc.Driver was not found, trying direct instantiation.
2021-10-24 19:04:45.184  INFO 22392 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
JDBC Connection [HikariProxyConnection@246168102 wrapping com.mysql.cj.jdbc.ConnectionImpl@10b1a751] will not be managed by Spring
==>  Preparing: SELECT COUNT(*) FROM user
==> Parameters: 
<==    Columns: COUNT(*)
<==        Row: 9
<==      Total: 1
==>  Preparing: SELECT id,name,age,email,create_time,update_time,version FROM user LIMIT ?
==> Parameters: 3(Long)
<==    Columns: id, name, age, email, create_time, update_time, version
<==        Row: 1, sxq, 18, test1@baomidou.com, null, null, null
<==        Row: 2, Jack, 120, test2@baomidou.com, null, null, null
<==        Row: 3, Tom, 28, test3@baomidou.com, null, null, null
<==      Total: 3
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@39008c9f]
1
[User(id=1, name=sxq, age=18, email=test1@baomidou.com, createTime=null, updateTime=null, version=null), User(id=2, name=Jack, age=120, email=test2@baomidou.com, createTime=null, updateTime=null, version=null), User(id=3, name=Tom, age=28, email=test3@baomidou.com, createTime=null, updateTime=null, version=null)]
3
9
3
true
false
2021-10-24 19:04:45.259  INFO 22392 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-10-24 19:04:45.262  INFO 22392 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```

## 分页插件
使用步骤
 1.配置分页插件 MybatisPlusInterceptor

```
package com.sxq.mpdemo.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;

import com.baomidou.mybatisplus.extension.plugins.inner.OptimisticLockerInnerInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author sxq
 * Mybatis-Plus配置类
 */
  @Configuration
  @MapperScan("com.sxq.mpdemo.mapper")
  public class MpConfig {


    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        /**
         * 乐观锁插件
         */
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        ===============================================================================
        /**
         * 分页插件
         */
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.H2));
        ==============================================================================
        return mybatisPlusInterceptor;
    }
}
```
- 2.编写分页代码，直接new Page对象，传入两个参数：当前页和每页显示的记录数


```
     /**
     * 分页查询测试
     */
    @Test
    void testPageHelper(){
        //1 new一个Page对象
        //2 传入两个参数：当前页和显示记录数
        Page<User> userPage = new Page<>(1,3);
        //调用map的分页查询方法
        //调用map分页查询的过程中，底层封装，将所有数据都封装到userPage对象中
        userMapper.selectPage(userPage,null);
    
        //通过userPage对象获取分页数据


        System.out.println(userPage.getCurrent());//获取当前页
        System.out.println(userPage.getRecords());//每页数据list集合
        System.out.println(userPage.getSize());   //每页数据记录数
        System.out.println(userPage.getTotal());  //获取表中所有记录数
        System.out.println(userPage.getPages());  //得到当前的总页数
    
        System.out.println(userPage.hasNext());   //是否有下一页
        System.out.println(userPage.hasPrevious());//是否有上一页
    
    }
```
## mp删除
### 物理删除
> 真实删除，将对应的数据从数据库中删除，之后查询不到这条数据
#### - 单个数据删除

```
  /**
     * 物理删除
     */
    @Test
    void testDeleteById(){
        int result = userMapper.deleteById(2l);
        System.out.println(result);
    }

#结果显示
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@1fba386c] was not registered for synchronization because synchronization is not active
2021-10-24 19:19:47.033  INFO 1064 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-10-24 19:19:47.039  WARN 1064 --- [           main] com.zaxxer.hikari.util.DriverDataSource  : Registered driver with driverClassName=com.mysql.jdbc.Driver was not found, trying direct instantiation.
2021-10-24 19:19:48.307  INFO 1064 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
JDBC Connection [HikariProxyConnection@1389984438 wrapping com.mysql.cj.jdbc.ConnectionImpl@4da6d664] will not be managed by Spring
==>  Preparing: DELETE FROM user WHERE id=?
==> Parameters: 2(Long)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@1fba386c]
1
2021-10-24 19:19:48.382  INFO 1064 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-10-24 19:19:48.385  INFO 1064 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.

```
#### -  批量删除

> int results = userMapper.deleteBatchIds(Arrays.asList(2, 3, 4));
#### - 简单条件删除

```
 /**
     * 物理删除-简单条件删除
     */
    @Test
    void testDeleteByMap() {
        HashMap<String, Object> map = new HashMap<>();
        map.put("name", "sxq");
        map.put("age", 18);
        int result = userMapper.deleteByMap(map);
        System.out.println(result);
    }
```
### 逻辑删除

> 假删除，将对应数据中代表是否被删除字段转改修改为"被删除状态",之后在数据库中仍然可以看到此条数据

- 1.在数据库中加入deleted(删除状态)字段
- 2.实体类上添加deleted字段并加上注解@TableLogic和@TableField(fill = FieldFill.INSERT)


```
    /**
     * @TableField(fill = FieldFill.INSERT):这个注解可以通过数据库默认填充0来替换
     */
    @TableField(fill = FieldFill.INSERT)
    @TableLogic
    private Integer deleted;
```
- 3.元对象处理器接口添加deleted的insert默认值


```
package com.sxq.mpdemo.Handler;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * @author sxq
 * Component:交给spring管理
 * 元对象处理
 */
    @Component
    public class MyMetaObjectHandler implements MetaObjectHandler {
    /**
     * 使用mp实现添加操作，这个方法就会执行
     * @param metaObject
     */
      @Override
      public void insertFill(MetaObject metaObject) {

        this.setFieldValByName("createTime",new Date(),metaObject);
        this.setFieldValByName("updateTime",new Date(),metaObject);
        this.setFieldValByName("version",1,metaObject);
        **this.setFieldValByName("deleted",0,metaObject);**
      }


    /**
     * 使用mp实现修改操作，这个方法就会执行
     * @param metaObject
     */
    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("updateTime",new Date(),metaObject);
    }
}

```
- 4.application.properties加入配置。可不加使用默认即可


```
存在就默认为0，删除就为1
mybatis-plus.global-config.db-config.logic-delete-value=1 
mybatis-plus.global-config.db-config.logic-not-delete-value=0
```

- 5.直接调用删除方法即可


```
#最终执行效果，将deleted的值从0变为1
    @Test
    void testDeleted(){
        int user = userMapper.deleteById(1452240624232214529l);
        System.out.println(user);
    }
#结果
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@45d64d27] was not registered for synchronization because synchronization is not active
2021-10-24 19:51:17.316  INFO 26084 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-10-24 19:51:17.321  WARN 26084 --- [           main] com.zaxxer.hikari.util.DriverDataSource  : Registered driver with driverClassName=com.mysql.jdbc.Driver was not found, trying direct instantiation.
2021-10-24 19:51:18.450  INFO 26084 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
JDBC Connection [HikariProxyConnection@1163664780 wrapping com.mysql.cj.jdbc.ConnectionImpl@a451491] will not be managed by Spring
==>  Preparing: UPDATE user SET deleted=1 WHERE id=? AND deleted=0
==> Parameters: 1452240624232214529(Long)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@45d64d27]
1
2021-10-24 19:51:18.521  INFO 26084 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-10-24 19:51:18.524  INFO 26084 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.

```

## 性能分析
> 性能分析拦截器，用于输出每条sql语句及其执行时间

sql性能执行分析，开发环境使用，超过指定时间，停止运行，有助于发现问题

### 使用方式
- 1.添加依赖


```
    <dependency>
            <groupId>p6spy</groupId>
            <artifactId>p6spy</artifactId>
            <version>最新版</version>
        </dependency>
```
- 2.修改application.properties文件中的url和driver-class-name


```
#driver-class-name变成了p6spy的Driver路径
spring.datasource.driver-class-name=com.p6spy.engine.spy.P6SpyDriver
#url在jdbc和mysql之间加了p6spy
spring.datasource.url=jdbc:p6spy:mysql://localhost:3306/mybatis_plus
```
- 3.在resources目录下编写spy.properties文件


```
#3.2.1以上使用
modulelist=com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory,com.p6spy.engine.outage.P6OutageFactory
#3.2.1以下使用或者不配置
#modulelist=com.p6spy.engine.logging.P6LogFactory,com.p6spy.engine.outage.P6OutageFactory
# 自定义日志打印
logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger
#日志输出到控制台
appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger
# 使用日志系统记录 sql
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
# 设置 p6spy driver 代理
deregisterdrivers=true
# 取消JDBC URL前缀
useprefix=true
# 配置记录 Log 例外,可去掉的结果集有error,info,batch,debug,statement,commit,rollback,result,resultset.
excludecategories=info,debug,result,commit,resultset
# 日期格式
dateformat=yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
#driverlist=org.h2.Driver
# 是否开启慢SQL记录
outagedetection=true
# 慢SQL记录标准 2 秒
outagedetectioninterval=2

```

- 使用查询全部结果测试


```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@36b310aa] was not registered for synchronization because synchronization is not active
2021-10-24 20:40:08.089  INFO 28324 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-10-24 20:40:09.318  INFO 28324 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
JDBC Connection [HikariProxyConnection@389703464 wrapping com.p6spy.engine.wrapper.ConnectionWrapper@1a22e0ef] will not be managed by Spring
==>  Preparing: SELECT id,name,age,email,create_time,update_time,version,deleted FROM user WHERE deleted=0
==> Parameters: 
 Consume Time：14 ms 2021-10-24 20:40:09<==这就是性能分析结果
 Execute SQL：SELECT id,name,age,email,create_time,update_time,version,deleted FROM user WHERE deleted=0

<==    Columns: id, name, age, email, create_time, update_time, version, deleted
<==        Row: 1, sxq, 18, test1@baomidou.com, null, null, null, 0
<==        Row: 3, Tom, 28, test3@baomidou.com, null, null, null, 0
<==        Row: 4, Sandy, 21, test4@baomidou.com, null, null, null, 0
<==        Row: 5, Billie, 24, test5@baomidou.com, null, null, null, 0
<==        Row: 1451837706610008066, liuton, 20, liutong@qq.com, null, null, null, 0
<==        Row: 1452206527627673602, lyh, 120, lyh@qq.com, 2021-10-24 17:33:12, 2021-10-24 17:38:31, null, 0
<==        Row: 1452207266764775426, mfx, 20, mfx@qq.com, 2021-10-24 17:36:08, 2021-10-24 17:36:08, null, 0
<==      Total: 7
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@36b310aa]
[User(id=1, name=sxq, age=18, email=test1@baomidou.com, createTime=null, updateTime=null, version=null, deleted=0), User(id=3, name=Tom, age=28, email=test3@baomidou.com, createTime=null, updateTime=null, version=null, deleted=0), User(id=4, name=Sandy, age=21, email=test4@baomidou.com, createTime=null, updateTime=null, version=null, deleted=0), User(id=5, name=Billie, age=24, email=test5@baomidou.com, createTime=null, updateTime=null, version=null, deleted=0), User(id=1451837706610008066, name=liuton, age=20, email=liutong@qq.com, createTime=null, updateTime=null, version=null, deleted=0), User(id=1452206527627673602, name=lyh, age=120, email=lyh@qq.com, createTime=Sun Oct 24 17:33:12 CST 2021, updateTime=Sun Oct 24 17:38:31 CST 2021, version=null, deleted=0), User(id=1452207266764775426, name=mfx, age=20, email=mfx@qq.com, createTime=Sun Oct 24 17:36:08 CST 2021, updateTime=Sun Oct 24 17:36:08 CST 2021, version=null, deleted=0)]
2021-10-24 20:40:09.420  INFO 28324 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-10-24 20:40:09.423  INFO 28324 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.

```

## mp复杂条件查询
## Wrapper：条件构造抽象类，最顶端父类

- AbstractWrapper:用于查询条件封装，生成sql的where条件
- QueryWrapper:Entity对象封装操作类，不是用lambada语法
- UpdateWrapper:Update条件封装，用于Entity对象更新操作
- AbstractLambdaWrapper:Lambda语法使用Wrapper同一处理解析lambda获取column
- LambdaQueryWrapper:看名称也知道是用于Lambda语法使用的查询Wrapper
- LambdaUpdateWrapper:Lambda更新封装Wrapper


## 详解

![image](https://note.youdao.com/yws/api/personal/file/WEBe48868a02dec88818807689bc1fda7ec?method=download&shareKey=0669e0ec7f62b3da8c5044e1358a2aa2)

## 实例使用
- ge gt le lt
