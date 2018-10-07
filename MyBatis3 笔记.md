# MyBatis3 学习笔记

## 1.环境搭建

### 1) 用maven建立工程，配置pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.hx.mybatis</groupId>
    <artifactId>MybatisTraning</artifactId>
    <version>1.0-SNAPSHOT</version>


    <dependencies>
        <!--mybatis 就一个包 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.6</version>
        </dependency>

        <!--mysql 的驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.37</version>
        </dependency>

        <!--为了输出日志，引入的，注意配合log4.properties文件-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <!--测试包-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

    </dependencies>
</project>
```

### 2)log4j.properties

```properties
### set log levels ###
log4j.rootLogger = DEBUG,Console,File

###  输出到控制台  ###
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.Target=System.out
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern= %d{ABSOLUTE} %5p %c{1}:%L - %m%n


### 输出到日志文件 ###
log4j.appender.File=org.apache.log4j.RollingFileAppender 
log4j.appender.File.File=${project}/WEB-INF/logs/app.log
log4j.appender.File.DatePattern=_yyyyMMdd'.log'
log4j.appender.File.MaxFileSize=10MB
log4j.appender.File.Threshold=ALL
log4j.appender.File.layout=org.apache.log4j.PatternLayout
log4j.appender.File.layout.ConversionPattern=[%p][%d{yyyy-MM-dd HH\:mm\:ss,SSS}][%c]%m%n

```

### 3)mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <!--将写好的sql映射文件，注册到全局配置中-->
    <mappers>
        <mapper resource="EmployeeMapper.xml"/>
    </mappers>
</configuration>
```

### 4)定义sql的Mapper

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--

  namespace：名称空间,随便写
  id：唯一标识
  resultType:返回值类型
  #{id}:从传递过来的参数中，取出id值

  唯一标识：建议用命名空间+id 组合成为唯一标识

-->
<mapper namespace="com.hx.mybatis.bean.EmployeeMapper">
    <select id="selectEmployee" resultType="com.hx.mybatis.bean.Employee">
       select id,last_name lastName,email,gender from tbl_Employee where id = #{id}
    </select>
</mapper>
```

### 5)定义实体类（pojo）

```java
public class Employee {
    private Integer id;
    private String lastName;
    private String gender;
    private String email;


    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", lastName='" + lastName + '\'' +
                ", gender='" + gender + '\'' +
                ", email='" + email + '\'' +
                '}';
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

### 6)测试

1. 得到SessionFactory

2. 得到SqlSession

3. openSession

4. sqlSession.selectOne 利用sqlSession来对数据库的crud

5. sqlSession 关闭

  **sqlSession代表和数据库的一次会话**

```java
public class MybatisTest {

    private SqlSessionFactory sqlSessionFactory = null;
    private SqlSession sqlSession = null;


    @Before
    public void before() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        sqlSession = sqlSessionFactory.openSession();
    }

    @After
    public void after(){
        sqlSession.close();
    }


    @Test
    public void test1(){
        //参数：sql的唯一标识，参数
        Employee employee = sqlSession.selectOne("com.hx.mybatis.bean.EmployeeMapper.selectEmployee", 1);
        System.out.println(employee);

    }
```

