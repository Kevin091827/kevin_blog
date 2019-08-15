mybatis逆向工程很早之前有用过，但是一直没有整理使用步骤，今天来回顾下在idea下springboot中整合mybatis逆向工程的实现步骤

# 什么是逆向工程？

所谓mybatis逆向工程，就是mybatis会根据我们设计好的数据表，自动生成pojo、mapper以及mapper.xml。本文将介绍两种方式实现mybatis的逆向工程。

# generatorConfig.xml配置文件

配置数据库驱动，连接以及映射

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!-- 数据库驱动:选择你的本地硬盘上面的数据库驱动包-->
    <classPathEntry  location="D:\apache-maven-3.5.4\maven-repository\mysql\mysql-connector-java\5.1.37\mysql-connector-java-5.1.37.jar"/>
    <context id="DB2Tables"  targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库链接URL，用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://127.0.0.1:3306/mybatis" userId="root" password="123">
        </jdbcConnection>
        <!--默认为false，把JDBC DECIMAL 和NUMERIC类型解析为Integer，为true时
	        把JDBC DECIMAL 和NUMERIC类型解析为java.math.BigDecimal-->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!-- 生成模型的包名和位置-->
        <javaModelGenerator targetPackage="com.kevin.lce.entity" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!-- 生成映射文件的包名和位置-->
        <sqlMapGenerator targetPackage="com.kevin.lce.mapper" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!-- 生成DAO的包名和位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.kevin.lce.mapper" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名-->
        <table tableName="tb_user_role"
               domainObjectName="UserRole">
        </table>
        <!--<table tableName="tb_content_category"-->
               <!--domainObjectName="ContentCategory"-->
               <!--enableCountByExample="false"-->
               <!--enableUpdateByExample="false"-->
               <!--enableDeleteByExample="false"-->
               <!--enableSelectByExample="false"-->
               <!--selectByExampleQueryId="false">-->
        <!--</table>-->
    </context>
</generatorConfiguration>
```

# 代码方式实现逆向工程

引入逆向工程依赖
```xml
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.2</version>
        </dependency>
```
引入插件
```xml
            <!-- mybatis generator 自动生成代码插件 -->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <configuration>
                    <configurationFile>${basedir}/src/main/resources/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
            </plugin>
```
xml读取
```xml
        <resources>
            <resource>
                <directory>src/main/resources</directory>
            </resource>

            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
        </resources>
```

手工代码实现逆向工程

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class LceApplicationTests {

    @Test
    public void contextLoads() {
        try {
            generator();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 代码方式实现逆向工程
     * @throws Exception
     */
    public void generator() throws Exception {
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        // 指定配置文件
        File configFile = new File("generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }
}
```

# maven方式启动逆向工程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190815102222995.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190815102329827.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190815102357790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

```shell
mybatis-generator:generate -e
```

最后apply - ok

启动刚刚的maven

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190815102504463.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

这样即完成