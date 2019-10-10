#                                                      MyBatis教程

## 一、入门了解

### 1、基本信息

​         MyBatis是一款优秀的持久层框架，它支持定制化SQL、存储过程以及高级映射。MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。MyBatis可以使用简单的XML或注解来配置和映射原生信息，将接口和java的POJOS(Plain Ordinary java Object,普通的java对象)映射成数据库中的记录。

### 2、背景

​		ORM（Object Relational Mapping，对象关系映射）是通过使用描述对象和数据库之间映射的元数据，将面向对象语言程序中的对象自动持久化到关系数据库中。

​		常见的ORM框架有：Hibernate、TopLink、Castor JDO、Apache OJB等。

​		MyBatis本身是apache的一个开源项目iBatis，2010年这个项目由apache software foundation 迁移到google code，并且该名为MyBatis。2013年11月迁移到Github。

​		每个MyBatis应用程序主要都是使用SqlSessionFactory实例可以通过SqlSessionFactoryBuilder获得。SqlSessionFactoryBuilder可以从一个xml配置文件或者一个预定义的配置类的实例获得。

##  二、环境搭建

### 1、引入依赖

~~~pom
<!--MyBatis的依赖-->
<dependency>    
    <groupId>org.mybatis</groupId>    			  		<artifactId>mybatis</artifactId>     				<version>3.5.2</version>
</dependency>
<!--    连接数据库的依赖    -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>
<!--    日志依赖    -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
~~~

### 2、建立实体类

~~~java
package xyz.liuyiblog.domain;

import java.io.Serializable;
import java.util.Date;

public class User implements Serializable {

    private Integer id;
    private String name;
    private int sex;
    private Date birthday;
    private String Address;

    public User() {
    }

    public User(Integer id, String name, int sex, Date birthday, String address) {
        this.id = id;
        this.name = name;
        this.sex = sex;
        this.birthday = birthday;
        Address = address;
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

    public int getSex() {
        return sex;
    }

    public void setSex(int sex) {
        this.sex = sex;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getAddress() {
        return Address;
    }

    public void setAddress(String address) {
        Address = address;
    }
}

~~~

### 3、建立dao层接口

~~~java
package xyz.liuyiblog.dao;

import xyz.liuyiblog.domain.User;

import java.util.List;

public interface UserDao {
    /**
     * 插入一条记录
     * @return 返回受影响的行数
     */
    public int insertUser();

    /**
     * 根据id查询用户信息
     * @param id
     * @return   查到的用户
     */
    public User findById(Integer id);

    /**
     * 查询所有用户信息
     * @return 用户列表
     */
    public List<User> findAll();

    /**
     * 修改用户的信息
     * @param user
     * @return 返回受影响的行数
     */
    public int update(User user);

    /**
     * 根据id删除用户
     * @param id
     * @return 返回受影响的行数
     */
    public int delete(Integer id);

}

~~~

### 4、建立mybatis的配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--MyBatis的主要配置文件-->
<configuration>
    <!--配置别名-->
    <typeAliases>
        <package name="xyz.liuyiblog.domain"/>
    </typeAliases>
    <!--配置环境-->
    <environments default="development">
        <!--配置mysql的环境-->
        <environment id="development">
            <!--配置事务的类型-->
            <transactionManager type="JDBC"/>
            <!--配置数据源（连接池）-->
            <dataSource type="POOLED">
                <!--配置连接数据库的4个基本信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/blog"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <!--指定mapper映射文件的位置-->
    <mapper resource="xyz/liuyiblog/dao/UserDao.xml"/>
    </mappers>

</configuration>
~~~

### 5、建立映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="xyz.liuyiblog.dao.UserDao">
	
    <select id="findById" resultType="user">
        select * from user where
    </select>

    <select id="findAll" resultType="user">
        select * from user
    </select>

    <insert id="insertUser" parameterType="user">
        insert into user values(#{id},#{name},#{sex},#{birthday},#{address})
    </insert>

    <update id="update" parameterType="user">
        update user
        <set>
            <if test="name!=null and name!='' ">
                name = #{name}
            </if>
            <if test="sex!=null and sex!='' ">
                sex = #{sex}
            </if>
            <if test="birthday!=null">
                birthday = #{birthday}
            </if>
            <if test="address!=null and address!='' ">
                address = #{address}
            </if>
        </set>
        where id = #{id}
    </update>

    <delete id="delete" parameterType="int">
        delete from user where id = #{id}
    </delete>
</mapper>
```

### 6、测试

~~~ java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import xyz.liuyiblog.dao.UserDao;
import xyz.liuyiblog.domain.User;

import java.io.IOException;
import java.io.InputStream;
import java.util.Date;
import java.util.List;

public class MyBatisTest {


    public static void main(String[] args) throws IOException {
        //1、读取配置文件
        InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2、创建SqlSessionFactory工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //3、使用工厂的SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //4、使用SqlSession对象创建Dao接口的代理对象
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        //5、使用代理对象执行方法
        /*  查找所有
        List<User> users = userDao.findAll();
        for (User user:users){
            System.out.println(user);
        }*/
        /*  根据id查找
        User user = userDao.findById(2);
        System.out.println(user);*/
        // 插入
        User user = new User(7, "扎饼", "男", new Date(),"广东佛山");
        int i = userDao.insertUser(user);
        sqlSession.commit();
        System.out.println("有"+i+"条记录受影响");
        //6、释放资源
        sqlSession.close();
        inputStream.close();
    }
}
~~~

在进行增删改操作时需要commit，查找不需要

### 7、注意事项

* mybatis的映射配置文件位置必须和dao接口的包结构相同

*  映射配置文件的mapper标签namespace属性的取值必须是dao接口的全限定类名
* 映射配置文件的操作配置（select），id属性的取值必须是dao接口的方法名

## 三、MyBatis的核心接口和类

<img src="F:\myblog\mybatislei.png" alt="mybatislei" style="zoom:200%;" />

* 每个MyBatis的应用程序都以一个SqlSessionFactory对象的实例为核心。
* 首先获取SqlSessionFactoryBuilder对象，可以根据XML配置文件或Configuration类的实例构建该对象。
* 然后获取SqlSessionFactory对象，该对象实例可以通过SqlSessionFactoryBuilder对象来获得。
* 使用SqlSessionFactory对象获取SqlSession实例。

### 1、SqlSessionFactoryBuilder

负责构建SqlSessionFactory，提供多个build()方法的重载，主要分为三个：

build(Reader reader,String environment,Properties properties)

build(InputStream inputStream,String environment,Properties properties)

build(Configuration config)

最大的特点是用过即丢，一旦创建了SqlSessionFactory对象之后，这个类就不再需要存在了，因此SqlSessionFactoryBuilder的最佳作用范围就是存在于方法体内，也就是局部变量。

### 2、SqlSessionFactory

相当于创建SqlSession实例的工厂，可以通过SqlSessionFactory提供的openSession()方法来获取SqlSession实例。

SqlSessionFactory对象一旦创建，就会在整个应用运行过程中始终存在，没有理由去销毁或再创建它，并且在应用运行中也不建议多次创建SqlSession。因此SqlSessionFactory的最佳作用与是Application，即随着应用的生命周期一同存在。这就是所谓的单例模式（指在应用运行期间有且仅有一个实例）。

### 3、SqlSession

SqlSession是用于执行持久化操作的对象，类似于JDBC中的Connection。它提供了面向数据库执行SQL命令所需的所有方法，可以通过SqlSession实例直接运行已映射的SQL语句。

SqlSession对应着一次数据库回话。由于数据库会话不是永久的，因此SqlSession的生命周期也不应该是永久的。相反，在每次访问数据库时都需要创建它。

需要注意的是，每个线程都有自己的SqlSession实例，SqlSession实例不能被共享，也不是线程安全的。因此最佳的作用域范围是request作用域或者方法体作用域内。