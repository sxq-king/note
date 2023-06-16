## 1.安装mongo

## 2.入门基本操作

-- 查看数据库
show databases;

-- 选择数据库 (有就使用，没有就创建)
use study;

-- 查看集合
show collections;

-- 创建集合
db.createCollection('s1');
db.createCollection('s2');

-- 删除集合
db.s1.drop();

-- 删除数据库 （需要先选择）
db.dropDatabase();

### 2.1 增删查改

##### 1.增加数据

>  如果集合存在，那么直接插入数据。如果集合不存在，那么会隐式创建。

```sql
-- 插入单条数据
db.集合名.insert(JSON数据)
-- 插入多条数据 数组中写入多个json数据即可
db.集合名.insert([{},{},{},...])
-- 快速插入10条记录 由于mongodb底层使用JS引擎实现的，所以支持部分js语法
for ( var i = 1 ; i <= 10 ; i++) {db.s2.insert({uname:"d"+i,age:i});}
```

例：s2集合中创建一个sxq的数据
```sql
db.s1.insert({uname:"sxq",age:11});
```

##### 2.删除数据

```sql
-- 单个删除
db.集合名.remove(条件,是否删除一条); true就删除，false就按条件删除
```

例：s2中删除记录d1的记录

```sql
db.s2.remove({uname:"d1"});
```

例：s2中删除第一条记录

```sql
db.s2.remove({},true); 
```

##### 3.查询数据

```sql
db.集合名.find(条件,查询的列);
```

| 查询的列(可选)  | 写法    |
| --------------- | ------- |
| 查询全部列      | 不写    |
| 只显示age       | {age:1} |
| 除了age列都显示 | {age:0} |

其他语法:

> db.集合名.find(
>
> ​	{
>
> ​		键:{运算符:值}
>
> ​	}
>
> )

| 运算符 | 作用     |
| ------ | -------- |
| $gt    | 大于     |
| $gte   | 大于等于 |
| $lt    | 小于     |
| $lte   | 小于等于 |
| $ne    | 不等于   |
| $in    | in       |
| $nin   | not in   |

例：查询全部数据 **系统的_id无论如何和都会在**

```sql
db.s1.find();
```

例：查询所有数据，只看uname列

```sql
db.s2.find({},{uname:1});
db.s2.find({},{age:0});
```

例：查询age大于5的数据

```sql
db.s2.find({age:{$gt:5}});
```

例：查询age是5，8，9的数据

```sql
db.s2.find({age:{$in:[5,8,9]}});
```

例：只看年龄列

```sql
db.s2.find({},{age:1});
```

##### 4.修改数据

```sql
db.集合名.update(条件，修改器，是否新增，是否修改多条);
```

- 新数据需要用修改器，如果不使用，那么将会用新数据替换原来的数据。

- db.集合名.update(条件,{修改器:{键：值}}[是否新增，是否修改多条]);


| 修改器  |   作用   |
| :-----: | :------: |
|  $inc   |   递增   |
| $rename |  重命名  |
|  $set   | 修改列值 |
| $unset  |  删除列  |

- 条件匹配不到的数据则插入

db.s2.update({uname:"d9"},{$set:{age:99}},true);


| 是否新增 |     作用     |
| :------: | :----------: |
|   true   |     插入     |
|  false   | 不插入(默认) |

- 将匹配成功的数据都修改

db.s2.update({uname:"d9"},{$set:{age:999}},false,true);


|    作用     | 是否修改多条 |
| :---------: | :----------: |
|     是      |     true     |
| 否 （默认） |    false     |

练习准备

```sql
use s3;
for(var i = 1; i<= 10; i++){
	db.s3.insert( {"uname":"s"+i,"age":i} );
}
```

例：将uname：“s1”改为s2

```sql
db.s3.update({uname:"s1"},{set:{uname:"s2"}});
```

例：给uname：“s10”的年龄加2岁 -2岁

```sql
db.s3.update({uname:"s10"},{$inc:{age:2}},false,true);
db.s3.update({uname:"s10"},{$inc:{age:-2}},false,true);
```



## 3.SpringBoot整合mongodb

1. 引入依赖

> ```java
> <dependency>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-starter-data-mongodb</artifactId>
> </dependency>
> ```

2. 写配置application.yml

```java
spring:
  data:
    mongodb:
      uri: mongodb://用户名:密码@服务器IP:端口/数据库名
      # 上方为明确指定某个数据的用户进行连接
      # 也可以使用admin 数据库中的用户进行连接  统一到admin 数据库进行认证
      # admin 用户认证 url 写法： mongodb://账户:密码@ip:端口/数据库名?authSource=admin&authMechanism=SCRAM-SHA-1

```

3. 写个测试的实体类student

```java
package com.sxq.springbootmongodb.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

/**
 * @author song
 * @version 1.0
 * @date 2023/6/15 17:54
 * @description: TODO 此处@Document注解中的 collection注意别写成collation
 */
//通过collection参数指定当前实体类对应的文档
@Document(collection = "student")
public class Student {
	//@Id 用来标识主键
    @Id
    private Long id;

    private String name;

    private String sex;

    private String age;

    private String introduce;

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public String getIntroduce() {
        return introduce;
    }

    public void setIntroduce(String introduce) {
        this.introduce = introduce;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
}
```

4. 开始操作数据

使用springboot整合MongoDB的专用客户端接口MongoTemplate来进行操作

```java
    @Autowired
    private MongoTemplate mongoTemplate;	
	/**
     * 新增信息
     * @param student
     */
    @Override
    public void save(Student student) {
        mongoTemplate.save(student);
    }

    /**
     * 修改信息
     * @param student
     */
    @Override
    public void update(Student student) {
        //修改的条件
        Query query = new Query(Criteria.where("id").is(student.getId()));

        //修改的内容
        Update update = new Update();
        update.set("name",student.getName());

        mongoTemplate.updateFirst(query,update,Student.class);
    }

    /**
     * 查询所有信息
     * @return
     */
    @Override
    public List<Student> findAll() {
        return mongoTemplate.findAll(Student.class);
    }

    /**
     * 根据id查询所有信息
     * @param id
     */
    @Override
    public void delete(Integer id) {
        Student byId = mongoTemplate.findById(1,Student.class);
        mongoTemplate.remove(byId);
    }
```

